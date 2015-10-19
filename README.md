# Telegraf - A native agent for InfluxDB [![Circle CI](https://circleci.com/gh/influxdb/telegraf.svg?style=svg)](https://circleci.com/gh/influxdb/telegraf)

Telegraf is an agent written in Go for collecting metrics from the system it's
running on, or from other services, and writing them into InfluxDB.

Design goals are to have a minimal memory footprint with a plugin system so
that developers in the community can easily add support for collecting metrics
from well known services (like Hadoop, Postgres, or Redis) and third party
APIs (like Mailchimp, AWS CloudWatch, or Google Analytics).

We'll eagerly accept pull requests for new plugins and will manage the set of
plugins that Telegraf supports. See the
[contributing guide](CONTRIBUTING.md) for instructions on
writing new plugins.

## Installation:

Due to a breaking change to the InfluxDB integer line-protocol, there
are some InfluxDB compatibility requirements:

* InfluxDB 0.9.3+ requires Telegraf 0.1.5+
* InfluxDB 0.9.2 and prior requires Telegraf 0.1.4

NOTE: Telegraf 0.1.9 will change the name of some CPU usage metrics, see
[CHANGELOG](CHANGELOG.md) for more details.

### Linux deb and rpm packages:

Latest:
* http://get.influxdb.org/telegraf/telegraf_0.1.9_amd64.deb
* http://get.influxdb.org/telegraf/telegraf-0.1.9-1.x86_64.rpm

0.1.4:
* http://get.influxdb.org/telegraf/telegraf_0.1.4_amd64.deb
* http://get.influxdb.org/telegraf/telegraf-0.1.4-1.x86_64.rpm

##### Package instructions:

* Telegraf binary is installed in `/opt/telegraf/telegraf`
* Telegraf daemon configuration file is in `/etc/opt/telegraf/telegraf.conf`
* On sysv systems, the telegraf daemon can be controlled via
`service telegraf [action]`
* On systemd systems (such as Ubuntu 15+), the telegraf daemon can be
controlled via `systemctl [action] telegraf`

### Linux binaries:

Latest:
* http://get.influxdb.org/telegraf/telegraf_linux_amd64_0.1.9.tar.gz
* http://get.influxdb.org/telegraf/telegraf_linux_386_0.1.9.tar.gz
* http://get.influxdb.org/telegraf/telegraf_linux_arm_0.1.9.tar.gz

##### Binary instructions:

These are standalone binaries that can be unpacked and executed on any linux
system. They can be unpacked and renamed in a location such as
`/usr/local/bin` for convenience. A config file will need to be generated,
see "How to use it" below.

### OSX via Homebrew:

```
brew update
brew install telegraf
```

### From Source:

Telegraf manages dependencies via `godep`, which gets installed via the Makefile
if you don't have it already. You also must build with golang version 1.4+

1. [Install Go](https://golang.org/doc/install)
2. [Setup your GOPATH](https://golang.org/doc/code.html#GOPATH)
3. run `go get github.com/influxdb/telegraf`
4. `cd $GOPATH/src/github.com/influxdb/telegraf`
5. run `make`

### How to use it:

* Run `telegraf -sample-config > telegraf.conf` to create an initial configuration
* Or run `telegraf -sample-config -filter cpu:mem -outputfilter influxdb > telegraf.conf`
to create a config file with only CPU and memory plugins defined, and InfluxDB output defined
* Edit the configuration to match your needs
* Run `telegraf -config telegraf.conf -test` to output one full measurement sample to STDOUT
* Run `telegraf -config telegraf.conf` to gather and send metrics to configured outputs.
* Run `telegraf -config telegraf.conf -filter system:swap`
to run telegraf with only the system & swap plugins defined in the config.

## Telegraf Options

Telegraf has a few options you can configure under the `agent` section of the
config.

* **hostname**: The hostname is passed as a tag. By default this will be
the value returned by `hostname` on the machine running Telegraf.
You can override that value here.
* **interval**: How often to gather metrics. Uses a simple number +
unit parser, e.g. "10s" for 10 seconds or "5m" for 5 minutes.
* **debug**: Set to true to gather and send metrics to STDOUT as well as
InfluxDB.

## Plugin Options

There are 5 configuration options that are configurable per plugin:

* **pass**: An array of strings that is used to filter metrics generated by the
current plugin. Each string in the array is tested as a prefix against metric names
and if it matches, the metric is emitted.
* **drop**: The inverse of pass, if a metric name matches, it is not emitted.
* **tagpass**: (added in 0.1.5) tag names and arrays of strings that are used to filter metrics by
the current plugin. Each string in the array is tested as an exact match against
the tag name, and if it matches the metric is emitted.
* **tagdrop**: (added in 0.1.5) The inverse of tagpass. If a tag matches, the metric is not emitted.
This is tested on metrics that have passed the tagpass test.
* **interval**: How often to gather this metric. Normal plugins use a single
global interval, but if one particular plugin should be run less or more often,
you can configure that here.

### Plugin Configuration Examples

This is a full working config that will output CPU data to an InfluxDB instance
at 192.168.59.103:8086, tagging measurements with dc="denver-1". It will output
measurements at a 10s interval and will collect totalcpu & percpu data.

```
[tags]
    dc = "denver-1"

[agent]
    interval = "10s"

# OUTPUTS
[outputs]
[outputs.influxdb]
    url = "http://192.168.59.103:8086" # required.
    database = "telegraf" # required.
    precision = "s"

# PLUGINS
[cpu]
    percpu = true
    totalcpu = true
```

Below is how to configure `tagpass` and `tagdrop` parameters (added in 0.1.5)

```
# Don't collect CPU data for cpu6 & cpu7
[cpu.tagdrop]
    cpu = [ "cpu6", "cpu7" ]

[disk]
[disk.tagpass]
    # tagpass conditions are OR, not AND.
    # If the (filesystem is ext4 or xfs) OR (the path is /opt or /home)
    # then the metric passes
    fstype = [ "ext4", "xfs" ]
    path = [ "/opt", "/home" ]
```

## Supported Plugins

**You can view usage instructions for each plugin by running**
`telegraf -usage <pluginname>`

Telegraf currently has support for collecting metrics from

* apache
* bcache
* disque
* elasticsearch
* exec (generic JSON-emitting executable plugin)
* haproxy
* httpjson (generic JSON-emitting http service plugin)
* kafka_consumer
* leofs
* lustre2
* memcached
* mongodb
* mysql
* nginx
* phpfpm
* ping
* postgresql
* procstat
* prometheus
* puppetagent
* rabbitmq
* redis
* rethinkdb
* zookeeper
* system
    * cpu
    * mem
    * io
    * net
    * netstat
    * disk
    * swap

## Supported Service Plugins

Telegraf can collect metrics via the following services

* statsd

We'll be adding support for many more over the coming months. Read on if you
want to add support for another service or third-party API.

## Output options

Telegraf also supports specifying multiple output sinks to send data to,
configuring each output sink is different, but examples can be
found by running `telegraf -sample-config`

## Supported Outputs

* influxdb
* kafka
* datadog
* opentsdb
* amqp (rabbitmq)
* mqtt

## Contributing

Please see the
[contributing guide](CONTRIBUTING.md)
for details on contributing a plugin or output to Telegraf
