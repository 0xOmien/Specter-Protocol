# Used for running integration tests on a simulated MPC network

FROM --platform=arm64 rust:latest AS chef

# Create a build dir and add local dependencies
WORKDIR /build

# Build the rust toolchain before adding any dependencies; this is the slowest
# step and we would like to cache it before anything else
COPY ./rust-toolchain ./rust-toolchain
RUN cat rust-toolchain | xargs rustup toolchain install

# Install cargo-chef
RUN cargo install cargo-chef

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /build/recipe.json recipe.json

# Install protoc, openssl, and pkg-config
RUN apt-get update && \
    apt-get install -y pkg-config && \
    apt-get install -y protobuf-compiler && \
    apt-get install -y libssl-dev && \
    apt-get install -y libclang-dev

# Disable compiler warnings and enable backtraces for panic debugging
ENV RUSTFLAGS=-Awarnings
ENV RUST_BACKTRACE=1

# Build only the dependencies to cache them in this layer
RUN cargo chef cook --release --recipe-path recipe.json

# Build the relayer
COPY . .
RUN cargo build --release

# Release stage
FROM --platform=arm64 debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y libssl-dev && \
    apt-get install -y ca-certificates

# Copy the binary from the build stage
COPY --from=builder /build/target/release/renegade-relayer /bin/renegade-relayer

ENTRYPOINT [ "/bin/renegade-relayer", "--config-file", "/config.toml" ]