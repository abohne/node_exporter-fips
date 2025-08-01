# Node exporter

[![CircleCI](https://circleci.com/gh/prometheus/node_exporter/tree/master.svg?style=shield)][circleci]
![bsd workflow](https://github.com/prometheus/node_exporter/actions/workflows/bsd.yml/badge.svg)
![golangci-lint workflow](https://github.com/prometheus/node_exporter/actions/workflows/golangci-lint.yml/badge.svg)
[![Docker Repository on Quay](https://quay.io/repository/prometheus/node-exporter/status)][quay]
[![Docker Pulls](https://img.shields.io/docker/pulls/prom/node-exporter.svg?maxAge=604800)][hub]
[![Go Report Card](https://goreportcard.com/badge/github.com/prometheus/node_exporter)][goreportcard]

Prometheus exporter for hardware and OS metrics exposed by \*NIX kernels, written
in Go with pluggable metric collectors.

The [Windows exporter](https://github.com/prometheus-community/windows_exporter) is recommended for Windows users.
To expose NVIDIA GPU metrics, [prometheus-dcgm
](https://github.com/NVIDIA/dcgm-exporter)
can be used.

## Installation and Usage

If you are new to Prometheus and `node_exporter` there is a [simple step-by-step guide](https://prometheus.io/docs/guides/node-exporter/).

The `node_exporter` listens on HTTP port 9100 by default. See the `--help` output for more options.

### Ansible

For automated installs with [Ansible](https://www.ansible.com/), there is the [Prometheus Community role](https://github.com/prometheus-community/ansible).

### Docker

The `node_exporter` is designed to monitor the host system. Deploying in containers requires
extra care in order to avoid monitoring the container itself.

For situations where containerized deployment is needed, some extra flags must be used to allow
the `node_exporter` access to the host namespaces.

Be aware that any non-root mount points you want to monitor will need to be bind-mounted
into the container.

If you start container for host monitoring, specify `path.rootfs` argument.
This argument must match path in bind-mount of host root. The node\_exporter will use
`path.rootfs` as prefix to access host filesystem.

```bash
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

For Docker compose, similar flag changes are needed.

```yaml
---
version: '3.8'

services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
```

On some systems, the `timex` collector requires an additional Docker flag,
`--cap-add=SYS_TIME`, in order to access the required syscalls.

## Collectors

There is varying support for collectors on each operating system. The tables
below list all existing collectors and the supported systems.

Collectors are enabled by providing a `--collector.<name>` flag.
Collectors that are enabled by default can be disabled by providing a `--no-collector.<name>` flag.
To enable only some specific collector(s), use `--collector.disable-defaults --collector.<name> ...`.

### Include & Exclude flags

A few collectors can be configured to include or exclude certain patterns using dedicated flags. The exclude flags are used to indicate "all except", while the include flags are used to say "none except". Note that these flags are mutually exclusive on collectors that support both.

Example:

```txt
--collector.filesystem.mount-points-exclude=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/.+)($|/)
```

List:

Collector | Scope | Include Flag | Exclude Flag
--- | --- | --- | ---
arp | device | --collector.arp.device-include | --collector.arp.device-exclude
cpu | bugs | --collector.cpu.info.bugs-include | N/A
cpu | flags | --collector.cpu.info.flags-include | N/A
diskstats | device | --collector.diskstats.device-include | --collector.diskstats.device-exclude
ethtool | device | --collector.ethtool.device-include | --collector.ethtool.device-exclude
ethtool | metrics | --collector.ethtool.metrics-include | N/A
filesystem | fs-types | --collector.filesystem.fs-types-include | --collector.filesystem.fs-types-exclude
filesystem | mount-points | --collector.filesystem.mount-points-include | --collector.filesystem.mount-points-exclude
hwmon | chip | --collector.hwmon.chip-include | --collector.hwmon.chip-exclude
hwmon | sensor | --collector.hwmon.sensor-include | --collector.hwmon.sensor-exclude
interrupts | name | --collector.interrupts.name-include | --collector.interrupts.name-exclude
netdev | device | --collector.netdev.device-include | --collector.netdev.device-exclude
qdisk | device | --collector.qdisk.device-include | --collector.qdisk.device-exclude
slabinfo | slab-names | --collector.slabinfo.slabs-include | --collector.slabinfo.slabs-exclude
sysctl | all | --collector.sysctl.include | N/A
systemd | unit | --collector.systemd.unit-include | --collector.systemd.unit-exclude

### Enabled by default

Name     | Description | OS
---------|-------------|----
arp | Exposes ARP statistics from `/proc/net/arp`. | Linux
bcache | Exposes bcache statistics from `/sys/fs/bcache/`. | Linux
bonding | Exposes the number of configured and active slaves of Linux bonding interfaces. | Linux
btrfs | Exposes btrfs statistics | Linux
boottime | Exposes system boot time derived from the `kern.boottime` sysctl. | Darwin, Dragonfly, FreeBSD, NetBSD, OpenBSD, Solaris
conntrack | Shows conntrack statistics (does nothing if no `/proc/sys/net/netfilter/` present). | Linux
cpu | Exposes CPU statistics | Darwin, Dragonfly, FreeBSD, Linux, Solaris, OpenBSD
cpufreq | Exposes CPU frequency statistics | Linux, Solaris
diskstats | Exposes disk I/O statistics. | Darwin, Linux, OpenBSD
dmi | Expose Desktop Management Interface (DMI) info from `/sys/class/dmi/id/` | Linux
edac | Exposes error detection and correction statistics. | Linux
entropy | Exposes available entropy. | Linux
exec | Exposes execution statistics. | Dragonfly, FreeBSD
fibrechannel | Exposes fibre channel information and statistics from `/sys/class/fc_host/`. | Linux
filefd | Exposes file descriptor statistics from `/proc/sys/fs/file-nr`. | Linux
filesystem | Exposes filesystem statistics, such as disk space used. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD
hwmon | Expose hardware monitoring and sensor data from `/sys/class/hwmon/`. | Linux
infiniband | Exposes network statistics specific to InfiniBand and Intel OmniPath configurations. | Linux
ipvs | Exposes IPVS status from `/proc/net/ip_vs` and stats from `/proc/net/ip_vs_stats`. | Linux
loadavg | Exposes load average. | Darwin, Dragonfly, FreeBSD, Linux, NetBSD, OpenBSD, Solaris
mdadm | Exposes statistics about devices in `/proc/mdstat` (does nothing if no `/proc/mdstat` present). | Linux
meminfo | Exposes memory statistics. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD
netclass | Exposes network interface info from `/sys/class/net/` | Linux
netdev | Exposes network interface statistics such as bytes transferred. | Darwin, Dragonfly, FreeBSD, Linux, OpenBSD
netisr | Exposes netisr statistics | FreeBSD
netstat | Exposes network statistics from `/proc/net/netstat`. This is the same information as `netstat -s`. | Linux
nfs | Exposes NFS client statistics from `/proc/net/rpc/nfs`. This is the same information as `nfsstat -c`. | Linux
nfsd | Exposes NFS kernel server statistics from `/proc/net/rpc/nfsd`. This is the same information as `nfsstat -s`. | Linux
nvme | Exposes NVMe info from `/sys/class/nvme/` | Linux
os | Expose OS release info from `/etc/os-release` or `/usr/lib/os-release` | _any_
powersupplyclass | Exposes Power Supply statistics from `/sys/class/power_supply` | Linux
pressure | Exposes pressure stall statistics from `/proc/pressure/`. | Linux (kernel 4.20+ and/or [CONFIG\_PSI](https://www.kernel.org/doc/html/latest/accounting/psi.html))
rapl | Exposes various statistics from `/sys/class/powercap`. | Linux
schedstat | Exposes task scheduler statistics from `/proc/schedstat`. | Linux
selinux | Exposes SELinux statistics. | Linux
sockstat | Exposes various statistics from `/proc/net/sockstat`. | Linux
softnet | Exposes statistics from `/proc/net/softnet_stat`. | Linux
stat | Exposes various statistics from `/proc/stat`. This includes boot time, forks and interrupts. | Linux
tapestats | Exposes statistics from `/sys/class/scsi_tape`. | Linux
textfile | Exposes statistics read from local disk. The `--collector.textfile.directory` flag must be set. | _any_
thermal | Exposes thermal statistics like `pmset -g therm`. | Darwin
thermal\_zone | Exposes thermal zone & cooling device statistics from `/sys/class/thermal`. | Linux
time | Exposes the current system time. | _any_
timex | Exposes selected adjtimex(2) system call stats. | Linux
udp_queues | Exposes UDP total lengths of the rx_queue and tx_queue from `/proc/net/udp` and `/proc/net/udp6`. | Linux
uname | Exposes system information as provided by the uname system call. | Darwin, FreeBSD, Linux, OpenBSD
vmstat | Exposes statistics from `/proc/vmstat`. | Linux
watchdog | Exposes statistics from `/sys/class/watchdog` | Linux
xfs | Exposes XFS runtime statistics. | Linux (kernel 4.4+)
zfs | Exposes [ZFS](http://open-zfs.org/) performance statistics. | FreeBSD, [Linux](http://zfsonlinux.org/), Solaris

### Disabled by default

`node_exporter` also implements a number of collectors that are disabled by default.  Reasons for this vary by
collector, and may include:
* High cardinality
* Prolonged runtime that exceeds the Prometheus `scrape_interval` or `scrape_timeout`
* Significant resource demands on the host

You can enable additional collectors as desired by adding them to your
init system's or service supervisor's startup configuration for
`node_exporter` but caution is advised.  Enable at most one at a time,
testing first on a non-production system, then by hand on a single
production node.  When enabling additional collectors, you should
carefully monitor the change by observing the `
scrape_duration_seconds` metric to ensure that collection completes
and does not time out.  In addition, monitor the
`scrape_samples_post_metric_relabeling` metric to see the changes in
cardinality.

Name     | Description | OS
---------|-------------|----
buddyinfo | Exposes statistics of memory fragments as reported by /proc/buddyinfo. | Linux
cgroups | A summary of the number of active and enabled cgroups | Linux
cpu\_vulnerabilities | Exposes CPU vulnerability information from sysfs. | Linux
devstat | Exposes device statistics | Dragonfly, FreeBSD
drm | Expose GPU metrics using sysfs / DRM, `amdgpu` is the only driver which exposes this information through DRM | Linux
drbd | Exposes Distributed Replicated Block Device statistics (to version 8.4) | Linux
ethtool | Exposes network interface information and network driver statistics equivalent to `ethtool`, `ethtool -S`, and `ethtool -i`. | Linux
interrupts | Exposes detailed interrupts statistics. | Linux, OpenBSD
ksmd | Exposes kernel and system statistics from `/sys/kernel/mm/ksm`. | Linux
lnstat | Exposes stats from `/proc/net/stat/`. | Linux
logind | Exposes session counts from [logind](http://www.freedesktop.org/wiki/Software/systemd/logind/). | Linux
meminfo\_numa | Exposes memory statistics from `/sys/devices/system/node/node[0-9]*/meminfo`, `/sys/devices/system/node/node[0-9]*/numastat`. | Linux
mountstats | Exposes filesystem statistics from `/proc/self/mountstats`. Exposes detailed NFS client statistics. | Linux
network_route | Exposes the routing table as metrics | Linux
pcidevice | Exposes pci devices' information including their link status and parent devices. | Linux
perf | Exposes perf based metrics (Warning: Metrics are dependent on kernel configuration and settings). | Linux
processes | Exposes aggregate process statistics from `/proc`. | Linux
qdisc | Exposes [queuing discipline](https://en.wikipedia.org/wiki/Network_scheduler#Linux_kernel) statistics | Linux
slabinfo | Exposes slab statistics from `/proc/slabinfo`. Note that permission of `/proc/slabinfo` is usually 0400, so set it appropriately. | Linux
softirqs | Exposes detailed softirq statistics from `/proc/softirqs`. | Linux
sysctl | Expose sysctl values from `/proc/sys`. Use `--collector.sysctl.include(-info)` to configure. | Linux
systemd | Exposes service and system status from [systemd](http://www.freedesktop.org/wiki/Software/systemd/). | Linux
tcpstat | Exposes TCP connection status information from `/proc/net/tcp` and `/proc/net/tcp6`. (Warning: the current version has potential performance issues in high load situations.) | Linux
wifi | Exposes WiFi device and station statistics. | Linux
xfrm | Exposes statistics from `/proc/net/xfrm_stat` | Linux
zoneinfo | Exposes NUMA memory zone metrics. | Linux

### Deprecated

These collectors are deprecated and will be removed in the next major release.

Name     | Description | OS
---------|-------------|----
ntp | Exposes local NTP daemon health to check [time](./docs/TIME.md) | _any_
runit | Exposes service status from [runit](http://smarden.org/runit/). | _any_
supervisord | Exposes service status from [supervisord](http://supervisord.org/). | _any_

### Perf Collector

The `perf` collector may not work out of the box on some Linux systems due to kernel
configuration and security settings. To allow access, set the following `sysctl`
parameter:

```
sysctl -w kernel.perf_event_paranoid=X
```

- 2 allow only user-space measurements (default since Linux 4.6).
- 1 allow both kernel and user measurements (default before Linux 4.6).
- 0 allow access to CPU-specific data but not raw tracepoint samples.
- -1 no restrictions.

Depending on the configured value different metrics will be available, for most
cases `0` will provide the most complete set. For more information see [`man 2
perf_event_open`](http://man7.org/linux/man-pages/man2/perf_event_open.2.html).

By default, the `perf` collector will only collect metrics of the CPUs that
`node_exporter` is running on (ie
[`runtime.NumCPU`](https://golang.org/pkg/runtime/#NumCPU). If this is
insufficient (e.g. if you run `node_exporter` with its CPU affinity set to
specific CPUs), you can specify a list of alternate CPUs by using the
`--collector.perf.cpus` flag. For example, to collect metrics on CPUs 2-6, you
would specify: `--collector.perf --collector.perf.cpus=2-6`. The CPU
configuration is zero indexed and can also take a stride value; e.g.
`--collector.perf --collector.perf.cpus=1-10:5` would collect on CPUs
1, 5, and 10.

The `perf` collector is also able to collect
[tracepoint](https://www.kernel.org/doc/html/latest/core-api/tracepoint.html)
counts when using the `--collector.perf.tracepoint` flag. Tracepoints can be
found using [`perf list`](http://man7.org/linux/man-pages/man1/perf.1.html) or
from debugfs. And example usage of this would be
`--collector.perf.tracepoint="sched:sched_process_exec"`.

### Sysctl Collector

The `sysctl` collector can be enabled with `--collector.sysctl`. It supports exposing numeric sysctl values
as metrics using the `--collector.sysctl.include` flag and string values as info metrics by using the
`--collector.sysctl.include-info` flag. The flags can be repeated. For sysctl with multiple numeric values,
an optional mapping can be given to expose each value as its own metric. Otherwise an `index` label is used
to identify the different fields.

#### Examples
##### Numeric values
###### Single values
Using `--collector.sysctl.include=vm.user_reserve_kbytes`:
`vm.user_reserve_kbytes = 131072` -> `node_sysctl_vm_user_reserve_kbytes 131072`

###### Multiple values
A sysctl can contain multiple values, for example:
```
net.ipv4.tcp_rmem = 4096	131072	6291456
```
Using `--collector.sysctl.include=net.ipv4.tcp_rmem` the collector will expose:
```
node_sysctl_net_ipv4_tcp_rmem{index="0"} 4096
node_sysctl_net_ipv4_tcp_rmem{index="1"} 131072
node_sysctl_net_ipv4_tcp_rmem{index="2"} 6291456
```
If the indexes have defined meaning like in this case, the values can be mapped to multiple metrics by appending the mapping to the --collector.sysctl.include flag:
Using `--collector.sysctl.include=net.ipv4.tcp_rmem:min,default,max` the collector will expose:
```
node_sysctl_net_ipv4_tcp_rmem_min 4096
node_sysctl_net_ipv4_tcp_rmem_default 131072
node_sysctl_net_ipv4_tcp_rmem_max 6291456
```

##### String values
String values need to be exposed as info metric. The user selects them by using the `--collector.sysctl.include-info` flag.

###### Single values
`kernel.core_pattern = core` -> `node_sysctl_info{key="kernel.core_pattern_info", value="core"} 1`

###### Multiple values
Given the following sysctl:
```
kernel.seccomp.actions_avail = kill_process kill_thread trap errno trace log allow
```
Setting `--collector.sysctl.include-info=kernel.seccomp.actions_avail` will yield:
```
node_sysctl_info{key="kernel.seccomp.actions_avail", index="0", value="kill_process"} 1
node_sysctl_info{key="kernel.seccomp.actions_avail", index="1", value="kill_thread"} 1
...
```

### Textfile Collector

The `textfile` collector is similar to the [Pushgateway](https://github.com/prometheus/pushgateway),
in that it allows exporting of statistics from batch jobs. It can also be used
to export static metrics, such as what role a machine has. The Pushgateway
should be used for service-level metrics. The `textfile` module is for metrics
that are tied to a machine.

To use it, set the `--collector.textfile.directory` flag on the `node_exporter` commandline. The
collector will parse all files in that directory matching the glob `*.prom`
using the [text
format](http://prometheus.io/docs/instrumenting/exposition_formats/). **Note:** Timestamps are not supported.

To atomically push completion time for a cron job:
```
echo my_batch_job_completion_time $(date +%s) > /path/to/directory/my_batch_job.prom.$$
mv /path/to/directory/my_batch_job.prom.$$ /path/to/directory/my_batch_job.prom
```

To statically set roles for a machine using labels:
```
echo 'role{role="application_server"} 1' > /path/to/directory/role.prom.$$
mv /path/to/directory/role.prom.$$ /path/to/directory/role.prom
```

### Filtering enabled collectors

The `node_exporter` will expose all metrics from enabled collectors by default.  This is the recommended way to collect metrics to avoid errors when comparing metrics of different families.

For advanced use the `node_exporter` can be passed an optional list of collectors to filter metrics. The parameters `collect[]` and `exclude[]` can be used multiple times (but cannot be combined).  In Prometheus configuration you can use this syntax under the [scrape config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#<scrape_config>).

Collect only `cpu` and `meminfo` collector metrics:
```
  params:
    collect[]:
      - cpu
      - meminfo
```

Collect all enabled collector metrics but exclude `netdev`:
```
  params:
    exclude[]:
      - netdev
```

This can be useful for having different Prometheus servers collect specific metrics from nodes.

## Development building and running

Prerequisites:

* [Go compiler](https://golang.org/dl/)
* RHEL/CentOS: `glibc-static` package.

Building:

    git clone https://github.com/prometheus/node_exporter.git
    cd node_exporter
    make build
    ./node_exporter <flags>

To see all available configuration flags:

    ./node_exporter -h

## Running tests

    make test

## TLS endpoint

**EXPERIMENTAL**

The exporter supports TLS via a new web configuration file.

```console
./node_exporter --web.config.file=web-config.yml
```

See the [exporter-toolkit web-configuration](https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md) for more details.

[travis]: https://travis-ci.org/prometheus/node_exporter
[hub]: https://hub.docker.com/r/prom/node-exporter/
[circleci]: https://circleci.com/gh/prometheus/node_exporter
[quay]: https://quay.io/repository/prometheus/node-exporter
[goreportcard]: https://goreportcard.com/report/github.com/prometheus/node_exporter
