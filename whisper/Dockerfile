FROM docker.io/rocker/r2u:22.04
LABEL org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.vendor="BNOSAC" \
      org.opencontainers.image.authors="Jan Wijffels <jwijffels@bnosac.be>"

## Common utitilities + dependencies for handling audio + OpenBLAS
RUN apt-get update \
    && apt-get install -y --no-install-recommends wget libcurl4-openssl-dev libssl-dev curl bash git \
    && apt-get install -y --no-install-recommends pkg-config libopenblas-dev libavfilter-dev software-properties-common dirmngr \
    && apt-get autoremove -y --quiet \
    && apt-get clean --quiet \
    && rm -rf /var/lib/apt/lists/*

## Install CUDA in case the hardware is a GPU - use CUDA 11-8 to be able to run Silero VAD with torch
RUN wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb \
    && apt-get update \
    && apt-get install -y --no-install-recommends software-properties-common dirmngr \
    && dpkg -i cuda-keyring_1.1-1_all.deb \
    && rm cuda-keyring_1.1-1_all.deb \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y nvidia-gds cuda-11-8 cuda-toolkit-11-8 cuda-drivers \
    && apt-get autoremove -y --quiet \
    && apt-get clean --quiet \
    && rm -rf /var/lib/apt/lists/*
ENV PATH=/usr/local/cuda-11.8/bin${PATH:+:${PATH}}
ENV LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
ENV CUDA_PATH=/usr/local/cuda-11.8
ENV CUDA=11.8

## General R packages
RUN apt-get update \
    && apt-get install -y --no-install-recommends r-cran-rcpp r-cran-av r-cran-data.table r-cran-jsonlite r-cran-zip \
    && apt-get autoremove -y --quiet \
    && apt-get clean --quiet \
    && rm -rf /var/lib/apt/lists/*

## Always enable OpenBLAS with audio.whisper
ENV WHISPER_OPENBLAS=1

## Install audio.whisper and VAD (audio.vadwebrtc/audio.vadsilero)
RUN R -e "install.packages('remotes')" \
    R -e "install.packages('audio')" \
    R -e "install.packages('abseil')" \
    R -e "install.packages('torch')"
RUN R -e "torch::install_torch()"
RUN R -e "remotes::install_github('bnosac/audio.vadwebrtc', ref = '0.2')" \
    R -e "remotes::install_github('bnosac/audio.vadsilero', ref = '0.2')"
RUN git clone https://github.com/bnosac/audio.whisper.git -b 0.4.1 \
    && R CMD build audio.whisper \
    && R CMD INSTALL audio.whisper_0.4.1.tar.gz \
    && rm -r audio.whisper

## Set the APP_DIR folder + download the medium model by default
ARG WHISPER_MODEL=medium
#ENV APP_DIR=/usr/local/src/whisper
ENV APP_DIR=/app
WORKDIR $APP_DIR
RUN mkdir $APP_DIR/models
ENV WHISPER_MODEL_DIR=$APP_DIR/models
RUN R -e "cat(sprintf('Downloading whisper gguf model <%s> to %s', Sys.getenv('WHISPER_MODEL', unset = 'medium'), Sys.getenv('WHISPER_MODEL_DIR')))"
RUN R -e "audio.whisper::whisper_download_model(Sys.getenv('WHISPER_MODEL', unset = 'medium'), version = '1.5.4')"

CMD ["R"]