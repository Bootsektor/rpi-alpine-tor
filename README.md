# rpi-alpine-tor
alpine:latest based TOR "The onion router" Docker image
this image was created on the latest alpine ( 3.7 ) and tor 3.3.2 or 3.2.9 release
image build:

cat << EOM >> torrc
Log notice stdout
SocksPort 0.0.0.0:9050
EOM

cat << EOM >> Dockerfile
# Tor 0.3.2.9/0.3.3.2-alpa STABLE & ALPHA
# Docker Image v0.1 20180221
# create your own torrc for your likings, sample torrc available

FROM alpine:latest

LABEL maintainer="bootsektor <https://hub.docker.com/u/bootsektor"

# TESTED ON RASPBERRYY PI 3 ( HOST OS: HypriotOS )

# EXPOSE PORT FOR EXTERNAL USE TORORT, TOR-DNS, TOR-DIRECTORY
EXPOSE 9050 9053 9001

# FETCH AND COPY TOR LATEST STABLE/BETA TO INTO SAME FOLDER
#
# wget https://www.torproject.org/dist/tor-0.3.2.9.tar.gz
# wget https://www.torproject.org/dist/tor-0.3.2.9.tar.gz.asc
# wget https://www.torproject.org/dist/tor-0.3.3.2-alpha.tar.gz
# wget https://www.torproject.org/dist/tor-0.3.3.2-alpha.tar.gz.asc

# COPY PREFERRED VERSION TO YOUR IMAGE
#
COPY tor-0.3.3.2-alpha.tar.gz /tmp/
COPY tor-0.3.3.2-alpha.tar.gz.asc /tmp/
# COPY tor-0.3.2.9.tar.gz /tmp/
# COPY tor-0.3.2.9.tar.gz.asc /tmp/

RUN build_pkgs=" \
        openssl-dev \
        zlib-dev \
        libevent-dev \
        gnupg \
        build-base \
        perl \
        "\
  && runtime_pkgs=" \
        libgcc \
        openssl \
        zlib \
        libevent \
        "\
  && apk --update add ${build_pkgs} ${runtime_pkgs} \
  && cd /tmp/ \
  && gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 \
  && gpg --keyserver keys.gnupg.net --recv-keys 4607B1FB \
  && gpg --verify tor*.asc \
  && tar xzf tor*.gz \
  && cd tor-* \
  && ./configure --prefix=/usr \
  && make -j4 \
  && make install-strip \
  && cd \
  && rm -rf /tmp/* \
  && apk del ${build_pkgs} \
  && rm -rf /var/cache/apk/*

RUN adduser -Ds /bin/sh tor

RUN mkdir /etc/tor
COPY torrc /etc/tor/

USER tor
CMD ["tor", "-f", "/etc/tor/torrc"]
EOM
