# AWS_EC2_guide
Guide to deploy fastapi on AWS EC2
# FastAPI Deployment on AWS Ubuntu

## 1. Basic System Setup

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install python3-pip python3-venv nginx git curl -y
```

## 2. Setup Application Directory

```bash
mkdir fastapi-app
cd fastapi-app
python3 -m venv venv
source venv/bin/activate
pip install fastapi uvicorn
```

## 3. Clone or Copy Your App

Ensure your `main.py` has the following:

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

## 4. Running FastAPI with `nohup` or `tmux`

### Option 1: Run with `nohup`

```bash
nohup uvicorn main:app --host 0.0.0.0 --port 8000 > output.log 2>&1 &
```

### Fix for `text file busy` Error

```bash
ps aux | grep uvicorn
killall uvicorn  # or kill <PID>
```

Avoid editing `main.py` while the server is running.

### Option 2: Use `tmux` (Preferred)

```bash
sudo apt install tmux
tmux new -s fastapi
uvicorn main:app --host 0.0.0.0 --port 8000
# Detach: Ctrl + B, then D
# Reattach:
tmux attach -t fastapi
```

##  5. AWS Security Group Setup

Go to EC2 → Security Groups → Inbound Rules:

| Type        | Protocol | Port Range | Source     |
|-------------|----------|------------|------------|
| Custom TCP  | TCP      | 8000       | 0.0.0.0/0  |
| SSH         | TCP      | 22         | YOUR_IP/32 |

##  6. Nginx Reverse Proxy (Optional)

```bash
sudo apt install nginx
sudo nano /etc/nginx/sites-available/fastapi
```

```nginx
server {
    listen 80;
    server_name YOUR_PUBLIC_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

##  7. Auto Restart with `systemd`

```bash
sudo nano /etc/systemd/system/fastapi.service
```

```ini
[Unit]
Description=FastAPI app
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/fastapi-app
ExecStart=/home/ubuntu/fastapi-app/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
```

##  8. Kill All FastAPI Related Processes

```bash
killall uvicorn
# or
ps aux | grep uvicorn
kill <PID>
```

```bash
ps aux | grep nohup
kill <PID>
```

##  Checklist

-  FastAPI app in working directory
-  Uvicorn installed in virtualenv
-  Port 8000 open in AWS Security Group
-  SSH access restricted
-  Use `tmux` or `systemd` instead of `nohup`
-  Use Nginx for reverse proxy and optional SSL
