FROM        amazoncorretto:11-alpine

LABEL       author="AshlynOrSomethin" maintainer="ashlyn@onlyhit.xyz"
LABEL       org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-images"
LABEL       org.opencontainers.image.licenses="MIT"

RUN         apk add --no-cache gcompat curl ffmpeg iproute2 git sqlite python3 tzdata ca-certificates bind-tools fontconfig freetype libstdc++ lsof jq build-base clang lld llvm musl-locales musl-locales-lang netcat-openbsd bash \
            && adduser -D -h /home/container container

ENV         CFLAGS="-O3 -march=native -flto"
ENV         CXXFLAGS="-O3 -march=native -flto"
ENV         LDFLAGS="-Wl,-O1 -Wl,--as-needed -flto"
ENV         CPPFLAGS="-O3 -march=native -flto"
ENV         CC="gcc"
ENV         CXX="g++"
ENV         LC_ALL=en_US.UTF-8
ENV         LANG=en_US.UTF-8
ENV         LANGUAGE=en_US.UTF-8

USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

COPY        ./entrypoint.sh /entrypoint.sh
CMD         [ "/bin/bash", "/entrypoint.sh" ]