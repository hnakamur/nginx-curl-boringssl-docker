FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
# golang is needed to build BoringSSL
RUN apt-get install -y build-essential git cmake golang libpcre3-dev zlib1g-dev

# Build BoringSSL
# adapted from https://everything.curl.dev/build/tls/boringssl
WORKDIR /src
RUN git clone --depth 1 https://github.com/google/boringssl
WORKDIR /src/boringssl
RUN mkdir build
WORKDIR /src/boringssl/build
RUN cmake ..
RUN make -j

# Build nginx with http3 support
WORKDIR /src
RUN git clone --depth 1 -b release-1.25.0 https://github.com/nginx/nginx
WORKDIR /src/nginx
RUN ./auto/configure --prefix=/usr \
                     --with-debug \
                     --with-http_v2_module \
                     --with-http_v3_module \
                     --with-cc-opt="-I../boringssl/include"     \
                     --with-ld-opt="-L../boringssl/build/ssl    \
                                    -L../boringssl/build/crypto"
RUN make -j
RUN make install

RUN mkdir -p /etc/nginx /var/log/nginx
RUN ln -s /dev/stdout /var/log/nginx/access.log
RUN ln -s /dev/stderr /var/log/nginx/error.log

CMD ["/usr/sbin/nginx", "-c", "/etc/nginx/nginx.conf", "-g", "daemon off;"]
