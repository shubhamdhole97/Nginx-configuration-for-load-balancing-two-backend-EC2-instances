# 🌸 Spring PetClinic Deployment with Nginx Load Balancer on AWS EC2

This guide walks through deploying the **Spring PetClinic** application
across **4 AWS EC2 servers** with a dedicated **MySQL Database server**
and an **Nginx Load Balancer**.

------------------------------------------------------------------------

## 🖥️ Server Architecture

We will use **4 servers**:

1.  **App Server 1** -- Runs Spring Boot PetClinic application
2.  **App Server 2** -- Runs Spring Boot PetClinic application
3.  **Database Server** -- Runs MySQL database
4.  **Load Balancer Server** -- Runs Nginx for load balancing

------------------------------------------------------------------------

## 📦 Setup on Each PetClinic Application Server

### 1️⃣ Install Java & Maven

``` bash
sudo apt update -y
sudo apt install -y openjdk-17-jdk maven

# Verify
java -version
javac -version
mvn -version
```

### 2️⃣ Set Java and Maven Environment Paths

``` bash
nano ~/.bashrc
```

Add:

``` bash
# Java
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

# Maven
export MAVEN_HOME=/usr/share/maven
export PATH=$MAVEN_HOME/bin:$PATH
```

Apply changes:

``` bash
source ~/.bashrc
echo $JAVA_HOME
echo $MAVEN_HOME
```

------------------------------------------------------------------------

## 🗄️ Setup on MySQL Database Server

### 3️⃣ Install & Configure MySQL

``` bash
sudo apt update
sudo apt install -y mysql-server
sudo systemctl status mysql
```

Login to MySQL:

``` bash
sudo mysql
```

Create database & user:

``` sql
CREATE DATABASE petclinic;
CREATE USER 'petclinic'@'%' IDENTIFIED BY 'shubham123';
GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'%';
FLUSH PRIVILEGES;
EXIT;
```

Allow remote connections:

``` bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Change:

``` ini
bind-address = 0.0.0.0
```

Restart MySQL:

``` bash
sudo systemctl restart mysql
```

------------------------------------------------------------------------

## ⚙️ Configure PetClinic Application

### 4️⃣ Clone & Build PetClinic

``` bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
```

Edit configuration:

``` bash
nano src/main/resources/application.properties
```

Add:

``` properties
# Database configuration
spring.datasource.url=jdbc:mysql://<DB-PRIVATE-IP>:3306/petclinic
spring.datasource.username=petclinic
spring.datasource.password=shubham123

# JPA configuration
spring.jpa.hibernate.ddl-auto=update
```

Build JAR:

``` bash
mvn clean package -DskipTests
```

Run:

``` bash
java -jar target/*.jar
```

(Optional) Create a `systemd` service for auto-start.

------------------------------------------------------------------------

## 🌐 Setup on Nginx Load Balancer Server

### 5️⃣ Install Nginx

``` bash
sudo apt update && sudo apt install -y nginx
```

### 6️⃣ Configure Load Balancer

Edit:

``` bash
sudo nano /etc/nginx/sites-available/default
```

Add:

``` nginx
upstream petclinic_backend {
    server <APP1-PRIVATE-IP>:8080;
    server <APP2-PRIVATE-IP>:8080;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://petclinic_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Test & reload:

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

------------------------------------------------------------------------

## 🔒 Security Groups Setup

-   **App Servers:** Allow inbound `8080` only from the LB's private
    IP.\
-   **DB Server:** Allow inbound `3306` only from the App servers'
    private IPs.\
-   **LB Server:** Allow inbound `80` (and `443` if using TLS) from the
    internet.

------------------------------------------------------------------------

## ✅ Testing

Access from browser:

    http://<LB-PUBLIC-IP>/

------------------------------------------------------------------------

## 📜 License

This project is based on the [Spring
PetClinic](https://github.com/spring-projects/spring-petclinic) sample
application.
