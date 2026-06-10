# Kubernetes Up and Runnning

## Chapter 1 - Introduction

### Definition

Kubernetes is an open source orchestrator for deploying containerized applications.

### Whose from

It was originally developed by Google, inspired by a decade of experience deploying scalable, reliable systems in containers via application-oriented APIs.

### When it was launch?

2014

### Topics

- distributed system
- reliable
- availability
- scalable

### Benefits

- Development velocity
- Scaling (of both software and teams)
- Abstracting your infrastructure
- Efficiency
- Cloud native ecosystem

### Velocity

- Software changed
  - Before
    - Shipping products as boxed CDs or DVDs
    - It was normal to have websites in maintenance mode
  - Now
    - Network via web-based services
    - Deployed hourly
    - The web sites must be up all the time

So Kubernetes have some characteristics that improves the velocity based on these concepts:

	- Immutability
	- Declarative configuration
	- Online self-healing systems
	- Shared reusable libraries and tools

### The Value of Immutability

Declarative x Imperative configuration

### Scaling Your Service and Your teams

#### Decoupling 

- Each team can focus on a single service to adhere the microservices architecture
#### Easy Scaling for Applications and Clusters

- Since it is a declarative configuration, it is easy to scale, cause it needs to create another containres from the same image and environment variables (of course with some alterations like port number)
#### Scaling Development Teams with Microservices

- two pizza team - small teams to develop software as fast as possible
	- good knowledge sharing
	- fast decision making
	- common sense of purpose
	- Larger teams suffer with
		-  hierarchy issues
		- poor visibility
		- infighting
- each small team develop a decoupled service and them, each service interact with the others via network APIs

Kubernetes help with it, how?

- pods: group of containers, can group together container images developed by different teams into a single deployable unit
- services: provide load balancing, naming, and discovery to isolate one microservice from another
- namespaces: provide isolation and access control, so that each microservice can control the degree to which other services interact with it.
- ingress: provide an easy-to-use frontend that can combine multiple microservices into a single externalized API surface area.

