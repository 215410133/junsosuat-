# junsosuat-
UTS sistem Terdistribusi dan terdesentralisasi
NAMA: JUNSO SUAT
NIM: 245410133

UTS Sistem Terdistribusi
1.  Teorema CAP dan BASE
.Teorema CAP dan BASE adalah konsep fundamental dalam desain sistem terdistribusi, terutama dalam konteks basis data.

Teorema CAP (Consistency, Availability, Partition Tolerance)
Teorema CAP, juga dikenal sebagai Teorema Brewer, menyatakan bahwa sistem terdistribusi tidak mungkin secara bersamaan menjamin tiga properti berikut:

Consistency (Konsistensi): Setiap pembacaan akan menerima data yang paling baru ditulis atau mendapatkan error. Dalam sistem terdistribusi, ini berarti semua node harus melihat data yang sama pada saat yang sama.

Availability (Ketersediaan): Setiap permintaan (request) non-error ke sistem akan menerima respons (tanpa menjamin bahwa data yang dikembalikan adalah yang paling baru). Artinya, sistem harus selalu aktif dan merespons.

Partition Tolerance (Toleransi Partisi): Sistem terus beroperasi meskipun terjadi kegagalan komunikasi (partisi jaringan) antara node-node sistem. Ini adalah prasyarat untuk setiap sistem terdistribusi yang nyata.

Keterbatasan: Dalam skenario adanya partisi jaringan, teorema CAP memaksa kita untuk memilih antara Konsistensi (C) atau Ketersediaan (A).

Pilih C dan P (CP System): Jika terjadi partisi, sistem akan memblokir atau mengembalikan error pada node yang tidak dapat disinkronkan. Ini menjamin konsistensi, tetapi mengorbankan ketersediaan pada node yang terisolasi. Contoh: Basis data SQL tradisional (MySQL, PostgreSQL), atau basis data NoSQL seperti MongoDB (sebelum sharding), HBase.

Pilih A dan P (AP System): Jika terjadi partisi, sistem akan terus menerima permintaan dan merespons, bahkan jika node tersebut mungkin mengembalikan data stale (kadaluarsa). Ini menjamin ketersediaan, tetapi mengorbankan konsistensi. Contoh: Basis data NoSQL seperti Cassandra, Couchbase, Redis (dalam mode cluster).
Prinsip BASE (Basically Available, Soft State, Eventual Consistency)
BASE adalah filosofi desain yang sering dianut oleh sistem AP (seperti NoSQL) sebagai alternatif terhadap ACID (yang terkait erat dengan sistem CP). BASE merupakan akronim dari:

Basically Available (Tersedia Secara Dasar): Sistem menjamin ketersediaan. Kegagalan sebagian mungkin terjadi, tetapi sistem secara keseluruhan tetap responsif. Ini selaras dengan properti Availability (A) dari CAP.

Soft State (Keadaan Lunak): Status sistem dapat berubah seiring waktu meskipun tidak ada input eksternal, karena perubahan sedang menyebar secara internal (eventual consistency).

Eventual Consistency (Konsistensi Akhir): Jika tidak ada penulisan data baru, sistem pada akhirnya akan menjadi konsisten; semua replika data akan bertemu pada nilai yang sama. Ini adalah kompromi terhadap Strong Consistency (C) dari CAP.

Keterkaitan CAP dan BASE
Teorema CAP menjelaskan batasan yang melekat pada sistem terdistribusi (harus memilih C atau A ketika P terjadi), sedangkan prinsip BASE menjelaskan cara sistem AP mengatasi batasan tersebut.

Sistem yang memilih A dan P dalam Teorema CAP sering kali mengadopsi filosofi BASE.

Basically Available dari BASE selaras dengan Availability dari CAP.

Eventual Consistency dari BASE adalah konsekuensi dari memilih Availability dan mengorbankan Strong Consistency selama partisi jaringan.

💡 Contoh Penggunaan
Saya pernah menggunakan MongoDB dalam arsitektur yang sangat terdistribusi.

Pada dasarnya, MongoDB adalah sistem yang cenderung CP. Ketika menggunakan Replica Set tanpa sharding yang ketat, Primary node harus mengonfirmasi penulisan sebelum replica mencerminkannya, dan jika primary gagal, pemilihan primary baru dapat menyebabkan downtime singkat (mengorbankan A untuk C).

Namun, ketika diimplementasikan dengan Sharding, MongoDB dapat dikonfigurasi untuk menjadi lebih AP (mengadopsi prinsip BASE). Dengan Write Concern yang rendah, sharding memungkinkan sistem untuk terus menulis data bahkan jika beberapa shard mengalami partisi atau tidak tersedia sepenuhnya (meningkatkan A). Dalam kasus ini, konsistensi data di seluruh shard hanya Eventually Consistent (BASE) dan bukan konsistensi yang kuat (Strong Consistency). Ini adalah contoh klasik dari penerapan BASE di bawah kendala CAP.

2. GraphQL dan Komunikasi Antar Proses (IPC)
GraphQL adalah bahasa kueri untuk API dan runtime untuk memenuhi kueri tersebut dengan data yang sudah ada. Meskipun sering digunakan untuk komunikasi Client-Server, GraphQL juga memiliki keterkaitan penting dalam komunikasi antar proses (Inter-Process Communication/IPC) dalam sistem terdistribusi, terutama dalam arsitektur Microservices.

Keterkaitan GraphQL dan IPC dalam Microservices
Dalam arsitektur microservices, aplikasi dibagi menjadi layanan-layanan kecil yang independen (proses) yang berkomunikasi satu sama lain. GraphQL bertindak sebagai lapisan Agregator Data atau API Gateway yang menyederhanakan dan mengoptimalkan bagaimana klien (atau proses lain) mendapatkan data dari berbagai microservices.

Peran GraphQL:

1. Orchestration dan Data Aggregation: Ketika klien membuat kueri tunggal, server GraphQL bertindak sebagai proses perantara. Ia akan memecah kueri tersebut menjadi kueri/permintaan yang lebih kecil dan mengirimkannya ke berbagai microservices (proses internal) menggunakan mekanisme IPC, seperti HTTP (REST/gRPC), Message Queuing, atau Remote Procedure Call (RPC). Ini mencegah klien harus berinteraksi dengan banyak microservices secara langsung.

2. Efficient Data Fetching (No Over/Under Fetching): GraphQL memungkinkan klien untuk menentukan secara tepat data yang dibutuhkan. Hal ini mengurangi overhead komunikasi antar proses internal karena server GraphQL hanya perlu mengambil field data spesifik dari microservice yang relevan.

3. Schema Stitching/Federation: Dalam implementasi yang lebih canggih, GraphQL dapat menggabungkan (stitch) schema dari berbagai microservice yang berbeda (setiap microservice mengekspos endpoint GraphQL-nya sendiri) menjadi schema terpadu tunggal. Klien hanya berinteraksi dengan schema terpadu, sementara komunikasi IPC internal (antar microservice dan Gateway) menangani resolusi field.

kode diagram keterkaitan. 
graph TD
    A[Client App] -->|GraphQL Query| B(GraphQL API Gateway / Server);
    B -->|IPC: Internal HTTP/gRPC/RPC| C[Microservice A (Users)];
    B -->|IPC: Internal HTTP/gRPC/RPC| D[Microservice B (Orders)];
    B -->|IPC: Internal HTTP/gRPC/RPC| E[Microservice C (Inventory)];
    C --> F((DB A));
    D --> G((DB B));
    E --> H((DB C));
    B -->|GraphQL Result| A;

    subgraph Distributed System (Microservices)
        C
        D
        E
        F
        G
        H
    end

    style B fill:#f9f,stroke:#333,stroke-width:2px;
    style A fill:#ccf,stroke:#333,stroke-width:2px;
    
Keterangan Diagram:

1.Client App mengirimkan satu GraphQL Query ke GraphQL API Gateway.
2. API Gateway adalah proses tunggal yang menerima kueri. Ini adalah titik awal IPC untuk internal.
3. Gateway memecah dan mendelegasikan pengambilan data ke Microservice A, B, dan C menggunakan mekanisme IPC (misalnya, HTTP di jaringan internal).
4. Setiap Microservice mengambil datanya dari basis data masing-masing.
5. Gateway mengumpulkan (aggregates) data dari ketiga microservice dan mengembalikannya sebagai satu GraphQL Result ke Klien.

3. Tentu, ini adalah jawaban untuk Ujian Tengah Semester Anda mengenai konsep sistem terdistribusi. Saya akan menyusun jawaban ini menggunakan format Markdown agar mudah dibaca.

1. Teorema CAP dan BASE
Teorema CAP dan BASE adalah konsep fundamental dalam desain sistem terdistribusi, terutama dalam konteks basis data.

📜 Teorema CAP (Consistency, Availability, Partition Tolerance)
Teorema CAP, juga dikenal sebagai Teorema Brewer, menyatakan bahwa sistem terdistribusi tidak mungkin secara bersamaan menjamin tiga properti berikut:

Consistency (Konsistensi): Setiap pembacaan akan menerima data yang paling baru ditulis atau mendapatkan error. Dalam sistem terdistribusi, ini berarti semua node harus melihat data yang sama pada saat yang sama.

Availability (Ketersediaan): Setiap permintaan (request) non-error ke sistem akan menerima respons (tanpa menjamin bahwa data yang dikembalikan adalah yang paling baru). Artinya, sistem harus selalu aktif dan merespons.

Partition Tolerance (Toleransi Partisi): Sistem terus beroperasi meskipun terjadi kegagalan komunikasi (partisi jaringan) antara node-node sistem. Ini adalah prasyarat untuk setiap sistem terdistribusi yang nyata.

Keterbatasan: Dalam skenario adanya partisi jaringan, teorema CAP memaksa kita untuk memilih antara Konsistensi (C) atau Ketersediaan (A).

Pilih C dan P (CP System): Jika terjadi partisi, sistem akan memblokir atau mengembalikan error pada node yang tidak dapat disinkronkan. Ini menjamin konsistensi, tetapi mengorbankan ketersediaan pada node yang terisolasi. Contoh: Basis data SQL tradisional (MySQL, PostgreSQL), atau basis data NoSQL seperti MongoDB (sebelum sharding), HBase.

Pilih A dan P (AP System): Jika terjadi partisi, sistem akan terus menerima permintaan dan merespons, bahkan jika node tersebut mungkin mengembalikan data stale (kadaluarsa). Ini menjamin ketersediaan, tetapi mengorbankan konsistensi. Contoh: Basis data NoSQL seperti Cassandra, Couchbase, Redis (dalam mode cluster).


Shutterstock
🧱 Prinsip BASE (Basically Available, Soft State, Eventual Consistency)
BASE adalah filosofi desain yang sering dianut oleh sistem AP (seperti NoSQL) sebagai alternatif terhadap ACID (yang terkait erat dengan sistem CP). BASE merupakan akronim dari:

Basically Available (Tersedia Secara Dasar): Sistem menjamin ketersediaan. Kegagalan sebagian mungkin terjadi, tetapi sistem secara keseluruhan tetap responsif. Ini selaras dengan properti Availability (A) dari CAP.

Soft State (Keadaan Lunak): Status sistem dapat berubah seiring waktu meskipun tidak ada input eksternal, karena perubahan sedang menyebar secara internal (eventual consistency).

Eventual Consistency (Konsistensi Akhir): Jika tidak ada penulisan data baru, sistem pada akhirnya akan menjadi konsisten; semua replika data akan bertemu pada nilai yang sama. Ini adalah kompromi terhadap Strong Consistency (C) dari CAP.

🔗 Keterkaitan CAP dan BASE
Teorema CAP menjelaskan batasan yang melekat pada sistem terdistribusi (harus memilih C atau A ketika P terjadi), sedangkan prinsip BASE menjelaskan cara sistem AP mengatasi batasan tersebut.

Sistem yang memilih A dan P dalam Teorema CAP sering kali mengadopsi filosofi BASE.

Basically Available dari BASE selaras dengan Availability dari CAP.

Eventual Consistency dari BASE adalah konsekuensi dari memilih Availability dan mengorbankan Strong Consistency selama partisi jaringan.

💡 Contoh Penggunaan
Saya pernah menggunakan MongoDB dalam arsitektur yang sangat terdistribusi.

Pada dasarnya, MongoDB adalah sistem yang cenderung CP. Ketika menggunakan Replica Set tanpa sharding yang ketat, Primary node harus mengonfirmasi penulisan sebelum replica mencerminkannya, dan jika primary gagal, pemilihan primary baru dapat menyebabkan downtime singkat (mengorbankan A untuk C).

Namun, ketika diimplementasikan dengan Sharding, MongoDB dapat dikonfigurasi untuk menjadi lebih AP (mengadopsi prinsip BASE). Dengan Write Concern yang rendah, sharding memungkinkan sistem untuk terus menulis data bahkan jika beberapa shard mengalami partisi atau tidak tersedia sepenuhnya (meningkatkan A). Dalam kasus ini, konsistensi data di seluruh shard hanya Eventually Consistent (BASE) dan bukan konsistensi yang kuat (Strong Consistency). Ini adalah contoh klasik dari penerapan BASE di bawah kendala CAP.

2. GraphQL dan Komunikasi Antar Proses (IPC)
GraphQL adalah bahasa kueri untuk API dan runtime untuk memenuhi kueri tersebut dengan data yang sudah ada. Meskipun sering digunakan untuk komunikasi Client-Server, GraphQL juga memiliki keterkaitan penting dalam komunikasi antar proses (Inter-Process Communication/IPC) dalam sistem terdistribusi, terutama dalam arsitektur Microservices.

🤝 Keterkaitan GraphQL dan IPC dalam Microservices
Dalam arsitektur microservices, aplikasi dibagi menjadi layanan-layanan kecil yang independen (proses) yang berkomunikasi satu sama lain. GraphQL bertindak sebagai lapisan Agregator Data atau API Gateway yang menyederhanakan dan mengoptimalkan bagaimana klien (atau proses lain) mendapatkan data dari berbagai microservices.

Peran GraphQL:

Orchestration dan Data Aggregation: Ketika klien membuat kueri tunggal, server GraphQL bertindak sebagai proses perantara. Ia akan memecah kueri tersebut menjadi kueri/permintaan yang lebih kecil dan mengirimkannya ke berbagai microservices (proses internal) menggunakan mekanisme IPC, seperti HTTP (REST/gRPC), Message Queuing, atau Remote Procedure Call (RPC). Ini mencegah klien harus berinteraksi dengan banyak microservices secara langsung.

Efficient Data Fetching (No Over/Under Fetching): GraphQL memungkinkan klien untuk menentukan secara tepat data yang dibutuhkan. Hal ini mengurangi overhead komunikasi antar proses internal karena server GraphQL hanya perlu mengambil field data spesifik dari microservice yang relevan.

Schema Stitching/Federation: Dalam implementasi yang lebih canggih, GraphQL dapat menggabungkan (stitch) schema dari berbagai microservice yang berbeda (setiap microservice mengekspos endpoint GraphQL-nya sendiri) menjadi schema terpadu tunggal. Klien hanya berinteraksi dengan schema terpadu, sementara komunikasi IPC internal (antar microservice dan Gateway) menangani resolusi field.

📊 Diagram Keterkaitan
Cuplikan kode

graph TD
    A[Client App] -->|GraphQL Query| B(GraphQL API Gateway / Server);
    B -->|IPC: Internal HTTP/gRPC/RPC| C[Microservice A (Users)];
    B -->|IPC: Internal HTTP/gRPC/RPC| D[Microservice B (Orders)];
    B -->|IPC: Internal HTTP/gRPC/RPC| E[Microservice C (Inventory)];
    C --> F((DB A));
    D --> G((DB B));
    E --> H((DB C));
    B -->|GraphQL Result| A;

    subgraph Distributed System (Microservices)
        C
        D
        E
        F
        G
        H
    end

    style B fill:#f9f,stroke:#333,stroke-width:2px;
    style A fill:#ccf,stroke:#333,stroke-width:2px;
Keterangan Diagram:

Client App mengirimkan satu GraphQL Query ke GraphQL API Gateway.

API Gateway adalah proses tunggal yang menerima kueri. Ini adalah titik awal IPC untuk internal.

Gateway memecah dan mendelegasikan pengambilan data ke Microservice A, B, dan C menggunakan mekanisme IPC (misalnya, HTTP di jaringan internal).

Setiap Microservice mengambil datanya dari basis data masing-masing.

Gateway mengumpulkan (aggregates) data dari ketiga microservice dan mengembalikannya sebagai satu GraphQL Result ke Klien.

3. Streaming Replication PostgreSQL dengan Docker Compose
Streaming Replication di PostgreSQL memungkinkan sinkronisasi data dari satu server Primary ke satu atau lebih server Standby (Replika). Ini sangat penting untuk high availability dan failover.

Berikut adalah langkah-langkah untuk menyiapkan streaming replication menggunakan Docker Compose. membuat satu Primary dan satu Standby (hot standby).
Langkah 1: Persiapan File docker-compose.yaml
Kita akan mendefinisikan dua layanan: primary_db dan standby_db.
kode YAML
version: '3.8'

services:
  primary_db:
    image: postgres:15-alpine
    container_name: primary_db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
      # Konfigurasi Replication
      # WAL (Write-Ahead Log) level harus replica atau lebih tinggi
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256" 
    ports:
      - "5432:5432"
    volumes:
      - primary_data:/var/lib/postgresql/data
      - ./primary_conf:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - postgres_net

  standby_db:
    image: postgres:15-alpine
    container_name: standby_db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
      # Variabel ini akan memicu inisialisasi sebagai Standby
      PGPRIMARYHOST: primary_db
      PGPRIMARYUSER: replicator
      PGPRIMARYPORT: 5432
      PGPASSWORD: password # Password untuk user replicator
    ports:
      - "5433:5432"
    volumes:
      - standby_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5
    depends_on:
      primary_db:
        condition: service_healthy
    networks:
      - postgres_net

networks:
  postgres_net:

volumes:
  primary_data:
  standby_data:
Langkah 2: Konfigurasi Primary (WAL & User Replication)
Kita perlu membuat file konfigurasi agar Primary mengizinkan koneksi replikasi. Buat direktori primary_conf dan dua file di dalamnya:
A. primary_conf/01-postgres.conf
File ini mengatur konfigurasi PostgreSQL untuk replikasi.
program:
# 01-postgres.conf
wal_level = replica
max_wal_senders = 10    # Jumlah proses maksimum untuk mengirim WAL
wal_keep_size = 64MB    # Menjaga WAL agar Standby tidak ketinggalan
hot_standby = on        # Untuk memungkinkan kueri pada Standby (Hot Standby)
listen_addresses = '*'

B. primary_conf/02-pg_hba.conf
File ini mengatur otorisasi untuk koneksi replikasi.

PROGRAM:

# 02-pg_hba.conf
host    all             all             0.0.0.0/0               scram-sha-256
host    replication     replicator      0.0.0.0/0               scram-sha-256

C. primary_conf/03-init-replication.sh

PROGRAM:
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'password';
EOSQL
Langkah 3: Eksekusi dan Verifikasi
A. JALANKAN COMPOSE
PROGRAM:

docker compose up -d

B. **Verifikasi Sinkronisasi (di Primary)
Setelah container berjalan, Anda dapat memeriksa status koneksi replikasi pada primary_db:**
PROGRAM:

docker exec -it primary_db psql -U user -d mydatabase -c "SELECT pid, usename, client_addr, application_name, state, sync_state FROM pg_stat_replication;"

Penjelasan Sinkronisasi: 
Jika berhasil, Anda akan melihat satu baris di mana application_name adalah standby_db dan sync_state adalah sync (sinkron) atau async (asinkron), menunjukkan bahwa standby_db telah terhubung dan menerima stream WAL (Write-Ahead Log).

C. Uji Data (Sinkronisasi)
Buat tabel dan isi data di Primary:
PROGRAM:
docker exec -it primary_db psql -U user -d mydatabase -c "CREATE TABLE test_sync (id int, data text);"
docker exec -it primary_db psql -U user -d mydatabase -c "INSERT INTO test_sync VALUES (1, 'Data dari Primary');"

2. Cek data di Standby:
   PROFRAM:

   # Koneksi ke Standby menggunakan port 5433
psql -h localhost -p 5433 -U user -d mydatabase -c "SELECT * FROM test_sync;"

Hasil dan Penjelasan: 
Jika replikasi berhasil, perintah di atas akan mengembalikan (1, 'Data dari Primary'). Ini menunjukkan bahwa semua perubahan yang terjadi pada Primary (transaksi, DDL, DML) telah direplikasi secara real-time (streaming) ke Standby melalui pengiriman WAL. Ini adalah esensi dari Sinkronisasi Streaming Replication di PostgreSQL.


