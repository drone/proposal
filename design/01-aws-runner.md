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

We will use the golang library for amazon services link here.

Global configuration:

```
Client string
Secret string
region string
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

Communitation with the provisioned EC2 instance will happen over SSH, public / private keys are generated for each new instance. 

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
  -  A pool can only be in one region. ??
- **proposal 2**: The configuration file is simpler ???

#### How do we connect to pooled systems

- **proposal 1**: SSH key pair are randomly generated once at the start of the daemon. When a build starts we use the shared ssh key pair. If there is nothing in the pool use a brand new key pair.
- **proposal 2**: The user specifies a ssh key pair in the configuration file / .env , it shared for all EC2 instances.
- **proposal 3**: The user uses files for ssh key pair in the configuration, These would have to be in a volume if the runner is in docker / kubernetes.

#### Lifecycle of the pool

- **proposal 1**: When a daemon starts it removes any instances that were previously there with the pool name. 
- **proposal 2**: When a daemon starts we reuse any instances that were previously there with that pool name. (depends on a ssh pair key being set once)

#### Lifecycle of an EC2 instance

- **proposal 1**: When a build completes the instance is destroyed. 
- **proposal 2**: When the build completes the instance is returned to the pool. This has security concerns and possible build interaction issues especially for failed builds.

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
- Scheduling, weekends ?
