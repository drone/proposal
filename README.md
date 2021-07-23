# Proposing Changes to Drone

## Introduction

The Drone project's development process is design-driven. Significant changes to the project, libraries, or tools must be first discussed, and sometimes formally documented, before they can be implemented.

This document describes the process for proposing, documenting, and implementing changes to the Drone project.

## The Proposal Process

### Goals

- Make sure that proposals get a proper, fair, timely, recorded evaluation with a clear answer.
- Make past proposals easy to find, to avoid duplicated effort.
- If a design doc is needed, make sure contributors know how to write a good one.

### Definitions

- A **proposal** is a suggestion filed as a GitHub issue, identified by having the Proposal label.
- A **design doc** is the expanded form of a proposal, written when the proposal needs more careful explanation and consideration.

### Scope

The proposal process should be used for any notable change or addition to the project, libraries and tools. Since proposals begin (and will often end) with the filing of an issue, even small changes can go through the proposal process if appropriate. Deciding what is appropriate is matter of judgment we will refine through experience. If in doubt, file a proposal.

### Yaml changes

Drone is a mature project with thousands of active installations, as such, significant or breaking yaml changes are unlikely to be accepted.

### Process

- [Create an issue](https://github.com/drone/proposal/issues/new) describing the proposal.

- The proposal is followed by an initial discussion about the suggestion.
	- The goal of the initial discussion is to reach agreement on the next step:
		(1) accept, (2) decline, or (3) ask for a design doc.
	- The discussion is expected to be resolved in a timely manner.
	- If the author wants to write a design doc, then they can write one.

- If a Proposal issue leads to a design doc:
	- The design doc should be checked in to [the proposal repository](https://github.com/drone/proposal/) as `design/{number}-{shortname}.md`, where `{number}` is the GitHub issue number and `{shortname}` is a short title.
	- The design doc should address any specific issues asked for during the initial discussion.
	- It is expected that the design doc may go through multiple checked-in revisions.
	- Comments on the proposal should be addressed to the related GitHub issue.

- Once comments and revisions on the design doc wind down, there is a final discussion about the proposal.
	- The goal of the final discussion is to reach agreement on the next step:
		(1) accept or (2) decline.
	- The discussion is expected to be resolved in a timely manner.

- The author (and/or other contributors) do the work as described by the "Implementation" section of the proposal.

## Help

If you need help with this process, please contact the project contributors by posting
to the [Slack channel](https://join.slack.com/t/harnesscommunity/shared_invite/zt-90wb0w6u-OATJvUBkSDR3W9oYX7D~4A).
