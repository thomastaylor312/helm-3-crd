# helm-3-crd
A proposal for Helm 3 using CRDs and a custom controller. This README comprises
the proposal for CRDs in Helm 3. In addition to this document, there are other
examples of what the objects will look like contained in this repo.

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
  easy way to use the current RBAC model for autorization.

## Overview

### The tl;dr version
The following points capture the high level changes compared to Helm 2. Further
details are available in the next subsection.

- State is no longer stored and releases are now diff'd against the current state
  in the cluster.
- Helm will come built in with common lifecycle management patterns and have a
  `Lifecycle` object for registering custom lifecycles 

### Details
TODO

Each lifecycle can be their own controller. We will have a "Lifecycle" type that
specifies its name and what "controller" should handle the lifecycle (internal or
external to the cluster). There will be built in lifecycles that are registered
this way and custom ones can be registered likewise

## Controller Logic
TODO: Flow diagram (or maybe just a list of steps)
Controller should watch for changes in the manifest, values, or secret specified
in the release. It should also watch for changes in the release object

## Open questions
This is a list of open questions that should be resolved before the proposal is
fully accepted. If a larger discussion is needed, we will open an [issue](https://github.com/thomastaylor312/helm-3-crd/issues)
and link it to the question.

- Should this be a simple controller or an extension API server? This could allow
  for custom storage backends but would be more difficult to program (and definitely
  harder for users to install). It may be the more robust option
- Should rendering take place locally or in the controller?
- How should we handle releases that are spread across multiple namespaces?
  Possibly a `GlobalRelease` object that is non-namespaced
- Should the controller enforce the given manifest again if changes are detected
  in the objects created by a release? This could be a feature that is optional
  and can be set by something like `enforce: true` in the `Release`

## Contributing to the proposal
This is an active proposal and feedback is definitely needed. If you see something
you'd like to change, please open a [Pull Request](https://github.com/thomastaylor312/helm-3-crd/pulls).
We can discuss the change you add and either merge it in or close based on the
outcome of the discussion.
