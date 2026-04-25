# How to Install Redis® on Ubuntu 26.04

**Updated on 25 April, 2026**  
Learn how to install Redis on Ubuntu 26.04 with step-by-step instructions. This guide covers installation methods, configuration, and basic Redis commands.

---

## Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Install Redis® on Ubuntu 26.04](#install-redis-on-ubuntu-2604)
- [Configure Redis® on Ubuntu 26.04](#configure-redis-on-ubuntu-2604)
- [Secure Redis®](#secure-redis)
- [Access the Redis® Server](#access-the-redis-server)
- [Conclusion](#conclusion)

---

## Introduction

Redis® (Remote Dictionary Server) is an open-source, in-memory key-value data store that works as a database, cache, or message broker. It integrates with modern web applications to improve performance and reduce server load by storing repeated queries, such as database queries, in memory. On Ubuntu 26.04, Redis® benefits from the system's stability, security updates, and package management, making it a reliable choice for high-performance caching and real-time data processing.

This article explains how to install Redis® on Ubuntu 26.04 and access the database to integrate it with other applications on your server. 

---

## Prerequisites

Before you begin:
- Deploy an Ubuntu 26.04 server on Vultr (or your preferred cloud provider)
- Access the server using SSH as a non-root user with sudo privileges

---

## Install Redis® on Ubuntu 26.04

Redis® is available in the default package repositories on Ubuntu 26.04, but for the latest version and enhanced security features, we recommend installing from the official Redis repository. Follow the steps below to install the latest Redis® package on your server.

### Step 1: Update Package Indexes

Update the server's package index and upgrade existing packages.

```bash
$ sudo apt update
$ sudo apt upgrade -y
```

### Step 2: Install Prerequisites

Install `lsb-release`, `curl`, and `gpg` which are needed for adding the Redis repository.

```bash
$ sudo apt-get install lsb-release curl gpg
```

### Step 3: Add the Redis GPG Key

Download and import the official Redis GPG key to verify packages from the Redis repository.

```bash
$ curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
```

Set appropriate permissions on the keyring file:

```bash
$ sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
```

### Step 4: Add the Redis Repository

Add the Redis repository to your APT sources list. This command dynamically adds the correct Ubuntu codename (e.g., `Resolute Raccoon` for 26.04).

```bash
$ echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
```

### Step 5: Update Package Index and Install Redis

Update the package index to include packages from the new Redis repository, then install Redis.

```bash
$ sudo apt-get update
$ sudo apt install redis
```

### Step 6: Enable and Start the Redis Service

Enable the Redis server to start automatically at boot time and start it immediately.

```bash
$ sudo systemctl enable redis-server
$ sudo systemctl start redis-server
```

Verify that the service is running:

```bash
$ sudo systemctl status redis-server
```

**Output:**
```
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; preset: enabled)
   Active: active (running) since Thu 2026-01-15 14:30:22 UTC
     Docs: http://redis.io/documentation, man:redis-server(1)
 Main PID: 24891 (redis-server)
   Status: "Ready to accept connections"
```

### Step 7: Verify the Installation

Check the Redis version installed on your server.

```bash
$ redis-server --version
```

**Output:**
```
Redis server v=8.6.2 sha=00000000:1 malloc=jemalloc-5.3.0 bits=64 build=72b111b4b7b0a7e7
```

---

## Configure Redis® on Ubuntu 26.04

Redis® listens for connection requests on the default localhost port **6379** on your server. In the following steps, configure Redis® to increase the default database limit and accept connections on your localhost `127.0.0.1` address.

### Step 1: Open the Configuration File

Open the main Redis® configuration file using a text editor such as nano or vim.

```bash
$ sudo nano /etc/redis/redis.conf
```

### Step 2: Review Key Settings

The default configuration is well-suited for most use cases, but let's verify and adjust a few key settings:

- **Network Binding** — Verify that Redis binds to `127.0.0.1` and `-::1` (IPv6 loopback):
  ```ini
  bind 127.0.0.1 -::1
  ```

- **Port** — Ensure the default port is set:
  ```ini
  port 6379
  ```

- **Protected Mode** — Protected mode is enabled by default, which restricts access to local connections only when no password is configured. This is a good security practice for development and production environments alike:
  ```ini
  protected-mode yes
  ```

- **Database Count** — The default number of databases has been increased from 16 in older Redis versions to accommodate modern workloads:
  ```ini
  databases 16
  ```

### Step 3: Save and Exit

Save the configuration file and exit the editor. If using `nano`, press `Ctrl+X`, then `Y` to confirm, then `Enter`.

---

## Secure Redis®

Redis® does not require authentication by default which enables unrestricted access for system users to available databases on the server. In the following steps, enable authentication to secure your Redis® server and only accept authorized user access.

### Step 1: Set a Password

Open the main Redis® configuration file again.

```bash
$ sudo nano /etc/redis/redis.conf
```

Find the `requirepass` directive near the end of the file **(around line 1080)**, uncomment it (remove the leading `#`), and replace `foobared` with a strong password of your choice:

```ini
requirepass YourStrongPassword123!
```

> **Security Note:** Choose a long, random password that is difficult to guess. Avoid using common words or predictable patterns.

### Step 2: Save and Restart

Save the file and restart the Redis® server to apply your configuration changes.

```bash
$ sudo systemctl restart redis-server
```

---

## Access the Redis® Server

The Redis® server accepts connection requests using the `redis-cli` utility or compatible application modules on your server. In the following steps, access the Redis® server and test access — write sample data to the default database to test your server configurations.

### Step 1: Connect to the Redis® Server

Connect to the Redis® server using the CLI tool.

```bash
$ redis-cli
```

**Output:**
```
127.0.0.1:6379> 
```

### Step 2: Test Access Without Authentication

Test access to the server without authentication — this should fail now that we've set a password.

```bash
127.0.0.1:6379> ping
```

**Output:**
```
(error) NOAUTH Authentication required.
```

### Step 3: Log in with Your Password

Log in to the Redis® server using your valid password. Replace `YourStrongPassword123!` with the actual password you set earlier.

```bash
127.0.0.1:6379> auth YourStrongPassword123!
```

**Output:**
```
OK
```

### Step 4: Test Access After Authentication

Test access to the database server again — this should now succeed.

```bash
127.0.0.1:6379> ping
```

**Output:**
```
PONG
```

### Step 5: Select a Database and Write Data

Select a database to use on your server. For example, select database `1`.

```bash
127.0.0.1:6379> SELECT 1
```

**Output:**
```
OK
```

Create a new sample key `testkey` with a value such as `Greetings from Vultr!`.

```bash
127.0.0.1:6379[1]> set testkey "Greetings from Vultr!"
```

**Output:**
```
OK
```

### Step 6: Query the Key Value

Query the key value from the database to verify your data was stored correctly.

```bash
127.0.0.1:6379[1]> get testkey
```

**Output:**
```
"Greetings from Vultr!"
```

### Step 7: Exit the Redis® CLI

Exit the Redis® CLI when you're done testing.

```bash
127.0.0.1:6379[1]> exit
```

---

## Conclusion

You have installed Redis® on your Ubuntu 26.04 server and configured it to accept secure connection requests with authentication enabled. You can integrate your Redis® server with your existing web applications to work as a database or cache to optimize your application and server performance. For more information and usage options, visit the [official Redis® documentation](https://redis.io/documentation).

