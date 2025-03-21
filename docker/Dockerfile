FROM ghcr.io/osgeo/gdal:ubuntu-full-3.10.2

ENV SHELL=bash \
    DEBIAN_FRONTEND=non-interactive \
    USE_PYGEOS=0 \
    SPATIALITE_LIBRARY_PATH='mod_spatialite.so' \
    GS_NO_SIGN_REQUEST=YES

RUN apt update --yes \
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt install --yes --fix-missing --no-install-recommends \
    # developer convenience
    fish \
    git \
    graphviz \
    htop \
    iproute2 \
    iputils-ping \
    jq \
    less \
    libtiff-tools \
    net-tools \
    openssh-client \
    postgresql \
    postgresql-client \
    python3-dev \
    python3-full \
    rsync \
    simpleproxy \
    sudo \
    tig \
    tini \
    tmux \
    vim \
    xz-utils \
    zip \
    # build psycopg2
    libpq-dev \
    # for cython to work need compilers
    build-essential \
    # install ffmpeg the normal way
    ffmpeg \
    nodejs \
    # Spatialite support
    libsqlite3-mod-spatialite \
    # install texlive
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-plain-generic

# Install yq
RUN wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq &&\
  chmod +x /usr/bin/yq

# install pandoc
RUN wget https://github.com/jgm/pandoc/releases/download/3.6.3/pandoc-3.6.3-1-amd64.deb \
  && apt --fix-broken install \
  && dpkg -i pandoc-3.6.3-1-amd64.deb \
  && rm pandoc-3.6.3-1-amd64.deb

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

# Configure environment
ENV HOME="/home/${NB_USER}"
ENV VIRTUAL_ENV="/opt/venv"
ENV PATH="${VIRTUAL_ENV}/bin:${PATH}"

# Delete existing user with UID="${NB_UID}" if it exists
RUN if grep -q "${NB_UID}" /etc/passwd; then \
        userdel --remove $(id -un "${NB_UID}"); \
    fi
# Create a non-root user
RUN useradd --no-log-init --create-home --shell /bin/bash --no-user-group --gid $NB_GID --uid $NB_UID  $NB_USER \
    && chown -R $NB_UID:$NB_GID $HOME
# Set up the python virtual environment PEP 668
RUN python3 -m venv $VIRTUAL_ENV \
    && chown -R $NB_UID:$NB_GID  $VIRTUAL_ENV

COPY assets/tflite_runtime-2.20.0-cp312-cp312-linux_x86_64.whl requirements.txt /

USER $NB_USER
RUN python -m pip install --upgrade pip pip-tools wheel
# Install required python packages
RUN pip install --no-cache-dir -r requirements.txt 
# Install the 3.12 compatible wheel for tflite-runtime
RUN pip install --no-cache-dir tflite_runtime-2.20.0-cp312-cp312-linux_x86_64.whl \
  # Ensure the latest odc-stats is installed
  && pip install --no-cache-dir --upgrade odc-stats
# Enable server extensions
RUN jupyter server extension enable --py --sys-prefix jupyterlab_iframe jupyter_resource_usage 
# Fix to ensure numpy-based raster support has been properly installed
RUN pip install --no-cache --force-reinstall gdal=="$(gdal-config --version).*"

USER root
ENV JUPYTERLAB_DIR=$VIRTUAL_ENV/share/jupyter/lab
COPY assets/sync_repo assets/with_bootstrap assets/jupyterhub-singleuser /usr/bin/
COPY assets/overrides.json $JUPYTERLAB_DIR/settings/
COPY assets/jupyter_lab_config.py /etc/jupyter/

WORKDIR $HOME

ARG WITH_SUDO="no"
RUN if [ "$WITH_SUDO" = "yes" ]; then \
  echo "${NB_USER} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
;fi

USER $NB_USER

ENTRYPOINT ["/bin/tini", "-s", "--", "with_bootstrap"]
CMD ["jupyter", "lab", \
"--ip=0.0.0.0", \
"--port=9988", \
"--no-browser", \
"--debug"]