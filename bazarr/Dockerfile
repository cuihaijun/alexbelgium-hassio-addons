#============================#
#  ALEXBELGIUM'S DOCKERFILE  #
#============================#
#           _.------.
#       _.-`    ('>.-`"""-.
# '.--'`       _'`   _ .--.)
#    -'         '-.-';`   `
#    ' -      _.'  ``'--.
#        '---`    .-'""`
#               /`
#=== Home Assistant Addon ===#

#################
# 1 Build Image #
#################

ARG BUILD_FROM
ARG BUILD_VERSION
FROM ${BUILD_FROM}

##################
# 2 Modify Image #
##################

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN \
    # Correct config folder
    grep -rl " /config" /etc/cont-init.d | xargs sed -i 's| /config| /config/addons_config/bazarr|g' \
    && grep -rl " /config" /etc/services.d | xargs sed -i 's| /config| /config/addons_config/bazarr|g' \
    \
    # Allow UID and GID setting
    && sed -i 's/bash/bashio/g' /etc/cont-init.d/10-adduser \
    && sed -i 's/{PUID:-911}/(bashio::config "PUID")/g' /etc/cont-init.d/10-adduser \
    && sed -i 's/{PGID:-911}/(bashio::config "PGID")/g' /etc/cont-init.d/10-adduser

##################
# 3 Install apps #
##################

# Add rootfs
COPY rootfs/ /

# Modules
ARG MODULES="00-banner.sh 92-local_mounts.sh 92-smb_mounts.sh"

# Automatic modules download
RUN if ! command -v bash >/dev/null 2>/dev/null; then (apt-get update && apt-get install -yqq --no-install-recommends bash || apk add --no-cache bash) >/dev/null; fi \
    && if ! command -v curl >/dev/null 2>/dev/null; then (apt-get update && apt-get install -yqq --no-install-recommends curl || apk add --no-cache curl) >/dev/null; fi \
    && mkdir -p /etc/cont-init.d \
    && for scripts in $MODULES; do echo "$scripts" && curl -f -L -s -S "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/$scripts" -o /etc/cont-init.d/"$scripts" && [ "$(sed -n '/\/bin/p;q' /etc/cont-init.d/"$scripts")" != "" ] || (echo "script failed to install $scripts" && exit 1); done \
    && chmod -R 755 /etc/cont-init.d || printf '%s\n' "${MODULES}" >/MODULESFILE

# Manual apps
ENV PACKAGES=""

# Automatic apps & bashio
RUN if ! command -v bash >/dev/null 2>/dev/null; then (apt-get update && apt-get install -yqq --no-install-recommends bash || apk add --no-cache bash) >/dev/null; fi \
    && if ! command -v curl >/dev/null 2>/dev/null; then (apt-get update && apt-get install -yqq --no-install-recommends curl || apk add --no-cache curl) >/dev/null; fi \
    && curl -f -L -s -S "https://raw.githubusercontent.com/alexbelgium/hassio-addons/master/.templates/automatic_packages.sh" --output /automatic_packages.sh \
    && chmod 777 /automatic_packages.sh \
    && eval /./automatic_packages.sh "${PACKAGES:-}" \
    && rm /automatic_packages.sh || printf '%s\n' "${PACKAGES:-}" > /ENVFILE

################
# 4 Entrypoint #
################

############
# 5 Labels #
############

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="alexbelgium (https://github.com/alexbelgium)" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Add-ons" \
    org.opencontainers.image.authors="alexbelgium (https://github.com/alexbelgium)" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/alexbelgium" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
