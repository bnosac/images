##
## Ubuntu 24.04 VS Code Server
##
FROM ghcr.io/astral-sh/uv:0.11.7 AS uv
FROM ghcr.io/rocker-org/rstudio:4.6.0 AS rstudio
FROM codercom/code-server:4.116.0-noble
LABEL org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.vendor="BNOSAC" \
      org.opencontainers.image.authors="Jan Wijffels <jwijffels@bnosac.be>"

## Timezone/Language
ENV TZ="Etc/UTC"
ENV LANG=en_US.UTF-8

## Julia version
ENV JULIA_VERSION="1.12.6"
## Python version is default Python from Ubuntu 24.04
ENV PYTHON_VERSION="3.12"
## R version
ENV R_VERSION="4.6.0"

##
## General system requirements
##
USER root
RUN apt-get update && DEBIAN_FRONTEND=noninteractive TZ=$TZ apt-get install -y --no-install-recommends \
    tzdata wget libcurl4-openssl-dev libssl-dev curl bash git acl sudo \
    pkg-config software-properties-common cmake \
    unixodbc-dev \    
    && rm -rf /var/lib/apt/lists/*
SHELL ["/bin/bash", "-c"]

##############################################################################################################
## SQL Server ODBC driver + command-line tools
## https://learn.microsoft.com/en-us/sql/linux/install-upgrade/setup-tools
##
RUN curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg \
    && echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/ubuntu/24.04/prod noble main" \
       > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql18 mssql-tools18 \
    && rm -rf /var/lib/apt/lists/*
ENV PATH="$PATH:/opt/mssql-tools18/bin"

##############################################################################################################
## Install R from source and littler to be able to run the installation of Julia / Python
##
COPY --from=rstudio /rocker_scripts/ /rocker_scripts/
ENV PATH="/rocker_scripts/bin:${PATH}"

ENV CRAN="https://p3m.dev/cran/__linux__/noble/latest"
RUN /rocker_scripts/install_R_source.sh
## littler and all kind of symlinks for it
RUN apt-get update && apt-get install -y libpcre2-dev libzstd-dev liblzma-dev libbz2-dev zlib1g-dev libdeflate-dev libicu-dev gcc && rm -rf /var/lib/apt/lists/*
ENV R_HOME="/usr/local/lib/R"
ENV CRAN="https://packagemanager.posit.co/cran/latest"
ENV CRAN="https://p3m.dev/cran/__linux__/noble/latest"
ENV CRAN="https://cran.r-project.org/"
RUN echo "options(repos = c(CRAN = '${CRAN}'), download.file.method = 'libcurl')" >>"${R_HOME}/etc/Rprofile.site"
RUN echo "options(HTTPUserAgent = sprintf('R/%s R (%s)', getRversion(), paste(getRversion(), R.version['platform'], R.version['arch'], R.version['os'])))" >> "${R_HOME}/etc/Rprofile.site"
RUN Rscript -e "install.packages(c('littler'), clean = TRUE)"
RUN /rocker_scripts/setup_R.sh

##
## Default R packages
##  note: stringr/stringi or to be able to do rmarkdown::run from the RStudio UI
##
USER root
RUN Rscript -e "install.packages(c('curl', 'openssl', 'remotes', 'httr'), clean = TRUE)"
RUN Rscript -e "install.packages(c('jsonlite', 'yaml', 'zip', 'data.table', 'Rcpp', 'reticulate'), clean = TRUE)"
RUN Rscript -e "install.packages(c('shiny', 'knitr', 'htmltools', 'markdown', 'rmarkdown', 'shinydashboard', 'flexdashboard', 'bslib', 'fontawesome', 'sass', 'farver', 'DT'), clean = TRUE)"
RUN Rscript -e "install.packages(c('stringr', 'stringi'), clean = TRUE)"
RUN Rscript -e "install.packages(c('languageserver', 'httpgd'), clean = TRUE)"
RUN Rscript -e "install.packages(c('odbc'), clean = TRUE)"

##############################################################################################################
## Install Quarto + Julia / Python / Jupyter / Geospatial tools
##
USER root
RUN /rocker_scripts/install_julia.sh
RUN /rocker_scripts/install_python.sh
RUN /rocker_scripts/install_quarto.sh
RUN /rocker_scripts/install_pandoc.sh
RUN /rocker_scripts/install_jupyter.sh
RUN mkdir /etc/cont-init.d
RUN /rocker_scripts/install_rstudio.sh
## RUN /rocker_scripts/install_geospatial.sh

##############################################################################################################
## Create user appuser which will be used as startup and which has rights to sudo with password
##
RUN useradd --create-home --home-dir /home/appuser --shell /bin/bash appuser
RUN echo "appuser ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers.d/appuser \
    && chmod 0440 /etc/sudoers.d/appuser

##############################################################################################################
## Install Python 3.12 and Python package dependencies. Use uv package manager
##    uv python install 3.12 && uv python pin 3.12
##
COPY --from=uv /uv /uvx /bin/
RUN uv python install $PYTHON_VERSION && uv python pin $PYTHON_VERSION
USER appuser
WORKDIR /home/appuser
COPY requirements.txt .
RUN uv venv .venv \      
    && source .venv/bin/activate \
    && uv pip install --no-cache --upgrade pip \      
    && uv pip install --no-cache -r requirements.txt \      
    && rm requirements.txt
ENV RETICULATE_PYTHON="/home/appuser/.venv"

##############################################################################################################
## Install VScode extensions (for that user)
##
USER appuser
RUN code-server --install-extension julialang.language-julia
RUN code-server --install-extension ms-python.python
RUN code-server --install-extension ms-toolsai.jupyter
RUN code-server --install-extension quarto.quarto
RUN code-server --install-extension REditorSupport.r
RUN code-server --install-extension RDebugger.r-debugger
RUN code-server --install-extension Posit.shiny
RUN code-server --install-extension ms-azuretools.vscode-docker
RUN code-server --install-extension GitLab.gitlab-workflow
RUN code-server --install-extension GitHub.vscode-pull-request-github
RUN code-server --install-extension ms-vsts.team

EXPOSE 8080
EXPOSE 8787
