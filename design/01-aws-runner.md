[This is a template for Drone's change proposal process, documented [here](../README.md).]

# Proposal: AWS runner 

Author(s): tphoney

Last updated: 2021/6/25

Discussion at https://github.com/drone/drone/issues/01

## Abstract

Add a new runner that provisions EC2 instances in AWS.

## Background

There is already the autoscaler to provision runners in AWS, but there is no way to provision EC2 instances.

## Proposal

Create a new runner based on the boilr template here. That can connect to a drone server providing EC2 instances to run a build against. It will have the basic functionality of other runners:
- compile
- daemon
- exec

To help mitigate the time to spin up instances the aws runner will allow the pooling of systems for hot-builds similar to how autoscaler works.

## Rationale

Adding custom functionality to the autoscaler is difficult intricate work, that does not always benefit all users. This is an attempt to follow the design pattern of the runners, to allow more configuration by the end user.

## Compatibility

This fully compatible with the existing runner framework.

## Implementation

- Basic linux / windows support, including setup of git : POC stage
- Installation of docker, for container usage: POC stage
- Pooling: design phase

## Open issues (if applicable)

[A discussion of issues relating to this proposal for which the author does not know the solution. This section may be omitted if there are none.]
