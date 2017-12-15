# Azure IoT Edge Template

This project will greatly simplify your Azure IoT Edge development.

It includes:
 - A script (**edge.py**) that simplifies runtime, modules and docker management.
 - A suggested folder structure for Edge projects including **modules**, **build**, and **config** folders.

## Edge Script
You will find the **edge.py** script in the root of this repository.  It has the following commands:

**runtime**

`edge.py runtime --help`
- `--start`               Starts Edge Runtime
- `--stop`              Stops Edge Runtime
- `--restart`             Restarts Edge Runtime
- `--setup`               Setup Edge Runtime using runtime.json in build/config directory
- `--status`              Edge Runtime Status
- `--logs`                Edge Runtime Logs
- `--set-container-registry` Pulls Edge Runtime from Docker Hub and pushes to container registry
- `--set-config`          Expands env vars in /config and copies to /build/config

**modules**

`edge.py modules --help`
- `--build`       Builds and pushes modules specified in ACTIVE_MODULES env var to container registry
- `--deploy`      Deploys modules to Edge device using modules.json in build/config directory
- `--set-config`  Expands env vars in /config and copies to /build/config

**docker**

`edge.py docker --help`
- `--setup-local-registry` Sets up a local Docker registry
- `--clean`              Removes all the Docker containers and Images
- `--remove-containers`  Removes all the Docker containers
- `--remove-images`      Removes all the Docker images

## Folder Structure

There are 3 main folders in this project:

1. **config** - Contains sample config files for both modules and runtime.

1. **build** - Contains the files outputted by the .NET Core SDK.

1. **modules** - Contains all of the modules for your Edge project.
    - The edge.py script assumes that you'll structure your Dockerfiles exactly like the filter-module sample.  Have a Docker folder in the root of the project, then subfolders within that to support multiple Docker files.
        
    > It is important that you follow this structure or the script will not work.

## Setup
### Azure Setup
1. [Create Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-csharp-csharp-getstarted#create-an-iot-hub)
1. [Create Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
    - Make sure you enable Admin Access when you create the Azure Container Registry
1. Create Edge Device using the Azure Portal

You can also deploy the IoT Hub and Container Registry with this **Deploy to Azure** script:

[![Azure Deployment](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonbgallant%2Fazure-iot-edge-template%2Fmaster%2Fassets%2Fdeploy%2FARMDeployment%2Fazuredeploy.json)

### Dev Machine Setup

Here's what you need to do to get `edge.py` running on your dev machine. If you are using a seperate Edge device, like a Raspberry Pi, you do not need to run all of these steps on your additional Edge device.  See the [Edge Device Setup](#edge-device-setup) section below for more information on setting up your Edge device.

> Note: See the ["Test Coverage"](#test-coverage) section below to see what this script has been tested with.

1. Install [Docker](https://docs.docker.com/engine/installation/)
    - Switch to Linux Containers

    Do not install via `sudo apt install docker.io`. Use the proper steps for [CE here](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce), or use the [convenience script](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-convenience-script).

1. Install [Python 2.7 or Python 3](https://www.python.org/downloads/)
    - You should already have this installed on Linux. Use the following command to check: `sudo apt install python-pip`

1. Install Pip

    1. Linux

    Python 2
        ```
        sudo apt install python-pip
        ```

    Python 3
        ```
        sudo apt install python3-pip
        ```

    2. Windows

        Pip should be installed by default.  `pip --version` should return a version number, 9+.

1. Install [.NET Core SDK](https://www.microsoft.com/net/core#windowscmd)
    - The .NET Core SDK does not run on ARM, so you do not need to install this on Raspberry Pi.

1. Clone This Repository

    `git clone https://github.com/jonbgallant/azure-iot-edge-template.git project-name`

    > Replace `project-name` with the name of your project.

1. Install Dependencies

    > You can also run under a Python Virtual Environment.  See the [Python Virtual Environment Setup](#python-virtual-environment-setup) instructions below for details on how to set that up.

    1. System Dependencies
        1. Mac:
            ```
            sudo easy_install pip
            brew install libffi
            ```

        1. Raspberry Pi:
            ```
            sudo pip install --upgrade setuptools pip
            sudo apt install python2.7-dev libffi-dev libssl-dev -y
            ```
    1. Script Dependencies
        > Use 'sudo' for Mac/Linux/RaspberryPi. You do not need to run this on the Raspberry Pi Edge device. See the [Edge Device Setup](#edge-device-setup) section below for more information on setting up your Edge device.

        `pip install -U -r requirements.txt`

1. Set Environment Variables

    > System or User Environment Variables take precedence over values in .env file.

    - Rename `.env.tmp` to `.env` or run the following command:

        `cp .env.tmp .env`

    - Open `.env` and set variables

    **Runtime Home Directory**
        - For Linux/Raspberry Pi, change: `RUNTIME_HOME_DIR="/etc/azure-iot-edge"`

    **Active Modules**
        - You can tell the script which modules you want to build by including them in the `ACTIVE_MODULES` setting.  Comma separated. 

    **Active Docker Directories**
        - You can tell the script which Docker files to build and push by including the Dockerfile's parent folder in the `ACTIVE_DOCKER_DIRS` setting.

1. Update Config

    If you are running on Raspberry Pi you need to use the arm32v7 Dockerfile. Open `config/modules.json`, find the `filter-module` line and replace `linux-x64` with `arm32v7`. 

    Replace this:
    `"image": "${CONTAINER_REGISTRY_SERVER}/filter-module:linux-x64-${CONTAINER_TAG}",`

    With this:
    `"image": "${CONTAINER_REGISTRY_SERVER}/filter-module:arm32v7-${CONTAINER_TAG}",`

### Local Docker Registry Setup

Instead of using a cloud based container registry, you can use a local Docker registry.  Here's how to get it setup.

1. Set `CONTAINER_REGISTRY_SERVER` in .env to `localhost:5000`. You can enter a different port if you'd like to.
1. Add `localhost:5000` and `127.0.0.1:5000` to Docker -> Settings -> Daemon -> Insecure Registries

The script will look for `localhost` in your setting and take care of the rest for you.

You can run the following command if you want to run the setup process on your own.

```
python edge.py docker --setup-local-registry
```

### Python Virtual Environment Setup

You can run this script inside a Python Vritual Environment. 

1. Install virtualenv

    `pip install virtualenv`

1. Create virtualenv

    Execute the following from the root of this repository.

    `virtualenv venv`

    > venv is just a project name that can be anything you want, but we recommend sticking with venv because the .gitignore file excludes it.

1. Activate the virtualenv

    Windows: `venv\Scripts\activate.bat`

    Posix: `source venv/bin/activate`

1. Install Dependencies

    Continue with the instructions above starting with the [Dev Machine Setup](#dev-machine-setup) -> Install Dependencies.

1. Deactivate the virtualenv

    When you are done with your virtualenv, you can deactivate it with the follow command:

    Windows: `venv\Scripts\deactivate.bat`

    Posix: `deactivate`

## Edge Device Setup

The `edge.py` script is intended to help with Edge development and doesn't necessarily need to be taken on as a dependency in production or integration environments, where you'll likely want to use the `iotedgectl` script directly. You can use `edge.py` to generate your runtime.json file on your dev machine, copy that to your Edge device and then use the following command to setup and start your Edge Runtime. 

```
iotedgectl setup --config-file runtime.json
iotedgectl start
```

Having said that, there's nothing stopping you from deploying `edge.py` to your Edge device. It may be helpful if you want to run the `edge.py docker --clean` command to clean up Docker containers and images. Or if you want to run `edge.py runtime --logs` to see all the log files on the device.

> Please note that the .NET Core SDK does not support ARM, so you will not be able to run `modules --build` or `modules --deploy` directly on a Raspberry Pi.

### Raspberry Pi

Whether you use `edge.py` or directly use `iotedgecgtl` on the Raspberry Pi, you will still need to run the following commands before you run the Edge Runtime.

    ```
    sudo pip install --upgrade setuptools pip
    sudo apt install python2.7-dev libffi-dev libssl-dev -y
    sudo pip install -U azure-iot-edge-runtime-ctl
    ```

## Script Usage

Each of the edge.py commands can be run individually or as a group.  Let's now build and deploy our modules and then setup and start our runtime. 

### Modules Build and Deploy

> Use `sudo` for Linux.  You __will not__ be able to build on the Raspberry Pi, because the .NET Core SDK does not support ARM. Build on a x86 based machine.

```
python edge.py modules --build --deploy
```

The `--build` command will build each module in the `modules` folder and push it to your container registry.  The `--deploy` command will apply the `build/modules.json` configuration file to your Edge device.

You can configure what modules will be built and deployed using the `ACTIVE_MODULES` env var in the `.env` file.

### Runtime Setup and Start

> Use 'sudo' for Linux/RaspberryPi

```
python edge.py runtime --setup --start
```

The `--setup` command will apply the `/build/runtime.json` file to your Edge device.  The `--start` command will start the Edge runtime.
   
### Monitor Messages

You can use the [Device Explorer](https://github.com/Azure/azure-iot-sdk-csharp/releases/download/2017-12-2/SetupDeviceExplorer.msi) to monitor the messages that are sent to your IoT Hub.

### Set Container Registry

You can also use the script to host the Edge runtime from your own Azure Container Registry.  Set the `.env` values for your Container Registry and run the following command. It will pull all the Edge containers from Dockerhub, tag them and upload them to the container registry you have specified in `.env`. 

> Use 'sudo' for Linux/RaspberryPi

```
python edge.py runtime --set-container-registry
```


### View Runtime Logs

The edge.py script also include a "Logs" command that will open a new command prompt for each module it finds in your Edge config.  Just run the following command:

> Note: I haven't figured out how to launch new SSH windows in a reliable way.  It's in the backlog.  For now, you must be on the desktop of the machine to run this command.

```
python edge.py runtime --logs
```

You can configure the logs command in the `.env` file with the `LOGS_CMD` setting.  The `.env.tmp` file provides two options, one for [ConEmu](https://conemu.github.io/) and one for Cmd.exe.

## Test Coverage

This script has been tested with the following:
- Windows 10 Fall Creators Update
- Raspberry Pi with Raspbian Stretch (Runtime Only, .NET Core SDK not supported on ARM.)
- Python 2.7.13 and Python 3.6.3
- Docker Version 17.09.1-ce-win42 (14687), Channel: stable, 3176a6a

## Troubleshooting

1. Invalid Reference Format
    ```
    500 Server Error: Internal Server Error for url: http+docker://localunixsocket/v1.30/images
    500 Server Error: Internal Server Error ("invalid reference format")
    ```

    Solution: You likely installed Docker via `sudo apt install docker.io`. Use the proper steps for [CE here](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce), or use the [convenience script](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-convenience-script).

1. Permissions Error

    ```
    The directory '/home/user/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
    ```
    
    Solution: Run pip install with -H `sudo -H pip install -U -r requirements.txt`

## Backlog

Please see the [GitHub project page](https://github.com/jonbgallant/azure-iot-edge-template/projects) for backlog tasks.

## Issues

Please use the [GitHub issues page](https://github.com/jonbgallant/azure-iot-edge-template/issues) to report any issues.

## Contributing

Please fork, branch and pull-request any changes you'd like to make.