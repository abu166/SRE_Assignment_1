# SRE_Assignment_1

# Task 1: Setting Up an SRE Environment

## Task 1.1: Installation of Docker

### 1. Research and Choose Docker

Docker is a containerization platform that allows you to run applications in isolated environments called containers. Unlike traditional virtualization, Docker shares the host OS kernel, making it lightweight and efficient.

- **Platform Options**: 
  - Docker Desktop is suitable for local development
  - Docker Engine is ideal for server environments

### 2. Download and Install Docker

Visit the official Docker website: https://www.docker.com/

**Download Process for Different Operating Systems:**
- **Windows**: 
  - Download Docker Desktop for Windows
  - Ensure WSL2 (Windows Subsystem for Linux) is enabled if using Windows 10 Home

- **macOS**: 
  - Download Docker Desktop for Mac 
  - Available for both Intel and Apple Silicon processors

- **Linux**: 
  - Follow distribution-specific instructions from Docker documentation
  - Detailed guide available at: https://docs.docker.com/engine/install/

### 3. Install Docker

1. Run the installer and follow the on-screen instructions.
2. Verify Docker installation by checking the version:
   ```bash
   docker --version
   ```
   This command should display the installed Docker version.

### 4. Run a Sample Container

Test Docker installation with the `hello-world` image:
```bash
docker run hello-world
```

**Expected Output:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

# Task 1.2: Configuration of Network Settings, Storage, and Backup Mechanisms

## 1. Network Configuration

Docker provides several networking modes:
- **Bridge Network**: Default network mode. Containers can communicate with each other and the host.
- **Host Network**: Containers share the host's network stack.
- **Overlay Network**: Used for multi-host communication in Docker Swarm or Kubernetes.
- **Macvlan Network**: Assigns a MAC address to a container, making it appear as a physical device.

### Network Configuration Steps:

1. Create a custom bridge network:
   ```bash
   docker network create my-bridge-network
   ```

2. Run a container attached to this network:
   ```bash
   docker run --name my-container --network my-bridge-network -d nginx
   ```

3. Verify the container is running:
   ```bash
   docker ps
   ```

## 2. Storage Configuration

Docker supports three types of storage:
- **Volumes**: Persistent storage outside the container lifecycle.
- **Bind Mounts**: Map a host directory to a container directory.
- **tmpfs Mounts**: Temporary storage inside the container.

### Storage Configuration Steps:

1. Create a volume:
   ```bash
   docker volume create my-volume
   ```

2. Run a container with the volume mounted:
   ```bash
   docker run --name my-container -v my-volume:/data -d nginx
   ```

3. Verify the volume is mounted:
   ```bash
   docker inspect my-container
   ```

## 3. Backup Mechanisms

Use Docker's built-in tools for backups:

### Commit Method
Save changes to a container as a new image:
```bash
docker commit my-container my-backup-image
```

### Docker Save/Load Method
Export and import images:
```bash
docker save -o my-backup.tar my-backup-image
docker load -i my-backup.tar
```

# Task 2: Introduction to SRE Tools

## Task 2.1: Installation of Essential SRE Tools

### 1. Prometheus
Prometheus is a monitoring tool that collects metrics and stores them as time-series data.

**Docker Installation:**
```bash
docker run -d --name prometheus -p 9090:9090 prom/prometheus
```

**Access:**
- Prometheus UI: `http://localhost:9090`

### 2. Alertmanager
Alertmanager handles alerts sent by Prometheus.

**Docker Installation:**
```bash
docker run -d --name alertmanager -p 9093:9093 prom/alertmanager
```

**Access:**
- Alertmanager UI: `http://localhost:9093`

### 3. ELK Stack
Run a pre-built ELK Docker image for log management and analysis.

**Docker Installation:**
```bash
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.17.0
docker run -d --name kibana -p 5601:5601 --link elasticsearch:elasticsearch docker.elastic.co/kibana/kibana:7.17.0
```

**Access:**
- Elasticsearch: `curl http://localhost:9200`

### 4. Grafana (Bonus)
Optional visualization tool for metrics and monitoring.

**Docker Installation:**
```bash
docker run -d --name grafana -p 3000:3000 grafana/grafana
```

**Access:**
- Grafana UI: `http://localhost:3000`

## Task 2.2: Overview and Demonstration of Tool Functionalities

### 1. Prometheus Monitoring

### Step 1: Configure `prometheus.yml`

1. Open the `prometheus.yml` configuration file.

2. Add the following configuration under `scrape_configs`:
   ```yaml
   scrape_configs:
     - job_name: "prometheus"
       static_configs:
         - targets: ["localhost:9090", "localhost:4000"]
       scrape_interval: 15s # Set the scrape interval to 15 seconds.
       scrape_timeout: 10s # Set the scrape timeout to 10 seconds.
   ```

**Configuration Explanation:**
- `job_name`: Identifies the scrape job
- `targets`: Specifies the endpoints to scrape metrics from
  - `localhost:9090`: Prometheus itself
  - `localhost:4000`: Another service or application
- `scrape_interval`: How often Prometheus collects metrics (15 seconds)
- `scrape_timeout`: Maximum time allowed for a scrape request (10 seconds)

### Step 2: Run Prometheus

Start Prometheus with the configuration file:
```bash
prometheus --config.file=prometheus.yml
```

### Step 3: Verify Target Status

Use the `up` expression in Prometheus to check target health:

- `up: 1` means the target is healthy and reachable
- `up: 0` indicates the target is down or unreachable

### Step 4: Query Metrics Using PromQL

Open another terminal and use `curl` to query metrics:
```bash
curl -X GET "http://localhost:9090/api/v1/query?query=up"
```

**Troubleshooting:**
To detect HTTP 500 errors:
```bash
curl -X GET "http://localhost:9090/api/v1/query?query=sum(rate(http_requests_total{status=~\"5..\"}[5m]))"
```

### 2. Alertmanager Configuration


1. Create an ```alertmanager.yml``` file:

**Creating an Alert Rule in Prometheus:**
```yaml
    cat <<EOF > alertmanager.yml
    route:
    receiver: 'team-email'
    group_by: ['severity']
    routes:
        - match:
            severity: 'critical'
        receiver: 'pagerduty'

    receivers:
    - name: 'team-email'
        email_configs:
        - to: 'team@example.com'
    - name: 'pagerduty'
        pagerduty_configs:
        - service_key: '<your-pagerduty-key>'
    EOF
```

**Troubleshooting:**
Simulate an alert by triggering it in Prometheus:
```bash
curl -X POST "http://localhost:9093/api/v2/alerts" \
  -H "Content-Type: application/json" \
  -d '[{"labels": {"alertname": "HighRequestLatency", "severity": "critical"}}]'
```

### 3. ELK Stack Configuration
### Step 1: Install Elasticsearch, Logstash, and Kibana

Pull and run the ELK Docker image:
```bash
# Pull the ELK Docker image
docker pull sebp/elk

# Run the ELK container
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it sebp/elk
```

**Port Mappings:**
- `5601`: Kibana web interface
- `9200`: Elasticsearch HTTP API
- `5044`: Logstash Beats input

### Step 2: Configure Logstash

Create a `logstash.conf` file:
```bash
    cat <<EOF > logstash.conf
    input {
    file {
        path => "/var/log/application.log"
        start_position => "beginning"
    }
    }
    output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "application-logs"
    }
    }
    EOF
```

Run Logstash inside the container:
```bash
# Enter the container
docker exec -it <container_id> /bin/bash

# Start Logstash with the configuration
/usr/share/logstash/bin/logstash -f /path/to/logstash.conf
```

### Step 3: Visualize Logs in Kibana

Access Kibana at `http://localhost:5601`:

1. **Create an Index Pattern:**
   - Navigate to Management > Stack Management > Index Patterns
   - Create a new index pattern: `application-logs`

2. **Build a Dashboard:**
   - Go to Dashboard > Create new dashboard
   - Add visualizations based on your log data
   - Save and customize your dashboard

## Troubleshooting and Log Search

Use Kibana's Dev Tools to search logs:
```json
    GET /application-logs/_search
    {
    "query": {
        "match": {
        "message": "error"
        }
    }
    }
```

### 4. Grafana Dashboard
- Create a dashboard
- Add panels to visualize metrics from Prometheus
