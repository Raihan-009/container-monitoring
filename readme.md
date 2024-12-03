# Docker Container Monitoring Guide

This guide walks through setting up and using different tools to monitor Docker containers, including Prometheus, cAdvisor, and Docker's built-in stats command.

## Part 1: Setting Up Prometheus with cAdvisor

### 1. Create Prometheus Configuration
First, create a `prometheus.yml` file in your root directory with the following configuration:

```yaml
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```

This configuration:
- Defines a job named "cadvisor"
- Sets the scraping interval to 5 seconds
- Configures Prometheus to scrape metrics from cAdvisor on port 8080

### 2. Set Up Docker Compose
Create a `docker-compose.yml` file with two services:

```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - 9090:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
      - cadvisor
    
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
```

Key components:
- Prometheus service:
  - Uses the latest Prometheus image
  - Maps port 9090
  - Mounts the prometheus.yml configuration file
  - Depends on the cAdvisor service

- cAdvisor service:
  - Uses the latest cAdvisor image
  - Maps port 8080
  - Mounts necessary system directories for container monitoring

### 3. Launch the Services
Start the containers in detached mode:

```bash
docker-compose up -d
```

You can now access:
- Prometheus UI at `http://localhost:9090`
- cAdvisor UI at `http://localhost:8080`

## Part 2: Using Docker Stats

### 1. Create the Stats Script
Create a shell script named `stats.sh` in your root directory:

```bash
#!/bin/bash
docker stats --format "table {{.Name}} {{.ID}} {{.MemUsage}} {{.CPUPerc}}"
```

This script will display:
- Container name
- Container ID
- Memory usage
- CPU percentage

### 2. Make the Script Executable
```bash
chmod a+x stats.sh
```

### 3. Run the Stats Monitor
```bash
./stats.sh
```

The output will show a real-time table of container metrics. Press Ctrl+C to exit the monitoring.

## Understanding the Metrics

### Prometheus/cAdvisor Metrics
- Container resource usage
- Historical performance data
- Custom metrics and alerting capabilities
- Detailed system-level metrics

### Docker Stats Metrics
- Real-time container performance
- Memory usage and limits
- CPU usage percentage
- Basic container identification

## Best Practices

1. **Regular Monitoring**
   - Check container metrics regularly
   - Set up alerts for abnormal patterns
   - Monitor system resource usage trends

2. **Resource Management**
   - Set appropriate container resource limits
   - Monitor for container resource exhaustion
   - Track historical performance patterns

3. **Security**
   - Use read-only volume mounts where possible
   - Restrict access to monitoring endpoints
   - Regularly update monitoring tools

## Troubleshooting

If you encounter issues:

1. Check container logs:
```bash
docker logs prometheus
docker logs cadvisor
```

2. Verify service accessibility:
```bash
curl localhost:9090/-/healthy
curl localhost:8080/healthz
```

3. Ensure proper volume permissions:
```bash
ls -la /var/run/docker.sock
```
