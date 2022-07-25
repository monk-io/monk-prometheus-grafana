Prometheus & Grafana meets Monk
===

This repository contains Monk.io template to deploy Grafana & Prometheus system either locally or on cloud of your choice (AWS, GCP, Azure, Digital Ocean).

It includes simple Grafana dashboards and few basic variables that will allow you to quickly spin up instance and add custom dashboards.

- [Prometheus \& Grafana meets Monk](#prometheus--grafana-meets-monk)
  - [Prerequisites](#prerequisites)
    - [Make sure monkd is running.](#make-sure-monkd-is-running)
    - [Clone Repository](#clone-repository)
    - [Load Template](#load-template)
    - [Verify if it's loaded correctly](#verify-if-its-loaded-correctly)
  - [Deploy Stack](#deploy-stack)
  - [Variables](#variables)
    - [Grafana](#grafana)
    - [Prometheus](#prometheus)
  - [Stop, remove and clean up workloads and templates](#stop-remove-and-clean-up-workloads-and-templates)
  - [Create your own template](#create-your-own-template)

## Prerequisites
- [Install Monk](https://docs.monk.io/docs/get-monk)
- [Register and Login Monk](https://docs.monk.io/docs/acc-and-auth)
- [Add Cloud Provider](https://docs.monk.io/docs/cloud-provider)
- [Add Instance](https://docs.monk.io/docs/multi-cloud)

### Make sure monkd is running.

```bash
foo@bar:~$ monk status
daemon: ready
auth: logged in
not connected to cluster
```

### Clone Repository

```bash
git clone git@github.com:CuteAnonymousPanda/monk-prometheus-grafana.git
```

### Load Template

```bash
cd monk-prometheus-grafana
monk load manifest.yaml
```

### Verify if it's loaded correctly

```bash
$ monk list -l monitoring

✔ Got the list
Type      Template                  Repository  Version  Tags
runnable  monitoring/grafana        local       -        -
runnable  monitoring/node-exporter  local       -        -
runnable  monitoring/prometheus     local       -        -
group     monitoring/stack          local       -        -
```

## Deploy Stack

```bash
$ monk run monitoring/stack
✔ Starting the job: local/monitoring/stack... DONE
✔ Preparing nodes DONE
✔ Checking/pulling images...
✔ [================================================] 100% docker.io/prom/prometheus:latest QmfWqNH8oktNNqaujf7XGSCRDmy4vMkcwshjyt37SE76ns
✔ [================================================] 100% quay.io/prometheus/node-exporter:latest QmfWqNH8oktNNqaujf7XGSCRDmy4vMkcwshjyt37SE76ns
✔ [================================================] 100% docker.io/grafana/grafana:latest QmfWqNH8oktNNqaujf7XGSCRDmy4vMkcwshjyt37SE76ns
✔ Checking/pulling images DONE
✔ Starting containers DONE
✔ Starting containers DONE
✔ Starting containers DONE
✔ New container local-23d4982c498a4242c04f299306-cal-monitoring-grafana-grafana created DONE
✔ New container local-a4277f5d0c8e60b2c304992412--monitoring-node-exporter-node created DONE
✔ Started local/monitoring/stack

🔩 templates/local/monitoring/stack
 └─🧊 Peer QmfWqNH8oktNNqaujf7XGSCRDmy4vMkcwshjyt37SE76ns
    ├─🔩 templates/local/monitoring/prometheus
    │  └─📦 local-8c491fc7f285432c7828a04139-cal-monitoring-prometheus-prom
    │     ├─🧩 docker.io/prom/prometheus:latest
    │     └─🔌 open 192.168.0.197:9090 -> 9090
    ├─🔩 templates/local/monitoring/grafana
    │  └─📦 local-23d4982c498a4242c04f299306-cal-monitoring-grafana-grafana
    │     ├─🧩 docker.io/grafana/grafana:latest
    │     └─🔌 open 192.168.0.197:3000 -> 3000
    └─🔩 templates/local/monitoring/node-exporter
       └─📦 local-a4277f5d0c8e60b2c304992412--monitoring-node-exporter-node
          ├─🧩 quay.io/prometheus/node-exporter:latest
          ├─💾 / -> /host
          └─🔌 open 192.168.0.197:9100 -> 9100

💡 You can inspect and manage your above stack with these commands:
        monk logs (-f) local/monitoring/stack - Inspect logs
        monk shell     local/monitoring/stack - Connect to the container's shell
        monk do        local/monitoring/stack/action_name - Run defined action (if exists)
💡 Check monk help for more!
```

## Variables

The variables are stored in `manifest.yaml` file.
You can quickly setup by editing the values there.

### Grafana

| Variable        | Description                               | Default                                             |
| --------------- | ----------------------------------------- | --------------------------------------------------- |
| admin-user      | Administrator username                    | admin                                               |
| admin-password  | Administrator password                    | adminz                                              |
| install_plugins | Default plugins to install with grafana   | grafana-clock-panel,grafana-simple-json-datasource  |
| anonymous       | Should we enable anonymous access         | false                                               |
| anonymous_role  | Role of the anonymous user                | Viewer                                              |
| prometheusHost  | Dynamic var to point to a Prometheus host | `<- get-hostname("monitoring/prometheus", "prom")`* |

* Only needs to be updated if using in your own namespace

### Prometheus

| Variable           | Description                                     | Default |
| ------------------ | ----------------------------------------------- | ------- |
| cmdParams          | Additional params to pass to Prometheus cmdline |         |
| additional_targets | Additional scrape targets for Prometheus*       |         |

* In a format: `'"another.lan:30303", "second.com:3333"'`
## Stop, remove and clean up workloads and templates

```bash
monk purge    monitoring/stack monitoring/grafana monitoring/prometheus monitoring/node-exporter
monk purge -x monitoring/stack monitoring/grafana monitoring/prometheus monitoring/node-exporter
```

## Create your own template

You can create your own template and inherit the defaults and add more changes that are needed by you.
This is an example manifest that you could adjust to your needs:

```yaml
namespace: /myOwnMonitoring

grafana:
  defines: runnable
  inherits: monitoring/grafana
  variables:
    prometheusHost:
      type: string
      value: <- get-hostname("myOwnMonitoring/prometheus", "prom")

    admin-password:
      type: string
      value: testing123

prometheus:
  defines: runnable
  inherits: monitoring/prometheus
  variables:
    additional_targets:
      type: string
      value: '"testing1.com:3333"'

stack:
  defines: process-group
  runnable-list:
    - /myOwnMonitoring/grafana
    - /myOwnMonitoring/prometheus
    - /monitoring/node-exporter
```
