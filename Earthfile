VERSION 0.6
FROM ubuntu:20.04
WORKDIR /workdir

install:
    RUN apt-get update \
     && apt-get upgrade -y \
     && apt-get install --no-install-recommends -y python3 g++ ca-certificates \
        apt-transport-https curl gnupg git libelf-dev libcap-dev
    RUN curl -fsSL https://bazel.build/bazel-release.pub.gpg \
        | gpg --dearmor > /usr/share/keyrings/bazel-archive-keyring.gpg \
     && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" \
        > /etc/apt/sources.list.d/bazel.list \
     && apt-get update \
     && apt-get install --no-install-recommends -y bazel
    RUN curl -fsSL https://go.dev/dl/go1.19.5.linux-amd64.tar.gz \
        | tar -xzf - -C /opt \
     && ln -s /opt/go/bin/go /usr/local/bin/go

build-pprof:
    FROM +install
    GIT CLONE https://github.com/google/pprof.git pprof
    WORKDIR pprof
    RUN go build
    SAVE ARTIFACT pprof AS LOCAL out/

build-perf-data-converter:
    FROM +install
    GIT CLONE https://github.com/google/perf_data_converter.git perf_data_converter
    WORKDIR perf_data_converter
    RUN bazel build src:perf_to_profile
    SAVE ARTIFACT bazel-bin/src/perf_to_profile AS LOCAL out/ 

build-perfetto:
    FROM +install
    GIT CLONE https://android.googlesource.com/platform/external/perfetto/ perfetto
    WORKDIR perfetto
    RUN tools/install-build-deps
    RUN tools/gn gen --args='is_debug=false' out/linux
    RUN tools/ninja -C out/linux tracebox
    SAVE ARTIFACT out/linux/tracebox AS LOCAL out/

build-all:
    BUILD +build-pprof
    BUILD +build-perf-data-converter
    BUILD +build-perfetto
