# junsosuat-
UTS sistem Terdistribusi dan terdesentralisasi
NAMA: JUNSO SUAT

NIM: 245410133

UTS Sistem Terdistribusi
Konsep Sistem Terdistribusi: CAP, BASE, dan GraphQ
 1. CAP Theorem vs. Filosofi B
CAP Theorem (Consistency, Availability, Partition Tolerance)
CAP Theorem* menyatakan bahwa dalam sistem terdistribusi, kita tidak dapat menjamin ketiga properti berikut secara bersamaan ketika terjadi pemisahan jaringan (network partition):
C — Consistency Semua pembaca melihat data yang sama setelah operasi penulisan selesai.
A — Availability Setiap permintaan menerima respons (bukan pesan timeoutatau error).
P — Partition Tolerance:** Sistem tetap berfungsi dan melanjutkan operasi meskipun terjadi kegagalan komunikasi antar node (pemisahan jaringan).
 Filosofi Desain BASE
BASE (*Basically Available, Soft-state, Eventual consistency*) adalah filosofi desain yang diadopsi oleh sistem yang memilih *Availability (A)* dan *Partition Tolerance (P)*dari CAP. Filosofi ini mengorbankan konsistensi instan demi ketersediaan yang tinggi.
*Basically Available:* Sistem menjamin sebagian layanan tersedia (walaupun mungkin tidak semua data terkini).
* **Soft-state:** Keadaan data bersifat sementara dan dapat berubah tanpa input baru hingga replikasi selesai (misalnya, data *out-of-date* di replika).
* **Eventual Consistency:** Sistem menjamin bahwa data **akhirnya** akan menjadi konsisten pada suatu waktu di masa depan, setelah partisi teratasi dan replikasi selesai.
 1.2. Keterkaitan CAP dan BASE
CAP menjelaskan *trade-off teoritis* yang harus dipilih oleh sistem terdistribusi. BASE adalah **kebijakan praktis** atau **model desain** yang secara eksplisit mengadopsi pilihan **AP** dari CAP.
| Konsep | Deskripsi | Pilihan CAP | Contoh Sistem |
| :--- | :--- | :--- | :--- |
| CAP | Teorema yang menjelaskan batas/trade-off. | AP (Availability + Partition Tolerance) | Sistem Feed/Sosial Media |
| BASE | Filosofi desain yang mewujudkan pilihan AP. | AP| MongoDB, Cassandra |
| Model CP | Memilih Konsistensi (C) ketat. | *CP* (Consistency + Partition Tolerance) | Sistem Perbankan, Zookeeper |
1.3. Contoh Nyata (Skenario AP + BASE)
Misalnya, pada aplikasi **Microblogging** (seperti yang Anda sebutkan):

* **Topologi:** Beberapa node aplikasi dan database/replika tersebar di **Region A** dan **Region B**.
* **Kebutuhan Bisnis:** Pengguna harus selalu bisa **Post** atau **Like** (Ketersediaan sangat penting).
Skenario Partisi
1.  **Network Partition Terjadi (P):** Region A dan Region B kehilangan koneksi satu sama lain.
2.  **Sistem Memilih A:** Sistem di Region A tetap menerima *postingan* baru dari pengguna lokal.
3.  **Konsistensi Terkorbankan (Non-C):** Pengguna di Region B **belum** bisa melihat *postingan* tersebut.
4.  **BASE Diterapkan:**
    * **Basically Available:** Layanan *post* tetap aktif.
    * **Eventual Consistency:** Setelah koneksi antar region terpulihkan, data di-sinkronisasi (*reconciliation*), dan akhirnya semua pengguna melihat *postingan* yang sama.
**Kontras:** Jika ini adalah sistem perbankan (**CP**), pada saat partisi, sistem di Region A mungkin akan **menolak** transaksi (mengorbankan A) demi menjamin saldo global tetap konsisten (C).
---
2. GraphQL dan Komunikasi Antar Proses (IPC)

2.1. Inti Hubungan

**GraphQL** adalah bahasa kueri dan *runtime* yang sering diimplementasikan sebagai lapisan **API Gateway** atau **Aggregator** di depan arsitektur mikroservis.
Hubungan GraphQL dengan IPC adalah sebagai berikut:
* **Dari Client ke Gateway:** Client mengirim **satu** kueri GraphQL ke *single endpoint* (API Gateway).
* **Dari Gateway ke Microservices (IPC):** Gateway GraphQL menerjemahkan kueri tunggal tersebut menjadi **beberapa panggilan internal** (IPC) ke berbagai mikroservis yang menaungi data yang diminta (misalnya, `UserService`, `PostService`, `CommentService`).
> 
**Bentuk IPC yang Digunakan:** Panggilan internal ini dapat berupa: **HTTP/REST**, **gRPC**, **RPC internal**, atau komunikasi melalui **Message Queue**.
 2.2. Manfaat dan Trade-off GraphQL
GraphQL memposisikan dirinya di antara klien dan layanan *backend* untuk mengoptimalkan komunikasi eksternal, namun hal ini memiliki implikasi pada kompleksitas IPC internal.
Manfaat
* **Single Endpoint:** Klien hanya perlu berinteraksi dengan satu URL.
* **Flexible Fetching:** Klien **secara presisi** menentukan *fields* yang dibutuhkan, mengurangi masalah **Overfetching** (mendapatkan data berlebihan) dan **Underfetching** (butuh *round-trip* tambahan).
* **Agregasi Data:** Efektif dalam menyatukan data yang tersebar di banyak layanan *backend* dalam satu respons terstruktur.
Trade-offs
* **Bottleneck Gateway:** GraphQL Gateway dapat menjadi titik tunggal kegagalan dan/atau *bottleneck* performa jika tidak di-optimasi.
* **Kompleksitas IPC:** Satu kueri kompleks dapat memicu *fan-out* banyak panggilan IPC internal, membuat *tracing* dan *observability* (latensi, *error rate*) menjadi lebih sulit.
* **Kebutuhan Optimasi:** Memerlukan teknik seperti **Caching** (di level gateway dan *backend*), **Batching** (misalnya dengan **DataLoader**) untuk mengurangi jumlah IPC yang dibuat, dan **Rate Limiting** yang ketat.
