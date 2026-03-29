# mde-linux-rhel-ansible-role

## 📦 Overview

An Ansible role to install, configure, and onboard **Microsoft Defender for Endpoint (MDE)** on RHEL-based Linux systems.

This role is designed to be:

* ✅ Idempotent
* 🔁 Retry-safe
* ⚙️ Production-ready
* 🧪 Lab-friendly

---

## 🧰 Development Environment Setup

This repository uses Python virtual environments for isolation. You can use either **uv (recommended)** or standard **Python venv + pip**.

---

# 🚀 Option 1 — Using uv (Recommended)

[`uv`](https://github.com/astral-sh/uv) is a fast Python package manager and virtual environment tool.

## 1. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Restart your shell or run:

```bash
source ~/.bashrc
```

---

## 2. Create virtual environment

```bash
uv venv .venv
```

---

## 3. Activate environment

```bash
source .venv/bin/activate
```

You should now see:

```text
(.venv)
```

---

## 4. Install dependencies

```bash
uv pip install ansible
```

---

## 5. Verify installation

```bash
ansible --version
```

---

## 6. (Optional) Lock dependencies

```bash
uv lock
```

---

# 🐍 Option 2 — Standard Python (venv + pip)

## 1. Ensure Python is installed

```bash
python3 --version
```

---

## 2. Create virtual environment

```bash
python3 -m venv .venv
```

---

## 3. Activate environment

```bash
source .venv/bin/activate
```

---

## 4. Install dependencies

```bash
pip install --upgrade pip
pip install ansible
```

---

## 5. Verify installation

```bash
ansible --version
```

---

# 🧪 Test Ansible

Run a quick test to confirm everything is working:

```bash
ansible localhost -m ping
```

Expected output:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# 📁 Project Structure

```text
.
|-- LICENSE
|-- README.md
|-- inventory.ini
|-- pyproject.toml
|-- requirements.txt
|-- roles
|   `-- mde_linux_rhel
|       |-- defaults
|       |   `-- main.yml
|       |-- files
|       |   `-- onboarding
|       |       `-- mdatp_onboard.json
|       |-- handlers
|       |   `-- main.yml
|       |-- meta
|       |   `-- main.yml
|       |-- tasks
|       |   |-- bootstrap.yml
|       |   |-- config.yml
|       |   |-- install.yml
|       |   |-- main.yml
|       |   |-- onboard.yml
|       |   `-- verify.yml
|       |-- templates
|       |   `-- mdatp_managed.json.j2
|       `-- vars
|           `-- main.yml
`-- site.yml
```

---

# ⚙️ Role Behaviour

## 🔐 Onboarding

* Uses onboarding file from:

  ```text
  roles/mde_linux_rhel/files/onboarding/
  ```

* Onboarding is **idempotent**:

    * Only runs if `org_id` is not present

---

## 🛡️ Configuration

* Managed via:

  ```text
  /etc/opt/microsoft/mdatp/managed/mdatp_managed.json
  ```

* Includes:

    * Antivirus mode (passive / real-time)
    * Cloud configuration
    * EDR tags

---

## 🔁 Service Handling

* `mdatp` is **only restarted when configuration changes**
* Implemented using Ansible **handlers**

```yaml
notify: restart mdatp
```

* Handlers are flushed before validation to ensure correct state

---

## 🏷️ Tagging

Example:

```yaml
mde_tags:
  - key: "GROUP"
    value: "RHEL-EDR"
  - key: "MANAGEMENT"
    value: "MDE-Management"
```

### ⚠️ Important

* Tag **keys must be unique**
* Duplicate keys will result in tags being ignored by MDE

---

## ⏱️ Eventual Consistency

MDE is **not instant**

* Config → Agent → Cloud → Agent
* Tags and state may take time to appear

The role accounts for this by:

* Waiting for daemon readiness
* Retrying health checks
* Avoiding brittle assumptions

---

## 🔍 Verification

The role verifies:

* MDE service is responsive
* Device is onboarded (`org_id`)
* Config file contains expected values

Example:

```bash
mdatp health
```

---

# 🚀 Usage

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# ⚠️ Notes

* Always activate the virtual environment before running Ansible
* Do **not** install dependencies globally
* `.venv/` should be in `.gitignore`
* MDE behaviour is **eventually consistent**, not immediate

---

# 🧹 Deactivate environment

```bash
deactivate
```

---

# 📌 Onboarding File

Place your onboarding file in:

```text
roles/mde_linux_rhel/files/onboarding/
```

This file is generated from the Microsoft Defender portal and must **not be modified**.

---

# 🧠 Design Principles

* Idempotent by default
* Minimal restarts
* Retry-aware
* Cloud-aware (eventual consistency)
* Clean separation of concerns

---

# 🧪 Lab vs Production

| Area             | Lab        | Production |
| ---------------- | ---------- | ---------- |
| Tag validation   | Relaxed    | Strict     |
| Retry timing     | Short      | Extended   |
| Logging          | Verbose    | Structured |
| Failure handling | Permissive | Strict     |

---

# 💬 Final Notes

This role is designed to reflect **real-world MDE behaviour**, including:

* Delayed Graph population
* Tag propagation delays
* Agent startup timing

If something appears “slow”, it’s usually **by design**, not failure.
