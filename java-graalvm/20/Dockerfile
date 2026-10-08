FROM        ghcr.io/graalvm/graalvm-community:20

LABEL       author="AshlynOrSomethin" maintainer="ashlyn@onlyhit.xyz"
LABEL       org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-images"
LABEL       org.opencontainers.image.licenses="MIT"

RUN         . /etc/os-release \
            && microdnf --enablerepo="ol${VERSION_ID%%.*}_codeready_builder" update -y \
            && microdnf install -y "oracle-epel-release-el${VERSION_ID%%.*}" \
            && microdnf --enablerepo="ol${VERSION_ID%%.*}_codeready_builder" install -y ffmpeg-free bash curl iproute git sqlite python3 tzdata ca-certificates bind-utils fontconfig freetype libstdc++ lsof sudo jq gcc gcc-c++ make automake libtool glibc-langpack-en nmap-ncat shadow-utils tar gzip openssl \
            && microdnf clean all \
            && useradd -m -d /home/container container

ENV         LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8
USER        container
ENV         USER=container HOME=/home/container
ENV         PATH="/home/container/.local/bin:${PATH}"
RUN         export FRESH_NO_DESKTOP=1; curl -fsSL https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
WORKDIR     /home/container
COPY        ./entrypoint.sh /entrypoint.sh
CMD         [ "/bin/bash", "/entrypoint.sh" ]
