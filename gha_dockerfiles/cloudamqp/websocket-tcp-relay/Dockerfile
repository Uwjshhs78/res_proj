FROM 84codes/crystal:1.1.1-alpine-latest AS builder

WORKDIR /tmp
COPY shard.yml shard.lock ./
RUN shards install --production
COPY src/ src/
RUN shards build --release --production --static
RUN strip bin/*

FROM scratch
USER 2:2
COPY --from=builder /etc/ssl/cert.pem /etc/ssl/
COPY --from=builder /tmp/bin/* /
EXPOSE 15670
ENTRYPOINT ["/websocket-tcp-relay"]
