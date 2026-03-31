FROM rust:1.94.1-bookworm

RUN rustup target install thumbv6m-none-eabi
RUN cargo install flip-link --version 0.1.12
RUN rustup component add rustfmt
RUN rustup component add llvm-tools
RUN cargo install cargo-binutils
RUN cargo install elf2uf2-rs --locked
