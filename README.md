

# 🚀 3-Tier Flask Web Application on Azure

## 📌 Project Overview

This project demonstrates a **production-style 3-tier architecture** on Azure using:

* **Web Tier (Frontend - Flask)**
* **App Tier (Backend API - Flask)**
* **Data Tier (Azure SQL Database with Private Endpoint)**

---

## 🏗️ Architecture

![Architecture](./images/architecture.png)

```text
User → Web VM → App VM → Azure SQL Database
```

---

## ⚙️ Tech Stack

* Python Flask
* Azure Virtual Machines
* Azure SQL Database
* VNet, NSG, Private Endpoint

---

# 🔹 Frontend (Web VM - Flask)

👉 File: `web_server/app.py`

```python
from flask import Flask, render_template, request, redirect
import requests

app = Flask(__name__)

APP_URL = "http://10.0.2.4:5001"  # App VM private IP

@app.route("/", methods=["GET","POST"])
def login():
    if request.method == "POST":
        res = requests.post(APP_URL + "/api/login", json={
            "username": request.form["username"],
            "password": request.form["password"]
        })
        if res.status_code == 200:
            return redirect("/doom")
        return "Login Failed"
    return render_template("login.html")

@app.route("/doom")
def doom():
    res = requests.get(APP_URL + "/api/doom")
    return res.json()

app.run(host="0.0.0.0", port=5000)
```

---

# 🔹 Backend (App VM - Flask API)

👉 File: `app_server/app.py`

```python
from flask import Flask, request, jsonify
import pyodbc

app = Flask(__name__)

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=doomsqlserver.database.windows.net;"
    "DATABASE=doomdb;"
    "UID=doom;"
    "PWD=YourPassword;"
    "Encrypt=yes;"
)

@app.route("/api/health")
def health():
    return jsonify({"status": "ok"})

@app.route("/api/login", methods=["POST"])
def login():
    data = request.json
    cursor = conn.cursor()
    cursor.execute(
        "SELECT * FROM Users WHERE username=? AND password=?",
        (data["username"], data["password"])
    )
    if cursor.fetchone():
        return jsonify({"status": "success"})
    return jsonify({"status": "fail"}), 401

@app.route("/api/doom")
def doom():
    return jsonify({
        "name": "Doctor Doom",
        "real_name": "Victor Von Doom",
        "about": "Marvel villain with genius intellect and advanced armor"
    })

app.run(host="0.0.0.0", port=5001)
```

---

# 🔐 NSG Rules (Important)

### Web Tier

* Allow 5000 (Internet)
* Allow SSH (Admin IP)
* Allow outbound → App VM (5001)

### App Tier

* Allow inbound **5001 from Web subnet**
* Destination: **10.0.2.4**
* Allow outbound → SQL (1433)

---

# 🧪 Testing

### Backend test (App VM)

```bash
curl http://localhost:5001/api/health
```

### Web → App test

```bash
curl http://10.0.2.4:5001/api/health
```

---

# 🌐 Access Application

```text
http://<Web-VM-Public-IP>:5000
```

Login:

```
username: doom
password: 123
```

---

# 🎯 Key Learnings

* Real-world 3-tier architecture
* Private Endpoint usage
* Secure NSG design
* Flask deployment on VMs
* API-based communication

---

# 🔮 Future Improvements

* Add HTTPS
* Use Load Balancer
* CI/CD pipeline
* Monitoring with Azure Monitor

---

# 👨‍💻 Author

Cloud / DevOps Project demonstrating real-world Azure architecture and deployment.
