FROM rust:latest

RUN cargo install mdbook --no-default-features --features output --vers "^0.3.7"

CMD ["mdbook", "build"]