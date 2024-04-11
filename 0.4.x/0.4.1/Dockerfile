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

# debug : 在安装SSH后，之后设置的所有环境变量都会失效，应该需要配置SSH，之后关注下
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

RUN \
    OPEN_RESTY_VERSION="1.25.3.1" && \
    mkdir $RESOURCES_PATH"/openresty" && \
    cd $RESOURCES_PATH"/openresty" && \
    apt-get update && \
    apt-get purge -y nginx nginx-common && \
    # libpcre required, otherwise you get a 'the HTTP rewrite module requires the PCRE library' error
    # Install apache2-utils to generate user:password file for nginx.
    apt-get install -y libssl-dev perl libpcre3 libpcre3-dev apache2-utils && \
    wget --no-verbose https://openresty.org/download/openresty-$OPEN_RESTY_VERSION.tar.gz  -O ./openresty.tar.gz && \
    tar xfz ./openresty.tar.gz && \
    rm ./openresty.tar.gz && \
    cd ./openresty-$OPEN_RESTY_VERSION/ && \
    # Surpress output - if there is a problem remove  > /dev/null
    ./configure --with-http_stub_status_module --with-http_sub_module > /dev/null && \
    make -j2 > /dev/null && \
    make install > /dev/null && \
    # create log dir and file - otherwise openresty will throw an error
    mkdir -p /var/log/nginx/ && \
    touch /var/log/nginx/upstream.log && \
    cd $RESOURCES_PATH && \
    rm -r $RESOURCES_PATH"/openresty" && \
    # Fix permissions
    chmod -R a+rwx $RESOURCES_PATH && \
    # Cleanup
    clean-layer.sh

ENV PATH=/usr/local/openresty/nginx/sbin:$PATH

COPY resources/nginx/lua-extensions /etc/nginx/nginx_plugins

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
    echo "${CONDA_ROOT}/lib" >> /etc/ld.so.conf && \
    ldconfig && \
    # Fix permissions
    fix-permissions.sh $CONDA_ROOT && \
    clean-layer.sh

ENV PATH=$CONDA_ROOT/bin:$PATH

# Will throw an error: OpenSSL version mismatch problem
# ENV LD_LIBRARY_PATH=$CONDA_ROOT/lib

# Install pyenv to allow dynamic creation of python versions
RUN git clone https://github.com/pyenv/pyenv.git $RESOURCES_PATH/.pyenv && \
    # Install pyenv plugins based on pyenv installer
    git clone https://github.com/pyenv/pyenv-virtualenv.git $RESOURCES_PATH/.pyenv/plugins/pyenv-virtualenv  && \
    git clone https://github.com/pyenv/pyenv-doctor.git $RESOURCES_PATH/.pyenv/plugins/pyenv-doctor && \
    git clone https://github.com/pyenv/pyenv-update.git $RESOURCES_PATH/.pyenv/plugins/pyenv-update && \
    git clone https://github.com/pyenv/pyenv-which-ext.git $RESOURCES_PATH/.pyenv/plugins/pyenv-which-ext && \
    apt-get update && \
    # TODO: lib might contain high vulnerability
    # Required by pyenv
    apt-get install -y --no-install-recommends libffi-dev && \
    clean-layer.sh

# Add pyenv to path
ENV PYENV_ROOT ${RESOURCES_PATH}/.pyenv
ENV PATH ${PYENV_ROOT}/bin:${PATH}

# Install pipx
RUN python -m pip install pipx && \
    python -m pipx ensurepath && \
    clean-layer.sh
ENV PATH=$HOME/.local/bin:$PATH

# Install node.js
RUN \
    apt-get update && \
    # https://nodejs.org/en/about/releases/ use even numbered releases, i.e. LTS versions
    curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash - && \
    apt-get install -y nodejs && \
    # As conda is first in path, the commands 'node' and 'npm' reference to the version of conda.
    # Replace those versions with the newly installed versions of node
    rm -f /opt/conda/bin/node && ln -s /usr/bin/node /opt/conda/bin/node && \
    rm -f /opt/conda/bin/npm && ln -s /usr/bin/npm /opt/conda/bin/npm && \
    # Fix permissions
    chmod a+rwx /usr/bin/node && \
    chmod a+rwx /usr/bin/npm && \
    # Fix node versions - put into own dir and before conda:
    mkdir -p /opt/node/bin && \
    ln -s /usr/bin/node /opt/node/bin/node && \
    ln -s /usr/bin/npm /opt/node/bin/npm && \
    # Update npm
    /usr/bin/npm install -g npm && \
    # Install Yarn
    /usr/bin/npm install -g yarn && \
    # Install typescript
    /usr/bin/npm install -g typescript && \
    # Install webpack - 32 MB
    /usr/bin/npm install -g webpack && \
    # Install node-gyp
    /usr/bin/npm install -g node-gyp && \
    # Update all packages to latest version
    /usr/bin/npm update -g && \
    # Cleanup
    clean-layer.sh

ENV PATH=/opt/node/bin:$PATH

# Java - removed

### END RUNTIMES ###

### PROCESS TOOLS ###

# Removed XRDP

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

### GUI TOOLS ###

# Install xfce4 & gui tools
RUN \
    # Use staging channel to get newest xfce4 version (4.16)
    add-apt-repository -y ppa:xubuntu-dev/staging && \
    apt-get update && \
    apt-get install -y --no-install-recommends xfce4 && \
    apt-get install -y --no-install-recommends gconf2 && \
    apt-get install -y --no-install-recommends xfce4-terminal && \
    apt-get install -y --no-install-recommends xfce4-clipman && \
    apt-get install -y --no-install-recommends xterm && \
    apt-get install -y --no-install-recommends --allow-unauthenticated xfce4-taskmanager  && \
    # Install dependencies to enable vncserver
    apt-get install -y --no-install-recommends xauth xinit dbus-x11 && \
    # Install gdebi deb installer
    apt-get install -y --no-install-recommends gdebi && \
    # Search for files
    apt-get install -y --no-install-recommends catfish && \
    apt-get install -y --no-install-recommends font-manager && \
    # vs support for thunar
    apt-get install -y thunar-vcs-plugin && \
    # Streaming text editor for large files - klogg is alternative to glogg
    apt-get install -y --no-install-recommends libqt5concurrent5 libqt5widgets5 libqt5xml5 && \
    wget --no-verbose https://github.com/variar/klogg/releases/download/v22.06/klogg-22.06.0.1289-Linux-amd64-jammy.deb -O $RESOURCES_PATH/klogg.deb && \
    dpkg -i $RESOURCES_PATH/klogg.deb && \
    rm $RESOURCES_PATH/klogg.deb && \
    # Disk Usage Visualizer
    apt-get install -y --no-install-recommends baobab && \
    # Lightweight text editor
    apt-get install -y --no-install-recommends mousepad && \
    apt-get install -y --no-install-recommends vim && \
    # Process monitoring
    apt-get install -y --no-install-recommends htop && \
    # Install Archive/Compression Tools: https://wiki.ubuntuusers.de/Archivmanager/
    apt-get install -y p7zip p7zip-rar && \
    apt-get install -y --no-install-recommends thunar-archive-plugin && \
    apt-get install -y xarchiver && \
    # DB Utils
    apt-get install -y --no-install-recommends sqlitebrowser && \
    # Install nautilus and support for sftp mounting
    apt-get install -y --no-install-recommends nautilus gvfs-backends && \
    # Install gigolo - Access remote systems
    apt-get install -y --no-install-recommends gigolo libglib2.0-bin && \
    # xfce systemload panel plugin - needs to be activated
    # apt-get install -y --no-install-recommends xfce4-systemload-plugin && \
    # Leightweight ftp client that supports sftp, http, ...
    apt-get install -y --no-install-recommends gftp && \
    # Install chrome
    # sudo add-apt-repository ppa:system76/pop
    add-apt-repository ppa:saiarcot895/chromium-beta && \
    apt-get update && \
    apt-get install -y chromium-browser chromium-browser-l10n chromium-codecs-ffmpeg && \
    ln -s /usr/bin/chromium-browser /usr/bin/google-chrome && \
    # Cleanup
    apt-get purge -y pm-utils xscreensaver* && \
    # Large package: gnome-user-guide 50MB app-install-data 50MB
    apt-get remove -y app-install-data gnome-user-guide && \
    clean-layer.sh

RUN echo "/lib/x86_64-linux-gnu" >> /etc/ld.so.conf && \
    echo "/usr/lib/x86_64-linux-gnu" >> /etc/ld.so.conf && \
    ldconfig

# Install VNC
RUN \
    apt-get update  && \
    # required for websockify
    # apt-get install -y python-numpy  && \
    cd ${RESOURCES_PATH} && \
    # Tiger VNC
    wget -qO- https://sourceforge.net/projects/tigervnc/files/stable/1.13.1/tigervnc-1.13.1.x86_64.tar.gz/download | tar xz --strip 1 -C / && \
    # Install websockify
    mkdir -p ./novnc/utils/websockify && \
    # debug : 原版是1.2.0，解压后是launch.sh，而1.4.0解压后是novnc_proxy，之后关注下是否会有VNC启动问题
    wget -qO- https://github.com/novnc/noVNC/archive/v1.4.0.tar.gz | tar xz --strip 1 -C ./novnc && \
    wget -qO- https://github.com/novnc/websockify/archive/v0.11.0.tar.gz | tar xz --strip 1 -C ./novnc/utils/websockify && \
    chmod +x -v ./novnc/utils/novnc_proxy && \
    # create user vnc directory
    mkdir -p $HOME/.vnc && \
    # Fix permissions
    fix-permissions.sh ${RESOURCES_PATH} && \
    # Cleanup
    clean-layer.sh

# Install Web Tools - Offered via Jupyter Tooling Plugin

## VS Code Server: https://github.com/coder/code-server
COPY resources/tools/vs-code-server.sh $RESOURCES_PATH/tools/vs-code-server.sh
RUN \
    /bin/bash $RESOURCES_PATH/tools/vs-code-server.sh --install && \
    # Cleanup
    clean-layer.sh

## ungit
COPY resources/tools/ungit.sh $RESOURCES_PATH/tools/ungit.sh
RUN \
    /bin/bash $RESOURCES_PATH/tools/ungit.sh --install && \
    # Cleanup
    clean-layer.sh

## netdata
COPY resources/tools/netdata.sh $RESOURCES_PATH/tools/netdata.sh
RUN \
    /bin/bash $RESOURCES_PATH/tools/netdata.sh --install && \
    # Cleanup
    clean-layer.sh

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

### CONFIGURATION ###

### END CONFIGURATION ###

# debug
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

ENTRYPOINT ["/tini", "-g", "--"]

# debug
CMD ["/usr/sbin/sshd", "-D"]