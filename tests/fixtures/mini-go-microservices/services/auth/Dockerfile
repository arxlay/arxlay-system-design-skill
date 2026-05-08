FROM golang:1.22-alpine AS build
WORKDIR /src
COPY . .
RUN go build -o /out/auth ./...

FROM alpine:3.19
COPY --from=build /out/auth /usr/local/bin/auth
EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/auth"]
