#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

    FROM wordpress

    # Avoid warnings by switching to noninteractive
    ENV DEBIAN_FRONTEND=noninteractive

    # Enable WP_DEBUG in wp-config.php
    ENV WORDPRESS_DEBUG=1

    # This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
    # property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
    # will be updated to match your local UID/GID (when using the dockerFile property).
    # See https://aka.ms/vscode-remote/containers/non-root-user for details.
    ARG USERNAME=vscode
    ARG USER_UID=1000
    ARG USER_GID=$USER_UID

    # Configure apt and install packages
    RUN apt-get update \
        && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
        #
        # install git iproute2, procps, lsb-release (useful for CLI installs)
        && apt-get -y install git iproute2 procps iproute2 lsb-release \
        #
        # Install xdebug
        && yes | pecl install xdebug \
        && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
        && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
        && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
        #
        # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
        && groupadd --gid $USER_GID $USERNAME \
        && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
        # [Optional] Add sudo support for the non-root user
        && apt-get install -y sudo \
        && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
        && chmod 0440 /etc/sudoers.d/$USERNAME \
        #
        # # Clean up
        && apt-get autoremove -y \
        && apt-get clean -y \
        && rm -rf /var/lib/apt/lists/*

    # Switch back to dialog for any ad-hoc use of apt-get
    ENV DEBIAN_FRONTEND=dialog

    # Install Composer
    RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

    # Install Oh My Zsh for personal preference
    RUN sh -c "$(curl -L https://github.com/deluan/zsh-in-docker/releases/download/v1.2.1/zsh-in-docker.sh)" -- \
    && chsh -s $(which zsh) vscode

    # Mark the WordPress directory as a safe Git directory to avoid permission issues when using Git
    RUN git config --global --add safe.directory /var/www/html

    # Add WP-CLI
    RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
        && php wp-cli.phar --info \
        && chmod +x wp-cli.phar \
        && mv wp-cli.phar /usr/local/bin/wp