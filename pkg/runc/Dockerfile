# Dockerfile to build linuxkit/runc for linuxkit
FROM linuxkit/alpine:e9313150d2cf07ad92b0adb86799dedd9a0476e4 as alpine
RUN \
  apk add \
  bash \
  gcc \
  git \
  libc-dev \
  libseccomp-dev \
  libseccomp-static \
  linux-headers \
  make \
  && true
ENV GOPATH=/go PATH=$PATH:/go/bin GO111MODULE=on GOFLAGS=-mod=vendor
COPY --from=golang:1.26.11-alpine3.23@sha256:db99415e2625ce0ee03eed09edf7f32ae8c58dc399f28cc602a87fe7238119e4 /usr/local/go /usr/local/go
ENV PATH=/usr/local/go/bin:$PATH GOTOOLCHAIN=local
ENV RUNC_COMMIT=491b69bab9fa206b984fb26ba07d3110d62e671f
RUN mkdir -p $GOPATH/src/github.com/opencontainers && \
  cd $GOPATH/src/github.com/opencontainers && \
  git clone https://github.com/opencontainers/runc.git
WORKDIR $GOPATH/src/github.com/opencontainers/runc
RUN git checkout $RUNC_COMMIT
RUN make static BUILDTAGS="seccomp" EXTRA_FLAGS="-buildmode pie" EXTRA_LDFLAGS="-extldflags \\\"-fno-PIC -static\\\""
RUN cp runc /usr/bin/

RUN mkdir -p /etc/init.d && ln -s /usr/bin/service /etc/init.d/010-onboot
RUN mkdir -p /etc/shutdown.d && ln -s /usr/bin/service /etc/shutdown.d/010-onshutdown

FROM scratch
WORKDIR /
ENTRYPOINT []
COPY --from=alpine /usr/bin/runc /usr/bin/
COPY --from=alpine /etc/init.d/ /etc/init.d/
COPY --from=alpine /etc/shutdown.d/ /etc/shutdown.d/
