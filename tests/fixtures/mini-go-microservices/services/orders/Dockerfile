FROM golang:1.22-alpine AS build
WORKDIR /src
COPY . .
RUN go build -o /out/orders ./...

FROM alpine:3.19
COPY --from=build /out/orders /usr/local/bin/orders
EXPOSE 8081
ENTRYPOINT ["/usr/local/bin/orders"]
