# Cluster Logging and OpenTelemetry

This is the protocol and semantic conventions documentation for Red Hat OpenShift Logging's OTEL support starting with Logging v6.1 which is considered **Tech-Preview**. This document should be considered as a work in progress and is subject to change until OTEL support graduates to **General Acceptance**.

## Forwarding and Ingestion Protocol

Red Hat OpenShift Logging provides a log collection and forwarding solution that is capable of writing logs to OpenTelemetry endpoints using OTLP. [OTLP](https://opentelemetry.io/docs/specs/otlp/) is the *protocol* for encoding, transporting, and delivering telemetry data.  This product additionally is capable of providing a Loki storage deployment that provides an OTLP endpont to ingest log streams.  This document defines the semantic conventions associated with the logs collected from the various sources of an OpenShift cluster.

**Note:** Logs are forwarded using OTLP/HTTP as defined by the OpenTelemetry Observability Framework.  It uses Protobuf payloads encoded in JSON format.

## Semantic Conventions

The log collector provided by this solution collects the following log streams:

* Container logs
* Cluster node journal logs
* Cluster node auditd logs
* Kubernetes and OpenShift API server logs
* OpenShift Virtual Network (OVN) logs

These streams are forwarded using the sementatic conventions defined by [OpenTelemetry semantic attributes](https://github.com/open-telemetry/semantic-conventions/tree/main/docs). The semantic conventions in OpenTelemetry define a *Resource* as an immutable representation of the entity producing telemetry as *Attributes*. For example, a process producing telemetry that is running in a container has a container_name, a cluster_id, a pod_name, a namespace, and possibly a deployment or app_name. All of these *Attributes* are included in the *Resource* object.  This grouping and reducing of common attributes is a powerful tool when sending logs as telemetry data.

The following sections define the attributes that are generally forwarded.

### Log Entry Structure

All log streams include the following [log data](https://opentelemetry.io/docs/specs/otel/logs/data-model/#log-and-event-record-definition) fields

The "Applicable Sources" column shows which log sources this field applies to:

* `all` is a field that is present on all logs
* `container` is a field that is present on Kubernetes Container logs (both application and infrastructure)
* `audit` is a field that is present on Kubernetes and OpenShift API
* `auditd` is a field that is present on Node auditd logs
* `journal` is a field that is present on Node journal logs
* `ovn log` is field that is present on OVN Logs

| Name |  Applicable Sources | Comment |
| :--- | :------------------ | :------ |
| `body` | all | |
| `observedTimeUnixNano` | all | |
| `timeUnixNano` | all | |
| `severityText` | container, journal | |
| `attributes` | all | Optional.  Present when forwarding stream specific attributes|

### Attributes

Log entries will have a set of resource, scope and log attributes depending on their source described by the following table.

The "Location" column can be one of the following:

* `resource` for a resource attribute
* `scope` for a scope attribute
* `log` for a log attribute

The "Storage" column shows whether the attribute is stored into a LokiStack using the default `openshift-logging` tenancy mode and where the attribute is stored:

* `stream label` (with an optional "required", if the Loki Operator will enforce this attribute in the configuration)
* `structured metadata`

| Name | Location | Applicable Sources | Storage (LokiStack) | Comment |
| :--- | :------- | :----------------- | :------------------ | :------ |
| `log_source` | resource | all | required stream label | **(DEPRECATED)** Compatibility attribute, contains same information as `openshift.log.source` |
| `log_type` | resource | all | required stream label | **(DEPRECATED)** Compatibility attribute, contains same information as `openshift.log.type` |
| `kubernetes.container_name` | resource | container | stream label | **(DEPRECATED)** Compatibility attribute, contains same information as `k8s.container.name` |
| `kubernetes.host` | resource | all | stream label | **(DEPRECATED)** Compatibility attribute, same information as `k8s.node.name` |
| `kubernetes.namespace_name` | resource | container | required stream label | **(DEPRECATED)** Compatibility attribute, contains same information as `k8s.namespace.name` |
| `kubernetes.pod_name` | resource | container | stream label | **(DEPRECATED)** Compatibility attribute, contains same information as `k8s.pod.name` |
| `openshift.cluster_id` | resource | all | | **(DEPRECATED)** Compatibility attribute, contains same information as `openshift.cluster.uid` |
| `level` | log | container, journal | | **(DEPRECATED)** Compatibility attribute, contains same information as `severityText` |
| `openshift.cluster.uid` | resource | all | required stream label | |
| `openshift.log.source` | resource | all | required stream label | |
| `openshift.log.type` | resource | all | required stream label | |
| `openshift.labels.*` | resource | all | structured metadata | |
| `k8s.node.name` | resource | all | stream label | |
| `k8s.namespace.name` | resource | container | required stream label | |
| `k8s.container.name` | resource | container | stream label | |
| `k8s.pod.labels.*` | resource | container | structured metadata | |
| `k8s.pod.name` | resource | container | stream label | |
| `k8s.pod.uid` | resource | container | structured metadata | |
| `k8s.cronjob.name` | resource | container | stream label | Conditionally forwarded based on creator of Pod |
| `k8s.daemonset.name` | resource | container | stream label | Conditionally forwarded based on creator of Pod |
| `k8s.deployment.name` | resource | container | stream label | Conditionally forwarded based on creator of Pod |
| `k8s.job.name` | resource | container | stream label | Conditionally forwarded based on creator of Pod |
| `k8s.replicaset.name` | resource | container | structured metadata | Conditionally forwarded based on creator of Pod |
| `k8s.statefulset.name` | resource | container | stream label | Conditionally forwarded based on creator of Pod |
| `log.iostream` | log | container | structured metadata | |
| `k8s.audit.event.level` | log | audit | structured metadata | |
| `k8s.audit.event.stage` | log | audit | structured metadata | |
| `k8s.audit.event.user_agent` | log | audit | structured metadata | |
| `k8s.audit.event.request.uri` | log | audit | structured metadata | |
| `k8s.audit.event.response.code` | log | audit | structured metadata | |
| `k8s.audit.event.annotation.*` | log | audit | structured metadata | |
| `k8s.audit.event.object_ref.resource` | log | audit | structured metadata | |
| `k8s.audit.event.object_ref.name` | log | audit | structured metadata | |
| `k8s.audit.event.object_ref.namespace` | log | audit | structured metadata | |
| `k8s.audit.event.object_ref.api_group` | log | audit | structured metadata | |
| `k8s.audit.event.object_ref.api_version` | log | audit | structured metadata | |
| `k8s.user.username` | log | audit | structured metadata | |
| `k8s.user.groups` | log | audit | structured metadata | |
| `process.executable.name` | resource | journal | structured metadata | |
| `process.executable.path` | resource | journal | structured metadata | |
| `process.command_line` | resource | journal | structured metadata | |
| `process.pid` | resource | journal | structured metadata | |
| `service.name` | resource | journal | stream label | |
| `systemd.t.*` | log | journal | structured metadata | |
| `systemd.u.*` | log | journal | structured metadata | |
| `k8s.ovn.component` | resource | OVN logs | stream label | The OVN component generating the log|
| `k8s.ovn.sequence` | log | OVN logs | structured metadata | The sequence number indicating the log entry's order|
| `k8s.ovn.message` | log | OVN logs | structured metadata | The main content of the log entry, can containing key-value pairs|
| `k8s.ovn.message.*` | log | OVN logs | structured metadata | Individual key-value pairs parsed from the `ovn.message` field|
| `k8s.auditd.event.type` | log | auditd logs | structured metadata | The type of auditd event (e.g., `SYSCALL`, `PROCTITLE`) |
| `k8s.auditd.event.*`| log | auditd logs | structured metadata | The audit event fields |



**Note:** Attributes marked as "Compatibility attribute" are added to support minimal backwards compatibility with the [ViaQ](https://github.com/openshift/cluster-logging-operator/blob/release-6.0/docs/reference/datamodels/viaq/v1.adoc) data model. These attributes should be considered deprecated and will be removed one release after **General Acceptance** of Red Hat OpenShift Logging.

**Note:** Loki changes the attribute names when persisting them to storage. They will be lower-cased and all characters in the set: (`.`,`/`,`-`) will be replaced by underscores (`_`). For example, `k8s.namespace.name` will become `k8s_namespace_name`.

**Note:** 

[Audit Record Types](https://access.redhat.com/articles/4409591#audit-record-types-2) 

[Audit Event Fields](https://access.redhat.com/articles/4409591#audit-event-fields-1)

## References

* [Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/)
* [Logs Data Model](https://opentelemetry.io/docs/specs/otel/logs/data-model/)
* [General Logs Attributes](https://opentelemetry.io/docs/specs/semconv/general/logs/)
* [Cluster Logging OTEL Support](https://github.com/openshift/enhancements/pull/1684)
