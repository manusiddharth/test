# Vehicle Rental Management System — Step-by-Step Solution Guide

Repository: `https://github.com/nikithamoturi/VehicleRentalManagementSystem.git`

---

## PART I – MAVEN WEB PROJECT DEVELOPMENT (40 Marks)

### Q1. Maven Project Setup and Build (20 Marks)

#### a) Clone and import into Eclipse as a Maven Web Project [4]

```bash
git clone https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
cd VehicleRentalManagementSystem
```

In Eclipse:
1. `File → Import → Maven → Existing Maven Projects`
2. Browse to the cloned folder, select the `pom.xml`, click **Finish**.
3. Eclipse imports it as a **Dynamic Web Project** (because `packaging` is `war`).

**Identify the JSP pages** — look under `src/main/webapp/`:

```bash
find src/main/webapp -name "*.jsp"
```

Typical pages for this project:
- `home.jsp` – landing page
- `login.jsp` – user login
- `register.jsp` – user registration
- `vehicle.jsp` – vehicle listing (name, type, rental price, availability)

#### b) Identify groupId, artifactId, version, packaging + 2 dependencies [4]

From `pom.xml`:

```xml
<groupId>KMIT</groupId>
<artifactId>VehicleRentalManagement</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>war</packaging>
```

| Field | Value |
|---|---|
| groupId | KMIT |
| artifactId | VehicleRentalManagement |
| version | 0.0.1-SNAPSHOT |
| packaging | war |

Two example dependencies and their purpose:

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>
```
> Purpose: gives access to `HttpServlet`, `HttpServletRequest/Response` classes needed to compile servlets. Scope `provided` because Tomcat already supplies this jar at runtime.

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.33</version>
</dependency>
```
> Purpose: JDBC driver used to connect the app to a MySQL database for storing users/vehicle data.

#### c) Execute Maven clean/build, fix pom.xml errors [7]

```bash
mvn clean
mvn clean package
```

**Common error & fix — missing Servlet API** (`cannot find symbol: class HttpServlet`):

```xml
<dependencies>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
  </dependency>
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

**Common error & fix — compiler release mismatch** (source/target 8 vs installed JDK):

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>8</maven.compiler.source>
  <maven.compiler.target>8</maven.compiler.target>
</properties>
```
If your local JDK is 17+, either install/point Maven to JDK 8, or bump these to match your JDK (e.g. `11`).

Re-run:

```bash
mvn clean install
```
Confirm `BUILD SUCCESS` and that `target/VehicleRentalManagement.war` is generated.

#### d) Verify build output, run on local server, check JSPs in browser [5]

```bash
ls target/*.war
```

Deploy to Tomcat:

```bash
cp target/VehicleRentalManagement.war $CATALINA_HOME/webapps/
$CATALINA_HOME/bin/startup.sh      # Linux/Mac
# or startup.bat on Windows
```

Access in browser:

```
http://localhost:8080/VehicleRentalManagement/home.jsp
http://localhost:8080/VehicleRentalManagement/login.jsp
http://localhost:8080/VehicleRentalManagement/register.jsp
http://localhost:8080/VehicleRentalManagement/vehicle.jsp
```

Verify each page loads with no HTTP 404/500 errors and confirm vehicle details (name, type, price, availability) render on `vehicle.jsp`.

---

### Q2. Maven Dependency and Build Troubleshooting (20 Marks)

#### a) Display dependency tree [4]

```bash
mvn dependency:tree
```

Sample output interpretation:
```
[INFO] KMIT:VehicleRentalManagement:war:0.0.1-SNAPSHOT
[INFO] +- javax.servlet:javax.servlet-api:jar:4.0.1:provided
[INFO] +- javax.servlet.jsp:javax.servlet.jsp-api:jar:2.3.3:provided
[INFO] +- javax.servlet:jstl:jar:1.2:compile
[INFO] +- mysql:mysql-connector-java:jar:8.0.33:compile
[INFO] \- junit:junit:jar:4.13.2:test
```

#### b) How Maven resolves dependencies / version conflicts [4]

- Maven reads `pom.xml`, resolves each `<dependency>` from the **local repo (`~/.m2`)** first; if absent, downloads from remote repos declared in `<repositories>` (default: Maven Central).
- Transitive dependencies (dependencies of your dependencies) are pulled in automatically and added to the dependency tree.
- **Conflict resolution:** if two dependencies require different versions of the same library, Maven applies the **"nearest wins"** rule — the version declared at the shallowest depth in the tree is used. If depths are equal, the **first declared** dependency in `pom.xml` wins.
- Conflicts can cause `NoSuchMethodError`/`ClassNotFoundException` at runtime if an incompatible version silently overrides the one your code was compiled against. Use `mvn dependency:tree` to detect this, and `<exclusions>` or explicit `<dependencyManagement>` versions to force a version.

#### c) Locate dependencies in local `.m2` repository [4]

```bash
# Default location
ls ~/.m2/repository/mysql/mysql-connector-java/8.0.33/
ls ~/.m2/repository/javax/servlet/javax.servlet-api/4.0.1/
```
Confirm the `.jar`, `.pom`, and checksum (`.sha1`) files exist for each dependency — this verifies a successful download.

#### d) Commands to clean, compile, package, troubleshoot [4]

```bash
mvn clean                 # removes target/ directory
mvn compile                # compiles source code only
mvn package                 # compiles + packages into WAR
mvn clean package -X        # -X = debug mode, verbose troubleshooting output
mvn clean package -e        # -e = show full error stack trace
mvn dependency:tree         # inspect dependency conflicts
mvn validate                 # checks project structure/pom correctness
```

When a build fails, read the first `[ERROR]` block (Maven stops at the root cause), check whether it's a **compilation error** (missing dependency/class), a **plugin/version error**, or a **test failure**, then fix `pom.xml` or source code accordingly and re-run `mvn clean package`.

#### e) Build succeeds but JSP doesn't open correctly — how to isolate the cause [4]

Checklist to isolate the layer:

1. **Maven build layer** — confirm `mvn clean package` ends with `BUILD SUCCESS` and the WAR contains the JSPs:
   ```bash
   jar tf target/VehicleRentalManagement.war | grep .jsp
   ```
   If JSPs are missing from the WAR → Maven/packaging issue (check `src/main/webapp` is correctly recognized as the web resources folder).

2. **Server configuration layer** — check Tomcat logs:
   ```bash
   tail -f $CATALINA_HOME/logs/catalina.out
   ```
   Look for deployment errors, port conflicts, or missing `web.xml`/context issues.

3. **Web application layer** — if the WAR deployed fine and Tomcat shows no errors but the page itself is broken (blank page, JSP compile error, 500 error), it's an application-level problem — check for:
   - JSP syntax errors (view the JSP error stack trace shown in the browser).
   - Broken relative paths to CSS/JS/images.
   - Missing scriptlet/bean/servlet-mapping in `web.xml`.

Conclusion approach: **Maven issue** → WAR missing files/won't build. **Server issue** → deployment/log errors, port/context problems. **Application issue** → WAR deploys and Tomcat is fine, but JSP itself throws an error or renders incorrectly.

---

## PART II – GIT & GITHUB INTEGRATION (40 Marks)

### Q3. Git Repository and Vehicle Page Modification (20 Marks)

#### a) Clone repo, verify with `git status` / `git remote -v` [4]

```bash
git clone https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
cd VehicleRentalManagementSystem
git status
git remote -v
```
Expected `git remote -v` output:
```
origin  https://github.com/nikithamoturi/VehicleRentalManagementSystem.git (fetch)
origin  https://github.com/nikithamoturi/VehicleRentalManagementSystem.git (push)
```

#### b) Create and check out a feature branch [4]

```bash
git checkout -b feature/vehicle-page-update
git branch          # confirms branch created and active (marked with *)
```

#### c) Modify `vehicle.jsp` to add Vehicle Type & Availability Status, verify with `git diff` [6]

Example addition inside the table/loop in `src/main/webapp/vehicle.jsp`:

```jsp
<table border="1">
  <tr>
    <th>Vehicle Name</th>
    <th>Vehicle Type</th>
    <th>Rental Price</th>
    <th>Availability Status</th>
  </tr>
  <c:forEach var="vehicle" items="${vehicleList}">
    <tr>
      <td>${vehicle.name}</td>
      <td>${vehicle.type}</td>
      <td>${vehicle.rentalPrice}</td>
      <td>
        <c:choose>
          <c:when test="${vehicle.available}">Available</c:when>
          <c:otherwise>Not Available</c:otherwise>
        </c:choose>
      </td>
    </tr>
  </c:forEach>
</table>
```

Verify changes:

```bash
git diff src/main/webapp/vehicle.jsp
```

#### d) Stage, commit, verify [6]

```bash
git add src/main/webapp/vehicle.jsp
git commit -m "Add Vehicle Type and Availability Status columns to vehicle.jsp"
git log -1
git show --stat HEAD
```

---

### Q4. Git Synchronization and Remote Repository (20 Marks)

#### a) Add a Rental Rate field / format change, verify with `git diff` [4]

```jsp
<th>Rental Rate (per day)</th>
...
<td>₹${vehicle.rentalRate}/day</td>
```

```bash
git diff src/main/webapp/vehicle.jsp
```

#### b) Pull latest changes from GitHub without overwriting local commits [4]

```bash
git fetch origin
git status                       # see if local branch is behind
git pull --rebase origin main    # replays local commits on top of latest remote commits
```
(`--rebase` avoids an unnecessary merge commit and keeps local commits on top instead of overwriting them.)

#### c) Compare feature branch with main [4]

```bash
git diff main feature/vehicle-page-update
# or file-scoped
git diff main feature/vehicle-page-update -- src/main/webapp/vehicle.jsp
git log main..feature/vehicle-page-update --oneline   # commits only on feature branch
```

#### d) Merge into main, push, verify on GitHub [8]

```bash
git checkout main
git pull origin main
git merge feature/vehicle-page-update
git push origin main
```

Verify remotely:

```bash
git log origin/main -1
```
Then open the repository on GitHub and confirm `vehicle.jsp` reflects the Vehicle Type, Availability Status, and Rental Rate changes.

---

## PART III – DOCKERIZATION AND CONTAINER MANAGEMENT (20 Marks)

### Q5. Dockerfile and Application Containerization (12 Marks)

#### a) Dockerfile with Tomcat base image [3]

```dockerfile
# Dockerfile
FROM tomcat:9.0-jdk8-openjdk
```

#### b) Build WAR and copy into Tomcat webapps [4]

```dockerfile
# --- Build stage ---
FROM maven:3.9-eclipse-temurin-8 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# --- Runtime stage ---
FROM tomcat:9.0-jdk8-openjdk
COPY --from=build /app/target/VehicleRentalManagement.war /usr/local/tomcat/webapps/
```

#### c) Expose port, start Tomcat [2]

```dockerfile
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

**Full Dockerfile:**

```dockerfile
FROM maven:3.9-eclipse-temurin-8 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM tomcat:9.0-jdk8-openjdk
COPY --from=build /app/target/VehicleRentalManagement.war /usr/local/tomcat/webapps/
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

#### d) Build the image, name/tag it, verify [3]

```bash
docker build -t vehiclerentalapp:1.0 .
docker images | grep vehiclerentalapp
```

---

### Q6. Run, Verify and Push Docker Image (8 Marks)

#### a) Run container mapping port 8080 → 8080 [3]

```bash
docker run -d --name vehiclerental-container -p 8080:8080 vehiclerentalapp:1.0
```

#### b) Verify container and access app in browser [2]

```bash
docker ps
docker logs -f vehiclerental-container
```
Browser:
```
http://localhost:8080/VehicleRentalManagement/home.jsp
```

#### c) Tag with Docker Hub username, login, push [2]

```bash
docker login
docker tag vehiclerentalapp:1.0 <your-dockerhub-username>/vehiclerentalapp:1.0
docker push <your-dockerhub-username>/vehiclerentalapp:1.0
```

#### d) Verify image on Docker Hub [1]

```bash
docker search <your-dockerhub-username>/vehiclerentalapp
```
Or log in to `https://hub.docker.com/repositories/<your-dockerhub-username>` and confirm `vehiclerentalapp` appears with tag `1.0`.

---

## Quick Command Reference

| Task | Command |
|---|---|
| Clone repo | `git clone <url>` |
| Maven clean build | `mvn clean package` |
| Dependency tree | `mvn dependency:tree` |
| New branch | `git checkout -b <branch>` |
| Diff changes | `git diff` |
| Stage & commit | `git add . && git commit -m "msg"` |
| Pull safely | `git pull --rebase origin main` |
| Compare branches | `git diff main <branch>` |
| Merge & push | `git merge <branch>` → `git push origin main` |
| Build Docker image | `docker build -t vehiclerentalapp:1.0 .` |
| Run container | `docker run -d -p 8080:8080 vehiclerentalapp:1.0` |
| Push to Docker Hub | `docker push <user>/vehiclerentalapp:1.0` |

> **Note:** Exact JSP filenames, entity/field names (`vehicleList`, `rentalRate`, etc.) should be adjusted to match whatever the actual cloned repository contains — clone it first and open `vehicle.jsp` to confirm the real variable/bean names before editing.
