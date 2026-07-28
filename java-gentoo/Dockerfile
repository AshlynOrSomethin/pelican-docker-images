ARG UPSTREAM_IMAGE=amazoncorretto:25-alpine

FROM ${UPSTREAM_IMAGE} AS java-upstream

RUN set -eux; \
    if [ -n "${JAVA_HOME:-}" ] && [ -d "${JAVA_HOME}" ]; then \
      src_java="${JAVA_HOME}"; \
    else \
      java_bin="$(command -v java)"; \
      src_java="$(dirname "$(dirname "${java_bin}")")"; \
    fi; \
    src_java="$(readlink -f "${src_java}")"; \
    [ -d "${src_java}" ]; \
    mkdir -p /opt/java-export/java; \
    cp -a "${src_java}/." /opt/java-export/java/

FROM gentoo/stage3:musl-llvm

LABEL author="AshlynOrSomethin" maintainer="admin@softwarenoob.com"
LABEL org.opencontainers.image.source="https://github.com/AshlynOrSomethin/pelican-docker-images"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.base.name="gentoo/stage3:musl-llvm"
LABEL org.opencontainers.image.base.stage3="https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-musl-llvm/"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG GENTOO_CFLAGS="-march=native -O2 -pipe"
ARG GENTOO_CXXFLAGS="-march=native -O2 -pipe"
ARG GENTOO_FCFLAGS="-march=native -O2 -pipe"
ARG GENTOO_FFLAGS="-march=native -O2 -pipe"
ARG GENTOO_RUSTFLAGS="-C target-cpu=native"
ARG GENTOO_LDFLAGS="-Wl,-O1 -Wl,--as-needed"
ARG GENTOO_LTO=""
ARG ENABLE_GENTOO_GURU_COMPAT=1

ENV FEATURES="parallel-fetch parallel-install splitdebug -news"

RUN set -eux; \
    mkdir -p /etc/portage /etc/portage/repos.conf; \
    mkdir -p /etc/portage/package.accept_keywords; \
    touch /etc/portage/make.conf; \
    if [[ ! -f /etc/portage/repos.conf/gentoo.conf && -f /usr/share/portage/config/repos.conf ]]; then \
      cp /usr/share/portage/config/repos.conf /etc/portage/repos.conf/gentoo.conf; \
    fi; \
    jobs="$(nproc)"; \
    if [[ "${jobs}" -gt 31 ]]; then jobs=31; fi; \
    if [[ "${jobs}" -lt 1 ]]; then jobs=1; fi; \
    lto_flags="${GENTOO_LTO}"; \
    if [[ -n "${lto_flags}" ]]; then \
      lto_flags="${lto_flags} -flto"; \
    fi; \
    cat > /etc/portage/make.conf <<EOF
COMMON_FLAGS="${GENTOO_CFLAGS}"
CFLAGS="${GENTOO_CFLAGS}${lto_flags}"
CXXFLAGS="${GENTOO_CXXFLAGS}${lto_flags}"
FCFLAGS="${GENTOO_FCFLAGS}"
FFLAGS="${GENTOO_FFLAGS}"
RUSTFLAGS="${GENTOO_RUSTFLAGS}"
LDFLAGS="${GENTOO_LDFLAGS}${lto_flags}"
FEATURES="${FEATURES}"
MAKEOPTS="-j${jobs} -l${jobs}"
EMERGE_DEFAULT_OPTS="--quiet-build=y --jobs=${jobs} --load-average=${jobs}"
EOF

RUN set -eux; \
    emerge-webrsync; \
  CFLAGS="-march=native -O2 -pipe" \
  CXXFLAGS="-march=native -O2 -pipe" \
  FCFLAGS="-march=native -O2 -pipe" \
  FFLAGS="-march=native -O2 -pipe" \
  RUSTFLAGS="-C target-cpu=native" \
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
    rm -rf /var/db/repos/gentoo; \
    mkdir -p /var/db/repos/gentoo; \
    emerge --sync

RUN set -eux; \
    if [ "${ENABLE_GENTOO_GURU_COMPAT}" = "1" ]; then \
      if ! eselect repository list -i | grep -qE '^[[:space:]]*guru[[:space:]]'; then \
        eselect repository enable guru; \
      fi; \
      emaint sync -r guru; \
      mkdir -p /etc/portage/package.accept_keywords /etc/portage/package.unmask; \
      printf '%s\n' 'sys-libs/libucontext ~amd64' 'sys-libs/gcompat::guru ~amd64' > /etc/portage/package.accept_keywords/guru-gcompat; \
      printf '%s\n' 'sys-libs/libucontext' 'sys-libs/gcompat::guru' > /etc/portage/package.unmask/guru-gcompat; \
      emerge --oneshot sys-libs/libucontext sys-libs/gcompat::guru; \
    else \
      echo "Skipping Guru overlay gcompat/libucontext install because ENABLE_GENTOO_GURU_COMPAT=${ENABLE_GENTOO_GURU_COMPAT}"; \
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