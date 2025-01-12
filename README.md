# Prometheus Kvrocks Metrics Exporter

This is a fork of oliver006/redis_exporter to export the kvrocks metrics.

## Building and running the exporter

### Build and run locally

```sh
git clone https://github.com/RocksLabs/kvrocks_exporter.git
cd kvrocks_exporter
go build .
./kvrocks_exporter --version
```

### Basic Prometheus Configuration

Add a block to the `scrape_configs` of your prometheus.yml config file:

```yaml
scrape_configs:
  - job_name: kvrocks_exporter
    static_configs:
    - targets: ['<<KVROCKS-EXPORTER-HOSTNAME>>:9121']
```

and adjust the host name accordingly.


### Kubernetes SD configurations

To have instances in the drop-down as human readable names rather than IPs, it is suggested to use [instance relabelling](https://www.robustperception.io/controlling-the-instance-label).

For example, if the metrics are being scraped via the pod role, one could add:

```yaml
          - source_labels: [__meta_kubernetes_pod_name]
            action: replace
            target_label: instance
            regex: (.*kvrocks.*)
```

as a relabel config to the corresponding scrape config. As per the regex value, only pods with "kvrocks" in their name will be relabelled as such.

Similar approaches can be taken with [other role types](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config) depending on how scrape targets are retrieved.

### Prometheus Configuration to Scrape Multiple Kvrocks Hosts

Run the exporter with the command line flag `--kvrocks.addr=` so it won't try to access the local instance every time the `/metrics` endpoint is scraped.

```yaml
scrape_configs:
  ## config for the multiple kvrocks targets that the exporter will scrape
  - job_name: 'kvrocks_exporter_targets'
    static_configs:
      - targets:
        - kvrocks://first-kvrocks-host:6666
        - kvrocks://second-kvrocks-host:6667
        - kvrocks://second-kvrocks-host:6668
        - kvrocks://second-kvrocks-host:6669
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <<KVROCKS-EXPORTER-HOSTNAME>>:9121

  ## config for scraping the exporter itself
  - job_name: 'kvrocks_exporter'
    static_configs:
      - targets:
        - <<KVROCKS-EXPORTER-HOSTNAME>>:9121
```

The kvrocks instances are listed under `targets`, the kvrocks exporter hostname is configured via the last relabel_config rule.\
If authentication is needed for the kvrocks instances then you can set the password via the `--kvrocks.password` command line option of
the exporter (this means you can currently only use one password across the instances you try to scrape this way. Use several
exporters if this is a problem). \
You can also use a json file to supply multiple targets by using `file_sd_configs` like so:

```yaml

scrape_configs:
  - job_name: 'kvrocks_exporter_targets'
    file_sd_configs:
      - files:
        - targets-kvrocks-instances.json
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <<KVROCKS-EXPORTER-HOSTNAME>>:9121

  ## config for scraping the exporter itself
  - job_name: 'kvrocks_exporter'
    static_configs:
      - targets:
        - <<KVROCKS-EXPORTER-HOSTNAME>>:9121
```

The `targets-kvrocks-instances.json` should look something like this:

```json
[
  {
    "targets": [ "kvrocks://kvrocks-host-01:6666", "kvrocks://kvrocks-host-02:6667"],
    "labels": { }
  }
]
```

Prometheus uses file watches and all changes to the json file are applied immediately.

## For Grafana 8.x

For Grafana 8.x, the default Prometheus data store access mode was `Server` which may have
the CORS issue, you can workaround this by choosing the `browser` mode or fix the CORS problem.

![image](https://user-images.githubusercontent.com/4987594/143570291-e4882b52-3a7a-4482-8bf1-ca6539a6b14c.png)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FG-Core%2Fkvrocks_exporter.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2FG-Core%2Fkvrocks_exporter?ref=badge_shield)

## What it looks like
Kvrocks Grafana dashboard template is available on [Grafana.com](https://grafana.com/grafana/dashboards/15286) and imports
the Dashboard with ID `15286` or download the JSON file.

Example Grafana screenshots:
![Grafana Example](https://grafana.com/api/dashboards/15286/images/11310/image)


## Communal effort

Open an issue or PR if you have more suggestions, questions or ideas about what to add.


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FG-Core%2Fkvrocks_exporter.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2FG-Core%2Fkvrocks_exporter?ref=badge_large)