# Chapter 1: Reliable, Scalable, and Maintainable Applications

---

- [Reliability](#reliability)
  - [Faults](#faults)
    - [Hardware Errors](#hardware-errors)
    - [Software Errors](#software-errors)
    - [Human Errors](#human-errors)
- [Scalability](#scalability)
- [Maintainability](#maintainability)
  - [Operability](#operability)
  - [Simplicity](#simplicity)
  - [Evolvability](#evolvability)
- [Summary](#summary)

---

- Many applications today are _data-intensive_ (as opposed to _compute-intensive_):

  - **Database**: store and retrieve data
  - **Cache**: remember result of expensive operation
  - **Search indexes**: filter data
  - **Stream processing**: handle dataasynchronously
  - **Batch processing**: periodically crunch large amount of accumulated data

- Data systems often combine multiple smaller, general-purpose data systems to create a new, special-purpose data system.

- Some factors that influence design:

  - Skills/experience of people
  - Legacy system dependencies
  - Time-scale for delivery
  - Tolerance for risk
  - Regulatory constraints
  - [Reliability](#Reliability)
  - [Scalability](#Scalability)
  - [Maintainability](#Maintainability)

## Reliability

**_TL;DR: Continuing to work correctly, even when things go wrong._**

_System should work correctly even in the face of adversity._

- Performs the expected function
- Tolerates mistakes and unexpected usage
- Performance is sufficient under expected load/data volume
- Prevents unauthorized access and abuse

### Faults

_Things that can go wrong._

**Fault** (one part fails) != **Failure** (whole system fails)

**Fault-tolerant** / **Resilient**: systems that anticipate faults and can cope with them

Often better to force faults instead of error handling.

- Error handling is hard
- Validates proper fault handling

However, some faults are dangerous (e.g. security).

#### Hardware Errors

##### Examples

- Hard disks crash
- RAM becomes faulty
- Power grid has a blackout
- Someone unplugs the wrong network cable

##### Mitigations

- Redundancy
  - Also useful for rolling upgrades.

#### Software Errors

##### Examples

- Software bug for bad input
- Runaway process that uses up some shared resource
- External dependency slows down, becomes unresponsive, or starts returning corrupted responses
- Cascading failures

##### Mitigations

No quick solution but:

- Carefully think about assumptions and interactions in the system
- Testing
- Process isolation
- Allowing processes to crash and restart
- Measuring, monitoring, and analyzing system behavior

#### Human Errors

Humans are unreliable.

##### Mitigations

- Minimize opportunities for error
  - Design interfaces that make it easy to do the right thing
- Decouple development environment from production environment
- Testing
- Allow quick and easy recovery
  - Rollbacks
  - Incremental rollout
  - Data restoration
- Telemetry
- Good management practice and training

## Scalability

**_TL;DR: System's ability to cope with increased load._**

_As system grows (in data volume, traffic volume, or complexity), there should be reasonable ways to dealing with the growth._

- **Load parameter**: numbers that describe load (for average or edge case)
  - requests per second
  - ratio of reads to writes
  - number of simultaneously active users
  - hit rate on a cache

Questions to consider:

- How is performance affected when load increases?
- How much more resources are needed when load is increased to keep performance unchanged?

Latency != Response time:

- **Latency** = duration that a request is waiting to be handled
- **Response time** = **Service time** (time to process request) + network delays + queueing delays

Instead of **average**, usually more valuable to define **percentiles**.

**Tail latency**: high percentile of response times

- Directly affects most users
- Not worth optimizing all 100% because
  - Too expensive
  - Difficult
  - Affected by random events outside control
  - Not beneficial enough

**Service Level Objectives (SLO)** and **Service Level Agreements (SLA)** define expected performance and availability of a service.

**Head-of-line blocking**: when requests hold up the processing of subsequent requests (due to insufficient parallelism)

- Leads to high **response times** even with low **latencies**

**Tail latency amplification**: the effect where a higher proportion of end-user requests end up being slow because each request requires multiple backend calls and multiple backend calls increases the chance of getting a slow call, slowing down the parent request

Scaling types

- **Scaling up** / **Vertical scaling**: moving to a more powerful machine
  - Simpler but expensive
- **Scaling out** / **Horizontal scaling** / **Shared-nothing architecture**: distributing the load across multiple smaller machines

Systems usually combine both.

- **Elastic**: automatically adding computing resources when detecting load increase; useful if load is unpredictable but more complex and has more operational surprises

Distributing stateless services is easy.

Distributing stateful services is hard.

No _magic scaling sauce_ (no one size fits all) but there are general-purpose building blocks.

## Maintainability

_Different people should be able to productively maintain current behavior and adapt the system to new use cases._

- Fixing bugs
- Keeping systems operational
- Investigating failures
- Adapting to new platforms
- Modifying for new use cases
- Repaying technical debt
- Adding new features

Three design principles:

- [Operability](#Operability)
- [Simplicity](#Simplicity)
- [Evolvability](#Evolvability) / **Extensibility** / **Modifiability** / **Plasticity**

### Operability

_Make it easy to keep the system running smoothly._

Operation teams are responsible for:

- Monitoring health of system and quickly restoring service if it goes into a bad state
- Tracking down cause of problems
- Keeping software and platforms up to date
- Avoiding problematic changes before they cause damage
- Anticipating and solving future problems before they occur
- Establishing good practices and tools
- Performing complex maintenance tasks
- Maintaining security of system as configuration changes are made
- Defining processes that make operations predictable and help keep production environment stable
- Preserving knowledge about the system, even as individual people come and go

Systems should make operations easy by:

- Monitoring
- Supporting automation and integration with standard tools
- Avoiding dependency on individual machines
- Documenting
- Providing good defaults with freedom to override them
- Self-healing where appropriate with ability to manually control when needed
- Exhibiting predictable behavior

### Simplicity

_Make it easy to understand the system._

Symptoms of complexity:

- Explosion of the state space
- Tight coupling of modules
- Tangled dependencies
- Inconsistent naming and terminology
- Hacks aimed at solving performance problems
- Special-casing to work around issues elsewhere

More risk with making a change.

**Abstractions** help reduce **accidental complexity**.

### Evolvability

_Make it easy to make changes to the system._

System requirements change:

- Learn new facts
- Previously unanticipated use cases emerge
- Business priorities change
- Users request new features
- New platforms replace old platforms
- Legal or regulatory requirements change
- Growth of the system

## Summary

Applications have:

- **Functional requirements**: what the system should do
- **Nonfunctional requirements**: security, reliability, compliance, scalability, compatibility, maintainability, etc.

**Reliability**: coping with faults (hardware, software, human)

**Scalability**: coping with load (data, traffic)

**Maintainability**: coping with complexity and changes
