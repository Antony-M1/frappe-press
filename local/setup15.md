# Summary
This document helps to setup the `press` into your local machine and help's to work with your own `Frappe Cloud` for dev purpose.


### Step 1: setup frappe docker

setup the `frappe_docker`

Clone and change directory to frappe_docker directory

```shell
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

Copy example devcontainer config from `devcontainer-example` to `.devcontainer`

```shell
cp -R devcontainer-example .devcontainer
```

Copy example vscode config for devcontainer from `development/vscode-example` to `development/.vscode`. This will setup basic configuration for debugging.

```shell
cp -R development/vscode-example development/.vscode
```

### Step 2: setup extention
Ignore the below step if its already installed

- Install Dev Containers for VSCode
  - through command line `code --install-extension ms-vscode-remote.remote-containers`
  - clicking on the Install button in the Vistual Studio Marketplace: [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  - View: Extensions command in VSCode (Windows: Ctrl+Shift+X; macOS: Cmd+Shift+X) then search for extension `ms-vscode-remote.remote-containers`
 
### Step 3: Setup Node
install `nvm` & `yarn`
```sh
nvm install v20
npm install -g yarn
nvm use v20
```

### Step 4: Setup python version
Get the right latest version of python-`3.10.x`

```
pyenv install --list | grep 3.10
```

![Pyenv Version Check](../media/pyenv_version_check.png)

for me the latest listed in the `pyenv` **3.10.19** so im choosing that version

```sh
pyenv uninstall 3.10.19
pyenv install 3.10.19
pyenv global 3.10.19
```

### Step 5: create bench

use the installed new version of python here 
```
PYENV_VERSION=3.10.19 bench init --skip-redis-config-generation --frappe-branch version-15 frappe-bench
```

```sh
cd frappe-bench
```

### Step 6: Setup hosts
```
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

### Step 7: Create site
```
bench new-site --db-root-password 123 --admin-password admin --mariadb-user-host-login-scope=% press15.localhost
```
```
bench use press15.localhost
```

```
bench --site press15.localhost set-config developer_mode 1
bench --site press15.localhost clear-cache
```

### Step 8: Install press

```
bench get-app press --resolve-deps
```
```
bench install-app press
```

### Step 9: Migrate & Build

```
bench migrate
```

```
bench build
```

### Step 10: Start
```
bench start
```
