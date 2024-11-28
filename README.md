### **Documentation for Setting Up the Application and Monitoring Stack**

This guide will walk you through deploying a monitoring stack, including **Prometheus**, **Grafana**, **cAdvisor**, and **NGINX Proxy Manager**, step by step. We’ll also highlight the purpose of each step so you can understand why it’s necessary.

---

### **1. File Structure Overview**

From the provided structure:
- **backend**: Contains backend application Dockerfiles and configurations.
- **frontend**: Contains frontend application files and an `nginx.conf` file.
- **monitoring**: Contains configuration files for monitoring tools like Prometheus, Loki, and Promtail. Also contains the docker compose file for the spinning them up and ensuring they interact with fullstack application services. 
- **docker-compose.yml**: Combines all services (monitoring, backend, and frontend) into a single deployment.
- **README.md**: Explains the purpose and usage of the project.
- **LICENSE**: Specifies licensing terms.

---

### **2. Prerequisites**

Before starting:
1. **Install Docker** and **Docker Compose** on your system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install docker.io -y
   sudo apt install docker-compose-v2 -y
   sudo usermod -aG docker $USER
   ```
2. Ensure your machine has at least:
   - 4GB RAM
   - Stable internet connection for pulling images.

---

### **3. Understanding the Components**

- **Backend and Frontend**:
  - Backend: Your application logic (e.g., API services).
  - Frontend: The user interface served by an NGINX server.
- **Monitoring Tools**:
  - **Prometheus**: Scrapes metrics from services.
  - **Grafana**: Visualizes the metrics for better insights.
  - **cAdvisor**: Collects container-specific metrics like CPU, memory, and disk usage.
- **NGINX Proxy Manager**:
  - Acts as a reverse proxy for routing and securing traffic.

---

### **4. File Details**

#### **4.1 Backend**
- Contains the backend application logic and **Dockerfile** for containerization.
- Purpose: Allows the backend to be run in a Docker container.

#### **4.2 Frontend**
- Contains `nginx.conf`, which configures how the frontend serves static files.
- Purpose: Provides the user-facing portion of the application.

#### **4.3 Monitoring**
- Contains configuration files for Prometheus, Grafana, and other monitoring services.
- Purpose: To monitor the health, performance, and resource usage of your applications.

#### **4.4 Docker Compose**
- Combines all services into one deployment.
- Purpose: Simplifies launching all services with one command.

---

### **5. Deployment Steps**

#### **Step 1: Clone the Repository**
Clone the repository to your local system:
```bash
git clone https://github.com/GideonIsBuilding/cv-challenge01.git
cd cv-challenge01
```

---

#### **Step 2: Navigate to the Directory and Start the Application Services**
Here, in the root folder, you will find the `docker-compose.yml` file. Enter and run the following command:
```bash
docker-compose -p cv-challenge -f docker-compose.yml up -d
```

---

#### **Step 3: Configure Prometheus**
Prometheus needs a configuration file (`prometheus.yml`) to define which targets to scrape metrics from.

1. Go to the **monitoring** folder.
2. Ensure there is a `prometheus.yml` file with the following contents:
   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: "prometheus"
       static_configs:
         - targets: ["localhost:9090"]
     - job_name: "cadvisor"
       static_configs:
         - targets: ["cadvisor:8080"]
   ```
   **Why?** This tells Prometheus to scrape metrics from itself (`localhost:9090`) and cAdvisor (`cadvisor:8080`).

---

#### **Step 4: Update Environment Variables for Grafana**
Grafana needs to know the root URL if it’s being accessed via a subpath (e.g., `/grafana`).

1. Open the `docker-compose.yml` file.
2. Under the **Grafana service**, add:
   ```yaml
   environment:
     - GF_SERVER_ROOT_URL=http://localhost/grafana
     - GF_SERVER_SERVE_FROM_SUB_PATH=true
   ```
   **Why?** This ensures Grafana correctly serves its dashboard when accessed at `/grafana`.

---

#### **Step 5: Start the Monitoring Stack**
Run the following command to bring up all services:
```bash
docker-compose -p cv-challenge -f docker-compose.yml up -d
```
- `-d`: Runs the containers in detached mode.

**Why?** This starts all services in the background, allowing them to interact with each other.

---

#### **Step 6: Configure Reverse Proxy in NGINX Proxy Manager**
If you’re using NGINX Proxy Manager, create a **single proxy host** for `localhost`, and add the following **custom locations**:
1. `/grafana` -> Forward to `http://grafana:3000`
2. `/prometheus` -> Forward to `http://prometheus:9090`
3. `/api` -> Forward to `http://backend:8000`
4. `/docs` -> Forward to `http://backend:8000`
- Then click on the gear button besides the location fild. This opens up a field to enter the advanced configurations. Enter in the following:
```
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

**Why?** This makes the services accessible through specific paths.

---

#### **Step 7: Access the Services**
- **Prometheus**: `http://localhost/prometheus`
- **Grafana**: `http://localhost/grafana`
.. and so on.

---

### **6. Key Features**

1. **Monitoring with Grafana and Prometheus**:
   - Add Prometheus as a data source in Grafana.
   - Use dashboards to visualize container metrics (CPU, memory, disk usage).

2. **Frontend and Backend Deployment**:
   - The backend API and frontend UI are deployed in separate containers.

3. **Reverse Proxy with NGINX Proxy Manager**:
   - Centralized routing for easy access to services.

---

### **7. Troubleshooting**

#### **Issue: Prometheus Target Shows 404**
- Check if `/metrics` is accessible:
  ```bash
  curl http://localhost:9090/metrics
  ```
- Fix the `prometheus.yml` file or ensure no subpath issues exist.

#### **Issue: Grafana Dashboard Not Loading**
- Verify `GF_SERVER_ROOT_URL` and `GF_SERVER_SERVE_FROM_SUB_PATH` are set correctly.

#### **Issue: Services Not Starting**
- Run:
  ```bash
  docker-compose logs <service-name>
  ```
  Check for error messages in the logs.

---

### **8. Next Steps**

1. **Secure the Setup**:
   - Use SSL certificates with NGINX Proxy Manager.
   - Restrict access to monitoring tools using basic authentication.

2. **Expand Monitoring**:
   - Add Node Exporter for host-level metrics.
   - Integrate Alertmanager with Prometheus for alerting.

3. **Automate Deployment**:
   - Use CI/CD pipelines to automate deployments for the backend, frontend, and monitoring stack.

---

### **Final Notes**

This guide simplifies deploying a monitoring stack for your containerized applications. If you encounter any issues, refer to the troubleshooting section or logs for insights. Let me know if you need further help! 😊