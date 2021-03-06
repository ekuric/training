deltarpm iptables-services

Docker images:
docker images | grep buildvm | awk {'print $3'} | xargs docker rmi -f

docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-haproxy-router
docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-deployer
docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-sti-builder
docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-docker-builder
docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-pod
docker pull docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-docker-registry
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-haproxy-router registry.access.redhat.com/openshift3_beta/ose-haproxy-router:v0.3.1
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-deployer registry.access.redhat.com/openshift3_beta/ose-deployer:v0.3.1
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-sti-builder registry.access.redhat.com/openshift3_beta/ose-sti-builder:v0.3.1
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-docker-builder registry.access.redhat.com/openshift3_beta/ose-docker-builder:v0.3.1
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-pod registry.access.redhat.com/openshift3_beta/ose-pod:v0.3.1
docker tag docker-buildvm-rhose.usersys.redhat.com:5000/openshift3_beta/ose-docker-registry registry.access.redhat.com/openshift3_beta/ose-docker-registry:v0.3.1

DOCKER_OPTIONS='--insecure-registry=0.0.0.0/0 -b=lbr0 --mtu=1450 --selinux-enabled'

## start master
cd ~/training
git pull origin master
mv -f ~/training/beta1/dnsmasq.conf /etc/
restorecon -rv /etc/dnsmasq.conf
sed -e '/^nameserver .*/i nameserver 192.168.133.2' -i /etc/resolv.conf
systemctl start dnsmasq
sed -i -e 's/^OPTIONS=.*/OPTIONS="--loglevel=4 --public-master=ose3-master.example.com"/' \
/etc/sysconfig/openshift-master
sed -i -e 's/^OPTIONS=.*/OPTIONS=-v=4/' /etc/sysconfig/openshift-sdn-master
sed -i -e 's/^MASTER_URL=.*/MASTER_URL=http:\/\/ose3-master.example.com:4001/' \
-e 's/^MINION_IP=.*/MINION_IP=192.168.133.2/' \
-e 's/^OPTIONS=.*/OPTIONS=-v=4/' \
-e 's/^DOCKER_OPTIONS=.*/DOCKER_OPTIONS="--insecure-registry=0.0.0.0\/0 -b=lbr0 --mtu=1450 --selinux-enabled"/' \
/etc/sysconfig/openshift-sdn-node
sed -i -e 's/OPTIONS=.*/OPTIONS="--loglevel=4 --master=ose3-master.example.com"/' \
/etc/sysconfig/openshift-node

systemctl start openshift-master; systemctl start openshift-sdn-master; systemctl start openshift-sdn-node;

## start node 1
sed -i -e 's/^MASTER_URL=.*/MASTER_URL=http:\/\/ose3-master.example.com:4001/' \
-e 's/^MINION_IP=.*/MINION_IP=192.168.133.3/' \
-e 's/^OPTIONS=.*/OPTIONS=-v=4/' \
-e 's/^DOCKER_OPTIONS=.*/DOCKER_OPTIONS="--insecure-registry=0.0.0.0\/0 -b=lbr0 --mtu=1450 --selinux-enabled"/' \
/etc/sysconfig/openshift-sdn-node
sed -i -e 's/OPTIONS=.*/OPTIONS="--loglevel=4 --master=ose3-master.example.com"/' \
/etc/sysconfig/openshift-node
rsync -av root@ose3-master.example.com:/var/lib/openshift/openshift.local.certificates /var/lib/openshift/

## start node 2
sed -i -e 's/^MASTER_URL=.*/MASTER_URL=http:\/\/ose3-master.example.com:4001/' \
-e 's/^MINION_IP=.*/MINION_IP=192.168.133.4/' \
-e 's/^OPTIONS=.*/OPTIONS=-v=4/' \
-e 's/^DOCKER_OPTIONS=.*/DOCKER_OPTIONS="--insecure-registry=0.0.0.0\/0 -b=lbr0 --mtu=1450 --selinux-enabled"/' \
/etc/sysconfig/openshift-sdn-node
sed -i -e 's/OPTIONS=.*/OPTIONS="--loglevel=4 --master=ose3-master.example.com"/' \
/etc/sysconfig/openshift-node
rsync -av root@ose3-master.example.com:/var/lib/openshift/openshift.local.certificates /var/lib/openshift/

docker run -i -t google/golang /bin/bash

alias denter='bash -c '"'"' nsenter --mount --uts --ipc --net --pid --target $(docker inspect --format {{.State.Pid}} $0)'"'"
