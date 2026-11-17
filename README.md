# Apache Maven Hands-On Practice | Java Spring Boot & AWS RDS

![Maven](https://img.shields.io/badge/Apache_Maven-C71A36?logo=apachemaven&logoColor=white) ![Java](https://img.shields.io/badge/Java-17%20%7C%2011-orange) ![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20RDS-232F3E?logo=amazonaws) ![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?logo=mysql&logoColor=white)

## Overview

This repository documents my **hands-on practice with Apache Maven**, including Java project builds, dependency management, build artifacts, and running a Spring Boot student-management application on AWS infrastructure with an Amazon RDS MySQL database.

**Application reference:** [Java-springboot-project](https://github.com/VENKATASUBBARAO13/Java-springboot-project)  
**Practice repository:** [maven-practice](https://github.com/VENKATASUBBARAO13/maven-practice)

> **Scope:** This is a learning and deployment exercise using an existing Spring Boot application, not a claim that the application was developed from scratch for this Maven practice. The walkthrough provides reproducible setup instructions and distinguishes documented outcomes from commands that may vary by environment.

## Technologies

| Technology | Purpose |
|---|---|
| Apache Maven | Java dependency management and build lifecycle |
| Java / JDK | Java compilation and application runtime |
| Spring Boot | Existing student management backend |
| Amazon EC2 | Linux host for build and application execution |
| Amazon RDS (MySQL) | Relational database |
| Git / GitHub | Source code and practice documentation |

## Important: Two Maven configurations

The **`maven-practice/pom.xml`** is a separate Maven exercise with `com.tcs:project:0.0.1-SNAPSHOT`, compiler release **17**, TestNG, MySQL Connector/J, Surefire, JAR and Sonar Maven plugins. Its manifest references `com.project.App`.

The **source application's `backend/pom.xml`** uses **Spring Boot 2.7.17**, `com.datastore:datastore:0.0.7`, Java **11**, Spring Web, Spring Data JPA, MySQL, and the Spring Boot Maven plugin. **Use the Spring Boot backend's POM to build and run the web application.** Do not assume that the standalone practice POM produces a runnable Spring Boot JAR.

## Step-by-step walkthrough

### Step 1 — Prepare an Amazon RDS MySQL database

1. In AWS Console, open **RDS → Databases → Create database**.
2. Choose **MySQL** and an instance size appropriate for your lab.
3. Set a database name, administrator username, and password. Store credentials securely.
4. Configure VPC, subnets, and a security group. Allow **TCP 3306 from the EC2 application's security group**, not from the entire internet.
5. Wait until the DB status is **Available** and record the endpoint.

**Verification:** The practice used an Amazon RDS MySQL database named `database-1`; exact configuration may differ in a fresh setup.

### Step 2 — Connect to an EC2 Linux environment

Launch or use an EC2 instance with network access to the RDS database. Connect through your approved SSH or browser terminal method. Install the required JDK, Git, and Maven for the project you intend to build.

Check the tools:

```bash
java -version
mvn -version
git --version
```

> For the Spring Boot application, use a compatible JDK (the source POM targets Java 11). The separate Maven practice POM targets Java 17.

### Step 3 — Clone the existing Spring Boot application

```bash
git clone https://github.com/VENKATASUBBARAO13/Java-springboot-project.git
cd Java-springboot-project
ls
cd backend
ls
```

The upstream repository contains `backend/`, `frontend/`, `compose/`, and CI/CD-related files. The Maven-based Spring Boot application lives in **`backend/`**.

### Step 4 — Inspect the Maven project and build structure

```bash
cat pom.xml
find src -maxdepth 4 -type f | head -40
```

The source POM declares Spring Boot dependencies, the MySQL driver, test dependencies, and the Spring Boot Maven plugin. Maven generates compiled classes and packaged artifacts in `target/`.

**Verification:** Inspect the generated build files to understand Maven's compilation output.

### Step 5 — Examine compiled output

After a successful compilation or build, inspect the `target` folder:

```bash
find target -maxdepth 5 -type f | head -60
```

**Verification:** The `target/` directory contains Maven-generated compiled output.

### Step 6 — Build the Spring Boot backend with Maven

Run the following **from `Java-springboot-project/backend`**:

```bash
mvn clean package
```

`clean` removes prior build output, while `package` compiles, tests, and packages the application. Do not skip tests unless there is a documented reason.

Inspect the results:

```bash
ls -lh target/
```

**Verification:** Check the Maven console for `BUILD SUCCESS` to verify that packaging completed.

### Step 7 — Configure database connectivity and start the application

The application needs a reachable MySQL database. Supply configuration through environment variables or the application's supported Spring configuration; avoid committing credentials to GitHub.

An **example** Spring Boot environment configuration is:

```bash
export SPRING_DATASOURCE_URL='jdbc:mysql://YOUR_RDS_ENDPOINT:3306/YOUR_DATABASE'
export SPRING_DATASOURCE_USERNAME='YOUR_DB_USER'
read -rs -p 'Database password: ' SPRING_DATASOURCE_PASSWORD; echo
export SPRING_DATASOURCE_PASSWORD
```

Run the packaged Spring Boot JAR, selecting the actual file produced by the build:

```bash
ls target/*.jar
java -jar target/datastore-0.0.7.jar
```

The JAR name above follows the source POM's artifact and version; **confirm the actual filename** with `ls` first. The source application's settings or deployment scripts may require additional environment variables.

**Verification:** The tested frontend ran on Streamlit port **8501**; the Spring Boot backend is a separate service and may run on a different port.

### Step 8 — Open the web application

For the pictured frontend running on Streamlit's port 8501, open:

```text
http://YOUR_EC2_PUBLIC_IP:8501
```

Your Spring Boot backend may run separately (commonly on port 8080); check its actual configuration and logs. Configure the EC2 security group to allow only the required application ports from appropriate trusted sources.

**Verification:** Open the student-management frontend in a browser and check that it loads correctly.

### Step 9 — Test student registration

1. Open **Add New Student**.
2. Enter a student name and age.
3. Submit the form.
4. Check for the application's confirmation message.

**Verification:** Verify that the registration form returns a success indication after submission.

### Step 10 — Verify the student directory

1. Open **View All Students**.
2. Confirm that the newly submitted record appears.
3. This demonstrates application-level write/read behavior; a direct SQL query can provide additional database verification.

**Verification:** Confirm that the student directory lists the newly registered record.

## Separate Maven fundamentals practice

This repository also includes its own `pom.xml`. To explore its lifecycle separately, run from the root of `maven-practice`:

```bash
mvn validate
mvn compile
mvn test
mvn package
mvn clean
```

These are **practice commands**, suggested for practicing the Maven lifecycle. Its JAR manifest specifies `com.project.App`, so a runnable JAR depends on that class being present and correctly compiled.

## Results demonstrated

- An available Amazon RDS MySQL database in AWS.
- Maven/Java build artifacts generated on a Linux host.
- A backend application supporting a browser-accessible student-management frontend.
- A student registration workflow and a populated student directory.

## What I learned

- Understanding Maven's `pom.xml`, dependency management, and build lifecycle.
- Distinguishing compiled output from packaged application artifacts.
- Building a Spring Boot backend with its own Maven configuration.
- Connecting an EC2-hosted application to an RDS MySQL database.
- Validating application startup and basic create/read functionality.

## Troubleshooting tips

| Issue | What to check |
|---|---|
| `mvn: command not found` | Maven installation and PATH |
| Java version mismatch | JDK version and the relevant POM's compiler target |
| Maven build fails | Full error log, dependencies, and tests |
| Database connection refused | RDS endpoint, security groups, VPC routing, credentials |
| Browser cannot reach application | EC2 security group, process status, port, and bind address |
| JAR not found | Correct directory and filename under `target/` |

## References

- [Maven practice repository](https://github.com/VENKATASUBBARAO13/maven-practice)
- [Java Spring Boot application repository](https://github.com/VENKATASUBBARAO13/Java-springboot-project)
- [Spring Boot backend POM](https://github.com/VENKATASUBBARAO13/Java-springboot-project/blob/main/backend/pom.xml)
- [Apache Maven documentation](https://maven.apache.org/guides/)

---

*Hands-on learning documentation based on the Maven practice and Spring Boot repository configuration.*
