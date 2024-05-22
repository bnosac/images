ARG R_VERSION=4.4.0
ARG IMAGE_REGISTRY=docker.io
ARG IMAGE_ORG=rocker
FROM $IMAGE_REGISTRY/$IMAGE_ORG/cuda:$R_VERSION
LABEL org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.vendor="BNOSAC" \
      org.opencontainers.image.authors="Jan Wijffels <jwijffels@bnosac.be>"

CMD ["R"]