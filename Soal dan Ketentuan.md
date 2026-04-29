# Soal dan Ketentuan
Praktikum Teknologi Komputasi Awan Modul 3: Ansible

## Anggota Kelompok B13
1. Imam Mahmud Dalil Fauzan - 5027241100
2. Khumaidi Kharis Az-Zacky - 5027241049    

## Latar Belakang
Anda merupakan mahasiswa IT yang Cloud Computing, untuk memahami workflow Infrastructure as Code (IaC) anda disuruh untuk deploy sebuah simulasi web login (Resource pada folder `Resource Soal Modul 3`) menggunakan Ansible.
Masalahnya, anda hanya mampu di satu skill! Oleh karena itu, anda memiliki ide mengunjungi diri anda sendiri dalam parallel universe berbeda dengan harapan anda menemukan versi lain dari diri anda dengan skill yang berbeda. Karena anda bekerja dengan diri anda sendiri, sudah pasti masing-masing versi dari diri anda memiliki dua node VM multipass dengan public key SSH yang siap untuk digunakan.

## Praktikan 1
Anda memiliki skillset dalam instalasi Docker dalam node Ansible, lakukan:
- Membuat inventory.yml untuk node yang sudah dipersiapkan, pecahkan kedua node tersebut dalam group yang berbeda.
- Menyiapkan role baru khusus untuk instalasi Docker Engine.
- Untuk memastikan semua telah disiapkan dengan benar, anda melakukan ping dengan Ansible kepada semua node yang tercatat dalam inventory.
- Dalam role yang sudah dipersiapkan sebelumnya, anda membuat playbook yang mampu:
    - Install seluruh keperluan Docker Engine.
    - Setup firewall agar hanya terbuka pada port 22.
- Menyiapkan playbook utama yang memberikan semua node dalam inventory sebuah role khusus instalasi Docker.
- Sebelum mengakhiri tugas anda, untuk memastikan semua lancar, anda:
    - SSH ke dalam kedua node anda.
    - Menjalankan image Docker “Hello World” secara manual

- Semua yang anda lakukan akan dilaporkan dalam sebuah video, sebelum anda berikan tugas berikutnya pada versi diri anda yang lain.

## Praktikan 2
Anda diberikan tugas oleh Praktikan 1 untuk melanjutkan tugas Cloud Computing, yaitu setup Backend. Lakukan:
- Menyiapkan role baru khusus untuk deploy backend.
- Memikirkan variabel apa saja yang akan diperlukan, masukkan keperluan berikut dalam vars atau vault untuk group khusus backend dalam inventory yang ada:
    - db_name
    - db_ username
    - backend_port
    - db_password
    - jwt_secret
- Melihat source code backend yang diberikan anda menyiapkan Dockerfile untuk build backend, mungkin image node akan membantu.
- Menyiapkan template Jinja2 untuk .env dan docker-compose.yml
- Dalam role yang sudah dipersiapkan sebelumnya, anda membuat playbook yang mampu:
    - Membuka port backend sesuai backend_port
    - Mampu menyalakan backend dengan Docker Compose:
        - Postgres untuk database
        - Service backend yang sudah menggunakan Dockerfile
    - Health check menggunakan perintah “uri”
- Melakukan modifikasi pada playbook utama untuk menjalankan role khusus backend kepada group backend.
- Sebelum mengakhiri tugas anda, untuk memastikan semua lancar, anda:
    - Curl health check
    - Curl register user
- Semua yang anda lakukan akan dilaporkan dalam sebuah video, sebelum anda berikan tugas berikutnya pada versi diri anda yang lain.

## Praktikan 3
Anda diberikan tugas oleh Praktikan 1 dan 2 untuk melanjutkan tugas Cloud Computing, yaitu setup Frontend. Lakukan:
- Menyiapkan role baru khusus untuk deploy frontend.
- Memikirkan variabel apa saja yang akan diperlukan, masukkan keperluan berikut dalam vars atau vault untuk group khusus backend dalam inventory yang ada:
    - frontend_port
    - backend_url (dilarang hardcode IP address, gunakan template berdasarkan Ansible inventory)
- Melihat source code frontend yang diberikan anda menyiapkan Dockerfile untuk build backend, mungkin image nginx akan membantu.
- Menyiapkan template Jinja2 untuk config.js dan docker-compose.yml
- Dalam role yang sudah dipersiapkan sebelumnya, anda membuat playbook yang mampu:
    - Membuka port frontend sesuai backend_port
    - Mampu menyalakan frontend dengan Docker Compose berdasarkan service frontend yang sudah menggunakan Dockerfile
    - Health check menggunakan perintah “uri”
- Melakukan modifikasi pada playbook utama untuk menjalankan role khusus backend kepada group frontend.
- Karena anda bangga dengan hasil kerjasama dengan berbagai diri anda sendiri, maka anda mencoba:
    - Register dengan user apapun
    - Register lagi dengan user sama
    - Login dengan salah password
    - Login berhasil

- **note**: bila memakai multipass dalam WSL akan ada double layer virtualization (karena windows -> WSL -> nodes) dan website tidak dibuka di browser utama, jadi uhh cari tau jalan keluarnya sendiri hehe (khusus ini boleh hardcode backend_url).
- Semua yang anda lakukan akan dilaporkan dalam sebuah video, sebelum anda melaporkan hasil pekerjaannya kepada diri kalian sendiri sehingga ketiga versi diri anda dapat lulus tugas ini dengan baik.