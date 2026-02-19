# Complete Server Workflow Guide

A comprehensive guide for deploying and managing Python-based applications (specifically FastAPI) on a Linux VPS from a Windows environment. This workflow covers everything from initial connection to continuous background execution using `systemd`.

## Table of Contents
- [1. Connecting from Windows](#1-connecting-from-windows)
- [2. Server Directory Structure](#2-server-directory-structure)
- [3. Project Workspace Setup](#3-project-workspace-setup)
- [4. Deploying a New Repository](#4-deploying-a-new-repository)
- [5. Python Environment Preparation](#5-python-environment-preparation)
- [6. Manual Testing](#6-manual-testing)
- [7. Background Service Configuration (systemd)](#7-background-service-configuration-systemd)
- [8. Updating Existing Projects](#8-updating-existing-projects)
- [9. Monitoring and Logs](#9-monitoring-and-logs)
- [10. Managing Multiple Projects](#10-managing-multiple-projects)
- [11. Useful Commands Reference](#11-useful-commands-reference)

---

## 1. Connecting from Windows
To access your Linux VPS, use the built-in Windows Command Prompt.

1. Open **CMD** (`Win + R`, type `cmd`, Enter).
2. Connect via SSH:
   ```bash
   ssh root@SERVER_IP
   ```
3. Enter your password when prompted. Once logged in, you will see the `root@linux:~#` prompt.

## 2. Server Directory Structure
Linux follows a specific hierarchy. For security and organization, projects should reside in the `/home` directory rather than `/root`.

- `/home`: User folders (Recommended for projects)
- `/etc`: System configurations
- `/var`: Logs
- `/root`: Root user's private folder

Navigate to your user directory:
```bash
cd /home
ls          # Check available users
cd <username>
```

## 3. Project Workspace Setup
Create a dedicated directory to house all your applications.

```bash
mkdir projects
cd projects
pwd # Should return /home/<username>/projects
```

## 4. Deploying a New Repository
Clone your project directly from GitHub into your workspace.

```bash
git clone https://github.com/username/repository.git
cd repository-name
```

## 5. Python Environment Preparation
Isolate your project dependencies using a virtual environment.

1. **Install venv tool** (one-time setup):
   ```bash
   sudo apt install python3-venv -y
   ```
2. **Create and activate environment**:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

## 6. Manual Testing
Before setting up a background service, verify the application runs correctly.

```bash
uvicorn main:app --host 0.0.0.0 --port 8002
```
Access the documentation at `http://SERVER_IP:8002/docs`. If the Swagger UI appears, the application is functional. Use `CTRL + C` to stop the test server.

## 7. Background Service Configuration (systemd)
Use `systemd` to ensure your application runs permanently and restarts automatically on failure.

1. **Create a service file**:
   ```bash
   sudo nano /etc/systemd/system/project-name.service
   ```
2. **Add the following configuration**:
   ```ini
   [Unit]
   Description=FastAPI Application
   After=network.target

   [Service]
   User=<username>
   WorkingDirectory=/home/<username>/projects/project-folder
   ExecStart=/home/<username>/projects/project-folder/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8002
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
3. **Enable and start the service**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start project-name
   sudo systemctl enable project-name
   ```
4. **Check status**:
   ```bash
   sudo systemctl status project-name
   ```

## 8. Updating Existing Projects
When code changes are pushed to GitHub, follow these steps to update the server:

```bash
cd /home/<username>/projects/project-folder
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart project-name
```

## 9. Monitoring and Logs
View real-time application logs for debugging:
```bash
journalctl -u project-name -f
```

## 10. Managing Multiple Projects
Each project should be isolated to avoid conflicts:
- **Separate Folders**: `/home/user/projects/app1`, `/home/user/projects/app2`
- **Separate Virtual Environments**: Each folder has its own `venv/`
- **Unique Ports**: Assign different ports (e.g., 8000, 8001, 8002)
- **Unique Services**: Create individual `.service` files in `/etc/systemd/system/`

## 11. Useful Commands Reference
| Command | Description |
|---------|-------------|
| `systemctl list-units --type=service` | List all running services |
| `sudo systemctl restart <name>` | Restart a specific service |
| `ss -tulnp` | Check all open ports and listening processes |
| `pwd` | Print current working directory |
| `ls -la` | List all files including hidden ones |

---
**Summary Flow**: SSH Login → Navigate to Workspace → Git Pull/Clone → Update Venv → Restart Systemd Service.
