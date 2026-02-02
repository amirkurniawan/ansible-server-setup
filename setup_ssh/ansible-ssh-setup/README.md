# Ansible SSH Key Setup

Deploy SSH public key ke multiple servers dengan Ansible.

---

## Project Structure

```
ansible-ssh-setup/
├── ansible.cfg          # Ansible configuration
├── inventory.ini        # Daftar 7 VMs (EDIT INI)
├── group_vars/
│   └── all.yml          # Public variables
├── secrets.yml          # SSH key & secrets (ENCRYPT INI)
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
ssh_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5XXXXXX your-email@example.com"
```

---

### 3. Encrypt Secrets (Recommended)

```bash
# Encrypt secrets.yml
ansible-vault encrypt secrets.yml

# Masukkan password, ingat password ini!
```

---

### 4. Test Koneksi Dulu

```bash
# Test semua server
ansible all -m ping

# Atau pakai playbook
ansible-playbook test-connection.yml
```

---

### 5. Deploy SSH Key

```bash
# Tanpa vault (jika secrets tidak encrypted)
ansible-playbook setup-ssh.yml

# Dengan vault (jika secrets encrypted)
ansible-playbook setup-ssh.yml --ask-vault-pass

# Dry run dulu (check mode)
ansible-playbook setup-ssh.yml --check

# Hanya server tertentu
ansible-playbook setup-ssh.yml --limit gitlab

# Beberapa server
ansible-playbook setup-ssh.yml --limit "gitlab,jenkins,k8s-master"
```

---

## File yang Perlu Diedit

| File | Apa yang Diedit |
|------|-----------------|
| `inventory.ini` | IP address dan username setiap VM |
| `secrets.yml` | Isi SSH public key kamu |
| `group_vars/all.yml` | Path ke SSH key (jika berbeda) |

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

## Ansible Vault Commands

```bash
# Encrypt file
ansible-vault encrypt secrets.yml

# Decrypt file (untuk edit)
ansible-vault decrypt secrets.yml

# Edit encrypted file langsung
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Change password
ansible-vault rekey secrets.yml
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
