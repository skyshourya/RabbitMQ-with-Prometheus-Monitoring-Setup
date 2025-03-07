# RabbitMQ with Prometheus Monitoring Setup
## This repository contains the steps to set up RabbitMQ with Prometheus monitoring using the rabbitmq-exporter to expose RabbitMQ metrics for Prometheus scraping.

## Prerequisites
### AWS EC2 instance running Ubuntu (or any Linux distribution).
### Docker and Docker Compose installed.
### A working RabbitMQ instance (either locally or on a remote server).
### Prometheus, Grafana, and Alertmanager for monitoring and alerting (optional but recommended).
### Steps to Set Up RabbitMQ with Prometheus Monitoring

## 1. Install RabbitMQ
### If you haven't installed RabbitMQ yet, you can follow these steps to set it up on your EC2 instance:

```bash
sudo apt-get update
sudo apt-get install -y rabbitmq-server
```

## Start RabbitMQ service:
```bash
sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server
```

## 2. Install RabbitMQ Management Plugin
### Enable the RabbitMQ management plugin to access the web UI and API:
```bash
sudo rabbitmq-plugins enable rabbitmq_management
```
## The RabbitMQ management plugin provides access to the web interface at http://<your_instance_ip>:15672 (replace <your_instance_ip> with your server's IP).

The default credentials for RabbitMQ are:

Username: `guest`
Password: `guest`
## 3. Install Docker on the EC2 Instance
###If Docker is not installed, follow these commands to install Docker:

## 4. Run RabbitMQ Exporter
### Now, let's use the rabbitmq-exporter to expose RabbitMQ metrics to Prometheus. We will run rabbitmq-exporter in a Docker container.
```bash
docker run -d --name rabbitmq-exporter --network=host -e RABBIT_URL="http://13.201.81.35:15672" -e RABBIT_USER="guest" -e RABBIT_PASSWORD="guest"  kbudde/rabbitmq-exporter
```

Replace <your_instance_ip> with the IP address of your EC2 instance.
This command exposes RabbitMQ metrics at http://<your_instance_ip>:9419/metrics.

## 5. Test RabbitMQ Exporter Metrics
### After running the container, you can test the Prometheus metrics endpoint by visiting:

`http://<your_instance_ip>:9419/metrics`

If everything is working correctly, you should see a list of available metrics exposed by the rabbitmq-exporter.

## 6. Configure Prometheus to Scrape Metrics
### Edit your Prometheus configuration file (prometheus.yml) to scrape RabbitMQ metrics from the exporter:
```
scrape_configs:
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['<your_instance_ip>:9419']
Replace <your_instance_ip> with the IP of your EC2 instance.
```


## 7. Set Up Grafana (Optional but Recommended)
## For visualizing the RabbitMQ metrics in Grafana:

Install Grafana on your EC2 instance.

Follow Grafana Installation for your specific platform.

Add Prometheus as a Data Source in Grafana.

Go to Grafana's web interface (default: http://<your_instance_ip>:3000) and add your Prometheus instance as a data source.

Import RabbitMQ Dashboard:

You can import a predefined RabbitMQ dashboard from Grafana's dashboard repository (ID: 10991) for a comprehensive view of your RabbitMQ metrics.

8. Set Up Alerts (Optional)
If you want to set up alerts for RabbitMQ metrics, you can use Alertmanager in combination with Prometheus.

Install Alertmanager.

Follow Alertmanager Installation to install it on your instance.

Configure Alertmanager in your `prometheus.yml`
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'
```
Create Alerts for important metrics (e.g., queue length, node health, etc.). You can define alerting rules in the Prometheus configuration:
```
groups:
  - name: rabbitmq-alerts
    rules:
      - alert: RabbitMQNodeDown
        expr: rabbitmq_node_up == 0
        for: 5m
        annotations:
          summary: "RabbitMQ node is down"
```
## Configure Slack or Email Notifications in Alertmanager by updating alertmanager.yml.


Conclusion
You've successfully set up RabbitMQ with Prometheus monitoring using the rabbitmq-exporter to expose metrics. Now, you can visualize RabbitMQ's health and performance metrics in Grafana and set up alerts in Prometheus and Alertmanager.
