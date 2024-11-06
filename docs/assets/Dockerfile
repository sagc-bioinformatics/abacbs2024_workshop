# Start with the rocker/rstudio base image
FROM rocker/rstudio:latest

RUN apt-get update && \
    apt-get install -y software-properties-common

RUN add-apt-repository ppa:deadsnakes/ppa

# Install system dependencies required for R packages and Python
RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libglpk-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    imagemagick \
    libmagick++-dev \
    libgit2-dev \
    python3.8 \
    python3.8-dev \
    python3.8-venv \
    python3-pip

# Create and activate Python virtual environment
RUN python3.8 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install JupyterLab
RUN pip install --no-cache-dir \
    jupyterlab \
    notebook \
    ipykernel \
    stereopy \
    matplotlib \
    pandas

# Install BiocManager and SpatialExperiment
RUN R -e 'if (!require("BiocManager", quietly = TRUE)) install.packages("BiocManager")' && \
    R -e 'BiocManager::install(c("SpatialExperiment", "ggspavis", "rhdf5"), ask=FALSE)'

# Install other common R packages
RUN R -e "install.packages(c(\
    'tidyverse', \
    'Seurat', \
    'devtools', \
    'rmarkdown', \
    'workflowr' \
    ), )"

# Configure Jupyter to not require a token or password
RUN mkdir -p /root/.jupyter && \
    echo 'c.ServerApp.password = "argon2:$argon2id$v=19$m=10240,t=10,p=8$wBq+71bzsd6/UYxua/5ELw$jsVOxs8QVEbnJeKd830oxlVUHMZ+HK96fVvoeW1z9nY"\nc.NotebookApp.notebook_dir = "/root"' > /root/.jupyter/jupyter_notebook_config.py

ENV JUPYTER_TOKEN="${PASSWORD}"

ENV USER="root"
ENV PASSWORD="abacbs24"

# Set password as specified
RUN echo "root:${PASSWORD}" | chpasswd

# Expose both RStudio and JupyterLab ports
EXPOSE 8080 8888

# Create a startup script
RUN echo '#!/bin/bash\n\
jupyter lab --ip 0.0.0.0 --port 8888 --allow-root --no-browser &\n\
/usr/lib/rstudio-server/bin/rserver --www-port=8080 --server-daemonize=0 --auth-minimum-user-id=0 \n\
' > /usr/local/bin/start-services.sh && \
    chmod +x /usr/local/bin/start-services.sh

# Use the startup script as the entry point
ENTRYPOINT ["/usr/local/bin/start-services.sh"]
