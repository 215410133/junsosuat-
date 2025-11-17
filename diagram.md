Diagram Alur Kerja (Rendered by Mermaid)
```mermaid
graph TD
    A[Mulai] --> B(Proses 1);
    B --> C{Keputusan?};
    C -- Ya --> D[Lakukan Aksi X];
    C -- Tidak --> E[Lakukan Aksi Y];
    D --> F[Selesai];
    E --> F;
