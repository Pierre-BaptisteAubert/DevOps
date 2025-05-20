# DevOps - Pierre-Baptiste Aubert
## TP 19/05/2025

### 1-1 For which reason is it better to run the container with a flag `-e` to give the environment variables rather than put them directly in the Dockerfile?

It is better to use `-e` to pass environment variables at runtime rather than putting them in the Dockerfile, because it makes the image more **generic**, **reusable**, and **secure**.

### 1-2 Why do we need a volume to be attached to our postgres container?

We need to attach a volume to our Postgres container to **persist the database data** outside the container's filesystem.

### 1-3 Document your database container essentials: commands and Dockerfile.

**1. Create the flask-app with a Dockerfile and an `init` folder with the SQL query**

In the Dockerfile:
- Use the base image:
  ```dockerfile
  FROM postgres:17.2-alpine
  ```
- Add a command to copy all the SQL queries to:
  ```dockerfile
  /docker-entrypoint-initdb.d
  ```

**2. Build the image**
```bash
docker build -t pbaubert/myfirstdatabase .
```

**3. Create the network**
```bash
docker network create app-network
```

**4. Run the Postgres container**
```bash
docker run -p 8888:5000 --name my-postgres-container --network app-network \
-e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd \
-v 'C:\Users\utilisateur\Desktop\Pb\EPF\S7\DevOps\DevOps\TP1-19-05-2025\datapostgres:/var/lib/postgresql/data' \
pbaubert/myfirstdatabase
```

**5. Pull Adminer image**
```bash
docker pull adminer
```

**6. Run Adminer container**
```bash
docker run -p 8090:8080 --net=app-network --name=adminer -d adminer
```

---

### 1-4 Why do we need a multistage build? And explain each step of this Dockerfile.

A **multi-stage build** in Docker allows us to separate the **build environment** (where we compile and package the code) from the **runtime environment** (where we run the app). This keeps the final image lightweight and secure.

#### Dockerfile explained

```dockerfile
# --- Build stage ---
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
```

- Uses Java 21 JDK Alpine image
- Installs Maven
- Copies project source files
- Builds the project and skips tests

```dockerfile
# --- Run stage ---
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

- Uses Java 21 JRE Alpine image (smaller)
- Copies only the final `.jar` from the build stage
- Defines the command to run the application

---

### 1-5 Why do we need a reverse proxy?

A **reverse proxy** acts as an intermediary between the client and backend services. It:
- Simplifies access (e.g., from `http://localhost:8080/` to `http://localhost/`)
- Unifies port usage
- Helps manage and secure connections

---

### 1-6 Why is Docker Compose so important?

Docker Compose is important because it **simplifies multi-container orchestration** using:
- A single YAML file
- Easy lifecycle management (`up`, `down`, `restart`, etc.)

---

### 1-7 Document Docker Compose most important commands

```bash
docker-compose up             # Start all services
docker-compose up --build    # Build before starting
docker-compose down          # Stop and remove services
docker-compose build         # Build services without starting
docker-compose ps            # List running services
docker-compose logs          # Show logs
docker-compose exec <svc> sh # Run command in container
docker-compose restart       # Restart services
```

---

### 1-8 Document your docker-compose file

#### 🔹 Services

**1. database (PostgreSQL)**  
- Build path: `./Database`  
- Image: `postgresdb`  
- Container name: `postgresdb1`  
- Env variables: from `.env` in `Database/`  
- Volume: `datapostgres`  
- Network: `app-network`  
- Restart policy: `unless-stopped`

**2. backend (Spring Boot API)**  
- Build path: `./simpleapi`  
- Image: `springapp`  
- Container name: `springapi1`  
- Env variables: from `.env` in `simpleapi/`  
- Depends on: `database`  
- Networks: `app-network`, `app-network2`  
- Restart policy: `on-failure:3`

**3. httpd (Apache Server)**  
- Build path: `./http-server`  
- Image: `httpproxy`  
- Container name: `apacheserver1`  
- Depends on: `backend`  
- Ports: `80:80`  
- Network: `app-network2`  
- Restart policy: `no`

#### 🔹 Networks

- `app-network`: between database and backend  
- `app-network2`: between backend and httpd

#### 🔹 Volumes

- `datapostgres`: to persist PostgreSQL data

### 1-9 Document your publication commands and published images in DockerHub

#### 🔹 Commands to publish an image

```bash
# 1. Log in to DockerHub
docker login

# 2. Tag your local image with your DockerHub username and image name
docker tag springapp pierrebaptisteaubert/springapp:1.0 
docker tag postgresdb pierrebaptisteaubert/postgresdb:1.0
docker tag httpproxy pierrebaptisteaubert/httpproxy:1.0 

# 3. Push the image to DockerHub
docker push pierrebaptisteaubert/springapp:1.0
docker push pierrebaptisteaubert/postgresdb:1.0
docker push pierrebaptisteaubert/httpproxy:1.0 
```
---

### 1-10 Why do we put our images into an online repo?

We publish Docker images to an **online repository** (like DockerHub) to:

- **Share** images with collaborators or the public
- **Deploy** images easily on remote servers or cloud environments
- **Version** our builds and ensure consistency across environments
- **Access** images from anywhere with `docker pull`

It enables **portability**, **automation**, and **reproducibility** of our applications.


## TP 19/05/2025

### 2-1: What are testcontainers?

**Answer:**  
They simply are java libraries that allow you to run a bunch of docker containers while testing. 