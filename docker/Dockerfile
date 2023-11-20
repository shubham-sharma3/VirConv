FROM nvidia/cudagl:11.1.1-devel-ubuntu20.04

ARG UID=1000
ARG GID=1000
ARG USERNAME=rosuser

# set shell to run other commands
SHELL ["/bin/bash", "-c"]

# Add User in container
RUN useradd -m $USERNAME && \
    echo "$USERNAME:$USERNAME" | chpasswd && \
    usermod --shell /bin/bash $USERNAME && \
    usermod -aG sudo $USERNAME && \
    mkdir -p /etc/sudoers.d && \
    touch /etc/sudoers.d/$USERNAME && \
    echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME && \
    usermod  --uid $UID $USERNAME && \
    groupmod --gid $GID $USERNAME && \
    touch /home/${USERNAME}/.sudo_as_admin_successful

# Install sudo for the user
RUN apt-get update && \
    apt-get install -y sudo

# Commands after this will run as the user and NOT as root
USER ${USERNAME}

# Install apt deps
RUN sudo apt-get -y install \
    python3 \
    python3-pip \
    python3-dev \
    python3-setuptools \
    apt-utils \
    git \
    bash-completion \
    acl

RUN echo 'debconf debconf/frontend select Noninteractive' | sudo debconf-set-selections

RUN sudo apt-get install -y \
    libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev \
    tk8.6-dev python3-tk libharfbuzz-dev libfribidi-dev libxcb1-dev

# updating alternatives to use python3 and pip3
RUN sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1

# Install python deps
RUN pip install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html

# Install python deps from requirements.txt
RUN echo "export PATH=/home/${USERNAME}/.local/bin:\$PATH" >> ~/.bashrc 
COPY --chown=${UID}:${GID} requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

# Set permissions for the user
RUN sudo setfacl -R -m u:${USERNAME}:rwx /usr/local/

WORKDIR /home/$USERNAME/ws