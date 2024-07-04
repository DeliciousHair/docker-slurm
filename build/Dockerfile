## SETTINGS ------------------------------------------------------------------
ARG USER_PRIVILEGE_LEVEL

# Settings for all images
ARG TIMEZONE
ARG UBUNTU_VERSION
ARG SCRIPT

# Settings for standard user images
ARG USERNAME
ARG PASSWORD
ARG UID


## BUILD IMAGE: ROOT USER ----------------------------------------------------
FROM ubuntu:${UBUNTU_VERSION} AS stage-root
ARG TIMEZONE

# Run system updates
RUN apt update && apt -y upgrade

# Set time zone
RUN ln -sn /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo ${TIMEZONE} > /etc/timezone \
    && apt -y install tzdata

# Set locale
RUN apt -y install locales \
    && locale-gen en_US.UTF-8 \
    && update-locale LANG=en_US.UTF-8 LC_MESSAGES=POSIX

# Install system packages
RUN DEBIAN_FRONTEND=noninteractive \
    apt -y install \
        munge \
        slurm-wlm \
        slurmdbd \
        supervisor

# Slurm configuration
COPY slurm.conf /etc/slurm/slurm.conf
COPY slurmdbd.conf /etc/slurm/slurmdbd.conf
RUN chmod 600 /etc/slurm/slurmdbd.conf
RUN mkdir -p /var/spool/slurmctld \
    && mkdir -p /var/spool/slurmd \
    && chown root:root /var/spool/slurmctld \
    && chown root:root /var/spool/slurmd

# Startup configuration
COPY startup_ctld.sh /etc/startup_ctld.sh
COPY startup_compute.sh /etc/startup_compute.sh
COPY startup_dbd.sh /etc/startup_dbd.sh
RUN chmod 555 /etc/startup_*

ADD supervisord-ctld.conf /etc/supervisor/conf.d/supervisord-ctld.conf
ADD supervisord-compute.conf /etc/supervisor/conf.d/supervisord-compute.conf
ADD supervisord-dbd.conf /etc/supervisor/conf.d/supervisord-dbd.conf

EXPOSE 6817
EXPOSE 6818
EXPOSE 6819

# Set working directory and default command when running container
WORKDIR /root
