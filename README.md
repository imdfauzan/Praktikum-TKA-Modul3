# Praktikum TKA Modul 3: Ansible

Repository ini berisi hasil pengerjaan Praktikum Modul 3 mata kuliah Teknologi Komputasi Awan (TKA). Praktikum ini berfokus pada penggunaan Ansible (Infrastructure as Code) untuk melakukan otomasi instalasi Docker, Firewall, serta *deployment* aplikasi Backend dan Frontend (menggunakan *container*).

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

## Panduan Pengerjaan & Walkthrough (Langkah demi Langkah)

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
```powershell
# Buka PowerShell, jalankan:
multipass launch file://C:/Images/ubuntu.img --name node-backend --disk 5G --memory 1G
multipass launch file://C:/Images/ubuntu.img --name node-frontend --disk 5G --memory 1G
```

**C. Setup SSH Key**
Agar Ansible bisa berkomunikasi tanpa *password*, kami membuat dan mendaftarkan SSH Key dari WSL ke masing-masing VM.
```bash
# Di WSL:
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub # Copy hasil teks public key

# Di PowerShell (Masuk ke tiap node):
multipass shell node-backend
echo "PUBLIC_KEY_ANDA" >> ~/.ssh/authorized_keys
```

*(Catatan khusus jaringan: Kami juga mengatur Port Proxy dan Firewall Windows menggunakan `netsh` agar WSL bisa berkomunikasi dengan VM Multipass melewati adapter jaringan Windows).*

---

### 1. Praktikan 1 (Setup Docker & Firewall)
Tujuan: Mengotomatisasi instalasi Docker dan Firewall UFW.

1. Lakukan tes *Ping* ke semua node untuk mengecek koneksi:
   ```bash
   ansible all -m ping -i inventory.yml
   ```
2. Jalankan playbook instalasi:
   ```bash
   ansible-playbook -i inventory.yml playbook.yml
   ```
3. **Pembuktian Manual:** Masuk ke dalam SSH Node dan jalankan *container* *Hello World*:
   ```bash
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
