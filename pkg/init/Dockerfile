# Dockerfile to build linuxkit/init for linuxkit
FROM linuxkit/alpine:e9313150d2cf07ad92b0adb86799dedd9a0476e4 AS build
RUN apk add --no-cache --initdb alpine-baselayout make gcc musl-dev git linux-headers

ADD usermode-helper.c ./
RUN LDFLAGS=-static CFLAGS=-Werror make usermode-helper

RUN apk add --no-cache musl-dev
COPY --from=golang:1.26.11-alpine3.23@sha256:db99415e2625ce0ee03eed09edf7f32ae8c58dc399f28cc602a87fe7238119e4 /usr/local/go /usr/local/go
ENV PATH=/usr/local/go/bin:$PATH GOTOOLCHAIN=local
ENV GOPATH=/go PATH=$PATH:/go/bin
ARG GOPKGVERSION
ENV ldflags="-X main.Version=$GOPKGVERSION"

# Hack to work around an issue with go on arm64 requiring gcc
RUN [ $(uname -m) = aarch64 ] && apk add --no-cache gcc || true

COPY . /go/src
RUN go-compile.sh /go/src/cmd/init
RUN go-compile.sh /go/src/cmd/rc.init
# this makes sure that the multi stage build copies as a symlink
RUN mkdir /tmp/bin && cd /tmp/bin/ && cp /go/bin/rc.init . && ln -s rc.init rc.shutdown

RUN go-compile.sh /go/src/cmd/service

# volumes link to start
RUN mkdir -p /etc/init.d && ln -s /usr/bin/service /etc/init.d/005-volumes

FROM linuxkit/alpine:e9313150d2cf07ad92b0adb86799dedd9a0476e4 AS mirror
RUN mkdir -p /out/etc/apk && cp -r /etc/apk/* /out/etc/apk/ && \
    printf '%s\n' 'https://dl-cdn.alpinelinux.org/alpine/v3.23/main' > /out/etc/apk/repositories
RUN apk add --no-cache --initdb -p /out \
    alpine-baselayout=3.7.2-r0 alpine-baselayout-data=3.7.2-r0 \
    busybox=1.37.0-r30 busybox-binsh=1.37.0-r30 musl=1.2.5-r24

# Add /etc/ssl/certs so it can be bind-mounted into metadata package
RUN mkdir -p /out/etc/ssl/certs

# Remove cache residuals. We retain apk for SBOM tools
RUN rm -rf /out/var/cache


FROM scratch
ENTRYPOINT []
CMD []
WORKDIR /
COPY --from=build /go/bin/init /
COPY --from=build /tmp/bin /bin/
COPY --from=build /go/bin/service /usr/bin/
COPY --from=build usermode-helper /sbin/
COPY --from=build /etc/init.d/ /etc/init.d/
COPY --from=mirror /out/ /
COPY etc etc/
