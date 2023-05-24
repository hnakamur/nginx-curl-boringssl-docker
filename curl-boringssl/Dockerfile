FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get install -y curl build-essential cmake git autoconf libtool pkg-config \
                       zlib1g-dev libzstd-dev libbrotli-dev libidn2-dev libnghttp2-dev

# Install rustup
# https://gist.github.com/gfx/2885785bac51b56fa5bf2d5c8dd9f7c6
RUN curl -sS https://sh.rustup.rs | sh -s -- -y --profile minimal

# Build curl with quiche
# https://curl.se/docs/http3.html#quiche-version

# Build quiche and BoringSSL:
WORKDIR /src
RUN git clone --depth 1 --recursive https://github.com/cloudflare/quiche
WORKDIR /src/quiche
RUN $HOME/.cargo/bin/cargo build --package quiche --release --features ffi,pkg-config-meta,qlog
RUN mkdir quiche/deps/boringssl/src/lib
RUN ln -vnf $(find target/release -name libcrypto.a -o -name libssl.a) quiche/deps/boringssl/src/lib/

# Build curl
WORKDIR /src
RUN git clone --depth 1 https://github.com/curl/curl
WORKDIR /src/curl
RUN autoreconf -fi
RUN LDFLAGS="-Wl,-rpath,$PWD/../quiche/target/release" ./configure --prefix=/usr --libdir=/usr/lib/$(uname -m)-linux-gnu --with-nghttp2 --with-openssl=$PWD/../quiche/quiche/deps/boringssl/src --with-quiche=$PWD/../quiche/target/release
RUN make -j
RUN make install

ENTRYPOINT ["/usr/bin/curl"]
