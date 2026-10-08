FROM        eclipse-temurin:17

LABEL       author="AshlynOrSomethin" maintainer="ashlyn@onlyhit.xyz"
LABEL       org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-images"
LABEL       org.opencontainers.image.licenses="MIT"

ENV         DEBIAN_FRONTEND=noninteractive
RUN         apt-get update \
            && apt-get -y install --no-install-recommends curl ffmpeg iproute2 git sqlite3 python3 tzdata ca-certificates dnsutils fontconfig libfreetype6 libstdc++6 lsof sudo jq build-essential locales netcat-openbsd login passwd \
            && apt-get clean \
            && rm -rf /var/lib/apt/lists/* \
            && useradd -m -d /home/container container \
            && sed -i 's/^# *en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
            && locale-gen en_US.UTF-8

ENV         LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8
USER        container
ENV         USER=container HOME=/home/container
ENV         PATH="/home/container/.local/bin:${PATH}"
RUN         export FRESH_NO_DESKTOP=1; curl -fsSL https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
WORKDIR     /home/container
COPY        ./entrypoint.sh /entrypoint.sh
CMD         [ "/bin/bash", "/entrypoint.sh" ]
