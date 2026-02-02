# Ansible SSH Key Setup

Deploy SSH public key ke multiple servers dengan Ansible.

---

## Project Structure

```
ansible-ssh-setup/
├── ansible.cfg          # Ansible configuration
├── inventory.ini        # Daftar 7 VMs (EDIT INI)
├── group_vars/
│   └── all.yml          # Public variables (semua server)
├── host_vars/           # Variables per server (password berbeda)
│   ├── vm1.yml          # Password vm1 (ENCRYPT INI)
│   ├── vm2.yml          # Password vm2 (ENCRYPT INI)
│   ├── vm3.yml          # Password vm3 (ENCRYPT INI)
│   ├── vm4.yml          # Password vm4 (ENCRYPT INI)
│   ├── vm5.yml          # Password vm5 (ENCRYPT INI)
│   ├── vm6.yml          # Password vm6 (ENCRYPT INI)
│   └── vm7.yml          # Password vm7 (ENCRYPT INI)
├── secrets.yml          # SSH public key (ENCRYPT INI)
├── setup-ssh.yml        # Main playbook
├── test-connection.yml  # Test koneksi
└── README.md
```

---

## Quick Start

### 1. Edit Inventory (Wajib!)

```bash
nano inventory.ini
```

Sesuaikan IP dan username untuk 7 VM kamu:

```ini
[all_servers]
gitlab     ansible_host=192.168.246.30 ansible_user=gitlabadmin
jenkins    ansible_host=192.168.246.31 ansible_user=jenkins
k8s-master ansible_host=192.168.246.32 ansible_user=kubeadmin
k8s-node1  ansible_host=192.168.246.33 ansible_user=kubeadmin
docker     ansible_host=192.168.246.34 ansible_user=dockeradmin
registry   ansible_host=192.168.246.35 ansible_user=registry
monitoring ansible_host=192.168.246.36 ansible_user=admin
```

---

### 2. Edit Secrets (Wajib!)

```bash
nano secrets.yml
```

Copy isi dari `~/.ssh/id_work.pub`:

```bash
# Lihat public key kamu
cat ~/.ssh/id_work.pub
```

Paste ke `secrets.yml`:

```yaml
---
# SSH Public Key yang akan di-deploy
ssh_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5XXXXXX your-email@example.com"

# SSH Password untuk initial setup (sebelum key ter-deploy)
# Hapus setelah SSH key berhasil di-deploy!
ansible_ssh_pass: "password-ssh-kamu"
ansible_become_pass: "password-sudo-kamu"
```

> ⚠️ **Catatan:** `ansible_ssh_pass` hanya dibutuhkan untuk **initial setup** saat SSH key belum ada di server. Setelah SSH key ter-deploy, hapus password dari secrets.yml!

---

### 3. Encrypt Secrets (Wajib!)

```bash
# Encrypt secrets.yml
ansible-vault encrypt secrets.yml

# Masukkan password vault, INGAT PASSWORD INI!
New Vault password: 
Confirm New Vault password:
```

Setelah di-encrypt, isi file akan terlihat seperti:
```
$ANSIBLE_VAULT;1.1;AES256
61626364656667686970...
```

---

### Ansible Vault Commands (Manage Secrets)

| Command | Fungsi |
|---------|--------|
| `ansible-vault encrypt secrets.yml` | Encrypt file |
| `ansible-vault decrypt secrets.yml` | Decrypt file (untuk edit) |
| `ansible-vault edit secrets.yml` | Edit langsung tanpa decrypt manual |
| `ansible-vault view secrets.yml` | Lihat isi tanpa decrypt |
| `ansible-vault rekey secrets.yml` | Ganti password vault |

#### Encrypt (pertama kali)
```bash
ansible-vault encrypt secrets.yml
```

#### Decrypt (untuk edit manual)
```bash
# Decrypt
ansible-vault decrypt secrets.yml

# Edit seperti biasa
nano secrets.yml

# Encrypt lagi setelah selesai
ansible-vault encrypt secrets.yml
```

#### Edit langsung (tanpa decrypt manual)
```bash
# Lebih aman - file tetap encrypted setelah save
ansible-vault edit secrets.yml
```

#### View tanpa decrypt
```bash
ansible-vault view secrets.yml
```

#### Ganti password vault
```bash
ansible-vault rekey secrets.yml
```

---

### 4. Test Koneksi Dulu

```bash
# Test semua server
ansible all -m ping --ask-vault-pass

# Atau pakai playbook
ansible-playbook test-connection.yml --ask-vault-pass
```

---

### 5. Deploy SSH Key

```bash
# Dengan vault password
ansible-playbook setup-ssh.yml --ask-vault-pass

# Dry run dulu (check mode)
ansible-playbook setup-ssh.yml --check --ask-vault-pass

# Hanya server tertentu
ansible-playbook setup-ssh.yml --limit gitlab --ask-vault-pass

# Beberapa server
ansible-playbook setup-ssh.yml --limit "gitlab,jenkins,k8s-master" --ask-vault-pass
```

---

## Initial Setup (SSH Key Belum Ada di Server)

Jika ini **pertama kali** dan SSH key belum ter-deploy ke server (masih pakai password):

### Option 1: Password di secrets.yml (Recommended)

1. Tambahkan password ke `secrets.yml`:
```yaml
---
ssh_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5XXXXXX your-email"

# Password untuk initial setup
ansible_ssh_pass: "password-ssh-kamu"
ansible_become_pass: "password-sudo-kamu"
```

2. Encrypt secrets:
```bash
ansible-vault encrypt secrets.yml
```

3. Run playbook:
```bash
ansible-playbook setup-ssh.yml --ask-vault-pass
```

4. **Setelah berhasil**, hapus password dari secrets:
```bash
ansible-vault edit secrets.yml
# Hapus baris ansible_ssh_pass dan ansible_become_pass
```

### Option 2: Password via Command Line

```bash
# Install sshpass dulu
sudo apt install sshpass -y

# Run dengan prompt password
ansible-playbook setup-ssh.yml --ask-pass --ask-become-pass
```

- `--ask-pass` → prompt password SSH
- `--ask-become-pass` → prompt password sudo

### Option 3: Password Berbeda per Server (host_vars)

Jika setiap server punya password berbeda, gunakan **host_vars**:

```
ansible-ssh-setup/
├── host_vars/
│   ├── vm1.yml    ← Password untuk vm1
│   ├── vm2.yml    ← Password untuk vm2
│   ├── vm3.yml    ← Password untuk vm3
│   ├── vm4.yml    ← Password untuk vm4
│   ├── vm5.yml    ← Password untuk vm5
│   ├── vm6.yml    ← Password untuk vm6
│   └── vm7.yml    ← Password untuk vm7
```

#### Step 1: Edit setiap file host_vars

```bash
nano host_vars/vm1.yml
```

```yaml
---
ansible_ssh_pass: "password-untuk-vm1"
ansible_become_pass: "sudo-password-vm1"
```

```bash
nano host_vars/vm2.yml
```

```yaml
---
ansible_ssh_pass: "password-untuk-vm2"
ansible_become_pass: "sudo-password-vm2"
```

... ulangi untuk vm3 - vm7

#### Step 2: Encrypt semua host_vars

```bash
# Encrypt semua sekaligus dengan password yang SAMA
ansible-vault encrypt host_vars/*.yml

# Atau satu-satu
ansible-vault encrypt host_vars/vm1.yml
ansible-vault encrypt host_vars/vm2.yml
# ... dst
```

#### Step 3: Run playbook

```bash
ansible-playbook setup-ssh.yml --ask-vault-pass
```

#### Edit password tertentu

```bash
# Edit password vm1
ansible-vault edit host_vars/vm1.yml

# View tanpa edit
ansible-vault view host_vars/vm3.yml
```

#### Decrypt semua (jika perlu)

```bash
ansible-vault decrypt host_vars/*.yml
```

> ⚠️ **Tips:** Gunakan vault password yang SAMA untuk semua file supaya mudah manage.

---

## File yang Perlu Diedit

| File | Apa yang Diedit |
|------|-----------------|
| `inventory.ini` | IP address dan username setiap VM |
| `secrets.yml` | SSH public key |
| `host_vars/vm*.yml` | Password SSH per server (jika berbeda) |
| `group_vars/all.yml` | Path ke SSH key (jika berbeda) |

---

## Password Options Summary

| Skenario | Solusi | File |
|----------|--------|------|
| Password **sama** semua server | Tambah di `secrets.yml` | `secrets.yml` |
| Password **beda** tiap server | Pakai `host_vars/` | `host_vars/vm1.yml`, `vm2.yml`, ... |
| **Tanpa** simpan password | Pakai `--ask-pass` flag | - |

---

## Inventory Format

```ini
[all_servers]
<nama_vm> ansible_host=<IP_ADDRESS> ansible_user=<SSH_USERNAME>
```

### Contoh Lengkap:

```ini
[all_servers]
# GitLab Server
gitlab ansible_host=192.168.246.30 ansible_user=gitlabadmin

# Jenkins CI/CD
jenkins ansible_host=192.168.246.31 ansible_user=jenkins

# Kubernetes Cluster
k8s-master ansible_host=192.168.246.32 ansible_user=kubeadmin
k8s-node1  ansible_host=192.168.246.33 ansible_user=kubeadmin
k8s-node2  ansible_host=192.168.246.34 ansible_user=kubeadmin

# Docker Registry
registry ansible_host=192.168.246.35 ansible_user=registry

# Monitoring (Grafana, Prometheus)
monitoring ansible_host=192.168.246.36 ansible_user=admin
```

---

## Ansible Vault Commands (Detail)

### Encrypt File (Pertama Kali)

```bash
ansible-vault encrypt secrets.yml
```

Output:
```
New Vault password: 
Confirm New Vault password:
Encryption successful
```

### Decrypt File (Untuk Edit Manual)

```bash
# Decrypt dulu
ansible-vault decrypt secrets.yml

# Edit seperti biasa
nano secrets.yml

# JANGAN LUPA encrypt lagi!
ansible-vault encrypt secrets.yml
```

### Edit Langsung (Recommended)

```bash
# File tetap encrypted setelah save
ansible-vault edit secrets.yml
```

Akan membuka editor (default: vim). Setelah save & quit, file otomatis encrypted lagi.

**Ganti editor default:**
```bash
# Pakai nano
EDITOR=nano ansible-vault edit secrets.yml

# Set permanent di .bashrc
echo 'export EDITOR=nano' >> ~/.bashrc
source ~/.bashrc
```

### View Tanpa Decrypt

```bash
ansible-vault view secrets.yml
```

### Ganti Password Vault

```bash
ansible-vault rekey secrets.yml
```

### Simpan Password di File (Opsional)

Supaya tidak perlu ketik password terus:

```bash
# Buat file password
echo "your-vault-password" > .vault_pass
chmod 600 .vault_pass

# Tambahkan ke .gitignore
echo ".vault_pass" >> .gitignore

# Run tanpa prompt
ansible-playbook setup-ssh.yml --vault-password-file .vault_pass
```

Atau set di `ansible.cfg`:
```ini
[defaults]
vault_password_file = .vault_pass
```

---

## Usage Examples

### Run ke Semua Server

```bash
ansible-playbook setup-ssh.yml --ask-vault-pass
```

### Run ke Server Tertentu

```bash
# Satu server
ansible-playbook setup-ssh.yml --limit gitlab --ask-vault-pass

# Beberapa server
ansible-playbook setup-ssh.yml --limit "gitlab,jenkins" --ask-vault-pass
```

### Dry Run (Check Mode)

```bash
ansible-playbook setup-ssh.yml --check --ask-vault-pass
```

### Verbose Output

```bash
ansible-playbook setup-ssh.yml -v --ask-vault-pass    # verbose
ansible-playbook setup-ssh.yml -vv --ask-vault-pass   # more verbose
```

---

## Troubleshooting

### Error: No inventory was parsed

```bash
# Pastikan ada inventory.ini
ls -la inventory.ini

# Atau specify manual
ansible-playbook setup-ssh.yml -i inventory.ini
```

### Error: Permission denied (publickey)

```bash
# Test SSH manual dulu
ssh -i ~/.ssh/id_work gitlabadmin@192.168.246.30

# Pastikan path SSH key benar di group_vars/all.yml
ssh_private_key_path: "~/.ssh/id_work"
```

### Error: Host key verification failed

```bash
# Sudah dihandle di ansible.cfg, tapi jika masih error:
ssh-keyscan 192.168.246.30 >> ~/.ssh/known_hosts
```

### Error: Vault password required

```bash
# Tambahkan --ask-vault-pass
ansible-playbook setup-ssh.yml --ask-vault-pass

# Atau simpan password di file
echo "your-vault-password" > .vault_pass
chmod 600 .vault_pass
ansible-playbook setup-ssh.yml --vault-password-file .vault_pass
```

---

## Expected Output

```
PLAY [Setup SSH Keys on All Servers] ********************************

TASK [Display target server info] ***********************************
ok: [gitlab] => 
  msg: 🖥️  Configuring gitlab (192.168.246.30) - User: gitlabadmin

TASK [Ensure .ssh directory exists] *********************************
ok: [gitlab]

TASK [Backup existing authorized_keys] ******************************
changed: [gitlab]

TASK [Add SSH public key from secrets] ******************************
changed: [gitlab]

TASK [Ensure correct permissions on authorized_keys] ****************
ok: [gitlab]

TASK [Display success message] **************************************
ok: [gitlab] => 
  msg: ✅ SSH key deployed successfully to gitlab

PLAY RECAP **********************************************************
gitlab : ok=6  changed=2  unreachable=0  failed=0
jenkins: ok=6  changed=2  unreachable=0  failed=0
...
```

---

## Security Notes

1. **Selalu encrypt `secrets.yml`** dengan ansible-vault
2. **Jangan commit** `secrets.yml` yang tidak encrypted ke Git
3. Tambahkan ke `.gitignore`:
   ```
   secrets.yml
   .vault_pass
   *.retry
   ```

---

## After Deployment

Setelah SSH key ter-deploy, test login:

```bash
# Test ke setiap server
ssh -i ~/.ssh/id_work gitlabadmin@192.168.246.30
ssh -i ~/.ssh/id_work jenkins@192.168.246.31
# ... dst
```

---

*Automate SSH key management dengan Ansible! 🔐*
