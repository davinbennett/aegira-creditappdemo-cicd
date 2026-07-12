# Aegira Credit App Demo CI/CD

`docker-compose.yml` menjalankan frontend `creditappdemo`, backend `aegira-loan-service`, PostgreSQL, dan Redis dalam satu stack. Frontend tersedia pada port `5173`, backend pada port `8080`.

## Menjalankan aplikasi lokal

Jalankan perintah berikut dari root repository di PowerShell.

| Command | Penjelasan singkat |
| --- | --- |
| `Copy-Item .env.example .env` | Membuat konfigurasi environment lokal. |
| `notepad .env` | Mengganti password database dan JWT secret. |
| `docker compose up -d --build` | Build image lalu menyalakan semua service. |
| `docker compose ps` | Melihat status container. |
| `docker compose logs -f` | Melihat log seluruh service secara langsung. |
| `docker compose down` | Menghentikan dan menghapus container serta network. |
| `docker compose down -v` | Menghapus container sekaligus data PostgreSQL dan Redis. |

Buka `http://localhost:5173` untuk frontend dan `http://localhost:8080/swagger-ui.html` untuk backend.

Frontend memakai `VITE_SCORING_MODE=mock` secara default. Endpoint `/api/v1/credit-scoring/simulate` belum tersedia pada `aegira-loan-service`, sehingga mengubahnya ke `uat` belum akan menghasilkan integrasi scoring yang berfungsi.

## Menjalankan GitHub Actions runner

Runner ini ditujukan untuk host Linux yang memiliki Docker Engine, karena memakai `/var/run/docker.sock`. `RUNNER_WORKDIR` sengaja dipasang dengan path host dan path container yang sama agar perintah Docker dari job Actions dapat mengakses checkout dengan benar.

| Command | Penjelasan singkat |
| --- | --- |
| `cp .env.runner.example .env.runner` | Membuat file konfigurasi runner di host Linux. |
| `chmod 600 .env.runner` | Membatasi akses file token runner. |
| `nano .env.runner` | Mengisi PAT GitHub baru dan menyesuaikan nama runner bila perlu. |
| `docker compose --env-file .env.runner -f docker-compose.runner.yml up -d` | Mendaftarkan dan menjalankan self-hosted runner. |
| `docker compose --env-file .env.runner -f docker-compose.runner.yml logs -f` | Melihat proses registrasi dan log runner. |
| `docker compose --env-file .env.runner -f docker-compose.runner.yml down` | Menghentikan runner. |

Sebelum menjalankan deploy, buat repository secrets `POSTGRES_PASSWORD` dan `JWT_SECRET` di GitHub. Workflow `.github/workflows/ci-cd.yml` menjalankan test backend dan frontend pada GitHub-hosted runner, lalu pada push ke `main` menjalankan deploy di runner berlabel `docker-local`.

Jangan menyimpan PAT di Git atau di GitHub Actions secret untuk bootstrap runner. Simpan hanya di `.env.runner` pada host, lalu rotasi token yang pernah terkirim di percakapan atau tersimpan di tempat lain.
