# camunda-springboot-docker-compose
Here's the content formatted as a `.md` file:

# Camunda 7 with MySQL Integration Using Docker

This guide outlines the process of configuring and deploying Camunda 7 with MySQL using Docker and Docker Compose. By the end of this setup, Camunda will utilize MySQL as its database for process and task management.

## Prerequisites
- Docker and Docker Compose installed on your system.
- Basic knowledge of YAML and Docker commands.

## Overview
You have two options for the Camunda Docker image:
1. **Custom Camunda Image**: Use your customized Camunda image with pre-configured plugins, libraries, or workflows.
2. **Official Camunda Image**: Use the official Camunda 7 image from Docker Hub for a quick and general-purpose setup.

Choose the image type based on your requirements.

## Steps

### Step 1: Prepare the Docker Compose File
1. Create a `docker-compose.yml` file in your project directory.
2. Replace the image name in the `camunda` service section:
   - Use your custom image (e.g., `your-dockerhub-username/camunda-custom-app:latest`), or
   - Use the official Camunda image (e.g., `camunda/camunda-bpm-platform:7.22.0`).

**Example `docker-compose.yml`:**
```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: camunda
      MYSQL_USER: camunda_user
      MYSQL_PASSWORD: camunda_pass
    volumes:
      - camunda-mysql-data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u camunda_user -p'camunda_pass' || exit 1"]
      interval: 10s
      retries: 5
    networks:
      - camunda-network

  camunda:
    image: camunda/camunda-bpm-platform:7.22.0
    container_name: camunda
    ports:
      - "8080:8080"
    environment:
      - DB_DRIVER=com.mysql.cj.jdbc.Driver
      - DB_URL=jdbc:mysql://mysql:3306/camunda?useSSL=false&serverTimezone=UTC
      - DB_USERNAME=camunda_user
      - DB_PASSWORD=camunda_pass
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - camunda-network

volumes:
  camunda-mysql-data:

networks:
  camunda-network:
```

---

### Step 2: Run Docker Compose
Run the following command in the directory containing `docker-compose.yml`:

```bash
docker-compose up -d
```

This will:
1. Start the MySQL container with the specified database and credentials.
2. Start the Camunda container configured to connect to the MySQL database.

---

### Step 3: Verify the Deployment
1. Open your browser and navigate to the Camunda Web App:
    - **URL**: `http://localhost:8080/camunda`
    - **Default Credentials**:
        - Username: `demo`
        - Password: `demo`
2. Confirm that Camunda is running and connected to the MySQL database.

---

### Step 4: Access the MySQL Database
To connect to the MySQL database, run the following command:

```bash
docker exec -it mysql mysql -u camunda_user -p camunda
```

Enter the password (`camunda_pass`) and inspect the Camunda tables:

```sql
SHOW TABLES;
```

---

### Step 5: Stopping and Cleaning Up
To stop and remove the containers, run:
```bash
docker-compose down
```

To remove volumes and erase all data, run:
```bash
docker-compose down --volumes
```

---

## Notes
- Refer to the [detailed documentation](https://docs.google.com/document/d/1_GoU0j9I2ICTpzsI2voxFDKDnapetyBqvYajcthDFAQ/edit?usp=sharing) for additional information and troubleshooting.
- For advanced use cases, consider building a custom Camunda image by extending the official one in a `Dockerfile`.

---

## Comparison: Custom vs. Official Camunda Image

| Aspect         | Custom Image                                   | Official Image                          |
|----------------|-----------------------------------------------|-----------------------------------------|
| **Use Case**   | Pre-configured workflows, plugins, or libraries | General-purpose, quick setup            |
| **Flexibility**| Requires building and maintaining manually     | Ready-to-use with minimal configuration |
| **Examples**   | `your-dockerhub-username/camunda-custom-app:latest` | `camunda/camunda-bpm-platform:7.22.0`  |
```
