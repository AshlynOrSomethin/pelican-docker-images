ARG UPSTREAM_IMAGE=eclipse-temurin:21-jdk

FROM ${UPSTREAM_IMAGE} AS java-upstream

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN set -eux; \
    if [[ -n "${JAVA_HOME:-}" && -d "${JAVA_HOME}" ]]; then \
      src_java="${JAVA_HOME}"; \
    else \
      java_bin="$(readlink -f "$(command -v java)")"; \
      src_java="$(dirname "$(dirname "${java_bin}")")"; \
    fi; \
    src_java="$(readlink -f "${src_java}")"; \
    test -d "${src_java}"; \
    mkdir -p /opt/java-export/java; \
    cp -a "${src_java}/." /opt/java-export/java/

FROM gentoo/stage3:musl-llvm

LABEL author="AshlynOrSomethin" maintainer="admin@softwarenoob.com"
LABEL org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-docker-images"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.base.name="gentoo/stage3:musl-llvm"
LABEL org.opencontainers.image.base.stage3="https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-musl-llvm/"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG GENTOO_CFLAGS="-O3 -pipe -flto=auto -fuse-linker-plugin -fno-plt -march=native -mtune=native"
ARG GENTOO_CXXFLAGS="-O3 -pipe -flto=auto -fuse-linker-plugin -fno-plt -march=native -mtune=native"
ARG GENTOO_LDFLAGS="-Wl,-O2 -Wl,--as-needed -Wl,--sort-common -flto=auto"

ENV FEATURES="parallel-fetch parallel-install splitdebug -news"

RUN set -eux; \
    mkdir -p /etc/portage /etc/portage/repos.conf; \
    mkdir -p /etc/portage/package.accept_keywords; \
    touch /etc/portage/make.conf; \
    if [[ ! -f /etc/portage/repos.conf/gentoo.conf && -f /usr/share/portage/config/repos.conf ]]; then \
      cp /usr/share/portage/config/repos.conf /etc/portage/repos.conf/gentoo.conf; \
    fi; \
    jobs="$(nproc)"; \
    if [[ "${jobs}" -lt 1 ]]; then jobs=1; fi; \
    cat > /etc/portage/make.conf <<EOF
COMMON_FLAGS="${GENTOO_CFLAGS}"
CFLAGS="${GENTOO_CFLAGS}"
CXXFLAGS="${GENTOO_CXXFLAGS}"
LDFLAGS="${GENTOO_LDFLAGS}"
FEATURES="${FEATURES}"
MAKEOPTS="-j${jobs}"
EMERGE_DEFAULT_OPTS="--quiet-build=y --jobs=${jobs} --load-average=${jobs}"
EOF

RUN set -eux; \
    emerge-webrsync; \
  CFLAGS="-O2 -pipe" \
  CXXFLAGS="-O2 -pipe" \
  LDFLAGS="-Wl,-O1 -Wl,--as-needed" \
  emerge --oneshot app-misc/ca-certificates dev-vcs/git app-eselect/eselect-repository

RUN set -eux; \
    if grep -q '^sync-type[[:space:]]*=' /etc/portage/repos.conf/gentoo.conf; then \
      sed -i 's|^sync-type[[:space:]]*=.*|sync-type = git|' /etc/portage/repos.conf/gentoo.conf; \
    else \
      printf '\nsync-type = git\n' >> /etc/portage/repos.conf/gentoo.conf; \
    fi; \
    if grep -q '^sync-uri[[:space:]]*=' /etc/portage/repos.conf/gentoo.conf; then \
      sed -i 's|^sync-uri[[:space:]]*=.*|sync-uri = https://github.com/gentoo-mirror/gentoo.git|' /etc/portage/repos.conf/gentoo.conf; \
    else \
      printf '\nsync-uri = https://github.com/gentoo-mirror/gentoo.git\n' >> /etc/portage/repos.conf/gentoo.conf; \
    fi; \
    emerge --sync

RUN set -eux; \
    if ! eselect repository list -i | grep -qE '^[[:space:]]*guru[[:space:]]'; then \
      eselect repository enable guru; \
    fi; \
    emaint sync -r guru; \
    if ! emerge --oneshot app-emulation/gcompat; then \
      echo "app-emulation/gcompat ~amd64" > /etc/portage/package.accept_keywords/gcompat; \
      emerge --oneshot app-emulation/gcompat::guru; \
    fi

RUN set -eux; \
    emerge --oneshot app-shells/bash \
      app-arch/tar \
      dev-db/sqlite \
      dev-libs/openssl \
      dev-vcs/git \
      media-libs/fontconfig \
      media-libs/freetype \
      net-misc/curl \
      net-misc/iputils \
      sys-apps/iproute2 \
      sys-apps/coreutils \
      sys-apps/shadow \
      sys-libs/timezone-data \
      sys-process/lsof; \
    rm -rf /var/cache/binpkgs/* /var/cache/distfiles/* /var/tmp/portage/*

COPY --from=java-upstream /opt/java-export/java /opt/java/openjdk

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV LANGUAGE=C.UTF-8

RUN useradd -m -d /home/container container

USER container
ENV USER=container HOME=/home/container
WORKDIR /home/container

COPY ./entrypoint.sh /entrypoint.sh
CMD ["/bin/bash", "/entrypoint.sh"]