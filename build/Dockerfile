from kalilinux/kali-rolling

RUN apt update && apt upgrade -y && apt dist-upgrade -y && \
  DEBIAN_FRONTEND=noninteractive apt install -y \
  kali-tools-top10 \
  pciutils \
  iputils-ping \
  openssh-server \
  hashcat \
  rdesktop \
  procps \
  wpasupplicant \
  isc-dhcp-common \
  isc-dhcp-client \
  arping \
  vim \
  tmux \
  screen \
  wordlists \
  pkg-config \
  libnl-3-dev \
  gcc \
  libssl-dev \
  libnl-genl-3-dev \
  build-essential \
  tk-dev \
  libncurses5-dev \
  libncursesw5-dev \
  libreadline6-dev \
  libdb5.3-dev \
  libgdbm-dev \
  libsqlite3-dev \
  libssl-dev \
  libbz2-dev \
  libexpat1-dev \
  liblzma-dev \
  zlib1g-dev \
  libffi-dev

RUN cd /root/ && \
  wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz && \
  tar xf Python-3.7.0.tar.xz && \
  cd Python-3.7.0 && \
  ./configure && \
  make -j4 && \
  make altinstall
RUN rm -rf /root/Python-3.7.0 && rm /root/Python-3.7.0.tar.xz

RUN cd /root/ && \
  git clone https://github.com/blunderbuss-wctf/wacker && \
  cd wacker/wpa_supplicant-2.8/wpa_supplicant/ && \
  cp defconfig_brute_force .config && \
  make -j4


# Set up SSH access
RUN mkdir /var/run/sshd
RUN echo 'root:ctf1234' | chpasswd
COPY content/sshd_config /etc/ssh/sshd_config

CMD ["/usr/sbin/sshd", "-D"]

EXPOSE 22/tcp
