# Proposal: AWS runner 

Author(s): tphoney

Last updated: 2021/6/25

Discussion at https://github.com/drone/drone/issues/02

## Abstract

Add a new runner that provisions EC2 instances in AWS.

## Background

There is already the autoscaler to provision drone runners in AWS, but there is no way to provision EC2 instances. The autoscaler is solid, but there are a number of customer requirements that are specific to that customer. This is an attempt to provide something more configurable by the customer.

## Proposal

Create a new runner based on the boilr template here. That can connect to a drone server providing EC2 instances to run a build against. It will have the basic functionality of other runners:

- compile
- daemon
- exec

#### AWS specific configuration

We will use the golang library for amazon services link `https://aws.amazon.com/sdk-for-go/`.

Global configuration:

```
AwsAccessKeyID     string
AwsAccessKeySecret string
AwsRegion          string
PrivateKeyFile     string
PublicKeyFile      string
ReusePool          bool
```

Instance configuration:

```
Image         string
Name          string
Region        string
Size          string
Subnet        string
Groups        []string
Device        string
PrivateIP     bool
VolumeType    string
VolumeSize    int64
VolumeIops    int64
IamProfileArn string
Userdata      string
Tags          map[string]string
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

- **proposal 1**: The configuration file mirrors the build file, with the same settings mentioned in `AWS specific configuration`. 
  -  Each pool only has one instance type. 
  -  There can be multiple pools. 1 pool == 1 pipeline
  -  The pool name == name of the pipeline.
  -  You can specify the size of the pool. 
  -  A pool can only be in one region.
- **proposal 2**: The configuration file is simpler ???

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

When an instance is created for a pool it has the aws tags:

```
drone=drone-runner-aws
pool=<pool name / pipeline name>
```
The instance will live until it is used for a build or is terminated at shutdown of the runner.

When a pooled instance is picked up for a build, it is tagged in aws with:

```
status=build in progress
```

The instance is removed when the build terminates

##### ad-hoc instance

This can happen if there are no instances in the pool, or the user hasnt set a ssh key pair.

When an instance is created for immediate use it has the aws tags:

```
drone=drone-runner-aws
pool=<pool name / pipeline name>
status=build in progress
```

The instance is removed when the build terminates

## Rationale

Adding custom functionality to the autoscaler is difficult intricate work, that does not always benefit all users. This is an attempt to follow the design pattern of the runners, to allow more configuration by the end user.

## Compatibility

This fully compatible with the existing runner framework.

## Implementation

- Basic linux / windows support, including setup of git / SSH : POC stage
- Installation of docker, for container usage: POC stage
- Pooling: design phase

## Open issues (if applicable)

- Windows - docker volumes
- Windows - docker services
- when is docker ready, currently we have a timer
- terminate instances on shutdown
