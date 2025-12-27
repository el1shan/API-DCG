## Table Of Contents
  - [Notations and Terminology](#notations-and-terminology)
  - [Purpose of the Work](#purpose-of-the-work)
  - [Architecture and Components](#architecture-and-components)
  - [API Specification](#api-specification)

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

## Purpose of the Work

The main idea of this project is to develop a Kubernetes Controller thath observes Kubernetes Objects
and checks them for Drift & Conformance Rules with defined rules. It publishes the results as 
CRD reports, Events, Conditions and metrics.

The project can be used in Kubernetes clusters for:

- quality control of manifests/deployments
- detecting configurations anti-patterns

## Architecture and Components

### Components

The project consists of several key components:

- *Controller*: responsible for generating and updating *DriftReport*, publishing *Conditions* and *Events*, 
  exporting metrics to Prometheus and implementing *Reconcile Loop* for *DriftPolicy* or target resources

- *CRD Resources*: defines *DriftPolicy* and *DriftReport*

- *Metrics Endpoint*: provides a `/metrics` endpoint for Prometheus scrapping

- *Tests*: includes unit, integration and end-to-end (e2e) tests

- *CI*: contains continuous integration pipelines for automated testing and validation

### Architecture

**TODO: add picture**

## API Specification

1. *DriftPolicy*
   
   There is namespaced resource. Expected fields are:
   
   - `spec.target`
     - `apiGroups` (required): List of Kubernetes API groups to match target resources
     - `kinds` (required): List of Kubernetes resource kinds to be evaluated
     - `namespaces` (optional): List of namespaces to which the policy applies
     - `labelSelector` (optional): Standart Kubernetes `LabelSelector` to filter target resources
   
   - `spec.rules[]`
     - `name` (required): Name of the rule to apply
     - `enabled` (required, default: `true`): Enables or disables the rule
     - `params` (optional): Configuration parameters

   - `spec.report`
     - `mode` (required): Report generation mode. Supported value: PerObject
     - `ttlSecondsAfterFinished` (optional): Time-to-live for completed reports, enforced by the controller
    
   - `spec.status`
     - `conditions`: Current status of the policy
     - `observedGeneration`: Most recent generation observed by the controller
     - `lastScanTime`: Timestamp of the last completed scan
     - `stats`: Aggregated statistics for the policy. Consist of `targetObjectsScanned` and `violationsTotal`
    
2. *DriftReport*

   One *DriftReport* corresponds to a single checked object. Expected fields are:

   - `spec.policyRef`
     - `name` (required): Name of the policy
  
   - `spec.subject`
     - `apiVersion` (required): API version of the evaluated Kubernetes object
     - `kind` (required): Kind of the evaluated Kubernetes object
     - `name` (required): Name of the evaluated Kubernetes object
     - `namespace` (required): Namespace of the evaluated Kubernetes object
    
   - `status.summary`
     - `violations`: number of detected rule violations
     - `passed`: number of rules that passed without violations
    
   - `status.violations[]`
     - `rule`: Name of the rule that was violated
     - `severity`: Severity level of the violation. Supported values: `low`, `medium`, `high`
     - `message`: Human-readable description of the violation
     - `fieldPath`: JSON field path within the subject object related to the violation
     - `docURL`: Reference URL to the rule documentation
     - `detectedAt`: Timestamp when the violation was detected
    
   - `status.conditions[]`
     - `Ready`: `True` if the report reflects the current evaluated state of the subject
     - `Stale`: `True` if the subject object is deleted or no longer accessible
