# Proposal: AWS runner 

Author(s): tphoney

First created: 2021/6/25

Last updated: 2021/7/21

Discussion at https://github.com/drone/drone/issues/02

## Abstract

Add a new runner that provisions EC2 instances in AWS.

## Background

There is already the autoscaler to provision drone runners in AWS, but there is no way to provision EC2 instances. The autoscaler is solid, but there are a number of customer requirements that are specific to that customer. This is an attempt to provide something more configurable by the customer.

## Proposal

Create a new runner based on the boilr template here. That can connect to a drone server providing EC2 instances to run a build against. It will have the basic functionality of other runners:

- compile
- daemon
- exec - only for development of the runner

#### AWS specific configuration

We will use the golang library for amazon services link `https://aws.amazon.com/sdk-for-go/`.

Global configuration along with other standard runner variables (in the .env file): 

```
DRONE_SETTINGS_AWS_ACCESS_KEY_ID      string
DRONE_SETTINGS_AWS_ACCESS_KEY_SECRET  string
DRONE_SETTINGS_AWS_REGION             string
DRONE_SETTINGS_PRIVATE_KEY_FILE       string
DRONE_SETTINGS_PUBLIC_KEY_FILE        string
DRONE_SETTINGS_REUSE_POOL             bool  - on startup shutdowm whether to kill existing instances in EC2
```

### Communication

Communitation with the provisioned EC2 instance will happen over SSH, If provided we will use the public / private key file, or generate keys for each new instance. 

### Docker support

Allow the use of:

- docker images in a step 
- drone plugins
- shared docker / temporary volumes
- drone services 
- drone detached steps.
- run in priviliged mode (windows cannot do this)

### Pooling

Add the ability to seed an instance, to save on spin up time. This provides some of the functionality of the autoscaler. 

#### Configuration of the pool

- simple configuration file, mirroring some of the build file.
-  .drone_pool.yml is the default file name
-  Each pool only has one instance type. 
-  There can be multiple pools. Different pipelines can use the same pool
-  You can specify the size of the pool. 
-  A pool can only be in one region.
-  Changing the pool configuration will mean removing the existing images and restarting the daemon.
-  If the pool is empty, it will trigger an adhoc instance.

#### Here are pool settings settings

A pool has:

```yaml
  name          string
	max_pool_size int
	platform      Platform
	account       Account
	instance      Instance
```

where Platform is(this is the same as plaform in other runners):

```yaml
  os      string
	arch    string
	variant string
	version string
```

where Account is (over-rides env settings):

```yaml
  access_key_id     string
	access_key_secret string
	region            string
```

where Instance contains aws instance specific ima:

```yaml
  ami              string
	tags             []string
	iam_profile_arn  string
  type             string
  disk             Disk
  network          Network
```

where Disk contains aws block information:

```yaml
  size int
  type string
  iops string
```

where Network contains aws network information:

```yaml
  vpc                 int
  vpc_security_groups []string
  security_groups     []string
  subnet_id           string
  private_ip          bool
```

EG, This `.drone_pool.yml` file configures 2 pools each with a maximum pool size of 1

```yaml
name: common
max_pool_size: 1

account:
  region: us-east-2

instance:
# ubuntu 18.04 t1-micro ohio
  ami: ami-051197ce9cbb023ea 
  private_key: ./private_key_file
  public_key: ./public_key_file
  type: t2.nano
  network:
    security_groups:
      - sg-0f5aaeb48d35162a4 

----
name: windows 2019
max_pool_size: 1

account:
  region: us-east-2

platform:
 os: windows  

instance:
# ubuntu 18.04 t1-micro ohio
  ami: ami-051197ce9cbb023ea 
  private_key: ./private_key_file
  public_key: ./public_key_file
  type: t2.nano
  network:
    security_groups:
      - sg-0f5aaeb48d35162a4 
```

#### How do we connect to systems

If we are using pools, the user specifies a ssh key pair in the configuration file / .env , it shared for all EC2 instances. 
or
If we are not using pools a new SSH key pair is randomly generated for each instance.

#### Lifecycle of the pool

If ReusePool is true, when a daemon starts we reuse any instances that were previously there with that pool name. (depends on a ssh pair key being set in config)
   ReusePool is false, When a daemon starts it removes any instances that have the tag pair drone=drone-runner-aws. 
   
On shutdown or termination:

If ReusePool is true, the instances are left running (depends on a ssh pair key being set in config)
   ReusePool is false, any instances that have the tag pair drone=drone-runner-aws. 

#### Lifecycle of an EC2 instance

##### Pool instance

Here is a build file that references uses a pool. 

```yaml
kind: pipeline
type: aws
name: default

pool:
  use: common
      
steps:
- name: display go version
  image: golang
  pull: if-not-exists
  commands:
  - go version
```

When an instance is created for a pool it has the aws tags:

```
drone=drone-runner-aws
pool=<pool name>
```
The instance will live until it is used for a build or is terminated at shutdown of the runner.

When a pooled instance is picked up for a build, it is tagged in aws with:

```
status=build in progress
```

***If there are no pool instances left, we will create an adhoc instance. We use the information from the pool file to create this system***

The instance is removed when the build terminates

Finally check to see if the pool should be re-populated and create a pool instance.

## Rationale

Adding custom functionality to the autoscaler is difficult intricate work, that does not always benefit all users. This is an attempt to follow the design pattern of the runners, to allow more configuration by the end user.

## Compatibility

This fully compatible with the existing runner framework.

## Implementation

- Basic linux / windows support, including setup of git / SSH : POC stage
- Installation of docker, for container usage: POC stage
- Pooling: POC stage

## Open Questions (if applicable)
- Should we remove running build instances on startup, always.
- Certain errors in engine will cause hard failures, killing any running builds. is this desireable 
