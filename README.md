# Smart Tour Router

Sistem optimalisasi rute & manajemen logistik perjalanan yang mengotomatisasi penyusunan jalur wisata multi-destinasi berdasarkan data koordinat geografis konvensional serta informasi event/kendala dinamis (seperti banjir atau penutupan wilayah).

Project ini mengimplementasikan arsitektur **polyglot persistence** yang mengintegrasikan tiga sistem basis data berbeda untuk menangani data spasial, dokumen, dan graf secara terdistribusi.

## Arsitektur Data

### 1. BaseX (XML Database & Web Service)
- Ekstraksi data spasial mentah (latitude, longitude, kategori, kota_kabupaten) dari `tourism_indonesia_main_data.csv`
- Parsing real-time agenda kebudayaan nasional & pengumuman penutupan wilayah dari `explore-indonesia.xml`
- 8 query XPath & XQuery untuk validasi keterbacaan dokumen XML
- HTTP Server BaseX (`basexhttp`) melayani 6 xQuery + 6 query berbasis API (2 GET, 2 POST, 2 Return JSON)
- Integrasi `explore-indonesia.xml` + data CSV pariwisata menjadi file XML baru: `integrated.xml`

### 2. MongoDB Atlas (Document Store)
- Skema dokumen dengan pendekatan **Normalized Reference Model** — memisahkan entitas geografis destinasi dari log pergerakan (`trip_logs.csv`)
- Distribusi beban melalui **Range Sharding** berdasarkan parameter provinsi/kota_kabupaten
- Simulasi kluster: 3 node replica, minimal 2 shard, 1 Mongos Router
- 8 query basic + 4 query agregasi geospasial (`$geoNear`) untuk mencari destinasi terdekat dari lokasi user

### 3. Neo4j Aura Cloud (Graph Database)
- Konversi hasil pemrosesan MongoDB ke representasi Graph DB
- Pipeline topologi integrasi multi-sumber data dengan 4 model relasi
- Minimal 4 graph model pendukung optimalisasi rute
- **Advanced Cypher Query — Dynamic Dijkstra Shortest Path with Constraints**: mencari rute terpendek antar destinasi lintas kota, secara dinamis memberi penalti bobot (weight penalty) atau mengabaikan node destinasi yang sedang terjangkit kendala real-time (misal: banjir/ditutup)

## Tech Stack
`BaseX` `MongoDB Atlas` `Neo4j Aura` `XQuery/XPath` `Cypher` `Postman`

## Alur Data
CSV & XML mentah → BaseX (validasi & integrasi XML) → MongoDB Atlas (dokumen + sharding) → Neo4j Aura (graph + shortest path dinamis) → Laporan
