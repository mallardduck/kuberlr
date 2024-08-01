ARG BCI_VERSION=15.6
FROM registry.suse.com/bci/bci-busybox:${BCI_VERSION} AS final
FROM registry.suse.com/bci/bci-base:${BCI_VERSION} AS build

ARG TARGETARCH
ENV TARGETARCH=$TARGETARCH

RUN mkdir /chroot
COPY --from=final / /chroot/

# The final image does not contain zypper, --installroot is used to
# install all artefacts within a dir (/chroot) that can then be copied
# over to a scratch image.
RUN zypper --non-interactive refresh && \
    zypper --installroot /chroot -n rm busybox-vi busybox-links && \
    zypper --installroot /chroot -n in bash-completion && \
    zypper --installroot /chroot clean -a && \
    rm -rf /chroot/var/cache/zypp/* /chroot/var/log/zypp/* /chroot/etc/zypp/

COPY ./dist /work-tmp
WORKDIR /work-tmp
RUN set -x \
    && if [ $TARGETARCH = "amd64" ]; then TARGETARCH="${TARGETARCH}_v1"; fi \
    && KUBERLR_PATH=kuberlr_linux_${TARGETARCH}/kuberlr \
    && mv $KUBERLR_PATH /chroot/bin/kuberlr \
    && cd /chroot/bin && ln -s ./kuberlr ./kubectl

RUN useradd -u 1000 -U -m kuberlr \
    && cp /etc/passwd /chroot/etc/passwd \
    && cp /etc/group /chroot/etc/group \
    && cp -r /home/kuberlr /chroot/home/kuberlr \
    && echo '. /etc/profile.d/bash_completion.sh' >> /chroot/home/kuberlr/.bashrc \
    && echo 'alias k="kubectl"' >> /chroot/home/kuberlr/.bashrc \
    && echo 'alias ks="kubectl -n kube-system"' >> /chroot/home/kuberlr/.bashrc \
    && echo 'source <(kubectl completion bash)' >> /chroot/home/kuberlr/.bashrc \
    && echo 'LANG=en_US.UTF-8' >> /chroot/home/kuberlr/.bashrc \
    && echo 'PS1="> "' >> /chroot/home/kuberlr/.bashrc \
    && mkdir /chroot/home/kuberlr/.kube \
    && mkdir /chroot/home/kuberlr/.kuberlr \
    && touch /chroot/home/kuberlr/.kuberlr/kuberlr.conf \
    && echo "AllowDownload = true" >> /chroot/home/kuberlr/.kuberlr/kuberlr.conf \
    && echo "Timeout = 12" >> /chroot/home/kuberlr/.kuberlr/kuberlr.conf \
    && chown -R 1000:1000 /chroot/home/kuberlr

FROM scratch
COPY --from=build /chroot /

USER kuberlr

WORKDIR /home/kuberlr
ENTRYPOINT ["/bin/kuberlr"]
CMD ["help"]