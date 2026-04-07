# GitHubAactions-springboot-docker-demo-customhost

STEP 1: Create Project (Manual – with pom.xml INCLUDED)

Create a folder:
mkdir springboot-docker-demo
cd springboot-docker-demo


STEP 2: Create FULL Project Structure
springboot-docker-demo/
├── src/
│   └── main/
│       ├── java/com/demo/app/
│       │   └── DemoApplication.java
│       └── resources/
│           └── application.properties
├── pom.xml
├── Dockerfile
└── .github/workflows/ci-cd.yml


STEP 3: Create pom.xml (FULLY WORKING)

Create file: pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo</groupId>
    <artifactId>springboot-docker-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-docker-demo</name>
    <description>DevOps CI/CD Demo</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>

        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Optional: Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <!-- Spring Boot Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

        </plugins>
    </build>

</project>


STEP 4: Create Java Application

👉 File:
src/main/java/com/demo/app/DemoApplication.java

package com.demo.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;

@SpringBootApplication
@RestController
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/")
    public String home() {
        return "Hello Manoj - CI/CD with Self Hosted Runner 🚀";
    }
}

STEP 5: application.properties

👉 File:
src/main/resources/application.properties
server.port=8080

STEP 6: Dockerfile

👉 File: Dockerfile

FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]

STEP 8: Push to GitHub
git init
git add .
git commit -m "initial setup"
git branch -M main
git remote add origin https://github.com/<your-username>/springboot-docker-demo.git
git push -u origin main

STEP 9: EC2 Setup (Self Hosted Runner)

SSH into EC2:
sudo apt update -y
sudo apt install openjdk-17-jdk maven docker.io -y

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker ubuntu
newgrp docker

STEP 10: Configure GitHub Runner

Go to:

👉 GitHub → Settings → Actions → Runners → New Runner

Run:
mkdir actions-runner && cd actions-runner

curl -o runner.tar.gz -L https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz

tar xzf runner.tar.gz

./config.sh --url https://github.com/<username>/springboot-docker-demo --token <TOKEN>

./run.sh

STEP 11: Add Secrets

GitHub → Settings → Secrets
DOCKER_USERNAME
DOCKER_PASSWORD

STEP 12: GitHub Actions Workflow

👉 File: .github/workflows/ci-cd.yml
name: CI-CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: self-hosted

    steps:

    - name: Checkout
      uses: actions/checkout@v3

    - name: Java Version
      run: java -version

    - name: Build JAR
      run: mvn clean package -DskipTests

    - name: Show Target
      run: ls -l target/

    - name: Docker Login
      run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

    - name: Build Image
      run: docker build -t ${{ secrets.DOCKER_USERNAME }}/springboot-demo:latest .

    - name: Push Image
      run: docker push ${{ secrets.DOCKER_USERNAME }}/springboot-demo:latest

      
      
        
STEP 13: Trigger Pipeline
git add .
git commit -m "added pipeline"
git push










