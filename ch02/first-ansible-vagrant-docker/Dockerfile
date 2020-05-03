#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Fortinet Corporation. All rights reserved.
# Licensed under the MIT License. See LICENSE in the project root for license information.
#-------------------------------------------------------------------------------------------------------------

# Stage python3.7/Ansible2.8/Stretch
FROM python:3.7-stretch

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vagrant
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Uncomment the following COPY line and the corresponding lines in the `RUN` command if you wish to
# include your requirements in the image itself. It is suggested that you only do this if your
# requirements rarely (if ever) change.
# COPY requirements.txt /tmp/pip-tmp/

# Configure apt and install packages
RUN apt-get update \
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    ######################################
    # Install and configure openssh-server
    && apt-get install -y openssh-server \
    && mkdir /var/run/sshd \
    # In case you still want password authentication
    # && echo $USERNAME':PASSWORD_FOR_USER_'$USERNAME | chpasswd \
    && sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd \
    ######################################
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /home/vagrant/.ssh
ADD authorized_keys /home/vagrant/.ssh/authorized_keys
RUN chmod 700 /home/vagrant/.ssh && \
    chmod 700 /home/vagrant/.ssh/authorized_keys && \
    chown vagrant:vagrant -R /home/vagrant/.ssh

#############################
# Expose SSH port 22
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
#############################

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
