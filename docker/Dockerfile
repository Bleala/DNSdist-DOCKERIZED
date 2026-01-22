# Global variables
ARG DEBIAN_VERSION="13.3-slim" \
    DNSDIST_VERSION="2.0.2" \
    IMAGE_VERSION="1.0.0"

# https://www.dnsdist.org/install.html#installing-from-source
# Set Debian as the base image
FROM debian:${DEBIAN_VERSION} AS dnsdist

# Build arguments
ARG DEBIAN_FRONTEND="noninteractive" \
    DEBIAN_VERSION \
    DNSDIST_VERSION \
    IMAGE_VERSION

# Set shell options
SHELL ["/bin/bash", "-e", "-u", "-x", "-o", "pipefail", "-c"]

# Set environment variables
ENV TZ="Europe/Vienna"

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD nslookup -type=NS -timeout=1 -retry=1 . 127.0.0.1 || exit 1

# Install runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl=8.14.1-2+deb13u2 \
        gpg=2.4.7-21+deb13u1+b1 \
        ca-certificates=20250419 \
        openssl=3.5.4-1~deb13u1 \
        bind9-dnsutils=1:9.20.15-1~deb13u1 \
        adduser=3.152 \
        procps=2:4.0.4-9 \
        tini=0.19.0-3+b5 \
        tzdata=2025b-4+deb13u1 && \
    # Clean up
    apt-get clean && \
    # Delete apt lists
    rm -rf /var/lib/apt/lists/*

# Install dnsdist from PowerDNS repository
    # Add PowerDNS GPG key
RUN install -d /etc/apt/keyrings; curl https://repo.powerdns.com/FD380FBB-pub.asc | \
      tee /etc/apt/keyrings/dnsdist-20-pub.asc > /dev/null && \
    # Add PDNS repository
    # https://repo.powerdns.com/
    echo "deb [signed-by=/etc/apt/keyrings/dnsdist-20-pub.asc] http://repo.powerdns.com/debian trixie-dnsdist-20 main" \
      > /etc/apt/sources.list.d/pdns.list && \
    # Set package pinning to prefer PowerDNS packages
    printf "Package: dnsdist*\nPin: origin repo.powerdns.com\nPin-Priority: 600" > /etc/apt/preferences.d/dnsdist-20 && \
    # Install dnsdist from PowerDNS repository
    apt-get update && \
    apt-get install -y --no-install-recommends dnsdist=2.0.2-1pdns.debian13 && \
    # Clean up
    apt-get clean && \
    # Delete apt lists
    rm -rf /var/lib/apt/lists/*

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
      org.opencontainers.image.description="DNSdist - a highly DNS-, DoS- and abuse-aware loadbalancer. DOCKERIZED!" \
      org.opencontainers.image.documentation="https://www.dnsdist.org/install.html" \
      org.opencontainers.image.url="https://hub.docker.com/r/bleala/dnsdist" \
      org.opencontainers.image.source="https://github.com/Bleala/DNSdist-DOCKERIZED"

# DNS
EXPOSE 53/tcp
EXPOSE 53/udp
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
# Web UI
EXPOSE 8083/tcp
# DNSCrypt
EXPOSE 8443/tcp
EXPOSE 8443/udp

# Run dnsdist
ENTRYPOINT ["/usr/bin/tini", "--", "/usr/bin/dnsdist"]
CMD ["--supervised", "--disable-syslog", "--uid", "_dnsdist", "--gid", "_dnsdist", "-C", "/etc/dnsdist/dnsdist.yml"]