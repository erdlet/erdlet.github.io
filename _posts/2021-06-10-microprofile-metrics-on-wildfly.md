---
layout: post
title:  "MicroProfile: Using MicroProfile Metrics in a WildFly standalone environment"
date:   2021-06-10 09:00:00 +0200
categories: jakarta-ee,wildfly,microprofile
author: Tobias Erdle
---

MicroProfile Metrics is an API that allows to get various (preconfigured) metrics via the runtime environment, as well as user-defined data via the application itself. This entry describes how to configure MicroProfile Metrics for a WildFly 23 in standalone mode. It is assumed that the standalone.xml is used. All steps show the use of the JBoss CLI as well as the corresponding XML fragment.

## 1. Install the extension

The provided standalone.xml already provides a very much simplified implementation for metrics. However, this does not comply with the MicroProfile Metrics specification and only provides base and vendor-specific data.

To enable MicroProfile Metrics, the appropriate Smallrye extension must first be added:

CLI: 

```bash
/extension=org.wildfly.extension.microprofile.metrics-smallrye:add()
```

XML: 

```xml
<extension module="org.wildfly.extension.microprofile.metrics-smallrye"/>
```

But by doing this, the extension won't work, because it isn't registered as subsystem, which will be described in the next section.

## 2. Activation of the subsystem

After the extension has been activated, it still needs to be activated as a subsystem:

CLI:
```bash
/subsystem=org.wildfly.extension.microprofile.metrics-smallrye:add()
```

XML:
```xml
<subsystem xmlns="urn:wildfly:microprofile-metrics-smallrye:2.0" exposed-subsystems="*" prefix="${wildfly.metrics.prefix:wildfly}"/>
```

Here it is important to note that authentication is required by default, as the metrics are provided via WildFly Management Port `9990`. The `security-enabled=false` attribute can be used to disable this authentication. However, this is not recommended on production systems as it can provide potential attackers with valuable data about the system. For use with authentication, a new management user must be created via the script `$WILDFLY_HOME/bin/add-user.sh`.

## 3. Use of the subsystem

After successful activation of the subsystem the metrics can be retrieved. Here are a few helpful CURL queries, assuming an secured API and a WildFly running on `localhost`:

Fetch all metrics in Prometheus compatible format:

```bash
curl --digest -u<yourUser>:<yourPassword> localhost:9990/metrics
```

Fetch vendor metrics in Prometheus compatible format:

```bash
curl --digest -u<yourUser>:<yourPassword> localhost:9990/metrics/application
```

Fetch application metrics in Prometheus compatible format:

```bash
curl --digest -u<yourUser>:<yourPassword> localhost:9990/metrics/application
```

Fetch all metrics in JSON:

```bash
curl -H "Accept: application/json" --digest -u<yourUser>:<yourPassword> localhost:9990/metrics
```

Fetch vendor metrics in JSON:

```bash
curl -H "Accept: application/json" --digest -u<yourUser>:<yourPassword> localhost:9990/metrics/application
```

Fetch application metrics in JSON:

```bash
curl -H "Accept: application/json" --digest -u<yourUser>:<yourPassword> localhost:9990/metrics/application
```

*Note:* For receiving application metrics someone must declare them in a deployed application. This won't be covered here.

## 4. Troubleshooting

Here are a few problems I stumbled over during the first time of configuring MicroProfile Metrics on WildFly.

### 4.1 /metrics/base, /metrics/vendor and /metrics/application do not filter
If all three queries lead to the same result and do not return any application metrics, the subsystem is not activated correctly. In this case, the corresponding configuration must be checked.

### 4.2 CDI Bean is no longer recognized by MetricRegistry during Inject
This can happen if the subsystem is not installed correctly. Also here one must check whether the entries for Extension and Subsystem exist.

## 5. More information

- [WildFly Admin Guide](https://docs.wildfly.org/23/Admin_Guide.html#MicroProfile_Metrics_SmallRye)
- [MicroProfile Metrics Spec](https://download.eclipse.org/microprofile/microprofile-4.0/microprofile-spec-4.0.html)