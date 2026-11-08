# 🍯 SSH Honeypot

A lightweight, dependency-free **SSH honeypot** written in pure Python. It listens on a TCP port, presents a fake OpenSSH banner and login prompt, and logs everything an attacker types — usernames, passwords, and commands — to daily JSON log files.

Built for **security research, threat intelligence, and education**.

![Python](https://img.shields.io/badge/python-3.6%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

---

## ✨ Features

- 🎭 Fake OpenSSH banner (`SSH-2.0-OpenSSH_7.4p1 Debian-10+deb9u7`)
- 🔑 Captures attempted **usernames** and **passwords**
- 💻 Captures the first **command** entered at the fake root prompt
- 🧵 Multi-threaded — handles multiple connections at once
- 📝 Structured **JSON-lines** logs, one file per day
- 🌐 Records attacker **IP address** and **timestamp** for every event
- 📦 Uses only the Python standard library

---

## 📋 Requirements

- Python 3.6 or newer
- No third-party packages

---

## 🚀 Installation

```bash
git clone https://github.com/whoami592/ssh-honeypot.git
cd ssh-honeypot
```

---

## ▶️ Usage

```bash
python3 ssh_honeypot.py
```

By default the honeypot listens on `0.0.0.0:2222`:

```
[*] Honeypot listening on 0.0.0.0:2222
```

Press `Ctrl+C` to stop it.

### Test it locally

```bash
nc 127.0.0.1 2222
```

### Custom configuration

Edit the bottom of `ssh_honeypot.py`:

```python
honeypot = SSHHoneypot(host='0.0.0.0', port=2222, log_dir='honeypot_logs')
honeypot.start()
```

| Parameter | Default          | Description                      |
|-----------|------------------|----------------------------------|
| `host`    | `0.0.0.0`        | Interface to bind to             |
| `port`    | `2222`           | TCP port to listen on            |
| `log_dir` | `honeypot_logs`  | Directory where logs are stored  |

> 💡 To catch real-world scans, forward external port `22` to port `2222` on the honeypot machine using your router or `iptables`. Never move your real SSH service to a different port without planning for it.

---

## 📂 Log Format

Logs are saved as `honeypot_logs/honeypot_YYYY-MM-DD.json`, with one JSON object per line:

```json
{"timestamp": "2026-11-08T12:30:45.123456", "client_ip": "203.0.113.5", "data": "Username attempted: admin"}
{"timestamp": "2026-11-08T12:30:47.654321", "client_ip": "203.0.113.5", "data": "Password attempted: 123456"}
{"timestamp": "2026-11-08T12:30:49.111111", "client_ip": "203.0.113.5", "data": "Command attempted: whoami"}
```

### Quick analysis

```bash
# Top attacking IPs
jq -r '.client_ip' honeypot_logs/*.json | sort | uniq -c | sort -rn | head

# Most common passwords tried
jq -r 'select(.data | startswith("Password")) | .data' honeypot_logs/*.json | sort | uniq -c | sort -rn | head
```

---

## 🔍 How It Works

1. Accepts a TCP connection and logs it.
2. Sends a fake OpenSSH version banner.
3. Prompts for `login:` and `Password:` and logs both responses.
4. Replies with *"Permission denied"*, then shows a fake `root@debian:~#` prompt.
5. Logs the command entered and closes the connection.

> ℹ️ This is a **low-interaction** honeypot. It imitates the *look* of an SSH service over plain text and does not implement the real SSH protocol, so advanced scanners and real SSH clients will detect it.

---

## 🛡️ Security Notes

- Run the honeypot as an **unprivileged user**, never as root.
- Isolate it (VM, container, or separate VLAN) from your production network.
- Attacker-supplied data is written to logs. Treat log contents as **untrusted input**.
- Consider adding rate limiting and connection timeouts before exposing it to the internet.

## 🗺️ Roadmap Ideas

- Real SSH protocol emulation (e.g. with `paramiko`)
- Fake shell with multiple commands
- GeoIP lookup and dashboards
- Alerts via Telegram / email
- Log rotation and connection limits

---

## ⚠️ Disclaimer

This tool is intended for **educational and defensive security research only**. Deploy it only on systems and networks you own or are authorized to monitor, and comply with all applicable laws and regulations. The author is not responsible for misuse.

---

## 👨‍💻 Author

**Coded by Mr Sabaz Ali Khan**
*Cyber Security Engineer / Ethical Hacker — Pakistan*

---

## 📄 License

Released under the [MIT License](LICENSE).
