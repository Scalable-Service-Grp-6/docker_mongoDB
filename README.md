# MongoDB Local Deployment with Docker Compose

&#x20;

A step-by-step guide to install Docker, set up Docker Desktop, and spin up a local MongoDB instance using Docker Compose.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Installing Docker & Docker Desktop](#installing-docker--docker-desktop)

   * [Windows](#windows)
   * [macOS](#macos)
   * [Linux](#linux)
4. [Project Structure](#project-structure)
5. [Configure ](#configure-docker-composeyml)[`docker-compose.yml`](#configure-docker-composeyml)
6. [Launching MongoDB Container](#launching-mongodb-container)
7. [Verifying Your Setup](#verifying-your-setup)
8. [Useful Docker Commands](#useful-docker-commands)
9. [Troubleshooting](#troubleshooting)

---

## 🛠️ Overview

This README will guide you through:

* Installing Docker Engine and Docker Desktop
* Preparing a `docker-compose.yml` to run MongoDB locally
* Starting and verifying your MongoDB container on a custom port (e.g., `8000`)

By the end, you’ll have a fully functional local MongoDB server for development and testing.

---

## 📦 Prerequisites

Before you begin, ensure you have:

* A **64-bit** operating system (Windows 10/11, macOS 10.15+, or a modern Linux distro)
* **4 GB+ RAM** (6 GB+ recommended)
* **Basic command-line** familiarity (Terminal / PowerShell)

---

## 🚀 Installing Docker & Docker Desktop

### Windows

1. Visit the [Docker Desktop for Windows](https://docs.docker.com/desktop/windows/install/) page.
2. Download the **Windows Installer** (Stable channel).
3. Run the installer `.exe` and follow the wizard.
4. After installation, launch **Docker Desktop** from the Start menu. Wait until the **Docker whale** icon is stable in the system tray.

### macOS

1. Go to [Docker Desktop for Mac](https://docs.docker.com/desktop/mac/install/).
2. Download the **Mac with Apple chip** or **Mac with Intel chip** `.dmg` as appropriate.
3. Open the `.dmg`, drag **Docker** to your **Applications** folder.
4. Launch **Docker** from the Applications directory. Accept any security prompts.

### Linux (Ubuntu example)

```bash
# 1) Update packages
sudo apt-get update

# 2) Install prerequisites
sudo apt-get install \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# 3) Add Docker’s official GPG key
distrib="$(lsb_release -is | tr '[:upper:]' '[:lower:]')" && \
codename="$(lsb_release -cs)" && \
mkdir -p /etc/apt/keyrings && \
curl -fsSL https://download.docker.com/linux/$distrib/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4) Add Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/$distrib \
    $codename stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5) Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 6) Verify
docker --version
docker-compose version
```

> **Tip:** On Linux you might need to `sudo usermod -aG docker $USER` then **logout/login** to run Docker without `sudo`.

---

## 📁 Project Structure

Your folder layout should look like this:

```
project-root/
├── mongodb/                  # Docker Compose for MongoDB
│   └── docker-compose.yml
└── (other project files)
```

In the **mongodb/** directory you will define your `docker-compose.yml` for MongoDB.

---

## ⚙️ Configure `docker-compose.yml`

Create a file at `mongodb/docker-compose.yml`:

```yaml
version: '3.8'

services:
  mongo:
    image: mongo:8.0
    container_name: ss_grp6_local-mongo

    ports:
      - "8000:27017"    # Host port 8000 → Container port 27017

    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
      MONGO_INITDB_DATABASE: movie_booking_db

    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data: {}
```

**What each line does:**

* `version: '3.8'`: Compose file format version.
* `services.mongo.image`: Use MongoDB 8.0 image.
* `container_name`: Friendly name for your Docker container.
* `ports`: Maps host port `8000` to container’s default `27017`.
* `environment`: Bootstraps an **admin/admin** user and creates `movie_booking_db`.
* `volumes`: Persists data in a named volume `mongo_data` under `/data/db`.

---

## ▶️ Launching MongoDB Container

1. Open a terminal and navigate to `mongodb/`:

   ```bash
   cd mongodb
   ```
2. Start the container in detached mode:

   ```bash
   docker-compose up -d
   ```
3. Check status:

   ```bash
   docker-compose ps
   ```

   You should see `ss_grp6_local-mongo` running, listening on port **8000**.

---

## 🔍 Verifying Your Setup

* **Via Docker CLI**:

  ```bash
  docker ps    # lists running containers
  docker logs ss_grp6_local-mongo
  ```

* **Via MongoDB Compass**:

  * **Host:** `localhost`
  * **Port:** `8000`
  * **Authentication Database:** `admin`
  * **Username/Password:** `admin` / `admin`

### Connecting from MongoDB Compass

1. Open **MongoDB Compass**.
2. Enter **Hostname**: `localhost` and **Port**: `8000`.
3. Under **Authentication**, select **Username / Password**.

   * **Username:** `admin`
   * **Password:** `admin`
   * **Authentication Database:** `admin`
4. Click **Connect** to access the `movie_booking_db` database.

### Connecting from Node.js Application

In your Node.js project’s `.env` file, add or update the `MONGO_DB_URL` variable:

```dotenv
MONGO_DB_URL=mongodb://admin:admin@localhost:8000/movie_booking_db?authSource=admin
```

Then in your application code (e.g., `dbService.ts`), load the environment variable and connect:

```js
import mongoose from 'mongoose';
import dotenv from 'dotenv';

dotenv.config();

const uri = process.env.MONGO_DB_URL;
if (!uri) throw new Error('MONGO_DB_URL not defined');

mongoose.connect(uri)
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('DB connection error:', err));
```

---

## ⚙️ Useful Docker Commands

| Command                        | Description                                       |
| ------------------------------ | ------------------------------------------------- |
| `docker-compose up -d`         | Start services in detached mode                   |
| `docker-compose down`          | Stop and remove containers & networks             |
| `docker-compose logs -f mongo` | Follow MongoDB container logs                     |
| `docker system prune -a`       | Clean up unused images/containers (use carefully) |
| `docker volume prune`          | Remove unused volumes                             |

---

## 🛠️ Troubleshooting

* **Port already in use**: Change host mapping (`8000:27017`) to another free port.
* **Authentication failed**: Ensure you use `admin/admin` and `authSource=admin` when connecting.
* **Data not persisting**: Check your named volume with `docker volume ls` and inspect it.

---

*Happy coding & enjoy your local MongoDB!*
