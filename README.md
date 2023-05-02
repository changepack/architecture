This is an architecture repository for [Changepack](https://github.com/changepack/changepack) maintained by Changepack’s core team.

- [Motivation](#motivation)
  - [Align the architecture of the system to the project goals](#align-the-architecture-of-the-system-to-the-project-goals)
  - [Find and recommend solutions to shared problems](#find-and-recommend-solutions-to-shared-problems)
  - [Share architecture and technical knowledge](#share-architecture-and-technical-knowledge)
- [How the core team works](#how-the-core-team-works)
  - [RFC](#rfc)
  - [Decision-making](#decision-making)
- [How the core team cooperates with others](#how-the-core-team-cooperates-with-others)
- [Benefits](#benefits)

## Motivation

### Align the architecture of the system to the project goals

When the architecture does not correspond to the goals of the organisation, implementing each new feature takes longer with time, or even becomes impossible. The core team holds the responsibility of keeping the architecture clean and aligned with the project.

### Find and recommend solutions to shared problems

Infrastructure and cross cutting concerns are shared between teams, so it makes sense to share the solutions to common problems among the organisation. The architecture guild is responsible for recommendations to issues that have long lasting consequences or a high cost of change. This is especially important if a decision by one team, influences other teams.

### Share architecture and technical knowledge

The architecture guild aims to provide recommendations for issues with long-lasting consequences or a high cost of change. This is particularly important in an open-source project, where decisions can impact various contexts, forks, and modifications by numerous contributors. By fostering an inclusive and collaborative environment, we encourage contributions from diverse perspectives to create a more robust and adaptable architecture.

## How the core team works

The core team works asynchronously. It discusses the problems and work required on GitHub, invites all the relevant people, creates an RFC document to gather all possible input from people with the knowledge on the issue matter, and chooses the person responsible for getting the decision done.

### RFC

RFC (Request for Comments) is a technique of efficient meeting-less communication and decision making. The goal is to allow the people with the knowledge to express their opinion, so that the final decision is well informed. Authors propose solution in a document in this repository, where anyone interested can add their comments or suggestion.

These documents capture:

- Context: what information did we consider while making this decision?
- Considered alternatives: what options did we see?
- Decision: what do we recommend?
- Rationale: why did we make that recommendation?
- Consequences: what are the known drawbacks?

It usually takes 1 week to get all the needed input, but that depends on the urgency of the issue. After all the input is gathered, the core team makes the decision.

### Decision-making

The decision is done after all the comments from RFC are gathered. The core team makes the decision together. If the people on the meeting cannot agree, the final decision is up to the Maintainer.

Architecture Decision Record is a way of documenting architectural and technical decisions. These documents are created after the discussion on the issue, and capture:

- Context: what information did we consider while making this decision?
- Considered alternatives: what options did we see?
- Research outcomes: what did we learn out of researching this problem and discussing it?
- Decision: what do we recommend?
- Rationale: why did we make that recommendation?
- Consequences: what are the known drawbacks?

## How the core team cooperates with others

All developers are welcome to work with the core team on Changepack’s architecture.
The core team is inclusive, that means it accepts new all members by default as contributors.
Everyone can report an issue to the core team and request for help, or provide insight into the problem.
People who join the core team will be held accountable for their input and may be requested to do the research or POCs on the best-effort basis.

## Benefits

If the process is followed, decisions nad recommendations made by the core team are well informed. No decision is made by a person who does not have the required architecture knowledge.
Because the decision making process is inclusive, no decision is made without the input of the people who have to live with the consequences of that decision.
A single person cannot have enough visibility into a large distributed system, to make sensible decision. Because the core team is composed of many people, from all sides of the system, decision take into account all the perspectives on the system we build.
