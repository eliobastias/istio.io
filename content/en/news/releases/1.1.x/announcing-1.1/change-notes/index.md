---
title: Istio 1.1 Change Notes
release: 1.1
subtitle: Minor Release
linktitle: 1.1 Change Notes
description: Istio 1.1 release notes.
weight: 10
publishdate: 2019-03-19
aliases:
    - /about/notes/1.1
---

## Incompatible changes from 1.0

In addition to the new features and improvements listed below, Istio 1.1 has introduced
a number of significant changes from 1.0 that can alter the behavior of applications.
A concise list of these changes can be found in the [upgrade notice](/news/releases/1.1.x/announcing-1.1/upgrade-notes).

## Upgrades

We recommend a manual upgrade of the control plane and data plane to 1.1. See
the [upgrades documents](/docs/setup/upgrade/) for more information.

{{< warning >}}
Be sure to check out the [upgrade notice](/news/releases/1.1.x/announcing-1.1/upgrade-notes) for a
concise list of things you should know before upgrading your deployment to Istio 1.1.
{{< /warning >}}

## Installation

- **CRD Install Separated from Istio Install**.  Placed Istio’s Custom Resource
  Definitions (CRDs) into the `istio-init` Helm chart. Placing the CRDs in
  their own Helm chart preserves the data continuity of the custom resource
  content during the upgrade process and further enables Istio to evolve beyond
  a Helm-based installation.

- **Installation Configuration Profiles**. Added several installation
  configuration profiles to simplify the installation process using well-known
  and well-tested patterns. Learn more about the better user experience
  afforded by the [installation profile feature](/docs/setup/additional-setup/config-profiles/).

- **Improved Multicluster Integration**. Consolidated the 1.0 `istio-remote`
  chart previously used for
  [multicluster VPN](https://archive.istio.io/v1.1/docs/setup/kubernetes/install/multicluster/vpn/) and
  [multicluster split horizon](https://archive.istio.io/v1.1/docs/examples/multicluster/split-horizon-eds/)
  remote cluster installation into the Istio Helm chart simplifying the operational experience.

## Traffic management

- **New `Sidecar` Resource**. The new [sidecar](/docs/concepts/traffic-management/#sidecars) resource
  enables more fine-grained control over the behavior of the sidecar proxies attached to workloads within a namespace.
  In particular it adds support to limit the set of services a sidecar will send traffic to.
  This reduces the amount of configuration computed and transmitted to
  the proxy, improving startup time, resource consumption and control-plane scalability.
  For large deployments, we recommend adding a sidecar resource per namespace. Controls are also
  provided for ports, protocols and traffic capture for advanced use-cases.

- **Restrict Visibility of Services**. Added the new `exportTo` feature which allows
  service owners to control which namespaces can reference their services. This feature is
  added to `ServiceEntry`, `VirtualService` and is also supported on a Kubernetes Service via the
  `networking.istio.io/exportTo` annotation.

- **Namespace Scoping**. When referring to a `VirtualService` in a Gateway we use DNS based name matching
  in our configuration model. This can be ambiguous when more than one namespace defines a virtual service
  for the same host name. To resolve ambiguity it is now possible to explicitly scope these references
  by namespace using a syntax of the form **`[{namespace-name}]/{hostname-match}`** in the `hosts` field.
  The equivalent capability is also available in `Sidecar` for egress.

- **Updates to `ServiceEntry` Resources**. Added support to specify the
  locality of a service and the associated SAN to use with mutual TLS. Service
  entries with HTTPS ports no longer need an additional virtual service to
  enable SNI-based routing.

- **Locality-Aware Routing**. Added full support for routing to services in the
  same locality before picking services in other localities.
  See [Locality Load Balancer Settings](/docs/reference/config/networking/destination-rule#LocalityLoadBalancerSetting)

- **Refined Multicluster Routing**. Simplified the multicluster setup and
  enabled additional deployment modes. You can now connect multiple clusters
  simply using their ingress gateways without needing pod-level VPNs, deploy
  control planes in each cluster for high-availability cases, and span a
  namespace across several clusters to create global namespaces. Locality-aware
  routing is enabled by default in the high-availability control plane
  solution.

- **Istio Ingress Deprecated**. Removed the previously deprecated Istio
  ingress. Refer to the [Securing Kubernetes Ingress with Cert-Manager](/docs/ops/integrations/certmanager/)
  example for more details on how to use Kubernetes Ingress resources with
  [gateways](/docs/concepts/traffic-management/#gateways).

- **Performance and Scalability Improvements**. Tuned the performance and
  scalability of Istio and Envoy. Read more about [Performance and Scalability](https://archive.istio.io/v1.1/docs/ops/deployment/performance-and-scalability/)
  enhancements.

- **Access Logging Off by Default**. Disabled the access logs for all Envoy
  sidecars by default to improve performance.

### Security

- **Readiness and Liveness Probes**. Added support for Kubernetes' HTTP
  [readiness and liveness probes](/about/faq/#k8s-health-checks) when
  mutual TLS is enabled.

- **Cluster RBAC Configuration**. Replaced the `RbacConfig` resource with the
  `ClusterRbacConfig` resource to implement the correct cluster scope. See
  [Migrating `RbacConfig` to `ClusterRbacConfig`](https://archive.istio.io/v1.1/docs/setup/kubernetes/upgrade/steps/#migrating-from-rbacconfig-to-clusterrbacconfig).
  for migration instructions.

- **Identity Provisioning Through SDS**. Added SDS support to provide stronger
  security with on-node key generation and dynamic certificate rotation without
  restarting Envoy.

- **Authorization for TCP Services**. Added support of authorization for TCP
  services in addition to HTTP and gRPC services. See [Authorization for TCP Services](/docs/tasks/security/authorization/authz-tcp)
  for more information.

- **Authorization for End-User Groups**. Added authorization based on `groups`
  claim or any list-typed claims in JWT. See [Authorization for JWT](/docs/tasks/security/authorization/authz-jwt/)
  for more information.

- **External Certificate Management on Ingress Gateway Controller**.
  Added a controller to dynamically load and rotate external certificates.

- **Custom PKI Integration**. Added Vault PKI integration with support for
  Vault-protected signing keys and ability to integrate with existing Vault PKIs.

- **Customized (non `cluster.local`) Trust Domains**. Added support for
  organization- or cluster-specific trust domains in the identities.

## Policies and telemetry

- **Policy Checks Off By Default**. Changed policy checks to be turned off by
  default to improve performance for most customer scenarios. [Enabling Policy Enforcement](https://istio.io/v1.6/docs/tasks/policy-enforcement/enabling-policy/)
  details how to turn on Istio policy checks, if needed.

- **Kiali**. Replaced the [Service Graph addon](https://github.com/istio/istio/issues/9066)
  with [Kiali](https://www.kiali.io) to provide a richer visualization
  experience. See the [Kiali task](/docs/tasks/observability/kiali/) for more
  details.

- **Reduced Overhead**. Added several performance and scale improvements
  including:

    - Significant reduction in default collection of Envoy-generated
      statistics.

    - Added load-shedding functionality to Mixer workloads.

    - Improved the protocol between Envoy and Mixer.

- **Control Headers and Routing**. Added the option to create adapters to
  influence the headers and routing of an incoming request. See the [Control Headers and Routing](https://istio.io/v1.6/docs/tasks/policy-enforcement/control-headers)
  task for more information.

- **Out of Process Adapters**. Added the out-of-process adapter functionality
  for production use. As a result, we deprecated the in-process adapter model
  in this release. All new adapter development should use the out-of-process
  model moving forward.

- **Tracing Improvements**. Performed many improvements in our overall tracing
  story:

    - Trace ids are now 128 bit wide.

    - Added support for sending trace data to Lightstep

    - Added the option to disable tracing for Mixer-backed services entirely.

    - Added policy decision-aware tracing.

- **Default TCP Metrics**. Added default metrics for tracking TCP connections.

- **Reduced Load Balancer Requirements for Addons**. Stopped exposing addons
  via separate load balancers. Instead, addons are exposed via the Istio
  gateway. To expose addons externally using either HTTP or HTTPS protocols,
  please use the [Addon Gateway documentation](/docs/tasks/observability/gateways/).

- **Secure Addon Credentials**. Changed storage of the addon credentials.
  Grafana, Kiali, and Jaeger passwords and username are now stored in
  [Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
  for improved security and compliance.

- **More Flexibility with `statsd` Collector**. Removed the built-in `statsd`
  collector. Istio now supports bring your own `statsd` for
  improved flexibility with existing Kubernetes deployments.

### Configuration management

- **Galley**. Added [Galley](https://archive.istio.io/v1.1/docs/concepts/what-is-istio/#galley) as the
  primary configuration ingestion and distribution mechanism within Istio. It
  provides a robust model to validate, transform, and distribute configuration
  states to Istio components insulating the Istio components from Kubernetes
  details. Galley uses the [Mesh Configuration Protocol](https://github.com/istio/api/tree/{{< source_branch_name >}}/mcp)
  to interact with components.

- **Monitoring Port**. Changed Galley's default monitoring port from 9093 to
  15014.

## `istioctl` and `kubectl`

- **Validate Command**. Added the [`istioctl validate`](/docs/reference/commands/istioctl/#istioctl-validate)
  command for offline validation of Istio Kubernetes resources.

- **Verify-Install Command**. Added the [`istioctl verify-install`](/docs/reference/commands/istioctl/#istioctl-verify-install)
  command to verify the status of an Istio installation given a specified
  installation YAML file.

- **Deprecated Commands**. Deprecated the `istioctl create`, `istioctl
  replace`, `istioctl get`, and `istioctl delete` commands. Use the
  [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl)
  equivalents instead. Deprecated the `istioctl gen-deploy` command too. Use a
  [`helm template`](https://archive.istio.io/v1.1/docs/setup/kubernetes/install/helm/#option-1-install-with-helm-via-helm-template)
  instead. Release 1.2 will remove these commands.

- **Short Commands**. Included short commands in `kubectl` for gateways,
  virtual services, destination rules and service entries.
