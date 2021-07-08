# Proposal: Report Agent Status to the Server 

Author(s): julienduchesne

Last updated: 2021/07/08

Initial proposal here: https://github.com/drone-runners/drone-runner-docker/pull/15#issuecomment-871286103

## Abstract

Have agents report their status regularly to the drone server

## Background

There is currently no way to know the status of all agents currently reporting to the Drone server. 
[A PR was opened to expose prometheus metrics on Drone agents directly](https://github.com/drone-runners/drone-runner-docker/pull/15#issuecomment-871286103) but it's not a generic enough solution since it would only be implemented for the Docker runner. 

## Proposal

Have the [`Ping` function](https://github.com/drone/runner-go/blob/master/client/http.go#L88) of `runner-go` send some information along with its ping:

* runner name
* hostname
* configured capacity
* available capacity

This information will be aggregated on the Drone server and exposed in multiple ways:

* API endpoint to list all agents/display that on the UI (runner/last report date/capacity table view)
* Prometheus metrics (maybe only show agents that have reported in the last 5 minutes, if ping is every minute?)
  * agent_configured_capacity{name="..."} 4
  * agent_available_capacity{name="..."} 4

Then modify the different runners to Ping regularly instead of only once at startup

## Rationale

By modifying the runner-go `Ping` function, we are ensuring that all runners will eventually have to implement the behavior

## Compatibility

This fully compatible with the existing runner framework.

## Implementation

- Gather additional information from pings on the server and save that into the database (allowing for pings without the info, for retro-compatibility)
- Modify the `Ping` function to send additional information in the POST body
- Modify runners to use the new runner-go code, implement a regular ping in each

## Open issues (if applicable)

