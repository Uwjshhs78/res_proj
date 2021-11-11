FROM rust:1-slim-buster AS base

RUN apt-get update \
  && apt-get install -y libssl-dev pkg-config \
  && rm -rf /var/lib/apt/lists/*

ENV USER=root

WORKDIR /code
RUN cargo init
COPY Cargo.toml /code/Cargo.toml
RUN cargo fetch

COPY src /code/src
COPY template /code/template

CMD [ "cargo", "test", "--offline" ]

FROM base AS builder

RUN cargo build --release --offline

FROM debian:buster-slim

LABEL org.label-schema.schema-version="1.0"
LABEL org.label-schema.docker.cmd="docker run -d -p 3000:3000 -e TEMPLATE_ROOT=/templates -e SMTP_LOCALHOST=localhost -e SMTP_PORT=25 -e SMTP_USERNAME=username -e SMTP_PASSWORD=password -e SMTP_POOL_MAX_SIZE=10 -e TEMPLATE_PROVIDER=local jdrouet/catapulte"
LABEL org.label-schema.vcs-url="https://jolimail.io"
LABEL org.label-schema.url="https://github.com/jdrouet/catapulte"
LABEL org.label-schema.description="Service to convert mrml to html and send it by email"
LABEL maintaner="Jeremie Drouet <jeremie.drouet@gmail.com>"

RUN apt-get update \
  && apt-get install -y ca-certificates libssl1.1 \
  && rm -rf /var/lib/apt/lists/*

ENV ADDRESS=0.0.0.0
ENV PORT=3000
ENV RUST_LOG=info
ENV TEMPLATE_ROOT=/templates

COPY --from=builder /code/target/release/catapulte /usr/bin/catapulte

EXPOSE 3000

ENTRYPOINT [ "/usr/bin/catapulte" ]
