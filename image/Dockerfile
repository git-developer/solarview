#
# This Dockerfile allows to package a single SolarView app. An app in this
# context is either the main application solarview-fb or one of the so called
# proxies (e.g. steca-b, d0-fb).
#
# This Dockerfile supports multiple architectures (e.g. x86, armhf).
#
# The image is created in two stages. The first stage downloads the binaries,
# selects the correct binaries for the architecture and saves version and release
# date. The second image prepares runtime dependencies and defines the start
# command.

# The RUNTIME_BASE_IMAGE is architecture dependent. It has to be declared before the
# first FROM to be used as variable in the FROM directive of the 'runtime' image
ARG RUNTIME_BASE_IMAGE

# the first image 'builder' prepares the runtime image
FROM alpine AS builder
ARG BUILD_ID
LABEL build="${BUILD_ID}"
ARG ARCH_ID
ARG APP_NAME
ARG APP_ARCHIVE_URL="http://www.solarview.info/downloads/${APP_NAME}.zip"
ARG APP_ARCHIVE_FILE="/build/${APP_NAME}.zip"
ARG APP_BINARIES="${APP_NAME}"
ARG APP_HOME="/opt/${APP_NAME}"

WORKDIR "${APP_HOME}"
ADD "${APP_ARCHIVE_URL}" "${APP_ARCHIVE_FILE}"
# the 'unzip' package is used because busybox-unzip does not retain timestamps
RUN apk add unzip && unzip "${APP_ARCHIVE_FILE}"
# select the correct binaries for the runtime architecture
RUN [ -z "${APP_BINARIES}" ] || for binary in ${APP_BINARIES}; do   \
      mv "${binary}" "${binary}.71xx"                             &&\
      chmod +x "${binary}.${ARCH_ID}"                             &&\
      ln -s "${binary}.${ARCH_ID}" "${binary}"                     ;\
    done
WORKDIR /build/arch-context
COPY "arch/${ARCH_ID}/build-context" .
RUN [ ! -x ./hook ] || ./hook
WORKDIR /build/app-context
COPY "app/${APP_NAME}/build-context" .
RUN [ ! -x ./hook ] || ./hook
WORKDIR "${APP_HOME}"
RUN >>/env echo "RELEASE_DATE=$(stat    -c %y "${APP_ARCHIVE_FILE}" | cut -d ' ' -f 1)"
RUN >>/env echo "APP_DATE=$(    stat -L -c %y "${APP_NAME}"         | cut -d ' ' -f 1)"

# the second image 'runtime' is the main image that is used at runtime
FROM "${RUNTIME_BASE_IMAGE}" AS runtime
ARG ARCH_ID
ARG APP_NAME
ENV APP_NAME="${APP_NAME}"
ENV APP_HOME="/opt/${APP_NAME}"
ENV APP_RUNTIME="/var/opt/${APP_NAME}"

COPY --from=builder "${APP_HOME}" "${APP_HOME}"
COPY --from=builder "/env" "/env"
WORKDIR /runtime/arch-context
COPY "arch/${ARCH_ID}/runtime-context" .
RUN [ ! -x ./hook ] || ./hook
WORKDIR /runtime/app-context
COPY "app/${APP_NAME}/runtime-context" .
RUN [ ! -x ./hook ] || ./hook

# a temp file is used to fetch the version, because SolarView apps hang when 
# they are piped
RUN log=$(mktemp) &&\
    "${APP_HOME}/${APP_NAME}" -v >>"${log}" &&\
    >>/env echo APP_VERSION=$(sed -n -E 's/.* V ([^\t ]+).*/\1/p' "${log}") &&\
    rm "${log}"

#
# When the container is started, the files from the release archive are copied to
# a runtime directory, which is mounted to the host. When a file on the host is
# newer, it is not overwritten. This technique allows to update the binaries and
# keep the user data without a special update mechanism.
#
# The combination of the main binary and 'tail' is necessary because the main
# binary goes to background right after its start, and nothing remains in the
# foreground. Without 'tail', the docker container would exit immediately.
#
WORKDIR "${APP_RUNTIME}"
CMD  [ "/bin/sh", "-c", "cp -u -a \"${APP_HOME}\"/* \"${APP_RUNTIME}/\" && \"./${APP_NAME}\" ${ARGS} && exec tail -f /dev/null" ]
VOLUME "${APP_RUNTIME}"
