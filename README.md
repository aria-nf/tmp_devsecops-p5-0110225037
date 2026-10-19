# Flaskr &mdash; Modern Blog & Article Management System

![DevSecOps Pipeline](https://github.com/aria-nf/devsecops-p5-0110225037/actions/workflows/devsecops.yml/badge.svg)

**Flaskr** adalah aplikasi web blogging dan manajemen konten berbasis Python & Flask. Aplikasi ini menyediakan fitur autentikasi pengguna (registrasi, login, session management), serta operasi CRUD (*Create, Read, Update, Delete*) artikel atau postingan blog secara penuh. Aplikasi ini telah diintegrasikan dengan arsitektur **DevSecOps Pipeline** lengkap menggunakan GitHub Actions, unit testing otomatis dengan Pytest (Quality Gate Code Coverage 100%), dan Static Application Security Testing (SAST) menggunakan Bandit, Semgrep, serta SonarCloud.

> **Sumber Basis Proyek (Attribution)**:
> Basis kode aplikasi web ini mengadopsi arsitektur resmi open-source dari tutorial proyek **[Pallets / Flaskr](https://github.com/pallets/flask/tree/main/examples/tutorial)** yang dikembangkan oleh Pallets Projects (pembuat resmi framework Flask), kemudian diadaptasikan dan diperkaya dengan pipeline otomatisasi keamanan DevSecOps terstandar industri.

---

## 1. Teknologi yang Digunakan

- **Bahasa & Framework**: Python 3.11 / 3.13, Flask 3.0.3
- **Basis Data**: SQLite (modul bawaan Python `sqlite3`, *zero-config relational database*)
- **Frontend**: Semantic HTML5, CSS3 Styling responsif
- **Pengujian Mutu (Testing & Quality Gate)**: Pytest 8.3.3, Pytest-Cov 5.0.0 (Coverage Gate $\ge$ 80%, tercapai 100%)
- **Keamanan Perangkat Lunak (SAST)**:
  - **Bandit**: Analisis keamanan kode statis Python (Security Gate: 0 temuan tingkat Medium/High)
  - **Semgrep**: *Multi-rule semantic scanner* untuk mendeteksi anti-pattern dan celah OWASP
  - **SonarCloud**: Analisis *code quality*, duplikasi, *maintainability*, dan *vulnerability*

---

## 2. Cara Menjalankan Aplikasi Secara Lokal

### Prasyarat
- Python versi 3.10 atau yang lebih baru.

### Langkah Instalasi & Menjalankan

```powershell
# 1. Clone repository
git clone https://github.com/aria-nf/devsecops-p5-0110225037.git
cd devsecops-p5-0110225037

# 2. Buat dan aktifkan virtual environment (opsional tapi disarankan)
py -m venv .venv
.\.venv\Scripts\Activate.ps1

# 3. Install seluruh dependensi
pip install -r requirements.txt

# 4. Inisialisasi basis data SQLite
flask --app flaskr init-db

# 5. Jalankan server aplikasi
python wsgi.py
# atau menggunakan CLI Flask:
flask --app flaskr run
```

Buka peramban (browser) dan akses: **`http://127.0.0.1:5000`**

### Akun Pengujian
Untuk mencoba fitur blog dan autentikasi:
1. Klik tombol **Register** untuk mendaftarkan akun baru.
2. Lakukan **Log In** dengan username dan password yang telah didaftarkan.
3. Anda dapat membuat artikel baru (**New**), mengedit (**Edit**), atau menghapus postingan milik Anda.

---

## 3. Menjalankan Pengujian & Security Scan Lokal

Pengembang dapat melakukan validasi mutu dan keamanan secara lokal sebelum melakukan `git push`:

```powershell
# Menjalankan unit test dengan laporan coverage dan Quality Gate (minimal 80%)
python -m pytest --cov=flaskr --cov-report=term --cov-report=xml --cov-fail-under=80

# Menjalankan pemindaian celah keamanan SAST (Bandit)
python -m bandit -r flaskr -ll
```

---

## 4. Overview Pipeline DevSecOps (CI/CD)

Repositori ini menerapkan pipeline DevSecOps terotomatisasi pada GitHub Actions ([.github/workflows/devsecops.yml](.github/workflows/devsecops.yml)) yang terpicu secara otomatis pada setiap event:
- `push` ke branch `main`
- `pull_request` ke branch `main`

### Alur Rantai Eksekusi (*Sequential Chaining*):
Runner yang digunakan adalah **`ubuntu-latest`**. Seluruh tahapan berjalan secara berantai menggunakan deklarasi ketergantungan `needs`:

1. **`build` (Build & Unit Testing)**:
   - Melakukan checkout source code dan setup Python 3.11.
   - Menginstalasi dependensi dari `requirements.txt`.
   - Mengeksekusi 24 test case Pytest dengan **Quality Gate Code Coverage minimal 80%** (pencapaian riil: 100%).
   - Menyimpan artefak laporan pengujian `coverage.xml`.

2. **`sast-bandit` (SAST Scan - Bandit)** (`needs: build`):
   - Berjalan setelah job `build` sukses.
   - Memindai seluruh kode sumber pada direktori `flaskr/` menggunakan engine Bandit.
   - Mengunggah artefak laporan `bandit-report.json`.
   - **Quality Gate**: Pipeline akan *FAIL* jika ditemukan kerentanan dengan tingkat keparahan *Medium* atau *High* (`bandit -r flaskr -ll`).

3. **`sast-semgrep` (SAST Scan - Semgrep)** (`needs: sast-bandit`):
   - Menjalankan container official `semgrep/semgrep`.
   - Memindai kepatuhan aturan keamanan kode OWASP dan CWE secara mendalam.

4. **`sonarcloud` (SonarCloud SAST)** (`needs: sast-semgrep`):
   - Menganalisis *maintainability*, *reliability*, dan *vulnerabilities* terintegrasi ke SonarCloud dashboard (berjalan kondisional saat secret `SONAR_TOKEN` aktif).

---

## 5. Struktur Berkas Repositori

```text
devsecops-p5-0110225037/
├── .github/
│   └── workflows/
│       └── devsecops.yml        # Konfigurasi pipeline GitHub Actions DevSecOps
├── flaskr/                      # Direktori utama kode sumber aplikasi (Modular)
│   ├── __init__.py              # Application factory & konfigurasi Flask
│   ├── auth.py                  # Blueprint otentikasi (Register, Login, Session)
│   ├── blog.py                  # Blueprint CRUD postingan artikel blog
│   ├── db.py                    # Utilitas koneksi & inisialisasi basis data SQLite
│   ├── schema.sql               # Skema tabel database (user, post)
│   ├── static/
│   │   └── style.css            # Berkas CSS styling antarmuka
│   └── templates/               # Berkas template antarmuka Jinja2
│       ├── base.html
│       ├── auth/
│       └── blog/
├── tests/                       # Automated Unit Test Suite (Pytest)
│   ├── conftest.py              # Pytest fixtures & mock in-memory SQLite DB
│   ├── data.sql                 # Data dummy untuk pengujian
│   ├── test_auth.py             # Pengujian skenario autentikasi & validasi input
│   ├── test_blog.py             # Pengujian fungsi CRUD artikel & otorisasi hak akses
│   ├── test_db.py               # Pengujian koneksi dan error handling database
│   └── test_factory.py          # Pengujian konfigurasi create_app & endpoint hello
├── wsgi.py                      # Entry point WSGI server
├── requirements.txt             # Daftar dependensi aplikasi dan testing
├── Dockerfile                   # Konfigurasi kontainerisasi aplikasi
└── sonar-project.properties     # Konfigurasi integrasi SonarCloud
```
.
