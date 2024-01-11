## SETTINGS ------------------------------------------------------------------
ARG USER_PRIVILEGE_LEVEL

# Settings for all images
ARG TIMEZONE
ARG UBUNTU_VERSION

# Settings for standard user images
ARG USERNAME
ARG PASSWORD
ARG UID


## BUILD IMAGE: ROOT USER ----------------------------------------------------
FROM ubuntu:${UBUNTU_VERSION} as stage-root
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
        slurmdbd

# Slurm configuration
COPY slurm.conf /etc/slurm/slurm.conf
COPY slurmdbd.conf /etc/slurm/slurmdbd.conf
RUN chmod 600 /etc/slurm/slurmdbd.conf
RUN mkdir -p /var/spool/slurmctld \
    && chown root:root /var/spool/slurmctld

# Startup configuration
COPY startup.sh /etc/startup.sh
RUN chmod 555 /etc/startup.sh

# Set working directory and default command when running container
WORKDIR /root
CMD /etc/startup.sh ; /bin/bash -l
