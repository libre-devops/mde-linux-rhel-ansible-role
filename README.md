# mde-linux-rhel-ansible-role

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

# ⚠️ Notes

* Always activate the virtual environment before running Ansible.
* Do **not** install dependencies globally.
* `.venv/` should be added to `.gitignore`.

---

# 🧹 Deactivate environment

```bash
deactivate
```

## Onboarding File

Place your onboarding file in:

files/onboarding/

This file is generated from the Microsoft Defender portal and must not be modified.
