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

## Configure your inventory.ini

Configure for your own use, this is an example

```ini
[rhel_mde_servers]
vm1 ansible_host=172.19.249.237

[rhel_mde_servers:vars]
ansible_user=root
```
Then

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
├── inventory.ini
├── LICENSE
├── pyproject.toml
├── README.md
├── requirements.txt
├── roles
│   └── mde_linux_rhel
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   ├── offboarding
│       │   │   └── mdatp_offboard.json
│       │   └── onboarding
│       │       └── mdatp_onboard.json
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── tasks
│       │   ├── bootstrap.yml
│       │   ├── config.yml
│       │   ├── install.yml
│       │   ├── main.yml
│       │   ├── offboard.yml
│       │   ├── onboard.yml
│       │   ├── uninstall.yml
│       │   └── verify.yml
│       ├── templates
│       │   └── mdatp_managed.json.j2
│       └── vars
│           └── main.yml
└── site.yml
```

---

# 🔄 Playbook Flow

```text
                ┌────────────────────┐
                │   bootstrap.yml    │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   install.yml      │
                │ (mdatp package)    │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   config.yml       │
                │ (managed config)   │
                └─────────┬──────────┘
                          │
                          ▼
        ┌───────────────────────────────────┐
        │ onboard.yml (if enabled)          │
        │ offboard.yml (if enabled)         │
        │ uninstall.yml (if absent state)   │
        └─────────┬─────────────────────────┘
                          │
                          ▼
                ┌────────────────────┐
                │   verify.yml       │
                │ health + tests     │
                └────────────────────┘
```

### 🧠 Behaviour

* Flow is **conditional**
* Controlled via variables (onboard/offboard/state)
* Safe to re-run multiple times
* Verification always runs last

---

# ⚙️ Role Behaviour

## 🔐 Onboarding

* Uses onboarding file from:

```text
roles/mde_linux_rhel/files/onboarding/
```

* Only runs if device is not already onboarded

---

## 🛡️ Configuration

* Managed via:

```text
/opt/microsoft/mdatp/managed/mdatp_managed.json
```

* Includes:

  * Antivirus mode
  * Cloud config
  * EDR tags

---

## 🔁 Service Handling

* Restart only when config changes:

```yaml
notify: restart mdatp
```

---

## 🏷️ Tagging

```yaml
mde_tags:
  - key: "GROUP"
    value: "MDE-Management"
  - key: "ROLE"
    value: "RHEL-EDR"
```

⚠️ Keys must be unique

---

## ⏱️ Eventual Consistency

MDE is not immediate:

```text
Config → Agent → Cloud → Agent
```

Expect delays in:

* Tags
* ManagedBy
* API visibility

---

# 🔍 Verification

The role validates:

* `mdatp health`
* `org_id` present
* connectivity test passes
* optional client analyzer
* quick scan completes

---

# ⚙️ Configuration Variables

| Variable                                        | Type   | Default             | Description            |
| ----------------------------------------------- | ------ | ------------------- | ---------------------- |
| configure_ssh                                   | bool   | false               | Configure SSH keys     |
| ssh_keys                                        | list   | []                  | SSH key definitions    |
| run_yum_update                                  | bool   | false               | Run system updates     |
| onboard_mde_agent                               | bool   | true                | Enable onboarding      |
| offboard_mde_agent                              | bool   | false               | Enable offboarding     |
| edr_enabled                                     | bool   | true                | Enable EDR features    |
| edr_supported_rhel_versions                     | list   | [8,9]               | Supported OS versions  |
| mde_onboarding_file                             | string | mdatp_onboard.json  | Onboarding file name   |
| mde_onboarding_remote_path                      | string | path                | Remote onboarding path |
| mde_offboarding_file                            | string | mdatp_offboard.json | Offboarding file       |
| mde_force_onboarding                            | bool   | false               | Force re-onboard       |
| mde_force_offboarding                           | bool   | false               | Force offboard         |
| mde_passive_mode                                | bool   | false               | Passive AV mode        |
| mde_tags                                        | list   | see defaults        | EDR tags               |
| mde_state                                       | string | present             | present/absent         |
| mde_region                                      | string | UK                  | Geo for connectivity   |
| run_mde_client_analyzer_connectivity_test       | bool   | true                | Run analyzer           |
| attempt_to_fix_rhel9_client_analyzer_ssl_errors | bool   | true                | Auto SSL fix           |
| mde_client_analyzer_path                        | string | path                | Analyzer binary path   |

---

# 🚀 Usage

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# ⚠️ Notes

* Activate venv before running
* Do not install globally
* `.venv/` should be ignored
* MDE is eventually consistent

---

# 🧹 Deactivate

```bash
deactivate
```

---

# 📌 Onboarding File

```text
roles/mde_linux_rhel/files/onboarding/
```

Must be downloaded from Defender portal.

---

# 🧠 Design Principles

* Idempotent
* Minimal restarts
* Retry-aware
* Cloud-aware
* Clean task separation

---

# 🚫 Offboarding

⚠️ Offboarding is currently **untested** — use with caution.
