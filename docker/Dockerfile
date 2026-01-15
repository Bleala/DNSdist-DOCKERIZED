# Global variables
ARG ALPINE_VERSION="3.23.2" \
    IMAGE_VERSION="1.0.0" \
    DNSDIST_VERSION="2.0.2" \
    TINYCDB_VERSION="0.81"

# -----------------------------------------------------------------------------------------------

# https://www.dnsdist.org/install.html#installing-from-source
# Set Alpine as the base image
FROM alpine:${ALPINE_VERSION} AS dnsdist-build

# Build arguments
ARG ALPINE_VERSION \
    QUICHE_DOWNLOAD="https://github.com/cloudflare/quiche" \
    TINYCDB_VERSION \
    TINYCDB_DOWNLOAD="https://www.corpit.ru/mjt/tinycdb/tinycdb-${TINYCDB_VERSION}.tar.gz"

# Set shell to ash
SHELL ["/bin/ash", "-eo", "pipefail", "-c"]

# Install runtime dependencies
RUN apk update && \
    apk add --no-cache --virtual .build-deps \
      build-base=~0.5 \
      pkgconf=~2.5.1 \
      curl=~8.17.0 \
      git=~2.52.0 \
      boost-dev=~1.84.0 \
      lua5.4-dev=~5.4.8 \
      libedit-dev=~20251016.3.1 \
      libevent-dev=~2.1.12 \
      fstrm-dev=~0.6.1 \
      gnutls-dev=~3.8.11 \
      libbpf-dev=~1.6.2 \
      libxdp-dev=~1.5.7 \
      libcap-dev=~2.77 \
      libsodium-dev=~1.0.20 \
      lmdb-dev=~0.9.33 \
      net-snmp-dev=~5.9.4 \
      nghttp2-dev=~1.68.0 \
      openssl-dev=~3.5.4 \
      re2-dev=~2025.11.05 &&\
    # Clean up apk cache
    rm -rf /var/cache/apk/* && \
    # Install newest Rust and Cargo version (für quiche, DoQ and DoH3)
    curl --proto '=https' --tlsv1.3 -sSf https://sh.rustup.rs | sh -s -- -y

# Set workdir
WORKDIR /tmp/quiche-build

# Download quiche for DNS over HTTP/3 and DNS over QUIC support
RUN git clone "${QUICHE_DOWNLOAD}" . && \
    cargo build --examples



# -----------------------------------------------------------------------------------------------

# Set Alpine as the base image
FROM alpine:${ALPINE_VERSION} AS dnsdist

# Build arguments
ARG ALPINE_VERSION \
    IMAGE_VERSION

# Config directory
RUN mkdir -p /etc/dnsdist

# Set working directory
WORKDIR /etc/dnsdist

# Set build date argument here, because caching would break otherwise
# It is set during the build process in the reusable workflow
ARG BUILD_DATE

# Set labels
LABEL org.opencontainers.image.authors="Bleala" \
      org.opencontainers.image.vendor="Bleala" \
      org.opencontainers.image.version="${IMAGE_VERSION}" \
      org.opencontainers.image.created="${BUILD_DATE}" \
      org.opencontainers.image.title="DNSdist-DOCKERIZED" \
      org.opencontainers.image.description="DNSdist - to synchronise calendars and contacts between different storages. DOCKERIZED!" \
      org.opencontainers.image.documentation="https://www.dnsdist.org/install.html" \
      org.opencontainers.image.url="https://hub.docker.com/r/bleala/dnsdist" \
      org.opencontainers.image.source="https://github.com/Bleala/DNSdist-DOCKERIZED"

# DNS
EXPOSE 53/udp
EXPOSE 53/tcp
# DNS over HTTPS
EXPOSE 443/tcp
# DNS over HTTP/3
EXPOSE 443/udp
# DNS over TLS
EXPOSE 853/tcp
# DNS over QUIC
EXPOSE 853/udp
# Console
EXPOSE 5199/tcp
# DNSCrypt
EXPOSE 8443/tcp
EXPOSE 8443/udp
# Web UI
EXPOSE 8083/tcp

# Run dnsdist
