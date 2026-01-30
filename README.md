# Implementasi Honeypot Cowrie sebagai Pendeteksi Serangan VPS

![Security Status](https://img.shields.io/badge/Security-Honeypot-green)
![OS Target](https://img.shields.io/badge/Target-Debian_12-red)
![OS Attacker](https://img.shields.io/badge/Attacker-CachyOS-blue)

Repository ini berisi dokumentasi, konfigurasi, dan laporan hasil pengujian implementasi **Honeypot Cowrie** sebagai mekanisme pertahanan aktif (*Active Defense*) pada Virtual Private Server (VPS). Proyek ini bertujuan untuk mensimulasikan dan menganalisis log serangan terhadap protokol SSH.

## 📋 Latar Belakang
Dalam administrasi server, serangan terhadap port SSH (22) adalah hal yang sangat umum. Proyek ini menggunakan **Cowrie**, sebuah *interaction honeypot* tingkat menengah, untuk memanipulasi penyerang agar mengira mereka telah berhasil membobol server, padahal segala aktivitasnya sedang direkam untuk analisis forensik.

## 🛠️ Arsitektur & Topologi

* **Target (Server):**
    * OS: Debian 12 (Virtual Machine)
    * IP: `192.168.0.102`
    * Honeypot: Cowrie (Port 2222 dialihkan seolah SSH asli)
* **Attacker (Penyerang):**
    * OS: CachyOS (Arch Based)
    * IP: `192.168.0.101`
    * Tools: Nmap, Hydra, Hping3

## ⚡ Skenario Pengujian

Pengujian dilakukan dengan melancarkan serangan dari mesin *Attacker* ke *Target* dengan skenario berikut:

### 1. Single Attack (Serangan Tunggal)
| Metode | Tools | Tujuan | Hasil Deteksi |
| :--- | :--- | :--- | :--- |
| **Port Scanning** | `Nmap` | Memetakan port terbuka | ✅ Log `New connection` tanpa login |
| **Brute Force** | `Hydra` | Menebak password root | ✅ Log `login.success` & `login.failed` |
| **DoS Attack** | `Hping3` | Membanjiri trafik (SYN Flood) | ✅ CPU Load 97% & Log Flooding |

### 2. Double Attack (Serangan Simultan)
* **Port Scanning & Brute Force:** Dijalankan bersamaan untuk melihat respon log.
* **Brute Force & DoS:** Menguji ketahanan pencatatan log saat sistem dalam kondisi beban tinggi (*High Load*).

## 📊 Hasil Analisis

### Deteksi Port Scanning
Sistem berhasil menipu Nmap. Port 2222 terdeteksi sebagai `OpenSSH 9.2p1 Debian`, menyembunyikan identitas asli Cowrie.

### Deteksi Brute Force
Honeypot dikonfigurasi untuk menerima password lemah (contoh: `root`/`password`).
> *Bukti Log:*
> ```json
> eventid: cowrie.login.success, username: root, password: password, src_ip: 192.168.0.101
> ```

### Dampak DoS Attack
Serangan SYN Flood menggunakan Hping3 menyebabkan lonjakan penggunaan CPU hingga **97%** pada proses `ksoftirqd` (Software Interrupts), membuktikan serangan berhasil membebani kernel meskipun aplikasi Honeypot mengalami *lag*.

## 📸 Dokumentasi / Bukti

*(Simpan screenshot kamu di folder bernama 'images' dan link di sini)*

| Log Brute Force | CPU Usage saat DoS |
| :---: | :---: |
| ![Log Bukti](images/log-bruteforce.png) | ![CPU Load](images/cpu-dos.jpg) |

## 🚀 Cara Menjalankan (Replikasi)

1.  **Instalasi Dependencies (Debian):**
    ```bash
    sudo apt install git python3-venv python3-pip libssl-dev libffi-dev build-essential
    ```
2.  **Setup User & Clone Cowrie:**
    ```bash
    sudo adduser --disabled-password cowrie
    sudo su - cowrie
    git clone [http://github.com/cowrie/cowrie](http://github.com/cowrie/cowrie)
    ```
3.  **Virtual Environment & Config:**
    ```bash
    cd cowrie
    python3 -m venv cowrie-env
    source cowrie-env/bin/activate
    pip install -r requirements.txt
    cp etc/cowrie.cfg.dist etc/cowrie.cfg # Edit port jadi 2222
    ```
4.  **Jalankan:**
    ```bash
    bin/cowrie start
    ```

## 👥 Kredit
* **Nama:** [Nama Kamu]
* **NIM:** [NIM Kamu]
* **Mata Kuliah:** Keamanan Jaringan

---
*Dibuat untuk memenuhi Tugas Besar Semester Ganjil 2026.*
