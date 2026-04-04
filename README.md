# MarkovIDS: Statistical SSH Anomaly Detection System

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active_Development-orange)

MarkovIDS is a lightweight Intrusion Detection System (IDS) that uses Markov Chain probabilistic modeling to detect sophisticated brute-force attacks and abnormal SSH login behavior in real-time. 

Unlike traditional rate-limiting tools (like Fail2Ban) that simply count failed attempts, MarkovIDS tracks the *sequence of states* for every connected IP address, calculating the mathematical probability of their session path to identify anomalous behavior.

---

## 🚀 Features

* **Real-time Log Tailing:** Continuously monitors `/var/log/auth.log` (or equivalent) without locking the file.
* **Probabilistic Threat Scoring:** Uses Markov Chain transition matrices to assign a dynamic "Trust Score" to incoming IP addresses.
* **Low False-Positive Rate:** Distinguishes between a forgetful admin (normal behavior) and a dictionary attack bot (anomalous behavior).
* **RESTful API:** Serves parsed data, active threats, and historical alerts via a JSON API.
* **Live Web Dashboard:** A responsive frontend to visualize attacks and monitor server health in real-time.

---

## 🏗️ Architecture

The system is decoupled into four primary components:
1. **The MarkovDaemon:** A background daemon that tails the log file, categorizes authentication states, and applies the Markov Chain algorithms.
2. **The Database:** A lightweight SQLite database storing historical session data, transition baselines, and active alerts.
3. **The API:** A Python-based web interface (e.g., FastAPI/Flask) that serves database metrics securely to the frontend.
4. **The Dashboard:** A dynamic HTML/JS frontend that visualizes the current threat landscape.

---

## ⚙️ Installation & Setup

### Prerequisites
* Python 3.8+
* Read access to SSH logs (usually requires `root` or `adm` group privileges).

### 1. Clone the Repository
```bash
git clone [https://github.com/yourusername/ssh-markov-ids.git](https://github.com/yourusername/ssh-markov-ids.git)
cd ssh-markov-ids
```

### 2. Install Dependencies
```python
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Configuration
Edit the `config/settings.py` file to match your environment:
```python
LOG_FILE_PATH = "/var/log/auth.log" # Use /var/log/secure for RHEL/CentOS
ALARM_THRESHOLD = 0.01              # Triggers alert if sequence probability drops below 1%
```
### 4. Run the System

You will need to run the Worker and the API simultaneously.

Start the Worker (requires log read permissions):
```bash
sudo python3 worker/tailer.py
```

Start the API Server (in a new terminal):
```bash
python3 api/main.py
```
The dashboard will now be accessible at `http://localhost:5000`.

# 🧠 The Math: How it Works

MarkovIDS maps SSH interactions into predefined states (e.g., `SUCCESS_LOGIN`, `FAILED_LOGIN`, `INVALID_USER`, `DISCONNECTED`).

During its baseline training, the system learns the standard transition probabilities between these states. In live operation, the system applies the Chain Rule of Probability to an IP's sequence of actions. As an IP traverses states, their initial Trust Score (1.0) is multiplied by the probability of each transition. If a bot attempts a highly unnatural sequence (e.g., chaining multiple INVALID_USER states), their Trust Score exponentially decays until it breaches the ALARM_THRESHOLD, triggering a defensive alert.
# 📝 License

This project is licensed under the MIT License.

## MIT License

Copyright (c) 2026 [Somenath Sebait/https://0x-s0M3n4th.github.io], [Nandhana RS/Social Link] 

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
