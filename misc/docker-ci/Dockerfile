FROM ubuntu:16.04

RUN apt-get --yes update

# huge packages go first (for better cacheability)
RUN apt-get install --yes bison ruby-dev
RUN apt-get install --yes php-cgi

# tools for building and testing
RUN apt-get install --yes apache2-utils cmake cmake-data git memcached netcat-openbsd nghttp2-client redis-server wget sudo
RUN apt-get install --yes libev-dev libc-ares-dev libnghttp2-dev libssl-dev libuv1-dev zlib1g-dev

# clang-4.0 for fuzzing
RUN apt-get install -y clang-4.0
ENV PATH=/usr/lib/llvm-4.0/bin:$PATH

# curl with http2 support
RUN wget --no-verbose -O - https://curl.haxx.se/download/curl-7.57.0.tar.gz | tar xzf -
RUN (cd curl-7.57.0 && ./configure --prefix=/usr/local --with-nghttp2 --disable-shared && make && sudo make install)

# cpan modules
RUN apt-get install --yes cpanminus
RUN apt-get install --yes libfcgi-perl libfcgi-procmanager-perl libipc-signal-perl libjson-perl liblist-moreutils-perl libplack-perl libscope-guard-perl libtest-exception-perl libwww-perl libio-socket-ssl-perl
ENV PERL_CPANM_OPT="--mirror https://cpan.metacpan.org/"
RUN sudo cpanm --notest Starlet Test::TCP Protocol::HTTP2

# h2get
RUN apt-get install --yes golang && mkdir -p /golang && ln -sf /usr/local/bin /golang/bin
ENV GOPATH=/golang
RUN (go get github.com/summerwind/h2spec && cd /golang/src/github.com/summerwind/h2spec/cmd/h2spec && git checkout v2.0.0^ && go install)

# use dumb-init
RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 \
 && chmod +x /usr/local/bin/dumb-init

# create user
RUN useradd --create-home ci
RUN echo 'ci ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
WORKDIR /home/ci
USER ci

ENTRYPOINT ["/usr/local/bin/dumb-init"]
