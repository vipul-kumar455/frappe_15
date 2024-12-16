# Frappe 15 Installation Guide

## Docker Installation

Follow these steps to install Docker on your system:

1. Update the package index:
    ```bash
    sudo apt-get update
    ```

2. Download and run the Docker installation script:
    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh --dry-run
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

3. Add a docker group and add your user to it:
    ```bash
    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```

4. Run a test Docker container:
    ```bash
    docker run hello-world
    ```

5. Enable Docker services:
    ```bash
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service
    ```

6. Set up Docker CLI plugins directory and download Docker Compose:
    ```bash
    DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
    mkdir -p $DOCKER_CONFIG/cli-plugins
    curl -SL https://github.com/docker/compose/releases/download/v2.19.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
    chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```

7. Verify Docker Compose installation:
    ```bash
    docker-compose --version
    ```

## Frappe and ERPNext Setup

Follow these steps to set up Frappe with ERPNext version 15:

1. Clone the Frappe Docker repository and navigate into it:
    ```bash
    git clone https://github.com/frappe/frappe_docker.git
    cd frappe_docker
    ```

2. Copy the devcontainer and vscode examples:
    ```bash
    cp -R devcontainer-example .devcontainer
    cp -R development/vscode-example development/.vscode
    ```

3. Install the Remote Containers extension for Visual Studio Code and open the current directory:
    ```bash
    code --install-extension ms-vscode-remote.remote-containers
    code .
    ```

4. Initialize a new Frappe bench with version 15 and move into the directory:
    ```bash
    bench init --skip-redis-config-generation --frappe-branch version-15 frappe-bench
    cd frappe-bench
    ```

5. Set database and Redis configurations:
    ```bash
    bench set-config -g db_host mariadb
    bench set-config -g redis_cache redis://redis-cache:6379
    bench set-config -g redis_queue redis://redis-queue:6379
    bench set-config -g redis_socketio redis://redis-queue:6379
    ```

6. Remove Redis from Procfile:
    ```bash
    sed -i '/redis/d' ./Procfile
    ```

7. Create a new site with specified passwords and configurations:
    ```bash
    bench new-site --mariadb-root-password 123 --admin-password admin --no-mariadb-socket development.localhost
    ```

8. Set root login and password for PostgreSQL:
    ```bash
    bench config set-common-config -c root_login postgres
    bench config set-common-config -c root_password '"123"'
    ```

9. Enable developer mode and clear cache for the site:
    ```bash
    bench --site development.localhost set-config developer_mode 1
    bench --site development.localhost clear-cache
    ```

10. Get and install the ERPNext app:
    ```bash
    bench get-app --branch version-15 --resolve-deps erpnext
    bench --site development.localhost install-app erpnext
    ```

11. Start the Frappe bench:
    ```bash
    bench start
    ```

Please ensure you have proper permissions and understand the implications of each command before executing them.
