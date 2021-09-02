# DeX Speed-Test Station

* This is a guide on how to setup Docker containers inside Alpine VM. Link Speed Test and Prometheus. Display Data on Grafana
  Requirements:
  - Have Alpine VM / Docker running on Termux. How to [https://github.com/mrp-yt/docker_and_portainer_on_dex](https://github.com/mrp-yt/docker_and_portainer_on_dex)

## Setup

*	Setting up config file
	Create speed_test_prometheus.yml file
	```
	apk add nano &&
	mkdir /root/prometheus &&
	nano /root/prometheus/speed_test_prometheus.yml
	```
	Copy / Paste code bellow in to `speed_test_prometheus.yml`
	```
	global:
  scrape_interval: 5s
	external_labels:
		monitor: 'node'
scrape_configs:
	- job_name: 'prometheus'
	static_configs:
		- targets: ['100.89.90.43:9090'] ## IP Address of the localhost. This will be used for Prometheus
	- job_name: 'node-exporter'
	static_configs:
		- targets: ['100.89.90.43:9100'] ## IP Address of the localhost. This will be used for Node-Exporter
	```
	*Replace IP address to your device IP*
	
*	Run command for docker to pull and start container `danopstech/speedtest_exporter:latest`
	```
	docker run -d -p 9080:9090 --name=speed_test --restart=unless-stopped danopstech/speedtest_exporter:latest 
	```

*	Setup Grafana.

	If you followed Part 2 you should have Grafana setup already. If not, run this command:
	```
	docker run -d -p 3000:3000 --name=grafana --restart=unless-stopped grafana/grafana:latest
	```

*	Setup Prometheus container.

	If you followed Part 2 you should have Prometheus running already. We are setting up 2nd instance to be used just for Speed-Test data
	```
	docker run -d -p 9091:9090 --name=speed_test_prometheus --restart=unless-stopped -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest &&
	```
	
