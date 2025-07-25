# Host Metrics Receiver

<!-- status autogenerated section -->
| Status        |           |
| ------------- |-----------|
| Stability     | [development]: logs   |
|               | [beta]: metrics   |
| Distributions | [core], [contrib], [k8s] |
| Issues        | [![Open issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aopen%20label%3Areceiver%2Fhostmetrics%20&label=open&color=orange&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aopen+is%3Aissue+label%3Areceiver%2Fhostmetrics) [![Closed issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aclosed%20label%3Areceiver%2Fhostmetrics%20&label=closed&color=blue&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aclosed+is%3Aissue+label%3Areceiver%2Fhostmetrics) |
| Code coverage | [![codecov](https://codecov.io/github/open-telemetry/opentelemetry-collector-contrib/graph/main/badge.svg?component=receiver_hostmetrics)](https://app.codecov.io/gh/open-telemetry/opentelemetry-collector-contrib/tree/main/?components%5B0%5D=receiver_hostmetrics&displayType=list) |
| [Code Owners](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/CONTRIBUTING.md#becoming-a-code-owner)    | [@dmitryax](https://www.github.com/dmitryax), [@braydonk](https://www.github.com/braydonk) |

[development]: https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/component-stability.md#development
[beta]: https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/component-stability.md#beta
[core]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol
[contrib]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-contrib
[k8s]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-k8s
<!-- end autogenerated section -->

The Host Metrics receiver generates metrics about the host system scraped
from various sources and host entity event as log. This is intended to be
used when the collector is deployed as an agent.

## Getting Started

The collection interval, root path, and the categories of metrics to be scraped can be
configured:

```yaml
hostmetrics:
  collection_interval: <duration> # default = 1m
  initial_delay: <duration> # default = 1s
  root_path: <string>
  scrapers:
    <scraper1>:
    <scraper2>:
    ...
```

The available scrapers are:

| Scraper      | Supported OSs                | Description                                            |
| ------------ | ---------------------------- | ------------------------------------------------------ |
| [cpu]        | All                          | CPU utilization metrics                                |
| [disk]       | All                          | Disk I/O metrics                                       |
| [load]       | All                          | CPU load metrics                                       |
| [filesystem] | All                          | File System utilization metrics                        |
| [memory]     | All                          | Memory utilization metrics                             |
| [network]    | All                          | Network interface I/O metrics & TCP connection metrics |
| [paging]     | All                          | Paging/Swap space utilization and I/O metrics          |
| [processes]  | Linux, Mac, FreeBSD, OpenBSD | Process count metrics                                  |
| [process]    | Linux, Windows, Mac, FreeBSD | Per process CPU, Memory, and Disk I/O metrics          |
| [system]     | Linux, Windows, Mac          | Miscellaneous system metrics                           |

[cpu]: ./internal/scraper/cpuscraper/documentation.md
[disk]: ./internal/scraper/diskscraper/documentation.md
[filesystem]: ./internal/scraper/filesystemscraper/documentation.md
[load]: ./internal/scraper/loadscraper/documentation.md
[memory]: ./internal/scraper/memoryscraper/documentation.md
[network]: ./internal/scraper/networkscraper/documentation.md
[paging]: ./internal/scraper/pagingscraper/documentation.md
[processes]: ./internal/scraper/processesscraper/documentation.md
[process]: ./internal/scraper/processscraper/documentation.md
[system]: ./internal/scraper/systemscraper/documentation.md

### Notes

Several scrapers support additional configuration:

### Disk

```yaml
disk:
  <include|exclude>:
    devices: [ <device name>, ... ]
    match_type: <strict|regexp>
```

### File System

```yaml
filesystem:
  <include_devices|exclude_devices>:
    devices: [ <device name>, ... ]
    match_type: <strict|regexp>
  <include_fs_types|exclude_fs_types>:
    fs_types: [ <filesystem type>, ... ]
    match_type: <strict|regexp>
  <include_mount_points|exclude_mount_points>:
    mount_points: [ <mount point>, ... ]
    match_type: <strict|regexp>
  include_virtual_filesystems: <false|true>
```

### Load

`cpu_average` specifies whether to divide the average load by the reported number of logical CPUs (default: `false`).

```yaml
load:
  cpu_average: <false|true>
```

### Network

```yaml
network:
  <include|exclude>:
    interfaces: [ <interface name>, ... ]
    match_type: <strict|regexp>
```

### Process

```yaml
process:
  <include|exclude>:
    names: [ <process name>, ... ]
    match_type: <strict|regexp>
  mute_process_all_errors: <true|false>
  mute_process_name_error: <true|false>
  mute_process_exe_error: <true|false>
  mute_process_io_error: <true|false>
  mute_process_user_error: <true|false>
  mute_process_cgroup_error: <true|false>
  scrape_process_delay: <time>
```

The following settings are optional:
- `mute_process_all_errors` (default: false): mute all the errors encountered when trying to read metrics of a process. When this flag is enabled, there is no need to activate any other error suppression flags.
- `mute_process_name_error` (default: false): mute the error encountered when trying to read a process name the collector does not have permission to read. This flag is ignored when `mute_process_all_errors` is set to true as all errors are muted.
- `mute_process_io_error` (default: false): mute the error encountered when trying to read IO metrics of a process the collector does not have permission to read. This flag is ignored when `mute_process_all_errors` is set to true as all errors are muted.
- `mute_process_cgroup_error` (default: false): mute the error encountered when trying to read the cgroup of a process the collector does not have permission to read. This flag is ignored when `mute_process_all_errors` is set to true as all errors are muted.
- `mute_process_exe_error` (default: false): mute the error encountered when trying to read the executable path of a process the collector does not have permission to read (Linux only). This flag is ignored when `mute_process_all_errors` is set to true as all errors are muted.
- `mute_process_user_error` (default: false): mute the error encountered when trying to read a uid which doesn't exist on the system, eg. is owned by a user that only exists in a container. This flag is ignored when `mute_process_all_errors` is set to true as all errors are muted.

## Advanced Configuration

### Filtering

If you are only interested in a subset of metrics from a particular source,
it is recommended you use this receiver with the
[Filter Processor](../../processor/filterprocessor).

### Different Frequencies

If you would like to scrape some metrics at a different frequency than others,
you can configure multiple `hostmetrics` receivers with different
`collection_interval` values. For example:

```yaml
receivers:
  hostmetrics:
    collection_interval: 30s
    scrapers:
      cpu:
      memory:

  hostmetrics/disk:
    collection_interval: 1m
    scrapers:
      disk:
      filesystem:

service:
  pipelines:
    metrics:
      receivers: [hostmetrics, hostmetrics/disk]
```

### Collecting host metrics from inside a container (Linux only)

Host metrics are collected from the Linux system directories on the filesystem.
You likely want to collect metrics about the host system and not the container.
This is achievable by following these steps:

#### 1. Bind mount the host filesystem

The simplest configuration is to mount the entire host filesystem when running
the container. e.g. `docker run -v /:/hostfs ...`.

You can also choose which parts of the host filesystem to mount, if you know
exactly what you'll need. e.g. `docker run -v /proc:/hostfs/proc`.

#### 2. Configure `root_path`

Configure `root_path` so the hostmetrics receiver knows where the root filesystem is.
Note: if running multiple instances of the host metrics receiver, they must all have
the same `root_path`.

Example:
```yaml
receivers:
  hostmetrics:
    root_path: /hostfs
```

## Resource attributes

Currently, the hostmetrics receiver does not set any Resource attributes on the exported metrics. However, if you want to set Resource attributes, you can provide them via environment variables via the [resourcedetection](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/resourcedetectionprocessor#environment-variable) processor. For example, you can add the following resource attributes to adhere to [Resource Semantic Conventions](https://opentelemetry.io/docs/reference/specification/resource/semantic_conventions/):

```
export OTEL_RESOURCE_ATTRIBUTES="service.name=<the name of your service>,service.namespace=<the namespace of your service>,service.instance.id=<uuid of the instance>"
```
## Entity Events

**Entity Events as logs are experimental** and might eventually be replaced by the result of [the OTEP](https://github.com/open-telemetry/oteps/blob/main/text/entities/0256-entities-data-model.md#entity-events). For now, the hostmetrics receiver can send the host entity event as a log records. By default, the hostmetrics receiver sends periodic EntityState events every 5 minutes. You can change that by setting `metadata_collection_interval`. Entity Events as logs are experimental. The result of the OTEP might eventually replace that.
