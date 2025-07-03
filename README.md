Great! Let’s proceed with **Project 2: Build a WAR file using Maven and deploy it to Apache Tomcat using Jenkins (Freestyle job)**.

---

# ✅ **Project 2: WAR Build & Deploy to Tomcat using Jenkins**

### 🎯 **Goal:**

* Clone a Java web app project from GitHub
* Build a `.war` file using Maven
* Deploy that `.war` to Apache Tomcat automatically from Jenkins 

---

## 🧰 Tools Used:

* Jenkins
* Git
* Maven
* Apache Tomcat 8/9
* Java (JDK)

---

## 🔧 Step-by-Step Setup

---

### 🔹 **Step 1: Prepare Java WAR Maven Project**

If you don’t have a Java web project, use this structure:

```bash
mkdir -p jenkins-war-sample/src/main/webapp/WEB-INF
cd jenkins-war-sample
```

#### Create `App.java`:

```bash
mkdir -p src/main/java/com/demo
nano src/main/java/com/demo/App.java
```

```java
package com.demo;

public class App {
    public static void main(String[] args) {
        System.out.println("WAR build test");
    }
}
```

#### Create a JSP File:

```bash
echo '<h1>Hello from WAR</h1>' > src/main/webapp/index.jsp
```

#### Create `web.xml`:

```bash
nano src/main/webapp/WEB-INF/web.xml
```

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0">
  <display-name>jenkins-war-sample</display-name>
</web-app>
```

#### Create `pom.xml`:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.demo</groupId>
  <artifactId>jenkins-war-sample</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <build>
    <finalName>jenkins-sample</finalName>
  </build>
</project>
```

Push this repo to GitHub.

---

### 🔹 **Step 2: Install Jenkins Plugins**

Go to **Manage Jenkins → Plugin Manager**, install:

* **Maven Integration Plugin**
* **Deploy to Container Plugin**

---

### 🔹 **Step 3: Configure Tomcat for Remote Deployment**

1. Edit `tomcat-users.xml`:

```bash
sudo nano /opt/tomcat/conf/tomcat-users.xml
```

Add:

```xml
<role rolename="manager-script"/>
<user username="admin" password="admin123" roles="manager-script"/>
```

2. Edit:

```bash
sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
```

Comment out the IP restriction:

```xml
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve" ... /> -->
```

3. Restart Tomcat:

```bash
sudo systemctl restart tomcat
```

Test by visiting: `http://<server-ip>:8080/manager/html`
Login with `admin / admin123`

---

### 🔹 **Step 4: Create Jenkins Freestyle Job**

1. Go to Jenkins Dashboard → **New Item**
2. Name: `WAR-Build-and-Deploy`
3. Type: **Freestyle project**

---

### 🔹 **Step 5: Configure Jenkins Job**

#### 📁 **Source Code Management**

* Select Git → Paste repo URL

#### ⚙️ **Build Section**

* Add **Invoke top-level Maven targets**
* Goals:

```bash
clean package
```

* Maven version: `Maven3`

---

### 🔹 **Step 6: Add Post-build Action (Deploy to Tomcat)**

1. Click: **Add post-build action → Deploy war/ear to a container**
2. WAR/EAR files:

```
target/*.war
```

3. Container:

   * **Tomcat 8.x**
   * URL: `http://<your-ip>:8080`
   * Credentials:

     * Username: `admin`
     * Password: `admin123`

---

### 🔹 **Step 7: Run the Job**

Click **Build Now**.

Check Console Output:

```
Uploading: http://localhost:8080/manager/text/deploy?path=/jenkins-sample...
OK - Deployed application at context path [/jenkins-sample]
```

---

### 🔹 **Step 8: Verify the Deployment**

Open:

```
http://<server-ip>:8080/jenkins-sample
```

You should see:

```
Hello from WAR
```

---

## 📌 What You Learned:

* Maven WAR packaging in Jenkins
* How to deploy to Tomcat from Jenkins
* WAR path, credentials, plugin usage

---
Perfect! Let’s build the complete **Project 2: WAR Build & Tomcat Deploy** using a declarative `Jenkinsfile` and proper repo structure.

---


















# ✅ **Project 2: WAR Build & Deploy to Tomcat with Jenkinsfile**

### 🎯 Goal:

* Build a WAR file from a Maven web project
* Deploy it to a remote Apache Tomcat server using Jenkins Pipeline

---

## 📁 Project Folder Structure:

```
jenkins-war-sample/
├── Jenkinsfile
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/demo/App.java
        └── webapp/
            ├── index.jsp
            └── WEB-INF/
                └── web.xml
```

---

## 📜 Jenkinsfile (Declarative Pipeline):

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK11'
    }

    environment {
        DEPLOY_USER = 'admin'
        DEPLOY_PASS = 'admin123'
        DEPLOY_HOST = 'http://<tomcat-server-ip>:8080'
        DEPLOY_PATH = '/manager/text/deploy?path=/jenkins-war-sample&update=true'
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/<your-user>/jenkins-war-sample.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = 'target/jenkins-sample.war'
                    sh """
                        curl -u $DEPLOY_USER:$DEPLOY_PASS \
                        --upload-file $warFile \
                        "$DEPLOY_HOST$DEPLOY_PATH"
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ WAR deployed successfully to Tomcat.'
        }
        failure {
            echo '❌ Deployment failed. Check console output.'
        }
    }
}
```

---

## 🧾 `pom.xml` (WAR Project):

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.demo</groupId>
    <artifactId>jenkins-sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <build>
        <finalName>jenkins-sample</finalName>
    </build>
</project>
```

---

## 🧾 `App.java`:

```java
package com.demo;

public class App {
    public static void main(String[] args) {
        System.out.println("Web WAR app running.");
    }
}
```

---

## 🧾 `index.jsp`:

```jsp
<%@ page language="java" contentType="text/html" %>
<html>
<head><title>Jenkins WAR</title></head>
<body>
    <h1>Hello from Jenkins WAR project!</h1>
</body>
</html>
```

---

## 🧾 `web.xml` (required by WAR):

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0">
    <display-name>Jenkins WAR App</display-name>
</web-app>
```

---

## 🧪 Usage:

1. Push this folder to GitHub (`jenkins-war-sample`)
2. Create a Jenkins Pipeline job
3. Configure SCM to point to your repo
4. Trigger the build — the `.war` will be deployed to Tomcat at:

```
http://<tomcat-ip>:8080/jenkins-war-sample
```

---

