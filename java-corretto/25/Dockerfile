FROM amazoncorretto:25

LABEL author="DonPedroTV" maintainer="admin@donpedrotv.com"
LABEL org.opencontainers.image.source="https://github.com/DonPedroTV/pterodactyl-images"
LABEL org.opencontainers.image.licenses="MIT"

RUN yum -y update \
 && yum -y install --allowerasing \
    curl ca-certificates openssl git tar \
    sqlite sqlite-libs \
    fontconfig freetype freetype-devel \
    gcc gcc-c++ make automake libtool \
    tzdata iproute \
    shadow-utils \
 && yum clean all \
 && rm -rf /var/cache/yum \
 && useradd -m -d /home/container container


ENV LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8

USER container
ENV USER=container HOME=/home/container
ENV PATH="/home/container/.local/bin:${PATH}"
RUN export FRESH_NO_DESKTOP=1; curl -fsSL https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
WORKDIR /home/container

COPY ./entrypoint.sh /entrypoint.sh
CMD ["/bin/bash", "/entrypoint.sh"]
