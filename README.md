# Praktikum TKA Modul 3: Ansible

Repository ini berisi hasil pengerjaan dan Walkthrough Praktikum Modul 3 Teknologi Komputasi Awan (TKA). Praktikum ini berfokus pada penggunaan Ansible (Infrastructure as Code) untuk melakukan otomasi instalasi Docker, Firewall, serta *deployment* aplikasi Backend dan Frontend (menggunakan *container*).

## Anggota Kelompok B13
1. Imam Mahmud Dalil Fauzan - 5027241100
2. Khumaidi Kharis Az-Zacky - 5027241049

## Struktur Folder
* `inventory.yml`: File inventory Ansible untuk mendefinisikan *group node* beserta IP dan konfigurasi SSH.
* `ansible.cfg`: Konfigurasi agar Ansible tidak perlu konfirmasi *host key* saat SSH.
* `playbook.yml`: Playbook utama untuk mengeksekusi *roles*.
* `group_vars/`: Berisi variabel seperti kredensial database dan port backend/frontend.
* `roles/`:
  * `docker_engine/`: Role Ansible untuk menginstall Docker Engine dan mengatur *firewall* (UFW) agar hanya terbuka port 22.
  * `backend/`: Role Ansible untuk membuka *firewall* port 3000, mengirim *source code*, me-render template `.env` & `docker-compose.yml`, serta menyalakan *service* menggunakan *Docker Compose*.

---

## Tahapan Pengerjaan & Walkthrough

### 0. Persiapan Awal (Install Ansible & Setup Multipass)
Skenario ini menggunakan **WSL (Ubuntu)** untuk menjalankan Ansible dan **Multipass (di Windows)** untuk simulasi node *Virtual Machine*.

**A. Install Ansible di WSL**
```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt-get install ansible
```

**B. Buat 2 Node VM Menggunakan Multipass**
Dikarenakan ada *double layer virtualization*, kami mengunduh *image* Ubuntu secara manual dan membuat 2 node dari *image* tersebut.
- Link Download image Ubuntu: https://cloud-images.ubuntu.com/noble/current/
- Pilih file `noble-server-cloudimg-amd64.img` size 601 MB
- Setelah didownload, rename file itu jadi `ubuntu.img`
- Buat folder `Images` di C:/
- Masukan file `ubuntu.img` tadi ke dalam folder `Images`
```powershell
# PowerShell A (Backend), jalankan:
multipass launch file://C:/Images/ubuntu.img --name node-backend --disk 5G --memory 1G
# PowerShell B (Frontend), jalankan:
multipass launch file://C:/Images/ubuntu.img --name node-frontend --disk 5G --memory 1G
```

**C. Setup SSH Key**
Agar Ansible bisa berkomunikasi tanpa *password*, kami membuat dan mendaftarkan SSH Key dari WSL ke masing-masing VM.
- Open 2 Terminal WSL baru (A dan B)
```bash
# Di WSL A:
ssh-keygen -t rsa
enter
enter
enter
cat ~/.ssh/id_rsa.pub # Copy hasil semua teks public key
```
```bash
# Di PowerShell A (node-backend):
multipass shell node-backend
echo "PUBLIC_KEY_ANDA" >> ~/.ssh/authorized_keys

# Di PowerShell B (node-frontend):
multipass shell node-frontend
echo "PUBLIC_KEY_ANDA" >> ~/.ssh/authorized_keys
```

*(P.S.: Kami juga mengatur Port Proxy dan Firewall Windows menggunakan `netsh` agar WSL bisa berkomunikasi dengan VM Multipass melewati adapter jaringan Windows). Jika Terminal WSL dan Powershell dalam 1 Laptop, Hiraukan ini :v*

**D. Sambungkan WSL ke Powershell dengan SSH**
- Buka Powershell, cek IP masing-masing VM
```bash
# Di Powershell:
multipass list
# Catat IP masing-masing node
```

- Buka WSL, sambungkan ke masing-masing VM
```bash
# Di WSL A (node-backend):
ssh ubuntu@[IP_NODE_BACKEND]

# Di WSL B (node-frontend):
ssh ubuntu@[IP_NODE_FRONTEND]
```

---

### 1. Praktikan 1 (Setup Docker & Firewall)
Tujuan: Mengotomatisasi instalasi Docker dan Firewall UFW.
-  Buka Terminal WSL di VS Code
1. Lakukan tes *Ping* ke semua node untuk mengecek koneksi:
   ```bash
   # Di WSL VS Code
   ansible all -m ping -i inventory.yml
   ```
   
2. Jalankan playbook instalasi:
   ```bash
   # Di WSL VS Code
   ansible-playbook -i inventory.yml playbook.yml
   ```

3. **Pembuktian Manual:** *Test* ke dalam SSH Node dan jalankan *container* *Hello World*:
   ```bash
   # di WSL A (node-backend), lakukan 2x
   sudo docker run hello-world
   
   # di WSL B (node-frontend), lakukan 2x
   sudo docker run hello-world
   ```

---

### 2. Praktikan 2 (Setup Backend)
Tujuan: Melakukan *deployment* aplikasi backend Node.js dan database PostgreSQL menggunakan Docker Compose via Ansible.

1. Pastikan role `backend` sudah ditambahkan di `playbook.yml`.
2. Jalankan *playbook* kembali:
   ```bash
   ansible-playbook -i inventory.yml playbook.yml
   ```
   *(Ansible akan otomatis membuka port 3000, mengirim source code, menyalin template environment, mem-build Dockerfile, dan menjalankan Docker Compose).*
3. **Pembuktian Manual:** Uji *endpoint* dari dalam SSH `node-backend` menggunakan cURL.

   **Health Check:**
   ```bash
   curl http://localhost:3000/
   ```
   
   **Register User:**
   ```bash
   curl -X POST http://localhost:3000/api/register \
   -H "Content-Type: application/json" \
   -d '{"username": "imam", "password": "111"}'
   ```

   **Login User (Sukses & Gagal):**
   ```bash
   # Gagal (Password salah)
   curl -X POST http://localhost:3000/api/login \
   -H "Content-Type: application/json" \
   -d '{"username": "imam", "password": "aaa"}'
   
   # Sukses
   curl -X POST http://localhost:3000/api/login \
   -H "Content-Type: application/json" \
   -d '{"username": "imam", "password": "111"}'
   ```
