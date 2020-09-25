# We will need GPU access in these images,
# so base on nvidia/cuda, which is in turn
# based on centos:7.
FROM nvidia/cuda:11.0-cudnn8-devel-centos7

MAINTAINER Claudio Kopper <kopper@ualberta.ca>

# Make the nvidia tools available on the system in case we
# log intothe container through ssh
COPY nvidia.sh  /etc/profile.d/nvidia.sh
COPY nvidia.csh /etc/profile.d/nvidia.csh

# Enable OpenCL
RUN yum install -y epel-release && \
    yum install -y ocl-icd ocl-icd-devel clinfo && \
    rm -rf /var/cache/yum && \
    mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd

# Install Singularity Dependencies
RUN yum groupinstall -y 'Development Tools' && \
	yum install -y epel-release && \
        yum install -y golang openssl-devel libuuid-devel \
        libseccomp-devel squashfs-tools cryptsetup wget

# Set the version of Singularity to install
ENV SINGULARITY_VERSION=3.5.1

# Create a Singularity RPM and install it
RUN wget https://github.com/sylabs/singularity/releases/download/v${SINGULARITY_VERSION}/singularity-${SINGULARITY_VERSION}.tar.gz && \
    rpmbuild -tb singularity-${SINGULARITY_VERSION}.tar.gz && \
    rpm -ivh ~/rpmbuild/RPMS/x86_64/singularity-${SINGULARITY_VERSION}-1.el7.x86_64.rpm && \
    rm -rf ~/rpmbuild singularity-${SINGULARITY_VERSION}*.tar.gz

# Configure Singularity (in directory-mode only, no loopback mounted images for now,
# those do not work well in docker [or need many more privileges...])
RUN sed -i 's/^[# ]*max loop devices.*/max loop devices = 0/g' /etc/singularity/singularity.conf && \
    sed -i 's/^[# ]*enable overlay.*/enable overlay = yes/g' /etc/singularity/singularity.conf && \
    sed -i 's/^[# ]*allow container squashfs.*/allow container squashfs = no/g' /etc/singularity/singularity.conf && \
    sed -i 's/^[# ]*allow container extfs.*/allow container extfs = no/g' /etc/singularity/singularity.conf && \
    sed -i 's/^[# ]*allow container dir.*/allow container dir = yes/g' /etc/singularity/singularity.conf && \
    yum -y clean all && rm -rf /var/cache/yum

# Install ssh, supervisord and ldap tools
RUN yum -y install openssh openssh-clients openssh-server && \
    mkdir -p /var/log/ssh && \
    mkdir -p /var/log/nslcd && \
    yum -y install epel-release && \
    yum -y install python-pip && \
    pip install supervisor supervisor-stdout && \
    yum -y install openldap-clients nss-pam-ldapd authconfig && \
    yum -y clean all && rm -rf /var/cache/yum && rm -rf /root/.cache

# Install compilers and other tools
RUN yum -y groupinstall \
           "Compatibility Libraries" \
           "Scientific Support" && \
    yum -y install nano astropy-tools bc binutils binutils-devel coreutils \
           curl fontconfig gcc gcc-c++ gcc-gfortran git glew-devel glib2-devel \
           glib-devel graphviz gsl-devel java-1.8.0-openjdk java-1.8.0-openjdk-devel \
           jq libgfortran libGLU libgomp libicu libquadmath libtool libtool-ltdl \
           libtool-ltdl-devel libX11-devel libXaw-devel libXext-devel \
           libXft-devel libxml2 libxml2-devel libXmu-devel libXpm libXpm-devel \
           libXt mesa-libGL-devel numpy octave octave-devel openssl098e p7zip \
           p7zip-plugins python-astropy python-devel R-devel redhat-lsb-core \
           rsync scipy subversion tcl-devel tcsh time tk-devel which \
           lzo squashfs-tools mc python-requests tmux screen mosh strace emacs \
           postgresql mariadb python2-pg8000 PyGreSQL valgrind cronie vim-enhanced zsh \
	   xorg-x11-xauth && \
    yum -y clean all && rm -rf /var/cache/yum

# Install Python 3 and Jupyter
RUN yum -y install python3 && \
    pip3 install --upgrade pip && \
    pip3 install jupyter