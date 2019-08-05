FROM centos:centos7
MAINTAINER The CentOS Project <cloud-ops@centos.org

ENV container docker
LABEL RUN="docker run -it --name NAME --privileged --ipc=host --net=host --pid=host -e HOST=/host -e NAME=NAME -e IMAGE=IMAGE -v /sys/fs/selinux:/sys/fs/selinux:ro -v /run:/run -v /var/log:/var/log -v /etc/localtime:/etc/localtime -v /:/host IMAGE"

RUN [ -e /etc/yum.conf ] && sed -i '/tsflags=nodocs/d' /etc/yum.conf || true

# Swap out the systemd-container package and install all useful packages
RUN yum -y install \
           git  \
           && yum clean all

# Set default command
CMD ["/usr/bin/bash"]
