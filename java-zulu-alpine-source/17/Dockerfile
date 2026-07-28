FROM alpine:3.20

LABEL author="AshlynOrSomethin" maintainer="ashlyn@onlyhit.xyz"
LABEL org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-images"
LABEL org.opencontainers.image.licenses="MIT"

RUN apk add --no-cache bash curl git tar xz wget ca-certificates build-base clang lld llvm musl-dev linux-headers file python3 perl tar zip unzip bzip2 gcompat tzdata fontconfig freetype sqlite sqlite-dev libffi-dev openssl openssl-dev libstdc++

ENV CFLAGS="-O3 -march=native -flto"
ENV CXXFLAGS="-O3 -march=native -flto"
ENV LDFLAGS="-Wl,-O1 -Wl,--as-needed -flto"
ENV CPPFLAGS="-O3 -march=native -flto"
ENV CC="gcc"
ENV CXX="g++"

RUN mkdir -p /opt/src && cd /opt/src && \
    wget -qO- https://cdn.azul.com/zulu/bin/zulu17.54.21-ca-jdk17.0.14-linux_musl_x64.tar.gz | tar -xz && \
    mv zulu17* /opt/java/openjdk

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV LANGUAGE=C.UTF-8

RUN adduser -D -h /home/container container
COPY ./entrypoint.sh /entrypoint.sh
USER container
WORKDIR /home/container
CMD ["/bin/bash", "/entrypoint.sh"]
