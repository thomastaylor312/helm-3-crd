# helm-3-crd
A proposal for Helm 3 using CRDs and a custom controller. This README comprises
the proposal for CRDs in Helm 3. In addition to this document, there are other
examples of what the objects will look like contained in this repo. This is not
meant to be a completely exhaustive description of all behavior, but is intended
to give a enough details as a starting point for actual implementation details
should this proposal be accepted.

## Caveat
NOTE: Although I am a core maintainer, this does not represent the direction
that the project _will_ go. It is solely a proposal for what I personally think
should be done for Helm 3

## Problem Statement
Originally, Helm was built as a way to share reusable components. However, as
its usage has grown, many want to use Helm more as a configuration management
tool. Also, many of the assumptions made when Helm 2 was first released have
not proved true now that Kubernetes has evolved. Below is the list of issues 
current users of Helm 2 face and that can be addressed by this proposal.

- Lack of application lifecycle management
- The current Tiller model does not follow any of the prescribed ways for 
  extending Kubernetes
- Security is difficult and not built in by default. Furthermore, there is no
  easy way to use the current RBAC model for authorization.
- The current method for storing releases is hard to read/understand and
  all parts of the release are stored together

## Overview

### Changes from Helm 2
The following points capture the high level changes compared to Helm 2.

- State is no longer stored and releases are now diff'd against the current state
  in the cluster.
- Helm will come built in with common lifecycle management patterns and have a
  `Lifecycle` object for registering custom lifecycles
- Complete backwards compatibility with the Helm 2 chart spec will be maintained.
  For example, hooks will be transparently converted to their equivalent `Lifecycle`
  object in Helm 3
- The current Release protobuf object has been decomposed into separate objects.
  Namely, `Release`, `Values`, `Manifest`, and `Secret` objects.

## Controller Logic
This is a simple list of logic that the controller will handle

- Tiller should watch for changes in the `Manifest`, `Values`, or `Secret` 
  specified in the `Release`. It should also watch for changes in the `Release`
  object. When a change is observed, a new release occurs.
- On the creation of a `Release` and on subsequent updates, a `ReleaseAudit`
  event is created to track the history of the release
- All Kubernetes resources created will have an `OwnerRef` that points to the 
  `Release` object that created it.
- Tiller will publish a `LifecycleEvent` with the configured `eventKey` for each
  `Lifecycle` specified in a `Release`. It will watch these events until the 
  lifecycle controller responsible for handling that event marks it as done (or
  errored), update the `Release` object, and then delete the event. If it doesn't
  complete after a certain amount of time, the event will be cleaned up and failure
  information added to the `Release` status.
- Each lifecycle will be handled by their own controller. The `Lifecycle` object
  specifies its name and what key the controller should watch for. This key will
  be used as the label value for filtering events in a lifecycle controller. Both
  built in and custom lifecycles will be registered in the same way.

## Open questions
This is a list of open questions that should be resolved before the proposal is
fully accepted. If a larger discussion is needed, we will open an [issue](https://github.com/thomastaylor312/helm-3-crd/issues)
and link it to the question.

- ~Should this be a simple controller or an extension API server? This could allow
  for custom storage backends but would be more difficult to program (and definitely
  harder for users to install). It may be the more robust option~ Just a controller
  for ease of use
- ~Should rendering take place locally or in the controller?~ locally
- How should we handle releases that are spread across multiple namespaces?
  Possibly a `ClusterRelease` object that is non-namespaced
- Should the controller enforce the given manifest again if changes are detected
  in the objects created by a release? This could be a feature that is optional
  and can be set by something like `enforce: true` in the `Release`
- With state no longer being stored in a config map, ​how do we handle objects
  that should be deleted because they are no longer in the manifests?
- How will things like lifecycles be configured by the user? One idea is to
  create a reserved key in `values.yaml` that can be used for configuring
  lifecycle behavior

## Contributing to the proposal
This is an active proposal and feedback is definitely needed. If you see something
you'd like to change, please open a [Pull Request](https://github.com/thomastaylor312/helm-3-crd/pulls).
We can discuss the change you add and either merge it in or close based on the
outcome of the discussion.
