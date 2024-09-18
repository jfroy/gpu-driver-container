ARG TALOS_VERSION=1.7.6

FROM ghcr.io/jfroy/talos/nvidia-open-gpu-kernel-modules-production-pkg:$TALOS_VERSION AS kernel-modules

FROM nvcr.io/nvidia/cuda:12.6.0-base-ubuntu24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV NVIDIA_VISIBLE_DEVICES=void

ARG DRIVER_TYPE=passthrough
ENV DRIVER_TYPE=$DRIVER_TYPE

ARG DRIVER_VERSION=550.90.07
ENV DRIVER_VERSION=$DRIVER_VERSION

ARG KERNEL_VERSION=6.10.6-talos
ENV KERNEL_VERSION=$KERNEL_VERSION

ENV TALOS_VERSION=$TALOS_VERSION

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        curl \
        kmod \
        rsync \
    && curl -s https://download.nvidia.com/XFree86/Linux-x86_64/${DRIVER_VERSION}/NVIDIA-Linux-x86_64-${DRIVER_VERSION}-no-compat32.run -o nvidia.run && \
    chmod +x nvidia.run && \
    ./nvidia.run \
        --silent \
        --no-rpms \
        --no-backup \
        --no-kernel-modules \
        --no-x-check \
        --no-distro-scripts \
        --no-wine-files \
        --no-kernel-module-source \
        --no-check-for-alternate-installs \
        # the driver container has a mount on /lib/firmware, so we can't install the firmware there
        --override-file-type-destination=FIRMWARE:/nvidia/firmware/${DRIVER_VERSION} \
        --no-systemd \
    && rm nvidia.run

# Copy kernel modules that were built alongside the kernel
COPY --from=kernel-modules /lib/modules/${KERNEL_VERSION}/ /lib/modules/${KERNEL_VERSION}/

COPY nvidia-driver /usr/local/bin
WORKDIR  /drivers
ENTRYPOINT ["nvidia-driver", "init"]
