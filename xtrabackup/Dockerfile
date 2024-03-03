FROM centos:7.8.2003
LABEL authors="hujiale"


ADD percona-xtrabackup-24-2.4.20-1.el7.x86_64.rpm /

RUN yum install -y /percona-xtrabackup-24-2.4.20-1.el7.x86_64.rpm && \
    yum clean all && \
    rm -rf /percona-xtrabackup-24-2.4.20-1.el7.x86_64.rpm
