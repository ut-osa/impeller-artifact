# Impeller artifact repository

### Structure of this repository ###
* `nexmark`: nexmark benchmark code for kafka streams
* `boki`: modified Boki
* `impeller`: impeller source code
* `impeller-experiments`: impeller experiments script

### Hardware and software dependencies ###

Our evaluation workloads run on AWS EC2 instances in `us-east-2` region.

EC2 VMs for running experiments use a public AMI (`ami-0c6de836734de3280`) from Boki,
which is based on Ubuntu 20.04 with necessary dependencies installed.

### Environment setup ###

#### Setting up the controller machine ####

A controller machine in AWS `us-east-2` region is required for running scripts executing experiments.
The controller machine can use very small EC2 instance type, as it only provisions and controls experiment VMs,
but does not affect experimental results.
In our own setup, we use a `t3.micro` EC2 instance installed with Ubuntu 20.04 as the controller machine.

On the controller machine, clone this repository with all git submodules
```bash
git clone --recursive https://github.com/ut-osa/impeller-artifact.git
```

To compile source code, the controller machine needs to install build dependencies by executing the following scripts. 
```bash
./impeller-experiments/scripts/install_deps.sh
```
This script installs latest version of AWS CLI version 1
and this [documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html)
details the recommanded way for installing AWS CLI version 1 if you decide to not using the latest one.
Once installed, AWS CLI has to be configured with region `us-east-2` and access key
(see this [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)).

Execute `./impeller-experiments/scripts/setup_sshkey.sh` to setup SSH keys that will be used to access experiment VMs.
Please read the notice in `scripts/setup_sshkey.sh` before executing it to see if this script works for your setup.

#### Setting up EC2 security group and placement group ####

Our VM provisioning script creates EC2 instances with security group `impeller` and placement group `impeller-experiments`.
The security group includes firewall rules for experiment VMs (including allowing the controller machine to SSH into them),
while the placement group instructs AWS to place experiment VMs close together.

Executing `scripts/aws_provision.sh` on the controller machine creates these groups with correct configurations.

#### Compiling source code
- Boki: 
  - build dependencies
    ```bash
    cd ./boki/tmp/ && ./build_deps_release.sh && cd -
    ```
  - build source code
    ```bash
    cd ./boki/ && make -j2 && cd -
    ```

- nexmark
  ```bash
  cd ./nexmark/nexmark-kafka-streams/ && task build
  ```

- impeller
  ```bash
  cd ./impeller/ && make -j4
  ```
