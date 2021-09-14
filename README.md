# DeX Speed-Test Station

![Speed_Test_Exporter](/assests/images/speedtest_tracking.jpg)

* This is a guide on how to setup Docker containers inside Alpine VM. Link Speed Test and Prometheus. Display Data on Grafana
  Requirements:
  - Have Alpine VM / Docker running on Termux. How to [https://github.com/mrp-yt/docker_and_portainer_on_dex](https://github.com/mrp-yt/docker_and_portainer_on_dex)
 
* NOTE
  - If you will use your device local IP address instead of TailScale IP, your internet monitoring system will only capture your home broadband speed. I am using TailScale which means it does if device local IP will change. TailScale IP will always be the same. That means IP link between data scrap will not disconect.

## Setup

### Setting up ports
Add these in to `startqemu.sh` located inside `~/alpine/`.\
If you already have Grafana running can ignore 1st line `hostfwd=tcp::3000-:3000,\`
```
hostfwd=tcp::3000-:3000,\
hostfwd=tcp::9091-:9091,\
hostfwd=tcp::9080-:9080,\
```
Ports used \
`3000` - Grafana \
`9091` - Prometheus container \
`9080` - Speed Test container

Start Alpine VM
```
cd ~/alpine/ && ./startqemu.sh
```

### Setting up Prometheus config file

Create Prometheus directory if not exist and `speed_test_prometheus.yml` config file inside.\
I have add `apk add nano` just in case if editor is missing. 
```
apk add nano &&
mkdir -p ~/prometheus/ &&
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
  - job_name: 'speed_test_prometheus'
    static_configs:
      - targets: ['100.89.90.43:9091'] ## IP Address of the localhost. This will be used for Prometheus
  - job_name: 'speed_test'
    static_configs:
      - targets: ['100.89.90.43:9080'] ## IP Address of the localhost. This will be used for Node-Exporter
```
**Replace IP address to your device IP**

### Setting up containers	
Containers for Speed-Test and Prometheus
```
docker run -d -p 9080:9090 --name=speed_test --restart=unless-stopped danopstech/speedtest_exporter:latest &&
docker run -d -p 9091:9090 --name=speed_test_prometheus --restart=unless-stopped -v /root/prometheus/speed_test_prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest &&
docker run -d -p 3000:3000 --name=grafana --restart=unless-stopped grafana/grafana:latest
```

### Setting up Grafana

1.	Open Grafana WEB UI\
`http://localhost:3000` or `http://<YOUR_DEVICE_IP>:3000`
2.	Click on gear icon on the left and then click on *Add data source*
3.	Select *Prometheus*
4.	Give a name. Under HTTP URL enter `http://<YOUR_DEVICE_IP>:9091`. Scroll down and click *Save & test*. You should get green test saying *Data source is working*
5. 	Mouse over *plus icon* and click *import*. Inside *Import via grafana.com* enter `14336` and click Load. 
6.	On next page give dashboard a name and in last option select data source which one you created in step 4. Click Import.\
	**All Done**

	Give some time for data to show up. I suggest 30m+. 

### Useful Info
-	If data not showing up after 1h+ check Prometheus dashboard.
	`http://<YOUR_DEVICE_IP>:9091/targets`\
	You should see two targers in the list `speed_test_prometheus` and `speed_test`. Both should have green `UP` in Status column. 
-	It takes time between Prometheus scraping data from Speed-Test container and showing up inside Grafana. If Prometheus/Targets page showing now errors and Grafana not showing any new data try to refresh Grafana dashboard by clicking *Refresh dashboard* button (top righthand coner inside Grafana Dashboard page).
