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
docker run -d --name elk -p 5601:5601 -p 9200:9200 sebp/elk
```

**Access:**
- Kibana: `http://localhost:5601`

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

**Setting Up a Monitor:**
- Add a target in `prometheus.yml` and restart the container

**PromQL Example Query:**
```promql
rate(http_requests_total[5m])
```

### 2. Alertmanager Configuration

**Creating an Alert Rule in Prometheus:**
```yaml
groups:
  - name: example
    rules:
      - alert: HighRequestLatency
        expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High request latency"
```

### 3. ELK Stack Configuration
- Configure Logstash to ingest logs
- Create a dashboard in Kibana to visualize logs

### 4. Grafana Dashboard
- Create a dashboard
- Add panels to visualize metrics from Prometheus
