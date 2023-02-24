# enpal-homeassistant-resources

## Introduction
This is a meta repository that collects instructions, hints, default configuration to setup the
default Enpal solar inverters to home assistant. Furthermore guidelines to setup the forwarding
of the data to an InfluxDB and Grafana will be shown.

The main goal is to setup you own (local) maintainace system so that you do not need the buggy
Enpal App.

## Preconditions
1. An [InfluxDB](https://www.influxdata.com/) instance running in version 2+ with administration rights
2. A [Grafana](https://grafana.com/) instance running in version 9+ with administration rights
3. A mini PC in range of the Huawei SUN2000 inverter that has two network adapters

## Home Assistant setup
1. Conenct the mini PC WiFi to that of the inverter
2. Install [home assistant](https://www.home-assistant.io/)
3. Install [HACS](https://github.com/hacs/integration)
4. Install the [custom integration](https://github.com/wlcrs/huawei_solar) for inverter

## InfluxDB intrgration
Make sure that the mini PC can reach the InfluxDB instance's `<hostname` with the second
network adapter.

Create a new organization (or use an existing one) in your InfluxDB instance. Then create
a bucket (or use an existing one) for your home assistant data. Write down you `<bucket-name>`.

You need the `<organization-id>` for the home assistant configuration yaml file. You can see
it in the about page of the organization.

As a last step, you need an api token. That can be created inside the _load-data/tokens_
screen of the organization. The token needs write access to your home assistant bucket.
Use that `<api-token>` in the following home assistant configuration file.

Use the following configuration to setup the integrated InfluxDB integration.
~~~yaml
influxdb:
  api_version: 2
  ssl: true
  host: <hostname>
  port: 443
  token: <api-key>
  organization: <organization-id>
  bucket: <bucket-name>
  tags:
    source: HA
  tags_attributes:
    - friendly_name
  default_measurement: units
  exclude:
    entities:
      - zone.home
    domains:
      - persistent_notification
      - person
  include:
    domains:
      - sensor
      - binary_sensor
~~~

## Grafana integration
As a last step, we need to setup the Grafana instance. Make sure, your Grafana instance
can reach your InfluxDB `<hostname>`.

For Grafana, you need to create another api token inside your InfluxDB instance. That new
token needs read access to your home assistant bucket.

When you have your new read token, you can create a new datasource inside Grafana. Use the
propper InfluxDB hostname, organization name (not id!), bucket name and token. Also make
sure you are using `flux` as query language.

![datasource](https://github.com/mustafauegly/enpal-homeassistant-resources/raw/main/grafana-datasource-configuration.png).

## See the data
If everything works, you can just use the
[example dashboard](https://github.com/mustafauegly/enpal-homeassistant-resources/blob/main/grafana-dashboard.json)
and load it into your Grafana instance.

