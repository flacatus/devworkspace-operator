FROM golang:1.13.7-alpine3.11 as builder

WORKDIR /devworkspace-operator

# Populate the module cache based on the go.{mod,sum} files.
COPY go.mod .
COPY go.sum .
RUN go mod download

# copy the rest of the sources code
COPY . .
# compile workspace controller binaries
RUN CGO_ENABLED=0 GOOS=linux go build \
  -o _output/bin/devworkspace-controller \
  -gcflags all=-trimpath=/ \
  -asmflags all=-trimpath=/ \
  cmd/manager/main.go

# Compile webhook binaries
RUN CGO_ENABLED=0 GOOS=linux go build \
  -o _output/bin/webhook-server \
  -gcflags all=-trimpath=/ \
  -asmflags all=-trimpath=/ \
  webhook/main.go

FROM registry.access.redhat.com/ubi8-minimal:8.2-349
COPY --from=builder /devworkspace-operator/_output/bin/devworkspace-controller /usr/local/bin/devworkspace-controller
COPY --from=builder /devworkspace-operator/_output/bin/webhook-server /usr/local/bin/webhook-server
COPY --from=builder /devworkspace-operator/internal-registry  internal-registry

ENV USER_UID=1001 \
    USER_NAME=devworkspace-controller

COPY build/bin /usr/local/bin
RUN  /usr/local/bin/user_setup

USER ${USER_UID}

ENTRYPOINT ["/usr/local/bin/entrypoint"]
CMD /usr/local/bin/devworkspace-controller
