FROM ubuntu:22.04

USER root

### BASICS ###
# Technical Environment Variables
ENV \
    SHELL="/bin/bash" \
    HOME="/root"  \
    # Nobteook server user: https://github.com/jupyter/docker-stacks/blob/main/images/base-notebook/Dockerfile
    NB_USER="root" \
    USER_GID=0 \
    XDG_CACHE_HOME="/root/.cache/" \
    XDG_RUNTIME_DIR="/tmp" \
    DISPLAY=":1" \
    TERM="xterm" \
    DEBIAN_FRONTEND="noninteractive" \
    RESOURCES_PATH="/resources" \
    SSL_RESOURCES_PATH="/resources/ssl" \
    WORKSPACE_HOME="/workspace"

WORKDIR $HOME

# Make folders
RUN \
    mkdir $RESOURCES_PATH && chmod a+rwx $RESOURCES_PATH && \
    mkdir $WORKSPACE_HOME && chmod a+rwx $WORKSPACE_HOME && \
    mkdir $SSL_RESOURCES_PATH && chmod a+rwx $SSL_RESOURCES_PATH

# Layer cleanup script
COPY resources/scripts/clean-layer.sh  /usr/bin/clean-layer.sh
COPY resources/scripts/fix-permissions.sh  /usr/bin/fix-permissions.sh

# Make clean-layer and fix-permissions executable
RUN \
    chmod a+rwx /usr/bin/clean-layer.sh && \
    chmod a+rwx /usr/bin/fix-permissions.sh

# Generate and Set locals
# https://stackoverflow.com/questions/28405902/how-to-set-the-locale-inside-a-debian-ubuntu-docker-container#38553499
RUN \
    apt-get update && \
    apt-get install -y locales && \
    # install locales-all?
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8 && \
    # Cleanup
    clean-layer.sh

ENV LC_ALL="en_US.UTF-8" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en"

# Install basics
RUN \
    # TODO add repos?
    # add-apt-repository ppa:apt-fast/stable
    # add-apt-repository 'deb http://security.ubuntu.com/ubuntu xenial-security main'
    apt-get update --fix-missing && \
    apt-get install -y sudo apt-utils && \
    apt-get upgrade -y && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    # This is necessary for apt to access HTTPS sources:
    apt-transport-https \
    gnupg-agent \
    gpg-agent \
    gnupg2 \
    ca-certificates \
    build-essential \
    pkg-config \
    software-properties-common \
    lsof \
    net-tools \
    libcurl4 \
    curl \
    wget \
    cron \
    openssl \
    iproute2 \
    psmisc \
    tmux \
    dpkg-sig \
    uuid-dev \
    csh \
    xclip \
    clinfo \
    time \
    libssl-dev \
    libgdbm-dev \
    libncurses5-dev \
    libncursesw5-dev \
    # required by pyenv
    libreadline-dev \
    libedit-dev \
    xz-utils \
    gawk \
    # Simplified Wrapper and Interface Generator (5.8MB) - required by lots of py-libs
    swig \
    # Graphviz (graph visualization software) (4MB)
    graphviz libgraphviz-dev \
    # Terminal multiplexer
    screen \
    # Editor
    nano \
    # Find files
    locate \
    # Dev Tools
    sqlite3 \
    # XML Utils
    xmlstarlet \
    # GNU parallel
    parallel \
    #  R*-tree implementation - Required for earthpy, geoviews (3MB)
    libspatialindex-dev \
    # Search text and binary files
    yara \
    # Minimalistic C client for Redis
    libhiredis-dev \
    # postgresql client
    libpq-dev \
    # mysql client (10MB)
    libmysqlclient-dev \
    # mariadb client (7MB)
    # libmariadbclient-dev \
    # image processing library (6MB), required for tesseract
    libleptonica-dev \
    # GEOS library (3MB)
    libgeos-dev \
    # style sheet preprocessor
    less \
    # Print dir tree
    tree \
    # Bash autocompletion functionality
    bash-completion \
    # ping support
    iputils-ping \
    # Map remote ports to localhosM
    socat \
    # Json Processor
    jq \
    rsync \
    # sqlite3 driver - required for pyenv
    libsqlite3-dev \
    # VCS:
    git \
    subversion \
    jed \
    # odbc drivers
    unixodbc unixodbc-dev \
    # Image support
    libtiff-dev \
    libjpeg-dev \
    libpng-dev \
    libglib2.0-0 \
    libxext6 \
    libsm6 \
    libxext-dev \
    libxrender1 \
    libzmq3-dev \
    # protobuffer support
    protobuf-compiler \
    libprotobuf-dev \
    libprotoc-dev \
    autoconf \
    automake \
    libtool \
    cmake  \
    fonts-liberation \
    google-perftools \
    # Compression Libs
    # also install rar/unrar? but both are propriatory or unar (40MB)
    zip \
    gzip \
    unzip \
    bzip2 \
    lzop \
    libarchive-tools \
    # unpack (almost) everything with one command
    unp \
    libbz2-dev \
    liblzma-dev \
    zlib1g-dev && \
    # Update git to newest version
    add-apt-repository -y ppa:git-core/ppa  && \
    apt-get update && \
    apt-get install -y --no-install-recommends git && \
    # Fix all execution permissions
    chmod -R a+rwx /usr/local/bin/ && \
    # configure dynamic linker run-time bindings
    ldconfig && \
    # Fix permissions
    fix-permissions.sh $HOME && \
    # Cleanup
    clean-layer.sh

# Add tini
RUN wget --no-verbose https://github.com/krallin/tini/releases/download/v0.19.0/tini -O /tini && \
    chmod +x /tini

# prepare ssh for inter-container communication for remote python kernel
RUN \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    openssh-client \
    openssh-server \
    # SSLH for SSH + HTTP(s) Multiplexing
    sslh \
    # SSH Tooling
    autossh \
    mussh && \
    chmod go-w $HOME && \
    mkdir -p $HOME/.ssh/ && \
    # create empty config file if not exists
    touch $HOME/.ssh/config  && \
    sudo chown -R $NB_USER:users $HOME/.ssh && \
    chmod 700 $HOME/.ssh && \
    printenv >> $HOME/.ssh/environment && \
    chmod -R a+rwx /usr/local/bin/ && \
    # Fix permissions
    fix-permissions.sh $HOME && \
    # Cleanup
    clean-layer.sh

### END BASICS ###

### RUNTIMES ###

# Install Miniconda: https://repo.continuum.io/miniconda/
ENV \
    CONDA_ROOT=/opt/conda \
    PYTHON_VERSION="3.10.14" \
    CONDA_PYTHON_DIR=/opt/conda/lib/python3.10 \
    MINICONDA_VERSION=24.1.2-0 \
    MINICONDA_MD5=122c8c9beb51e124ab32a0fa6426c656 \
    CONDA_VERSION=24.1.2-0

RUN wget --no-verbose https://repo.anaconda.com/miniconda/Miniconda3-py310_${CONDA_VERSION}-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -u -p $CONDA_ROOT && \
    export PATH=$CONDA_ROOT/bin:$PATH && \
    rm ~/miniconda.sh && \
    # Configure conda
    # TODO: Add conde-forge as main channel -> remove if testted
    # TODO, use condarc file
    $CONDA_ROOT/bin/conda config --system --add channels conda-forge && \
    $CONDA_ROOT/bin/conda config --system --set auto_update_conda False && \
    $CONDA_ROOT/bin/conda config --system --set show_channel_urls True && \
    $CONDA_ROOT/bin/conda config --system --set channel_priority strict && \
    # Deactivate pip interoperability (currently default), otherwise conda tries to uninstall pip packages
    $CONDA_ROOT/bin/conda config --system --set pip_interop_enabled false && \
    # Update conda
    $CONDA_ROOT/bin/conda update -y -n base -c defaults conda && \
    $CONDA_ROOT/bin/conda update -y setuptools && \
    $CONDA_ROOT/bin/conda install -y conda-build && \
    # Update selected packages - install python 3.10.x
    $CONDA_ROOT/bin/conda install -y --update-all python=$PYTHON_VERSION && \
    # Link Conda
    ln -s $CONDA_ROOT/bin/python /usr/local/bin/python && \
    ln -s $CONDA_ROOT/bin/conda /usr/bin/conda && \
    # Update
    $CONDA_ROOT/bin/conda install -y pip && \
    $CONDA_ROOT/bin/pip install --upgrade pip && \
    chmod -R a+rwx /usr/local/bin/ && \
    # Cleanup - Remove all here since conda is not in path as of now
    # find /opt/conda/ -follow -type f -name '*.a' -delete && \
    # find /opt/conda/ -follow -type f -name '*.js.map' -delete && \
    $CONDA_ROOT/bin/conda clean -y --packages && \
    $CONDA_ROOT/bin/conda clean -y -a -f  && \
    $CONDA_ROOT/bin/conda build purge-all && \
    # Fix permissions
    fix-permissions.sh $CONDA_ROOT && \
    clean-layer.sh

ENV PATH=$CONDA_ROOT/bin:$PATH

# Install pipx
RUN python -m pip install pipx && \
    python -m pipx ensurepath && \
    clean-layer.sh
ENV PATH=$HOME/.local/bin:$PATH

### END RUNTIMES ###

### PROCESS TOOLS ###

# Install supervisor for process supervision
RUN \
    apt-get update && \
    # Create sshd run directory - required for starting process via supervisor
    mkdir -p /var/run/sshd && chmod 400 /var/run/sshd && \
    # Install rsyslog for syslog logging
    apt-get install -y --no-install-recommends rsyslog && \
    pipx install supervisor && \
    pipx inject supervisor supervisor-stdout && \
    # supervisor needs this logging path
    mkdir -p /var/log/supervisor/ && \
    # Cleanup
    clean-layer.sh

### END PROCESS TOOLS ###

### JUPYTER ###

# Install and activate ZSH
COPY resources/tools/oh-my-zsh.sh $RESOURCES_PATH/tools/oh-my-zsh.sh

RUN \
    # Install ZSH
    /bin/bash $RESOURCES_PATH/tools/oh-my-zsh.sh --install && \
    # Make zsh the default shell
    # Initialize conda for command line activation
    # TODO do not activate for now, opening the bash shell is a bit slow
    # conda init bash && \
    conda init zsh && \
    chsh -s $(which zsh) $NB_USER && \
    # Install sdkman - needs to be executed after zsh
    curl -s https://get.sdkman.io | bash && \
    # Cleanup
    clean-layer.sh

### END JUPYTER ###

# debug
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

ENTRYPOINT ["/tini", "-g", "--"]

# debug
CMD ["/usr/sbin/sshd", "-D"]