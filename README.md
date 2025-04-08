# Java WebApp CI/CD Pipeline using GitHub Actions, SonarQube, Nexus & Tomcat

![architechture](project.png)

This project demonstrates how to build a complete CI/CD pipeline using **GitHub Actions** to deploy a Java web application to **Apache Tomcat** on an EC2 server. The pipeline also integrates with **SonarQube** for code quality analysis, **Nexus Repository** for artifact storage, and includes optional **notifications** via email or Slack.

---

## 🚀 Tech Stack Used
- GitHub Actions
- Apache Tomcat 9
- SonarQube (Docker)
- Nexus Repository Manager 3
- Maven
- Java 11/17
- Ubuntu Server
- Notifications via Email (Optional)

---

## 📌 Step 1: Setup Tomcat, SonarQube & Nexus Artifactory Server

![step1](steps/step1a.png)

### 🔧 Install Apache Tomcat
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-17-jdk -y
sudo apt install tomcat9 tomcat9-admin tomcat9-common -y
sudo systemctl start tomcat9
sudo systemctl enable tomcat9
```

**Edit `tomcat-users.xml` to add user roles:**
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="tomcat123" roles="manager-gui,manager-script"/>
```

**Edit `context.xml` to allow remote deployment:**
Comment out this line:
```xml
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
     allow="127\..+|::1|0:0:0:0:0:0:0:1" /> -->
```


### 🔧 Install SonarQube via Docker
```bash
sudo apt update -y
sudo apt install docker.io -y
sudo docker run -itd --name sonarqube -p 9000:9000 sonarqube:lts
```
![step1](steps/step1b.png)

### 🔧 Install Nexus Repository
```bash
sudo apt update -y
sudo apt install openjdk-8-jdk -y
cd /opt
sudo wget https://sonatype-download.global.ssl.fastly.net/repository/downloads-prod-group/3/nexus-unix-x86-64-3.78.0-14.tar.gz
sudo tar -xvf nexus-*.tar.gz
sudo mv nexus-3.78.0-14 nexus
sudo /opt/nexus/bin/nexus start
```
![step1](steps/step1c.png)

---

## 📌 Step 2: Create SonarQube Token

- Log in to SonarQube
- Navigate to **My Account > Security**
- Generate a new token

![step2](steps/step2a.png)

---

## 📌 Step 3: Create Repository Secrets

Go to your GitHub Repo:
**Settings > Secrets and Variables > Actions > New Repository Secret**

Add the following:
| Key              | Value                            |
|------------------|----------------------------------|
| SONARQUBE_URL    | http://<SONARQUBE_HOST>:9000     |
| SONAR_TOKEN      | Your SonarQube token             |
| NEXUS_USER       | Nexus username                   |
| NEXUS_PASSWORD   | Nexus password or token          |
| TOMCAT_USER      | Tomcat username                  |
| TOMCAT_PASSWORD  | Tomcat password                  |
| TOMCAT_HOST      | Tomcat server IP:8080            |
| EMAIL_TO         | Email address to receive alerts  |

![step3](steps/step3a.png)
![step3](steps/step3a2.png)
![step3](steps/step3b.png)
![step3](steps/step3c.png)

---

## 📌 Step 4: Update `pom.xml`

- Add `distributionManagement` to deploy to Nexus:
```xml
<distributionManagement>
  <repository>
    <id>nexus</id>
    <url>http://<NEXUS_HOST>:8081/repository/maven-releases/</url>
  </repository>
</distributionManagement>
```

- Or use `maven-snapshots` for snapshot versions.

![step4](steps/step4a.png)

---

## 📌 Step 5: Create GitHub Actions Workflow

- File: `.github/workflows/cicd.yml`
- Define CI/CD pipeline: Checkout > Build > SonarQube > Nexus Upload > Tomcat Deploy > Notify (Optional)

![step5](steps/step5a.png)
![step5](steps/step5b.png)


Notify stage :

### 📧 Email Notification Example
```yaml
- name: Send Email
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 587
    username: your-email@gmail.com
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: GitHub Actions Build Failed
    to: ${{ secrets.EMAIL_TO }}
    from: GitHub CI/CD
    body: "Build failed for MyWebApp. Please check logs."
```

---

## 📌 Step 6: Verify the Stages

### ✅ GitHub Workflow Status
![step6](steps/step6a.png)

### ✅ SonarQube Quality Analysis
![step6](steps/step6b.png)

### ✅ Nexus Artifactory Upload
![step6](steps/step6c.png)

### ✅ Tomcat Web Deployment
Access your deployed app using:
```
http://<TOMCAT_HOST>:8080/MyWebApp
```
![step6](steps/step6d.png)

---

## ✅ Conclusion
This project showcases a full CI/CD integration pipeline using GitHub Actions with automated:
- Code Quality checks via SonarQube
- Artifact management via Nexus
- Application deployment to Tomcat server
- Optional Email notifications for pipeline status

You can extend this to use Docker/Kubernetes or add stages like Trivy scanning, notifications, or Helm deployments.
