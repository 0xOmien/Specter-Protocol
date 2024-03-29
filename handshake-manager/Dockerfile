FROM --platform=arm64 rust:latest AS chef

# Build dir, use cargo chef to cache dependencies
WORKDIR /build
COPY ./rust-toolchain ./rust-toolchain
RUN cat rust-toolchain | xargs rustup toolchain install

# Install protoc, openssl, and pkg-config
RUN apt-get update && \
    apt-get install -y pkg-config && \
    apt-get install -y protobuf-compiler && \
    apt-get install -y libssl-dev && \
    apt-get install -y libclang-dev

# Install chef and generate a recipe
RUN cargo install cargo-chef

COPY . .
RUN cargo chef prepare --recipe-path recipe.json

# Disable compiler warnings and enable backtraces for panic
ENV RUSTFLAGS=-Awarnings
ENV RUST_BACKTRACE=1

# Build only the dependencies to cache them in this layer
RUN cargo chef cook --tests --recipe-path recipe.json

# Copy back in the full sources and build the tests
WORKDIR /build
COPY . .

WORKDIR /build/workers/handshake-manager-tests
RUN cargo build 

# Copy in the deployments file from the sequencer
COPY --from=sequencer /deployments.json /deployments.json