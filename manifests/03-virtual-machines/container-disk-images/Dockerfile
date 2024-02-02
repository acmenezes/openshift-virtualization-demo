FROM registry.access.redhat.com/ubi8/ubi:latest AS builder
ADD --chown=107:107 Fedora-Cloud-Base-39-1.5.x86_64.qcow2 /disk/ 
RUN chmod 0440 /disk/*

FROM scratch
COPY --from=builder /disk/* /disk/