# Build rust
FROM rust:bookworm as rust-build

RUN apt-get update && \
	apt-get install pkg-config musl-tools libssl-dev

RUN rustup update
ENV RUSTFLAGS='--remap-path-prefix $HOME=~ -C target-feature=+crt-static'

COPY . /src
WORKDIR /src

RUN rustup target add x86_64-unknown-linux-musl
RUN cargo build --target x86_64-unknown-linux-musl --release

# DUMMY web static file
FROM alpine as web
RUN mkdir /web && echo 'Piss off' > /web/index.html

# Final container
FROM scratch
WORKDIR /

COPY --from=rust-build /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=rust-build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

COPY --from=rust-build /src/target/x86_64-unknown-linux-musl/release/m3u-filter /m3u-filter
COPY --from=web /web /web
# config should be mounted as volume
# COPY ./config /config

ENTRYPOINT ["/m3u-filter"]
CMD ["-s", "-p", "/config"]
