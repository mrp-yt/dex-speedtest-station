# DeX Speed-Test Station

* This is a guide on how to setup Docker containers inside Alpine VM. Link Speed Test and Prometheus. Display Data on Grafana
  Requirements:
  - Have Alpine VM / Docker running on Termux. How to [https://github.com/mrp-yt/docker_and_portainer_on_dex](https://github.com/mrp-yt/docker_and_portainer_on_dex)
  - If you followed [my dex-monitoring-station guide](https://github.com/mrp-yt/dex-monitoring-station), you already have Grafana setup. Some of the steps can be skipped. 

## Setup

### Setting up ports
Add these in to `startqemu.sh` located inside `~/alpine/`. If you already have Grafana setup done, you can ignore 1st line `hostfwd=tcp::3000-:3000,\`
```
hostfwd=tcp::3000-:3000,\
hostfwd=tcp::9091-:9091,\
hostfwd=tcp::9080-:9080,\
```
Ports used for \
`3000` - Grafana \
`9091` - Prometheus container \
`9080` - Speed Test container

### Setting up Prometheus config file

If you have `/prometheus/` folder already
```
apk add nano &&
nano /root/prometheus/speed_test_prometheus.yml
```
If you don't have prometheus folder:
```
apk add nano &&
mkdir ~/alpine/prometheus/
nano /root/prometheus/speed_test_prometheus.yml
```
Copy / Paste code bellow in to `speed_test_prometheus.yml`
```
global:
  scrape_interval: 15m
  scrape_timeout: 2m
  external_labels:
    monitor: 'node'
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['100.89.90.43:9091'] ## IP Address of the localhost. This will be used for Prometheus
  - job_name: 'node-exporter'
	static_configs:
      - targets: ['100.89.90.43:9080'] ## IP Address of the localhost. This will be used for Node-Exporter
```
**Replace IP address to your device IP**

### Setting up containers	
Containers for Speed-Test and Prometheus
```
docker run -d -p 9080:9090 --name=speed_test --restart=unless-stopped danopstech/speedtest_exporter:latest &&
docker run -d -p 9091:9090 --name=speed_test_prometheus --restart=unless-stopped -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest
```

If you don't have Grafana already setup
```
docker run -d -p 3000:3000 --name=grafana --restart=unless-stopped grafana/grafana:latest
```
	
