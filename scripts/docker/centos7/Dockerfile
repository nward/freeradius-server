ARG from=centos:centos7
FROM ${from}

ARG source=https://github.com/FreeRADIUS/freeradius-server.git
ARG release=v3.0.x

#
#  Install build tools
#
RUN yum groupinstall -y "Development Tools"
RUN yum install -y rpmdevtools
RUN yum install -y openssl

#
#  Create build directory
#
RUN mkdir -p /usr/local/src/repositories
WORKDIR /usr/local/src/repositories

#
#  Shallow clone the FreeRADIUS source
#
RUN git clone --depth 1 --single-branch --branch ${release} ${source}
WORKDIR freeradius-server

#
#  Other requirements
#

# Use LTB's openldap packages intead of the distribution version to avoid linking against NSS
RUN echo $'[ltb-project]\n\
name=LTB project packages\n\
baseurl=https://ltb-project.org/rpm/$releasever/$basearch\n\
enabled=1\n\
gpgcheck=1\n\
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-LTB-project'\
> /etc/yum.repos.d/ltb-project.repo
RUN rpm --import https://ltb-project.org/lib/RPM-GPG-KEY-LTB-project

# EPEL repository for freetds and hiredis
RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

#
#  Install build dependencies
#
RUN [ -e redhat/freeradius.spec ] && yum-builddep -y redhat/freeradius.spec

#
#  Create RPM build environment
#
ENV BUILDDIR=/root/rpmbuild
RUN rpmdev-setuptree

RUN ./configure
RUN make freeradius-server-$(cat VERSION).tar.bz2
RUN cp freeradius-server-$(cat VERSION).tar.bz2 $BUILDDIR/SOURCES/
RUN cp -r redhat/* $BUILDDIR/SOURCES/
RUN cp -r redhat/freeradius.spec $BUILDDIR/SPECS/
WORKDIR $BUILDDIR

#
#  Build the server
#
ENV QA_RPATHS=0x0003
RUN rpmbuild -bb --define '_release $release' "$BUILDDIR/SPECS/freeradius.spec"

RUN mkdir /root/rpms
RUN mv $BUILDDIR/RPMS/*/*.rpm /root/rpms/

#
#  Clean environment and run the server
#
FROM ${from}
COPY --from=0 /root/rpms /root/rpms

# Use LTB's openldap packages intead of the distribution version to avoid linking against NSS
RUN echo $'[ltb-project]\n\
name=LTB project packages\n\
baseurl=https://ltb-project.org/rpm/$releasever/$basearch\n\
enabled=1\n\
gpgcheck=1\n\
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-LTB-project'\
> /etc/yum.repos.d/ltb-project.repo
RUN rpm --import https://ltb-project.org/lib/RPM-GPG-KEY-LTB-project
# EPEL repository for freetds and hiredis
RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

RUN yum install -y /root/rpms/*.rpm

EXPOSE 1812/udp 1813/udp
CMD ["/usr/sbin/radiusd", "-X"]
