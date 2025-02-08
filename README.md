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
In our own setup, we use a `t3.micro` EC2 instance installed with Ubuntu 20.04 as the controller machine. If you also
want to compiling the code on the controller machine, we use a `c5.xlarge` EC2 instance.

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

#### Compiling exp tool
- latency
  ```bash
  cd ./impeller-experiments/latency/ && cargo build --release && cd -
  ```

#### Example command
- run query 1 for 60 seconds with 1 iterations
  ```bash
  cd ./impeller-experiments/nexmark_impeller/ && ./run_q1_quick.sh && cd -
  ```

#### Rerun experiments that produce figure 7
- experiments on Impeller 
  ```bash
  cd ./impeller-experiments/nexmark_impeller/
  # run ./run_q1.sh to ./run_q8.sh
  ```
- experiments on Kafka Streams
  ```bash
  cd ./impeller-experiments/nexmark_kafka-streams
  # run ./run_q1.sh to ./run_q8.sh
  ```
Serially execute these scripts are estimated to take 6300 mins. 

#### Rerun experiments that produce figure 8
  ```bash
  cd ./impeller-experiments/nexmark_impeller/
  # run ./run_q1_commit_interval.sh to ./run_q8_commit_interval.sh
  ```
Serially execute these scripts are estimated to take 1600 mins. 

#### Using latency command to collect the experiment result
For Kafka Stream results, query 1
```bash
latency scan --prefix q1_sink_ets --output $output_dir $q1_exp_dir # the exp dir is the dir that contains logs
```
For q2 to q8, change the prefix from q1_sink_ets to q2_sink_ets .. q8_sink_ets

For impeller experiments,
- query 1
```bash
latency scan --prefix query1 --suffix .json.gz --output $output_dir $q1_exp_dir
```
- query 2
```bash
latency scan --prefix query2 --suffix .json.gz --output $output_dir $q2_exp_dir
```
- query 3
```bash
latency scan --prefix q3JoinTable --suffix .json.gz --output $output_dir $q3_exp_dir
```
- query 4
```bash
latency scan --prefix q4Avg --suffix .json.gz --output $output_dir $q4_exp_dir
```
- query 5
```bash
latency scan --prefix q5maxbid --suffix .json.gz --output $output_dir $q5_exp_dir
```
- query 6
```bash
latency scan --prefix q6Avg --suffix .json.gz --output $output_dir $q6_exp_dir
```
- query 7
```bash
latency scan --prefix q7JoinMaxBid --suffix .json.gz --output $output_dir $q7_exp_dir
```
- query 8
```bash
latency scan --prefix q8JoinStream --suffix .json.gz --output $output_dir $q8_exp_dir
```



