ARG BUILD_FROM
FROM $BUILD_FROM

# Add env
ENV LANG=C.UTF-8
# Set CUPS environment variables for data persistence
ENV CUPS_DATADIR=/data/cups
ENV CUPS_CACHEDIR=/data/cups/cache
ENV CUPS_LOGDIR=/data/cups/logs
ENV CUPS_STATEDIR=/data/cups/state
ENV CUPS_SERVERROOT=/data/cups/config

# Setup base
RUN \
    echo "http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main" > /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community" >> /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories && \
    apk update && \
    apk add --no-cache \
        avahi \
        avahi-compat-libdns_sd \
        cups \
        cups-filters \
        dbus \
        ghostscript \
        hplip \
        libjpeg-turbo \
        net-snmp

# Copy data
COPY rootfs /

# Fix permissions for s6-overlay scripts
RUN chmod -R 755 /etc/services.d && \
    chmod -R 755 /etc/cont-init.d

# Expose CUPS web interface port
EXPOSE 631

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s \
  CMD /usr/bin/lpstat -r || exit 1
