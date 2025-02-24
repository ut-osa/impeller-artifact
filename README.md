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

The controller needs to set an IAM role. The IAM role needs to have these permission policy: AmazonEC2FullAccess and IAMReadOnlyAccess. 
In additional to the AWS managed permission policies, we also set up a custom policy pass-iam-role:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::<account_id>:role/*"
    }
  ]
}
```

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
  sudo apt-get install -y fontconfig libfontconfig-dev
  cd ./impeller-experiments/latency/ && cargo build --release && cd -
  ```

### Experiments ###

#### Example command
- run query 1 for 60 seconds with 1 iterations
  ```bash
  cd ./impeller-experiments/nexmark_impeller/ && ./run_q1_quick.sh && cd -
  ```
  The experiment outputs lots of logs detailing each steps. If you don't want to see those, you can disable the base log print by removing `set -x`.
  You may see the line similar to `fail to remove nexmark network` or `fail to remove a service`. This is expected for a clean start. The script is trying
  to kill the previous running experiment and on the fresh start, there's nothing to kill and it will output such message. The result log is in `result.log`.
  For a successful run, you should see metrics and stats for each task. At the end, you should see the script downloads the stats like this:
  ```bash
  source-2.json.gz                                                                                                                                                                    100%  118   584.9KB/s   00:00
  source-3.json.gz                                                                                                                                                                    100%  118   573.9KB/s   00:00
  source-0.json.gz                                                                                                                                                                    100%  118   731.6KB/s   00:00
  query1-3.json.gz                                                                                                                                                                    100%   34KB  70.8MB/s   00:00
  source-1.json.gz                                                                                                                                                                    100%  117   714.3KB/s   00:00
  query1-0.json.gz                                                                                                                                                                    100%   34KB  86.2MB/s   00:00
  query1-2.json.gz                                                                                                                                                                    100%   34KB  87.9MB/s   00:00
  query1-1.json.gz                                                                                                                                                                    100%   34KB  89.7MB/s   00:00
  ```

#### Rerun experiments that produce figure 7
- Artifact evaluator: Skip this step
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
- Artifact evaluator: Skip this step
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

#### Collect result for 10ms, 25ms and 50ms for figure 8
```bash
# Impeller
# for example, query 1, for the other query, use q2 to q8
f=10 # or 25 or 50
python3 parse_e2e_latency.py \
    --dir $q1_exp_dir \
    --out_stats $out_dir/epoch/${f}ms/ --app q1 --target epoch

# Kafka Streams transaction on Impeller
python3 parse_e2e_latency.py \
    --dir $q1_exp_dir \
    --out_stats $out_dir/remote_2pc/${f}ms/ --app q1 --target remote_2pc
```

### License ###
All of our repository included in the artifact are in Apache 2.0 License.
We use a script docker-stack-wait.sh which is districuted in MIT License.

### Paper ###
Impeller: Stream Processing on Shared Logs

### Known limitations ###
- For long running experiments, sometimes Boki cluster might fail to setup and if you see the experiment emits a timeout error for waiting the Boki to finish setup, you need to record the current progress and restart the experiment.
  If you see a log similar to this, it means the generator starts outputing to Boki log before Boki fully setsup. If you hits this case, kill the experiment and starts again.
  ```
  [FATAL] Failed to receive aux buffer header: EOF 
  ```
- Impeller is a research quality product which might have multiple sharp edges. Do not use this repo in production environment. 
