# Ansible Server Configuration

Automate server setup menggunakan Ansible untuk configuration management.

Project reference: [roadmap.sh/projects/configuration-management](https://roadmap.sh/projects/configuration-management)

---

## Kenapa Harus Ansible?

### Masalah Tanpa Configuration Management

```
Manual Setup = Masalah!

Server 1: Install nginx, configure, deploy app... ✅ (30 menit)
Server 2: Install nginx, configure, deploy app... ✅ (30 menit)  
Server 3: Install nginx, configure, deploy app... ❌ (Lupa 1 step!)
Server 4: Install nginx, configure, deploy app... ✅ (30 menit)
...
Server 100: 😱 (50 jam kerja, banyak error!)
```

### Solusi: Ansible

```
Dengan Ansible:

ansible-playbook setup.yml

Server 1: ✅ (otomatis)
Server 2: ✅ (otomatis)
Server 3: ✅ (otomatis)
Server 4: ✅ (otomatis)
...
Server 100: ✅ (semua sama, 5 menit!)
```

---

## Keuntungan Menggunakan Ansible

| Keuntungan | Penjelasan |
|------------|------------|
| **Agentless** | Tidak perlu install software apapun di server target. Cukup SSH! |
| **Idempotent** | Jalankan berkali-kali, hasilnya tetap sama. Aman untuk di-run ulang. |
| **Human Readable** | Pakai YAML, mudah dibaca dan dipahami |
| **Version Control** | Simpan di Git, track perubahan, rollback jika error |
| **Reusable** | Buat sekali, pakai di banyak server/project |
| **Scalable** | 1 server atau 1000 server? Sama saja! |
| **Dokumentasi Hidup** | Playbook = dokumentasi setup server yang selalu up-to-date |

### Perbandingan Manual vs Ansible

| Aspek | Manual | Ansible |
|-------|--------|---------|
| Setup 1 server | 30 menit | 30 menit (pertama kali) |
| Setup 10 server | 5 jam | 5 menit |
| Setup 100 server | 50 jam | 5 menit |
| Konsistensi | ❌ Human error | ✅ Selalu sama |
| Dokumentasi | ❌ Sering outdated | ✅ Playbook = docs |
| Reproducible | ❌ Susah | ✅ Mudah |
| Rollback | ❌ Manual | ✅ Git revert |
| Audit trail | ❌ Tidak ada | ✅ Git history |

### Kapan Harus Pakai Ansible?

✅ **Pakai Ansible jika:**
- Manage lebih dari 1 server
- Setup server yang sama berulang kali
- Butuh konsistensi antar environment (dev, staging, prod)
- Ingin dokumentasi yang selalu up-to-date
- Bekerja dalam tim

❌ **Mungkin tidak perlu jika:**
- Hanya 1 server yang jarang diubah
- One-time setup yang tidak akan diulang
- Belajar Linux basic (manual dulu biar paham)

---

## Apa itu Ansible?

Ansible adalah **configuration management tool** yang memungkinkan kamu untuk:
- Automate server setup
- Manage multiple servers sekaligus
- Reproducible infrastructure (Infrastructure as Code)
- Agentless (tidak perlu install agent di server)

```
┌─────────────────┐      SSH      ┌─────────────────┐
│   Control Node  │ ───────────► │  Target Server  │
│   (Laptop)      │              │  (VM/Cloud)     │
│                 │              │                 │
│  ansible-playbook setup.yml    │  ✅ Updated     │
│                 │              │  ✅ Nginx       │
│                 │              │  ✅ App deployed│
└─────────────────┘              └─────────────────┘
```

### Ansible vs Tools Lain

| Tool | Agentless | Bahasa | Learning Curve |
|------|-----------|--------|----------------|
| **Ansible** | ✅ Ya | YAML | Mudah |
| Puppet | ❌ Agent | DSL | Sedang |
| Chef | ❌ Agent | Ruby | Sulit |
| SaltStack | Optional | YAML | Sedang |
| Terraform | ✅ Ya | HCL | Sedang |

**Ansible paling cocok untuk pemula** karena:
- Tidak perlu install agent
- YAML mudah dibaca
- Dokumentasi lengkap
- Komunitas besar

---

## Project Structure

```
ansible-server-setup/
├── ansible.cfg           # Ansible configuration
├── inventory.ini         # Target servers list
├── setup.yml            # Main playbook
├── group_vars/
│   └── all.yml          # Global variables
└── roles/
    ├── base/            # Basic server setup
    │   ├── tasks/
    │   ├── handlers/
    │   └── templates/
    ├── ssh/             # SSH key management
    │   └── tasks/
    ├── nginx/           # Nginx installation
    │   ├── tasks/
    │   ├── handlers/
    │   └── templates/
    └── app/             # App deployment
        ├── tasks/
        ├── templates/
        └── files/
```

---

## Roles

| Role | Fungsi |
|------|--------|
| `base` | Update system, install utilities, setup fail2ban |
| `ssh` | Add SSH public key to authorized_keys |
| `nginx` | Install & configure Nginx web server |
| `app` | Deploy static website (tarball atau git) |

---

## Prerequisites

### 1. Install Ansible di Control Node (laptop)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Verify
ansible --version
```

### 2. SSH Access ke Target Server

Pastikan bisa SSH ke server:
```bash
ssh gitlabadmin@192.168.246.30
```

---

## Quick Start

### 1. Clone/Download Project

```bash
git clone https://github.com/amirkurniawan/ansible-server-setup.git
cd ansible-server-setup
```

### 2. Edit Inventory

```bash
nano inventory.ini
```

Sesuaikan IP dan user:
```ini
[webservers]
server1 ansible_host=192.168.246.30 ansible_user=gitlabadmin

[webservers:vars]
ansible_ssh_private_key_file=~/.ssh/id_work
```

### 3. Edit Variables

```bash
nano group_vars/all.yml
```

Sesuaikan:
```yaml
ssh_public_key: "ssh-ed25519 AAAA... your-email@example.com"
nginx_server_name: "yourdomain.com"
```

### 4. Test Connection

```bash
ansible all -m ping
```

Output yang diharapkan:
```
server1 | SUCCESS => {
    "ping": "pong"
}
```

### 5. Run Playbook

```bash
# Run semua roles
ansible-playbook setup.yml

# Atau dry-run dulu (check mode)
ansible-playbook setup.yml --check
```

---

## Usage

### Run All Roles

```bash
ansible-playbook setup.yml
```

### Run Specific Role

```bash
# Base role only (update, utilities, fail2ban)
ansible-playbook setup.yml --tags "base"

# Nginx role only
ansible-playbook setup.yml --tags "nginx"

# App deployment only
ansible-playbook setup.yml --tags "app"

# SSH key only
ansible-playbook setup.yml --tags "ssh"
```

### Run Multiple Roles

```bash
ansible-playbook setup.yml --tags "base,nginx"
```

### Skip Specific Role

```bash
ansible-playbook setup.yml --skip-tags "app"
```

### Verbose Output

```bash
ansible-playbook setup.yml -v    # verbose
ansible-playbook setup.yml -vv   # more verbose
ansible-playbook setup.yml -vvv  # debug level
```

---

## App Deployment Methods

### Method 1: Tarball (Default)

Edit `group_vars/all.yml`:
```yaml
app_deploy_method: "tarball"
app_source: "files/website.tar.gz"
```

Letakkan tarball di `roles/app/files/website.tar.gz`

### Method 2: Git Repository (Stretch Goal)

Edit `group_vars/all.yml`:
```yaml
app_deploy_method: "git"
app_git_repo: "https://github.com/username/repo.git"
app_git_branch: "main"
```

---

## Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `timezone` | Asia/Jakarta | Server timezone |
| `ssh_public_key` | - | SSH public key to add |
| `nginx_server_name` | - | Domain/server name |
| `nginx_root` | /var/www/html | Web root directory |
| `app_name` | mywebsite | Application name |
| `app_deploy_method` | tarball | Deployment method (tarball/git) |
| `fail2ban_maxretry` | 3 | Max login attempts |
| `fail2ban_bantime` | 3600 | Ban duration (seconds) |

---

## Directory Explanation

### ansible.cfg

Konfigurasi Ansible:
- Default inventory file
- SSH settings
- Privilege escalation

### inventory.ini

Daftar target servers:
```ini
[webservers]
server1 ansible_host=192.168.246.30 ansible_user=gitlabadmin
server2 ansible_host=192.168.246.31 ansible_user=ubuntu
```

### group_vars/all.yml

Variables yang berlaku untuk semua hosts.

### roles/

Modular tasks yang bisa di-reuse.

---

## Output Example

```
PLAY [Configure Web Server] ************************************

TASK [Display server info] *************************************
ok: [server1] => 
  msg: Configuring server1 (192.168.246.30)

TASK [base : Update apt cache] *********************************
ok: [server1]

TASK [base : Install base packages] ****************************
changed: [server1]

TASK [base : Install fail2ban] *********************************
ok: [server1]

TASK [nginx : Install Nginx] ***********************************
ok: [server1]

TASK [nginx : Configure nginx site] ****************************
changed: [server1]

TASK [app : Extract website tarball] ***************************
changed: [server1]

TASK [Display completion message] ******************************
ok: [server1] => 
  msg: |-
    ✅ Server configuration complete!
    🌐 Website: http://192.168.246.30
    📊 Services: nginx, fail2ban

PLAY RECAP *****************************************************
server1 : ok=15  changed=5  unreachable=0  failed=0  skipped=2
```

---

## Troubleshooting

### Connection Failed

```bash
# Test SSH manually
ssh -i ~/.ssh/id_work gitlabadmin@192.168.246.30

# Check inventory
ansible-inventory --list
```

### Permission Denied (sudo)

Pastikan user bisa sudo tanpa password:
```bash
# Di server
sudo visudo
# Tambahkan:
gitlabadmin ALL=(ALL) NOPASSWD: ALL
```

### Module Not Found

```bash
# Install collection jika diperlukan
ansible-galaxy collection install ansible.builtin
```

---

## Add New Server

1. Edit `inventory.ini`:
```ini
[webservers]
server1 ansible_host=192.168.246.30 ansible_user=gitlabadmin
server2 ansible_host=10.0.0.5 ansible_user=ubuntu  # NEW
```

2. Run playbook:
```bash
ansible-playbook setup.yml
```

Semua server akan dikonfigurasi dengan setup yang sama!

---

## What I Learned

1. **Ansible Basics** - Inventory, playbooks, roles
2. **Configuration Management** - Automate server setup
3. **Idempotency** - Run multiple times, same result
4. **Jinja2 Templates** - Dynamic configuration files
5. **Role Structure** - Modular and reusable code
6. **Tags** - Selective execution

---

## References

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Pre-built roles

---

*Automate everything with Ansible! 🚀*
