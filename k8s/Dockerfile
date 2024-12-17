FROM golang:1.23.4 AS builder
WORKDIR /app
COPY . .
RUN git apply ./patches/delete-after-scrape.patch
RUN make build

FROM debian:bullseye-slim
WORKDIR /app
COPY --from=builder /app/graphite_exporter .
EXPOSE 9108 9109
ENTRYPOINT [ "/app/graphite_exporter" ]
