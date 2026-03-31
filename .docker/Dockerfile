FROM rust:1.86.0-bullseye

RUN rustup target install thumbv6m-none-eabi
RUN cargo install flip-link --version 0.1.10
RUN rustup component add rustfmt
RUN rustup component add llvm-tools
RUN cargo install cargo-binutils
RUN cargo install elf2uf2-rs --locked
