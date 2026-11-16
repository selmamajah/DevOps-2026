# DevOps & DevSecOps Lab — 2026

Repositori ini memuat implementasi praktikum mata kuliah **DevSecOps dan Cloud Infrastructure**, yang mencakup arsitektur *Centralized Logging*, *Resource & Container Monitoring*, sistem *Alerting*, serta integrasi analitik multi-service berbasis Docker.

* **Penulis:** Selma Humayra ([@selmamajah](https://github.com/selmamajah))
* **Institusi:** Politeknik Elektronika Negeri Surabaya (PENS)
* **Tahun Akademik:** 2026

---

## Struktur Repositori

```text
DevOps-2026/
├── bab-06/                 # Modul Centralized Logging (Studi Kasus PMB)
│   ├── app/                # Source code Registration API, Auth Service, & Worker
│   ├── fluent-bit/         # Konfigurasi parser dan pipeline log Fluent Bit
│   ├── init-app/           # Skrip inisialisasi basis data aplikasi
│   ├── init-log/           # Skrip inisialisasi basis data log terpusat
│   ├── queries/            # Query SQL untuk analisis insiden keamanan
│   ├── scripts/            # Skrip otomatisasi validasi, verifikasi, dan demo
│   ├── compose.yaml        # Definisi multi-container stack Bab 6
│   └── .env.example        # Template konfigurasi variabel lingkungan
├── bab-07/                 # Modul Resource Monitoring & Alerting
│   ├── prometheus/         # Konfigurasi Prometheus, recording rules, dan alert rules
│   │   ├── rules/          # Rule PromQL turunan dan alert threshold
│   │   └── tests/          # Unit test deklaratif promtool
│   ├── grafana/            # Provisioning deklaratif data sources & dashboards
│   │   ├── dashboards/     # Definisi JSON Dashboard Bab 7 Overview
│   │   └── provisioning/   # Deklarasi data sources (Prometheus & PostgreSQL)
│   ├── scripts/            # Skrip verifikasi stack dan koneksi cross-network
│   ├── compose.yaml        # Definisi stack monitoring terisolasi
│   └── .env.example        # Template konfigurasi akun Grafana & PostgreSQL
├── reports/                # Laporan praktikum dan dokumen evaluasi akademik
│   ├── bab-06/             # Laporan Praktikum Bab 6 (.docx & .md)
│   └── bab-07/             # Laporan Praktikum Bab 7
├── .gitignore              # Proteksi kebocoran kredensial dan artefak runtime
└── README.md               # Dokumentasi utama repositori
```

---

## Ringkasan Modul

### 1. Bab 06 — Centralized Logging dengan Fluent Bit & PostgreSQL

Modul ini mengimplementasikan pemisahan basis data transaksional dan basis data log, serta pengumpulan log otomatis (*log forwarding*) dari berbagai microservices secara *asynchronous* dan aman.

* **Komponen:**
  * `registration-api` & `auth-service`: Layanan aplikasi PMB berbasis Python/Flask.
  * `verification-worker`: Worker pemrosesan verifikasi berkas secara background.
  * `fluent-bit`: Forwarder log yang membaca log container dari Docker socket/JSON log dan meneruskannya ke PostgreSQL.
  * `app-db` & `log-db`: Dua instance PostgreSQL 16 terisolasi untuk memisahkan data transaksi dari data audit/log.
* **Cara Menjalankan:**
  ```bash
  cd bab-06
  cp .env.example .env
  docker compose up -d
  ./scripts/verify.sh
  ```

---

### 2. Bab 07 — Resource Monitoring dengan Prometheus, cAdvisor, Node Exporter, dan Grafana

Modul ini mengimplementasikan observabilitas metrik host Linux dan container Docker secara real-time, evaluasi ambang batas (*alerting*), serta dashboard visualisasi interaktif.

* **Komponen:**
  * `prometheus`: Time-series database penarik (*pull*) metrik dengan interval scrape 15s.
  * `node-exporter`: Kolektor metrik resource host sistem operasi Linux.
  * `cadvisor`: Kolektor metrik konsumsi resource per kontainer (CPU, memory, throttled time).
  * `grafana`: Platform visualisasi analitik terpadu (dashboard metrik PromQL & log SQL).
  * `load-generator`: Kontainer simulasi beban komputasi CPU untuk menguji alerting threshold.
* **Alur Arsitektur:**
  ```mermaid
  flowchart TD
      N["Node Exporter"] -->|"host metrics"| P["Prometheus"]
      C["cAdvisor"] -->|"container metrics"| P
      P -->|"PromQL"| G["Grafana"]
      D["PostgreSQL Bab 6"] -.->|"log query (read-only)"| G
      L["Load generator"] -->|"resource pressure"| C
  ```
* **Cara Menjalankan:**
  ```bash
  cd bab-07
  cp .env.example .env
  docker compose up -d
  ./scripts/verify-stack.sh
  ```
* **Integrasi dengan Bab 06:**
  Menghubungkan Grafana dengan database log Bab 6 menggunakan role *least privilege* (`grafana_reader`):
  ```bash
  ./scripts/connect-bab6.sh
  ```

---

## Penerapan Prinsip DevSecOps

1. **Principle of Least Privilege (PoLP):**
   * Integrasi monitoring dengan log database menggunakan akun basis data read-only (`grafana_reader`) dengan hak akses terbatas hanya pada `SELECT` skema `logs`.
   * Layanan Grafana dan Node Exporter dijalankan dengan `security_opt: no-new-privileges:true`.
2. **Network Segregation (Segmentasi Jaringan):**
   * Exporter internal (`cadvisor:8080`, `node-exporter:9100`) tidak dipublikasikan ke host port publik; hanya Prometheus yang dapat mengakses melalui jaringan bridge internal.
3. **Secret & Credential Management:**
   * Seluruh rahasia, password database, dan admin credential didelegasikan ke file `.env` lokal yang diproteksi oleh `.gitignore`. File yang dipublikasikan ke repositori hanya berupa template `.env.example`.
4. **Reproducibility & Immutable Infrastructure:**
   * Seluruh container image dipin (*pinned tags*) untuk menjamin konsistensi build dan deployment antar lingkungan.
   * Dashboard dan Data Source Grafana diprovisi secara deklaratif (*Infrastructure as Code*) melalui YAML.

---

## Lisensi & Hak Cipta

Proyek ini dikembangkan untuk keperluan akademik praktikum DevSecOps 2026. Seluruh hak cipta materi praktikum mengacu pada silabus perkuliahan.
