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
  python3

RUN cd /root/ && \
  git clone https://github.com/blunderbuss-wctf/wacker && \
  cd wacker && cp defconfig wpa_supplicant-2.10/wpa_supplicant/.config && \
  git apply wpa_supplicant.patch && cd wpa_supplicant-2.10/wpa_supplicant && \
  make -j4


# Set up SSH access
RUN mkdir /var/run/sshd
RUN echo 'root:ctf1234' | chpasswd
COPY content/sshd_config /etc/ssh/sshd_config

CMD ["/usr/sbin/sshd", "-D"]

EXPOSE 22/tcp
