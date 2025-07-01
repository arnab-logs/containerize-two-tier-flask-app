# containerize-two-tier-flask-app
Flask + MySQL two-tier app containerized with Docker and connected using a custom network.
---

```markdown
# Two-Tier Flask App with MySQL using Docker

This project demonstrates a **two-tier application architecture**, where a Flask-based web server connects to a MySQL database. Both services are containerized using Docker and communicate through a shared user-defined bridge network.

Through this project, I explored and applied core containerization concepts including:
- Multi-container application design
- Dockerfile creation and image building
- Inter-container communication using Docker networks
- Runtime configuration via environment variables
- Manual container debugging and log analysis

---

## Tech Stack

- **Flask** – Python web framework (backend logic + UI)
- **MySQL** – Relational database for persistent storage
- **Docker** – Containerization tool for isolated environments
- **Docker Bridge Network** – Custom network for service discovery

---

## Project Structure

.
├── app.py                # Main Flask application
├── templates/            # HTML UI templates
│   └── index.html        # Web interface for input/output
├── requirements.txt      # Python dependencies
├── Dockerfile            # Docker config for building Flask app container
└── README.md             # Project documentation (this file)


````


## Running the Application with Docker

### Step 1: Clone the Repository

```bash
git clone https://github.com/arnab-logs/containerize-two-tier-flask-app.git
cd containerize-two-tier-flask-app
````

---

### Step 2: Create a Docker Bridge Network

This network allows the two containers (Flask + MySQL) to resolve each other by name.

```bash
docker network create two-tier
```

---

### Step 3: Run the MySQL Container

```bash
docker run -d \
  --name mysql \
  --network two-tier \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  mysql
```

---

### Step 4: Build the Flask App Image

```bash
docker build -t two-tier-backend .
```

---

### Step 5: Run the Flask App Container

```bash
docker run -d -p 5000:5000 \
  --name flask-backend \
  --network two-tier \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  two-tier-backend
```

---

### Step 6: Access the App

Open your browser and go to:

```
http://localhost:5000
```

* Submit messages via the form
* View them live as they’re stored in MySQL

![image](https://github.com/user-attachments/assets/e0aa2692-79d0-475a-8711-78571e25ffbf)


---

## 🔍 Verifying Data in MySQL

To manually inspect the database inside the MySQL container:

```bash
docker exec -it mysql bash
mysql -u root -p
# Enter password: root

USE devops;
SELECT * FROM messages;
```
![image](https://github.com/user-attachments/assets/1fe3ffc4-c354-43c9-b8ff-d23f5d87e756)

---

## Environment Variables Overview

| Variable              | Used In   | Purpose                                   |
| --------------------- | --------- | ----------------------------------------- |
| `MYSQL_HOST`          | Flask App | Hostname of the MySQL container (`mysql`) |
| `MYSQL_USER`          | Flask App | Username to connect to MySQL              |
| `MYSQL_PASSWORD`      | Flask App | Password for MySQL user                   |
| `MYSQL_DB`            | Flask App | Database name to connect to               |
| `MYSQL_ROOT_PASSWORD` | MySQL     | Root password used to initialize MySQL    |
| `MYSQL_DATABASE`      | MySQL     | Creates this DB on first container run    |

---

## Challenges I Faced & What I Learned


### Challenge 1: Flask Container Kept Exiting

#### What Happened

After building and running the Flask container, it would exit almost immediately.

#### How I Investigated

* Ran `docker ps -a` and saw the status as `Exited`.

* Checked logs using:

  ```bash
  docker logs <container_id>
  ```

* Found this error:

  ```
  MySQLdb.OperationalError: (2005, "Unknown server host 'mysql' (-2)")
  ```

#### What I Realized

The Flask container couldn’t resolve `mysql` because it was on a separate default network from the MySQL container.

#### How I Fixed It

* Created a custom bridge network with:

  ```bash
  docker network create two-tier
  ```

* Ran both containers using the `--network two-tier` flag.

Now Flask could resolve and connect to MySQL.

---

### Challenge 2: MySQL Connection Timing Issue

#### What Happened

Even with networking fixed, I saw this error:

```
MySQLdb.OperationalError: (2005, "Unknown server host 'mysql'")
```

#### What I Investigated

* Flask was starting **before** MySQL was fully initialized and ready to accept connections.

#### What I Learned

* **Container startup order matters.**
* Apps might crash if dependent services aren’t ready.

#### How I Fixed It

* Started MySQL container first
* Waited a few seconds
* Then started the Flask container

That allowed MySQL to be ready before Flask attempted a connection.

---

### Final Result

* Flask app successfully running on `http://localhost:5000`
* Messages persisted in MySQL container
* Verified using SQL inside the MySQL container shell

---

## Concepts Demonstrated

* ✅ Building and running multi-container Docker apps
* ✅ Environment-based app configuration
* ✅ Docker network management for inter-service communication
* ✅ Real-world container debugging with `docker logs` and `docker exec`
* ✅ Manual DB verification using MySQL CLI inside a container

---

## Cleanup Commands

Stop and remove containers:

```bash
docker stop flask-backend mysql
docker rm flask-backend mysql
```

Remove the custom network:

```bash
docker network rm two-tier
```

---

This project was created as part of my DevOps learning journey — to explore Docker networking, container orchestration, and real-world debugging workflows.

---
