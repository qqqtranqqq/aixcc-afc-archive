# Build infer image
FROM gcr.io/oss-fuzz-base/base-builder AS infer_build

RUN apt-get update && mkdir -p /usr/share/man/man1 && apt-get install --yes --no-install-recommends autoconf automake bubblewrap bzip2 cmake curl g++ gcc git libc6-dev libgmp-dev libmpfr-dev libsqlite3-dev sqlite3 make opam openjdk-11-jdk-headless patch patchelf pkg-config python3 python3-distutils unzip xz-utils zlib1g-dev

ARG OPAM_VERSION=2.2.0
RUN curl -fsSL "https://github.com/ocaml/opam/releases/download/${OPAM_VERSION}/opam-${OPAM_VERSION}-x86_64-linux" -o /usr/local/bin/opam && \
    chmod +x /usr/local/bin/opam


RUN opam init --reinit --bare --disable-sandboxing --yes --auto-setup



# TODO this should probably be a specific tag/commit

#ARG STDCOMPAT_VERSION=19.0

#ARG CAMLZIP_VERSION=1.13

#RUN opam pin add -n stdcompat 19.0

RUN cd / && git clone https://github.com/facebook/infer/
WORKDIR /infer

RUN git checkout 1b1366e64ae46f1c886626e8cf51d81ca5ee4322

COPY ./patches/build-infer.sh /infer/build-infer.sh
COPY ./patches/infer.opam.locked /infer/opam/infer.opam.locked


RUN update-alternatives --install /usr/bin/c++ c++ /usr/local/bin/clang++ 100 && update-alternatives --install /usr/bin/cc cc /usr/local/bin/clang 100
RUN ./build-infer.sh --only-setup-opam

COPY ./patches/macro.patch /infer/macro.patch
RUN git apply macro.patch
RUN JOBS=$(nproc) NO_CMAKE_STRIP=1 CXXFLAGS='-pthread' CFLAGS='-pthread' LDFLAGS='-pthread' LIBS='-pthread' ./build-infer.sh -y clang

# copy patches after the opam setup and clang build--that takes a long time and we don't touch it
COPY ./patches/p0.patch /infer/p0.patch
COPY ./patches/p1.patch /infer/p1.patch
COPY ./patches/p2.patch /infer/p2.patch
RUN git apply p0.patch && git apply p1.patch && git apply p2.patch

RUN JOBS=$(nproc) NO_CMAKE_STRIP=1 CXXFLAGS='-pthread' CFLAGS='-pthread' LDFLAGS='-pthread' LIBS='-pthread' ./build-infer.sh -y clang
