##
## Ubuntu 24.04 VS Code Server
##
FROM ghcr.io/astral-sh/uv:0.11.7 AS uv
FROM ghcr.io/rocker-org/rstudio:4.6.1 AS rstudio
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
ENV R_VERSION="4.6.1"

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
## Install R from https://r2u.stat.illinois.edu/ubuntu
##
RUN apt update -qq \
    && apt install --yes --no-install-recommends wget ca-certificates gnupg sudo \
    && wget -q -O- https://eddelbuettel.github.io/r2u/assets/dirk_eddelbuettel_key.asc | sudo tee -a /etc/apt/trusted.gpg.d/cranapt_key.asc \
    && wget -q -O- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc    | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc \
    && echo "deb [arch=amd64] https://r2u.stat.illinois.edu/ubuntu noble main" | sudo tee -a  /etc/apt/sources.list.d/cranapt.list \
    && echo "deb [arch=amd64] https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/" | sudo tee -a   /etc/apt/sources.list.d/cran_r.list \
    && sudo apt-get update \
    && sudo apt-get install -y --no-install-recommends wget curl bash git pkg-config \
    && sudo apt-get install -y --no-install-recommends software-properties-common dirmngr \
    && sudo apt-get install -y --no-install-recommends r-base r-base-core \
    && sudo apt-get install -y --no-install-recommends r-cran-littler r-cran-docopt \
    && sudo apt-get install -y --no-install-recommends r-cran-remotes \
    && sudo apt-get install -y --no-install-recommends r-cran-languageserver r-cran-httpgd \
    && sudo apt-get install -y --no-install-recommends r-cran-reticulate \
    && sudo apt-get install -y --no-install-recommends r-cran-rjson r-cran-juliaconnector r-cran-juliacall \
    && sudo apt-get install -y --no-install-recommends r-cran-curl r-cran-openssl r-cran-httr \ 
    && sudo apt-get install -y --no-install-recommends r-cran-odbc \
    && sudo apt-get install -y --no-install-recommends r-cran-jsonlite r-cran-yaml r-cran-zip r-cran-data.table r-cran-rcpp \
    && sudo apt-get install -y --no-install-recommends r-cran-httpuv r-cran-shiny r-cran-crayon r-cran-knitr r-cran-htmltools r-cran-markdown r-cran-rmarkdown r-cran-shinydashboard r-cran-flexdashboard r-cran-bslib r-cran-fontawesome r-cran-sass r-cran-farver r-cran-dt \
    && sudo apt-get install -y --no-install-recommends r-cran-stringr r-cran-stringi \
    && sudo apt-get install -y --no-install-recommends r-cran-xml2 r-cran-fs r-cran-digest r-cran-rlang r-cran-sys r-cran-openssl r-cran-mime r-cran-dbi \
    && sudo apt-get autoremove -y --quiet \
    && sudo apt-get clean --quiet \
    && sudo rm -rf /var/lib/apt/lists/* \
    && R -e "print(sessionInfo())" \
    && R -e "installed.packages()[, 'Version']"


##############################################################################################################
## Install R from source and littler to be able to run the installation of Julia / Python
##
COPY --from=rstudio /rocker_scripts/ /rocker_scripts/
ENV PATH="/rocker_scripts/bin:${PATH}"

# ENV CRAN="https://p3m.dev/cran/__linux__/noble/latest"
# RUN /rocker_scripts/install_R_source.sh
# ## littler and all kind of symlinks for it
# RUN apt-get update && apt-get install -y libpcre2-dev libzstd-dev liblzma-dev libbz2-dev zlib1g-dev libdeflate-dev libicu-dev gcc && rm -rf /var/lib/apt/lists/*
# ENV R_HOME="/usr/local/lib/R"
# ENV CRAN="https://packagemanager.posit.co/cran/latest"
# ENV CRAN="https://p3m.dev/cran/__linux__/noble/latest"
# ENV CRAN="https://cran.r-project.org/"
# RUN echo "options(repos = c(CRAN = '${CRAN}'), download.file.method = 'libcurl')" >>"${R_HOME}/etc/Rprofile.site"
# RUN echo "options(HTTPUserAgent = sprintf('R/%s R (%s)', getRversion(), paste(getRversion(), R.version['platform'], R.version['arch'], R.version['os'])))" >> "${R_HOME}/etc/Rprofile.site"
# RUN Rscript -e "install.packages(c('littler'), clean = TRUE)"
# RUN /rocker_scripts/setup_R.sh

##############################################################################################################
## Install Quarto + Julia / Python / Jupyter / Geospatial tools
##
USER root
RUN sed -i '/^strip /s/^/#/' /rocker_scripts/install_julia.sh
RUN /rocker_scripts/install_julia.sh
RUN sed -i '/^strip /s/^/#/' /rocker_scripts/install_python.sh
RUN /rocker_scripts/install_python.sh
RUN /rocker_scripts/install_quarto.sh
RUN /rocker_scripts/install_pandoc.sh
RUN /rocker_scripts/install_texlive.sh
#RUN /rocker_scripts/install_jupyter.sh
RUN mkdir /etc/cont-init.d
RUN /rocker_scripts/install_rstudio.sh
## RUN /rocker_scripts/install_geospatial.sh

##############################################################################################################
## Default R packages
##  note: stringr/stringi or to be able to do rmarkdown::run from the RStudio UI
##
USER root
# RUN apt-get update && apt-get install -y libpng-dev libxml2-dev libz-dev libcurl4-openssl-dev libuv1-dev libfontconfig1-dev && rm -rf /var/lib/apt/lists/*
# RUN Rscript -e "install.packages(c('curl', 'openssl', 'remotes', 'httr'), clean = TRUE)"
# RUN Rscript -e "install.packages(c('languageserver', 'httpgd'), clean = TRUE)"
# RUN Rscript -e "install.packages(c('odbc'), clean = TRUE)"
# RUN Rscript -e "install.packages(c('jsonlite', 'yaml', 'zip', 'data.table', 'Rcpp', 'reticulate'), clean = TRUE)"
# RUN Rscript -e "install.packages(c('shiny', 'knitr', 'htmltools', 'markdown', 'rmarkdown', 'shinydashboard', 'flexdashboard', 'bslib', 'fontawesome', 'sass', 'farver', 'DT'), clean = TRUE)"
# RUN Rscript -e "install.packages(c('stringr', 'stringi'), clean = TRUE)"


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
ENV UV_PROJECT_ENVIRONMENT=/home/appuser/.venv
USER appuser
WORKDIR /home/appuser
COPY requirements.txt .
RUN uv venv $UV_PROJECT_ENVIRONMENT \
    && source .venv/bin/activate \
    && uv pip install --no-cache --upgrade pip \
    && uv pip install --no-cache -r requirements.txt \
    && rm requirements.txt

##############################################################################################################
## Install VScode extensions (for that user)
##
USER appuser
RUN code-server --install-extension julialang.language-julia \
    && code-server --install-extension ms-python.python \
    && code-server --install-extension ms-toolsai.jupyter \
    && code-server --install-extension quarto.quarto \
    && code-server --install-extension REditorSupport.r \
    && code-server --install-extension RDebugger.r-debugger \
    && code-server --install-extension Posit.shiny \
    && code-server --install-extension ms-azuretools.vscode-docker \
    && code-server --install-extension GitLab.gitlab-workflow \
    && code-server --install-extension GitHub.vscode-pull-request-github \
    && code-server --install-extension ms-vsts.team

EXPOSE 8080
EXPOSE 8787