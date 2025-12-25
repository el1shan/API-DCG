## Table Of Contents
  - [Notations And Terminology](#notations-and-terminology)

## Notations And Terminology

### Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Terminology

This specification defines the following terms:

- *Kubernetes*: open source system for managing containerized applications across
  multiple hosts. It provides basic mechanisms for the deployment, maintenance, 
  and scaling of applications.

- *Kubernets API Object*: there are persistent entities in the Kubernetes system.
  Kubernetes usese these entities to represent the state of your cluster.

- *Kubernetes Resource*: a resource is an endpoint in the Kubernetes API that stores
  a collection of API objects. For example, the built-in pods resource contains
  a collection of Pod objects.

- *Kubernetes Custom Resources*: an extension of Kubernetes API that is not necessarily
  available in a default Kubernetes installation. It represents a customization of a
  particular Kubernetes installation. However, many core functions are now built using
  custom resources, making Kubernetes more modular.

- *Kubernetes Controllers*: keeps the current state of Kubernetes objects in sync with
  your declared desired state.
  Custom Kubernetes Controllers can work with any kind of resource, but they are especially
  effective when combined with custom resources.

- *DriftPolicy*: CRD which describes rules and target-resources.

- *DriftReport*: CRD which contains audit results for a specific object.

- *Rule*: atomic check. For example, constraint on `:latest` tag.

- *Violation*: the fact of violation of the rule by the object.
