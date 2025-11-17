# sample
Nice — I’ll give you **complete, copy-pasteable, step-by-step instructions** for all three projects: Portfolio (Jenkins pipeline + webhook deploy), Travel Agency (Docker automation), and Blog (Maven + Java). Each project includes:

* minimal working files (HTML/CSS/Java)
* Git / GitHub steps
* Jenkinsfile(s) / pipeline examples
* Dockerfile / Docker commands where needed
* webhook configuration steps
* commands to verify the site is running

If you want me to adapt any pipeline to a specific host/port, tell me the port. For now I’ll show common ports and note where to change them.

---

# 1) Portfolio Website — Deploy using Jenkins Pipeline (static 2-page site)

### Goal

a) Two-page HTML/CSS site
b) Push to GitHub
c) Jenkins pipeline builds & deploys automatically (using Docker/nginx)
d) Configure GitHub webhook to trigger the pipeline on push

---

### A — Local site files (2 pages)

Create a folder `portfolio-site/` with:

`index.html`

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Varad — Portfolio</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <header><h1>Varad</h1><nav><a href="index.html">Home</a> | <a href="about.html">About</a></nav></header>
  <main>
    <section>
      <h2>Welcome</h2>
      <p>Welcome to my portfolio. I build things with HTML, CSS and more.</p>
    </section>
    <section>
      <h3>Projects</h3>
      <ul>
        <li>Project A — small description</li>
        <li>Project B — small description</li>
      </ul>
    </section>
  </main>
  <footer>© 2025 Varad</footer>
</body>
</html>
```

`about.html`

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>About — Varad</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <header><h1>About Me</h1><nav><a href="index.html">Home</a> | <a href="about.html">About</a></nav></header>
  <main>
    <h2>About</h2>
    <p>I am a developer who likes building websites and automations.</p>
    <h3>Contact</h3>
    <p>Email: varad@example.com</p>
  </main>
  <footer>© 2025 Varad</footer>
</body>
</html>
```

`styles.css`

```css
*{box-sizing:border-box;margin:0;padding:0;font-family:Arial,Helvetica,sans-serif}
body{line-height:1.6;color:#222;padding:1rem;background:#f5f7fa}
header{background:#2b6cb0;color:#fff;padding:1rem;border-radius:6px;margin-bottom:1rem}
nav a{color: #fff;text-decoration:none;margin-left:.5rem}
main{background:#fff;padding:1rem;border-radius:6px;box-shadow:0 2px 6px rgba(0,0,0,.06)}
footer{text-align:center;margin-top:1rem;color:#666}
```

---

### B — Git initialization & push to GitHub

1. Create repo on GitHub (e.g. `varad-portfolio`).
2. Locally:

```bash
cd portfolio-site
git init
git add .
git commit -m "Initial portfolio site"
git branch -M main
git remote add origin https://github.com/<your-username>/varad-portfolio.git
git push -u origin main
```

---

### C — Jenkins Pipeline (build & deploy)

Goal: Jenkins clones repo, builds a Docker image that serves the static site with nginx and runs container.

**Jenkins prerequisites**

* Jenkins installed and reachable from GitHub (public URL or accessible using tunneling)
* Plugins: Pipeline, Git, Docker Pipeline (or Docker plugin), GitHub Integration (or GitHub Branch Source)
* Docker installed on Jenkins agent (or use Docker-in-Docker)

**Add Credentials (if private repo)**

* In Jenkins → Credentials → add GitHub username/password or better: SSH key or personal access token.

**Jenkinsfile (Declarative)** — place in repo root as `Jenkinsfile`:

```groovy
pipeline {
  agent any
  environment {
    IMAGE = "varad-portfolio:${env.BUILD_ID}"
    CONTAINER_NAME = "varad-portfolio"
    PORT = "9090" // change to your host port if you want
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.build(env.IMAGE)
        }
      }
    }
    stage('Stop/Remove Old Container') {
      steps {
        sh "docker rm -f ${env.CONTAINER_NAME} || true"
      }
    }
    stage('Run Container') {
      steps {
        sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.PORT}:80 ${env.IMAGE}"
      }
    }
  }
  post {
    success { echo "Deployed on port ${env.PORT}" }
    failure { echo "Build or deploy failed" }
  }
}
```

**Dockerfile** (in same repo root)

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
# optional: expose 80
EXPOSE 80
```

**Notes**

* This pipeline builds a Docker image (nginx serving static files) and runs it mapping host port `9090` to container 80. Modify `PORT` in Jenkinsfile as needed.
* Ensure Jenkins agent can run Docker commands (jenkins user in docker group or use Docker socket with appropriate security).

---

### D — Configure GitHub webhook to trigger Jenkins

1. In GitHub repo → Settings → Webhooks → Add webhook.
2. **Payload URL**: `http://<your-jenkins-domain>/github-webhook/`
   (If Jenkins is behind auth or different path, use the GitHub plugin recommended URL.)
3. **Content type**: `application/json`
4. **Secret**: optional — if used, set same in Jenkins job config
5. **Which events?**: choose `Just the push event`
6. Save.

**Jenkins job setup**:

* Create a Pipeline job (or Multibranch Pipeline)
* In job configuration, select "GitHub hook trigger for GITScm polling" (for classic Pipeline). For Multibranch, GitHub Branch Source plugin will manage webhooks automatically.
* Point Pipeline SCM to your GitHub repo or leave Jenkinsfile in the repo (recommended).
* After enabling webhook, a push → GitHub will send payload → Jenkins will trigger pipeline.

**Quick test**

* Make a small change locally, `git commit -am "test deploy"` and `git push`.
* Check GitHub webhook deliveries (repo Settings → Webhooks → recent deliveries) for 200 responses.
* Check Jenkins build console.

---

# 2) Travel Agency Website — Docker Automation (static 2-page site)

### Goal

a) Two-page static site
b) Push to GitHub
c) Build Docker image from Dockerfile (in repo)
d) Run Docker container, verify site

---

### A — Files (2 pages)

Folder `travel-site/`

`index.html`

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"/><meta name="viewport" content="width=device-width,initial-scale=1"/><title>Travel Agency</title><link rel="stylesheet" href="styles.css"/></head>
<body>
<header><h1>Sunrise Travel</h1><nav><a href="index.html">Home</a> | <a href="destinations.html">Destinations</a></nav></header>
<main>
  <h2>Explore the world with Sunrise Travel</h2>
  <p>Best packages. Personalized itineraries. 24/7 support.</p>
</main>
<footer>Contact: contact@sunrisetravel.example</footer>
</body>
</html>
```

`destinations.html` (same layout, list of destinations)

`styles.css` - simple CSS similar to the portfolio.

---

### B — Push to GitHub

Same steps as portfolio:

```bash
cd travel-site
git init
git add .
git commit -m "Initial travel site"
git branch -M main
git remote add origin https://github.com/<user>/travel-site.git
git push -u origin main
```

---

### C — Dockerfile (build image from GitHub source)

`Dockerfile` (static site served by nginx):

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```

**Build image locally**

```bash
docker build -t sunrise-travel:1.0 .
```

**Build image from GitHub with Docker (on a machine that has Git)**

```bash
# clone then build
git clone https://github.com/<user>/travel-site.git
cd travel-site
docker build -t sunrise-travel:1.0 .
```

**Or — build directly using Docker remote build (docker client needs to clone first)**

---

### D — Run container & verify

Run:

```bash
docker run -d --name sunrise -p 8080:80 sunrise-travel:1.0
```

Open `http://localhost:8080` and verify pages. If you want port 9090, map `-p 9090:80`.

To stop/remove:

```bash
docker rm -f sunrise
```

---

### Optional: Automate build in GitHub Actions or Jenkins

If you want automatic Docker image build & push on GitHub push:

* Use GitHub Actions to build and push to Docker Hub/GitHub Container Registry.
* Or use Jenkins pipeline that checks out repo, runs `docker build`, `docker push` and restarts the container.

Example minimal Jenkinsfile for this repo:

```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build Image') { steps { sh 'docker build -t yourdockerhub/sunrise-travel:${BUILD_ID} .' } }
    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId:'docker-hub', usernameVariable:'USER', passwordVariable:'PW')]) {
          sh 'echo $PW | docker login -u $USER --password-stdin'
          sh 'docker push yourdockerhub/sunrise-travel:${BUILD_ID}'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'docker rm -f sunrise || true'
        sh 'docker run -d --name sunrise -p 8080:80 yourdockerhub/sunrise-travel:${BUILD_ID}'
      }
    }
  }
}
```

---

# 3) Blog Website using Maven Build Tool (HTML, CSS, Java)

### Goal

a) Two-page blog web app (HTML/CSS + Java)
b) Push to GitHub
c) Configure Maven for build automation & dependencies
d) Use Maven to build deployable artifact

I'll give a simple Spring Boot app (executable JAR) — minimal and standard for Java web apps. This is easier to run (no external servlet container required).

---

### A — Project structure (Spring Boot)

Use `spring-boot` initializer or manual pom. Create project `blog-app`.

Directory layout:

```
blog-app/
 ├─ src/
 │  ├─ main/
 │  │  ├─ java/com/example/blog/BlogApplication.java
 │  │  └─ java/com/example/blog/controller/HomeController.java
 │  └─ resources/
 │      ├─ templates/index.html
 │      └─ templates/post.html
 └─ pom.xml
```

`pom.xml` (minimal)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>blog-app</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.0</version> <!-- adjust to a stable version -->
    <relativePath/>
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

`BlogApplication.java`

```java
package com.example.blog;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BlogApplication {
  public static void main(String[] args) { SpringApplication.run(BlogApplication.class, args); }
}
```

`HomeController.java`

```java
package com.example.blog.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
  @GetMapping("/")
  public String index(Model m) {
    m.addAttribute("title","Home");
    return "index";
  }
  @GetMapping("/post")
  public String post(Model m) {
    m.addAttribute("title","Blog Post");
    return "post";
  }
}
```

`src/main/resources/templates/index.html`

```html
<!doctype html>
<html xmlns:th="http://www.thymeleaf.org">
<head><meta charset="utf-8"/><title>Blog - Home</title></head>
<body>
  <header><h1 th:text="${title}">Home</h1><nav><a th:href="@{/}">Home</a> | <a th:href="@{/post}">Post</a></nav></header>
  <main>
    <h2>Latest Posts</h2>
    <article>
      <h3>My First Post</h3>
      <p>This is a simple example blog built with Spring Boot.</p>
      <a th:href="@{/post}">Read more</a>
    </article>
  </main>
</body>
</html>
```

`src/main/resources/templates/post.html` — similar with content.

---

### B — Push to GitHub

```bash
cd blog-app
git init
git add .
git commit -m "Initial Spring Boot blog app"
git branch -M main
git remote add origin https://github.com/<user>/blog-app.git
git push -u origin main
```

---

### C — Configure Maven (already via `pom.xml`)

Maven handles dependencies (spring-boot-starter-web, thymeleaf). Ensure you have JDK 17+ and Maven installed.

Important `spring-boot-maven-plugin` in `pom.xml` will create an executable jar.

---

### D — Build and run with Maven

From project root:

**Build**

```bash
mvn clean package
```

Successful build creates `target/blog-app-0.0.1-SNAPSHOT.jar`.

**Run**

```bash
java -jar target/blog-app-0.0.1-SNAPSHOT.jar
```

By default Spring Boot runs on port `8080`. Open `http://localhost:8080`.

**Dockerize the Spring Boot app (optional)**
Add `Dockerfile`:

```dockerfile
FROM eclipse-temurin:17-jre
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8080
```

Build and run:

```bash
mvn clean package
docker build -t blog-app:1.0 .
docker run -d --name blog -p 8080:8080 blog-app:1.0
```

---

### Jenkins pipeline for Maven project

Place a `Jenkinsfile` in repo:

```groovy
pipeline {
  agent any
  tools { maven 'M3' jdk 'jdk17' } // must configure tools in Jenkins global config
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') { steps { sh 'mvn -B clean package' } }
    stage('Test') { steps { sh 'mvn -B test' } }
    stage('Deploy (Run Jar)') {
      steps {
        sh 'pkill -f blog-app || true' // optional: stop old process
        sh 'nohup java -jar target/*.jar > app.log 2>&1 &'
      }
    }
  }
  post {
    success { echo 'App built and started.' }
    failure { echo 'Build failed.' }
  }
}
```

Notes:

* In Jenkins, configure `M3` Maven installation and `jdk17` JDK installation names.
* If you prefer containerized deploy, add Docker build/push steps and run container as in previous Jenkinsfiles.

---

# Verification and troubleshooting

* **Confirm port mapping**: if container maps `-p 9090:80`, open `http://localhost:9090`.
* **If Jenkins cannot run docker**: ensure Jenkins user has docker permissions or use a Docker-enabled agent.
* **If webhook not triggering**: check GitHub webhook delivery logs and Jenkins access logs (403/401 often indicate authentication or URL issue).
* **If build fails due to Java version**: install matching JDK on Jenkins/build host (Spring Boot 3 requires Java 17+).

---

# Quick checklist (copy & paste)

### For each static site (portfolio/travel)

1. Put `index.html`, second page, `styles.css`, `Dockerfile`, and `Jenkinsfile` in repo root.
2. `git add .` → `git commit` → `git push`.
3. Configure Jenkins Pipeline job to use repo (or Multibranch).
4. Add GitHub webhook to `http://<jenkins>/github-webhook/`.
5. Push a commit — confirm Jenkins build and container start.
6. Browse `http://<jenkins-host>:<port>` (or host where container runs).

### For blog (Maven)

1. Create Spring Boot project files and `pom.xml`.
2. `mvn clean package` locally to test.
3. Push to GitHub.
4. Create Jenkins pipeline job with Maven tool configured.
5. Configure webhook and build.
6. After build, visit `http://<server>:8080`.

---

If you want, I can:

* produce full repo trees and `curl` test commands,
* generate GitHub Actions workflows instead of Jenkins pipelines,
* adapt Jenkinsfiles for your actual Jenkins setup (agent labels, credentials IDs, Docker host),
* change the deployment port to `9090` (I used 9090 in examples where noted).

Tell me which one to tailor first and I’ll give the exact, final files adjusted to your environment (no waiting).
