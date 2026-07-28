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

RUN mkdir -p /opt/src /opt/java && cd /opt/src && \
    wget -qO /tmp/openjdk25.tar.gz https://github.com/adoptium/temurin25-binaries/releases/download/jdk-25.0.1%2B9/OpenJDK25U-jdk_x64_alpine-linux-musl_25.0.1_9.tar.gz && \
    tar -xzf /tmp/openjdk25.tar.gz -C /opt/java && \
    mv /opt/java/jdk-25* /opt/java/openjdk

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV LANGUAGE=C.UTF-8

RUN adduser -D -h /home/container container
USER container
WORKDIR /home/container
CMD ["/bin/bash"]
