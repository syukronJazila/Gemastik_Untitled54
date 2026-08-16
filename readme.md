# Dataset Geodemografi Koperasi Desa Merah Putih (KDMP) Kabupaten Sukoharjo

Repositori ini memuat kumpulan data geodemografi, spasial, dan infrastruktur ekonomi terintegrasi yang digunakan untuk membangun **Sistem Rekomendasi Jenis Koperasi Desa Merah Putih (KDMP)**. Pendekatan analitik pada repositori ini bertumpu pada arsitektur *Graph Attention Network v2* (GATv2Conv) dan *Gaussian Mixture Model* (GMM).

## 1. Metodologi dan Proses Konstruksi Data
Tim peneliti mengawali tahapan konstruksi dataset dengan menetapkan Kabupaten Sukoharjo sebagai wilayah studi utama. Pemilihan wilayah ini didasarkan pada tingkat ketersediaan instrumen data yang paling komprehensif, mencakup rekam data KDMP, akurasi peta geografi, serta presisi data demografi lokal. Perjalanan konstruksi data dilakukan melalui tahapan berikut:

*   **Pengumpulan Data Induk (Web-Scraping & LLM Annotation):** Langkah pertama dilakukan dengan mengekstraksi data dari portal resmi (https://trade.simkopdes.go.id). Proses *scraping* menghasilkan profil dasar seperti `cooperative_id`, koordinat (*latitude/longitude*), dan potensi lokal. Kami kemudian menggunakan *Large Language Model* (LLM) melalui antarmuka *OpenRouter API* untuk menstandarisasi pelabelan kolom potensi.
*   **Pengayaan Fitur Spasial (OpenStreetMap):** Data diperkaya secara berlapis (*layering*) menggunakan data *OpenStreetMap* (OSM) untuk mengekstraksi metrik jaringan jalan. Variabel yang didapatkan meliputi rata-rata jarak antar-KDMP, tingkat kepadatan jalan dalam radius 1 kilometer, serta proksimitas terhadap jalur transportasi utama.
*   **Pemetaan Aktivitas Ekonomi Wilayah (Google Maps API):** Kami mengekstraksi fasilitas *Point of Interest* (POI) dalam radius 1 km dari setiap titik KDMP. Proses ini berhasil menghimpun **70.343** data POI yang kemudian dikategorikan ke dalam lima kelompok: **Komersial UMKM, Logistik Eksternal, Bank Besar, Keuangan Mikro, dan Lainnya**.
*   **Feature Engineering POI:** Hasil kategorisasi POI dikonversi menjadi indikator kuantitatif berbasis jarak dan agregasi, mencakup Kepadatan Fasilitas (jumlah POI) dan Jarak Minimum (proksimitas terdekat).
*   **Integrasi Data Demografi, Pertanian, dan Eksternal Lainnya:** Struktur geodemografi disempurnakan dengan menggabungkan metrik kependudukan, pertanian, dan ekonomi dari portal data Kabupaten Sukoharjo (https://data.sukoharjokab.go.id) serta BPS Kabupaten Sukoharjo. Metrik mencakup total populasi, rasio jenis kelamin, luas wilayah, data partisipasi Keluarga Berencana, produktivitas sektor agraris (tanaman pangan), dan pendapatan asli desa.
*   **Konsolidasi & Preprocessing (Dataset Final):** Seluruh data dibersihkan dari anomali, diselaraskan indeksnya berdasarkan `cooperative_id`, dan disatukan menjadi satu *dataset* final (162 entitas KDMP) yang siap digunakan untuk *training* model GMM.

---

## 2. Struktur Repositori (Deskripsi Berkas CSV)
Repositori ini terdiri dari beberapa berkas CSV historis yang merepresentasikan tahapan pemrosesan data, hingga berkas akhir yang digunakan untuk pemodelan:

*   **`Data_Titik_Kopdes_Sukoharjo.csv`**
    Berisi data mentah hasil *web-scraping* dari portal SIMKOPDES. Memuat identitas dasar, koordinat geografis, serta hasil anotasi potensi awal menggunakan LLM.
*   **`Data_Spasial_Kopdes_Sukoharjo.csv`**
    Memuat variabel keruangan tingkat dasar, seperti konektivitas geografi, jarak ke jalan utama, dan kalkulasi jarak rata-rata antar KDMP menggunakan geometri OSM.
*   **`Data_Ruas_Jalan_Kopdes_Sukoharjo.csv`**
    Dataset teknis yang memuat metrik kepadatan ruas jaringan jalan yang dihitung secara bertahap dalam perbesaran radius (dari `ruas 100` meter hingga `ruas 1000` meter).
*   **`Data_POI_Kopdes_Sukoharjo.csv`**
    Himpunan pangkalan data masif berisi ** ~71.000** titik *Point of Interest* (POI) mentah hasil ekstraksi dari *Google Maps API*, lengkap dengan koordinat, nama tempat, ulasan, dan tag lokasi.
*   **`Data_Demografis_Sukoharjo.csv`**
    Dataset yang bersumber dari BPS dan portal daerah yang mencakup metrik kependudukan komprehensif, seperti total populasi, rasio jenis kelamin, rata-rata usia, luas wilayah, kepadatan penduduk, hingga rasio jumlah Kepala Keluarga (KK) di setiap desa/kelurahan.
*   **`Data_Pertanian_Sukoharjo.csv`**
    Dataset sektoral yang memuat indikator produktivitas agribisnis tingkat desa, mencakup metrik volume tanaman pangan, jumlah rumah tangga usaha pertanian, serta usaha pertanian perorangan.
*   **`Data_Pendapatan_Sukoharjo.csv`**
    Data agregasi finansial tingkat desa yang berisi rekam Pendapatan Asli Desa (PAD) yang berfungsi sebagai tolok ukur kapasitas perputaran ekonomi lokal.
*   **`Data_Kesehatan_Sukoharjo.csv`**
    Dataset pelengkap dari BPS yang memuat indikator kesehatan demografi, seperti tingkat partisipasi masyarakat pada program Keluarga Berencana.
*   **`Dataset_Kopdes_Sukoharjo_Final.csv`**
    **[DATASET UTAMA]** Berkas konsolidasi akhir (*feature-rich*) yang menggabungkan seluruh metrik spasial, ekonomi (POI), kependudukan, pertanian, kesehatan, dan pendapatan. Berisi **162** entitas KDMP beserta **34 variabel** terintegrasi yang siap dimasukkan ke dalam *pipeline* Machine Learning.

---

## 3. Definisi Operasional Variabel Penelitian

Berikut adalah rincian ruang lingkup dan deskripsi pengukuran untuk setiap fitur pada **Dataset Final** (`Dataset_Kopdes_Sukoharjo_Final.csv`):

| Nama Variabel | Definisi Operasional |
| :--- | :--- |
| **Variabel Identifikasi** | |
| `cooperative_id` | Identitas unik (*primary key*) dari setiap Koperasi Desa Merah Putih (KDMP) |
| **Variabel Spasial & Infrastruktur** | |
| `rata_jarak_antar_kopdes_m` | Rata-rata jarak proksimitas (dalam meter) antara satu titik KDMP dengan KDMP di sekitarnya |
| `kepadatan_jaringan_jalan` | Nilai agregat kepadatan seluruh infrastruktur jaringan jalan di sekitar titik KDMP |
| `ruas 100` s.d. `ruas 1000` | Metrik densitas infrastruktur ruas jalan yang diukur secara bertahap pada radius 100 meter hingga 1.000 meter |
| **Variabel Aktivitas Ekonomi (POI)** | |
| `Kepadatan_Komersial_UMKM` | Total jumlah fasilitas komersial, perdagangan, dan jasa (minimarket, restoran, dll) dalam radius 1 km |
| `Jarak_Min_Komersial_m` | Jarak proksimitas terdekat (dalam meter) ke titik fasilitas komersial/UMKM |
| `Kepadatan_Logistik_Eksternal` | Total jumlah fasilitas jasa pengiriman/ekspedisi (Kantor Pos, JNE, dll) dalam radius 1 km |
| `Jarak_Min_Logistik_m` | Jarak proksimitas terdekat (dalam meter) ke fasilitas jasa logistik eksternal |
| `Kepadatan_Bank_Besar` | Total jumlah titik kantor cabang bank komersial nasional atau mesin ATM dalam radius 1 km |
| `Jarak_Min_Bank_m` | Jarak proksimitas terdekat (dalam meter) ke fasilitas perbankan skala besar |
| `Kepadatan_Keuangan_Mikro` | Total jumlah lembaga keuangan skala pedesaan (BPR, Pegadaian, BMT) dalam radius 1 km |
| `Jarak_Min_Keuangan_Mikro_m` | Jarak proksimitas terdekat (dalam meter) ke lembaga permodalan/keuangan mikro |
| `Pendapatan_Asli_Desa_Rp` | Total Pendapatan Asli Desa (PAD) dalam satuan Rupiah sebagai metrik kapasitas sirkulasi kas wilayah |
| **Variabel Demografi & Kependudukan** | |
| `populasi` | Total jumlah penduduk yang bermukim di desa/kelurahan setempat |
| `persenatase perempuan` | Persentase proporsi penduduk berjenis kelamin perempuan di wilayah tersebut |
| `rata-rata usia` | Rata-rata usia harapan/profil kependudukan di wilayah desa tersebut |
| `jumlah_kk` | Total akumulasi Kepala Keluarga (KK) terdaftar di wilayah desa sasaran |
| `luas_km2` | Luas bentang wilayah administratif desa/kelurahan (dalam kilometer persegi) |
| `rata-rata anggota keluarga` | Indeks rata-rata besaran/jumlah anggota individu di dalam satu Kepala Keluarga |
| `kepadatan penduduk (jiwa/km2)` | Rasio jumlah penduduk per satu kilometer persegi ruang administratif wilayah |
| `Peserta Keluarga Berencana` | Total penduduk yang menjadi partisipan aktif program kesehatan Keluarga Berencana (KB) |
| `Bukan Peserta Keluarga Berencana` | Total penduduk usia subur yang tidak mengikuti program kesehatan Keluarga Berencana |
| **Variabel Sektor Agraris** | |
| `Tanaman Pangan` | Indikator yang merepresentasikan volume hasil/rumah tangga dari sektor agribisnis tanaman pangan |
| `Rumah Tangga Usaha Pertanian` | Total jumlah rumah tangga kependudukan yang memiliki mata pencaharian utama dalam usaha pertanian |
| `Usaha Pertanian Perorangan` | Total jumlah usaha/pengelolaan lahan pertanian yang dijalankan secara independen/perorangan |

---

## 4. Konteks Penelitian
Penelitian ini menggunakan pendekatan kuantitatif berbasis data sains yang didasarkan pada data sekunder keruangan (spasial) dan geodemografi dalam bentuk data *cross-section*. Unit analisis dalam ekosistem ini mencakup **162 entitas Koperasi Desa Merah Putih (KDMP)** yang tersebar secara geografis di seluruh kecamatan dalam kawasan wilayah studi Kabupaten Sukoharjo. 

Seluruh variabel di dalam himpunan data ini dipilih secara metodologis berdasarkan relevansinya terhadap identifikasi kelayakan dan potensi ekonomi wilayah desa, baik sebagai indikator proksimitas infrastruktur penunjang maupun sebagai faktor geodemografi yang mempengaruhi sirkulasi kapital. Dataset final tidak mengandalkan target label manual, melainkan bertindak secara holistik sebagai variabel fitur independen yang akan diekstraksi ke dalam algoritma untuk menemukan pola klasterisasi (*unsupervised learning*) guna merumuskan spesifikasi jenis operasi unit koperasi yang paling presisi.

Sumber utama dalam penelitian ini merupakan hasil harmonisasi dari ragam penyedia data terbuka (*Open Data*), di antaranya berasal dari ekstraksi *web-scraping* portal resmi simkopdes, publikasi resmi pemerintah desa Kabupaten Sukoharjo, hingga portal statistik nasional (BPS). Data spasial infrastruktur wilayah dan pemetaan persaingan bisnis (POI) divalidasi dan diolah menggunakan antarmuka *OpenStreetMap* serta pelacakan titik koordinat dari *Google Maps API*.

Definisi operasional dari setiap variabel telah dirangkum secara transparan pada tabel untuk memberikan standardisasi ruang lingkup dan kerangka pengukuran masing-masing atribut fitur dalam menyokong sistem rekomendasi ini.
