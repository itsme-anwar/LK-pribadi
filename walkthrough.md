# Walkthrough: Pembaruan Fitur Aplikasi Pengelola Keuangan Pribadi

Pembaruan telah selesai disesuaikan pada file utama aplikasi GitHub Pages ([`index.html`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi/index.html)) serta versi PHP lokal ([`index.php`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi-php/index.php) & [`api.php`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi-php/api.php)).

---

## 📋 Status Pembaruan & Konfigurasi Aktif

### 1. Tabel Baru HUTANG (HUT, ZIS, DC) [Fitur Terbaru]
- **Posisi Tabel**:
  - Ditempatkan tepat di atas tabel **Transaksi Extra** (antara tabel *3. Transaksi Lainnya* dan tabel *Transaksi Extra*).
- **Sumber Data & Kategori**:
  - Menampilkan kategori: `HUT` (Hutang), `ZIS` (Zakat/Infaq/Shodaqoh), dan `DC` (Dana Cadangan).
  - Ketiga kategori ini kini dipisahkan dari tabel *Transaksi Lainnya* agar data tidak berulang dan fokus terkumpul pada tabel khusus HUTANG.
- **Struktur Kolom & Perhitungan**:
  1. **Kategori**: Menampilkan nama kategori laporan (`HUTANG`, `ZIS`, `DC`) beserta nama pelengkap dalam tanda kurung jika ada.
  2. **Bulan Lalu**: Akumulasi saldo sebelum bulan berjalan. Khusus untuk `ZIS`, otomatis mencakup akumulasi alokasi 3% dari kategori `IN` pada bulan-bulan sebelumnya.
  3. **Masuk**: Pemasukan pada bulan berjalan. Khusus untuk `ZIS`, otomatis terhitung sebesar 3% dari total Penghasilan (`IN`) bulan tersebut.
  4. **Keluar**: Pengeluaran riil pada bulan berjalan.
  5. **Selisih**: Dihitung berdasarkan rumus saldo mutasi berjalan:
     $$\text{Selisih} = \text{Bulan Lalu} + \text{Masuk} - \text{Keluar}$$
- **Baris Total (`tfoot`)**:
  - Menampilkan rekapitulasi `TOTAL HUTANG`: Total Bulan Lalu, Total Masuk, Total Keluar, dan Total Selisih.
- **Dukungan Cetak PDF**:
  - Karena berada dalam kontainer laporan utama, tabel HUTANG otomatis tercetak secara rapi saat tombol **🖨️ Cetak PDF** ditekan.

---

### 2. Fitur Edit Batas Anggaran Bulanan (Budgeting)
- **Tombol Edit (✏️) pada Kartu Anggaran**:
  - Pada setiap kartu anggaran di panel **Batas Anggaran Bulanan (Budgeting)**, terdapat tombol Edit (`✏️`) di samping tombol Hapus (`🗑️`).
- **Modal Dinamis Tambah / Edit Anggaran**:
  - Saat tombol Edit ditekan, modal anggaran terbuka dengan formulir yang otomatis terisi data kategori dan nominal limit yang telah disetel sebelumnya.
  - Judul modal otomatis berubah menjadi **✏️ Edit Batas Anggaran Bulanan** dan tombol submit menjadi **Perbarui Anggaran**.
  - Saat tombol "+ Pasang Anggaran" ditekan untuk menambah anggaran baru, formulir dan modal otomatis dikembalikan ke status awal (**🎯 Pasang Batas Anggaran Bulanan**).
- **Penanganan Pembaruan Data**:
  - **Versi Supabase (`index.html`)**: Memperbarui record anggaran terpilih dengan query `update({ category_id, monthly_limit }).eq('id', editId)`. Jika kategori diubah ke kategori lain yang sudah memiliki anggaran, duplikat lama dibersihkan secara otomatis.
  - **Versi PHP / MySQL (`index.php` & `api.php`)**: Endpoint `save_budget` kini menerima parameter `id` / `edit_id`. Jika `id` disediakan, API melakukan validasi duplikasi kategori dan memperbarui `category_id` serta `monthly_limit` dengan aman.

---

### 3. Ekspor Excel/CSV & Cetak PDF Sesuai Filter pada Tabel Riwayat Mutasi
- **Ekspor Excel/CSV Dinamis (`exportFilteredTransactions`)**:
  - Tombol **📊 Ekspor Excel/CSV** pada kartu Riwayat Mutasi Transaksi **hanya mengekspor transaksi yang cocok dengan kriteria filter aktif**:
    - **Bulan & Tahun**: Bulan terpilih atau seluruh bulan pada tahun berjalan.
    - **Kategori**: Semua kategori atau kategori spesifik yang difilter (misal `ZIS`, `HOM`, `TAG`).
    - **Rekening**: Semua rekening atau rekening tertentu (misal `CASH`, `BCA`).
    - **Folder**: Semua folder atau kode folder tertentu (misal `📁 F01`).
    - **Pencarian Teks**: Transaksi yang mengandung kata kunci pencarian.
  - Kolom CSV rapi dan lengkap: `No`, `Tanggal`, `Rekening`, `Kategori`, `Deskripsi`, `Kode File`, `Masuk`, `Keluar`, `Nominal`.
  - Penamaan file otomatis mengikuti filter aktif, contoh: `mutasi_2026-03_zis_cash_f01_2026-09-20.csv`.
- **Cetak PDF Lengkap Sesuai Filter (`triggerPrint('transactions')`)**:
  - Tombol **🖨️ Cetak PDF** pada Riwayat Mutasi Transaksi mencetak **seluruh data transaksi hasil filter** (tidak lagi terpotong hanya 15 baris halaman pertama).
  - Tampilan cetak PDF profesional dengan:
    1. **Header Lengkap**: Menampilkan judul laporan, tanggal/waktu cetak, serta badge parameter filter aktif.
    2. **3 Widget Ringkasan**: Menampilkan **Total Masuk** (termasuk perhitungan otomatis 3% IN jika filter ZIS), **Total Keluar**, dan **Selisih Bersih (Netto)**.
    3. **Tabel Mutasi Penuh**: Nomor urut, Tanggal, Rekening, Kategori, Deskripsi, Folder, dan Nominal dengan warna dan tanda (+/-).
    4. **Footer Tabel**: Rekapitulasi Total Masuk, Total Keluar, dan Selisih.

---

### 4. Perhitungan Otomatis 3% Kategori IN pada Widget MASUK untuk Filter ZIS
- Saat pengguna memilih filter **Kategori = ZIS** pada tabel Riwayat Mutasi Transaksi (Tab 1):
  - Widget **📥 MASUK** secara otomatis menghitung dan menampilkan **3% dari total Penghasilan (kategori IN)** pada bulan/periode yang dipilih:
    $$\text{MASUK}_{\text{ZIS}} = 3\% \times \text{Total IN Periode Tersebut}$$
  - Ditampilkan badge penanda **`(3% IN)`** serta tooltip informasi di samping label MASUK.
  - Widget **📤 KELUAR** menghitung riil seluruh transaksi penyaluran/pengeluaran kategori ZIS pada periode tersebut.
  - Widget **⚖️ SELISIH** menghitung sisa alokasi dana ZIS:
    $$\text{SELISIH}_{\text{ZIS}} = \text{MASUK}_{\text{ZIS}} - \text{KELUAR}_{\text{ZIS}}$$

---

### 5. Penambahan 3 Widget Ringkasan Dinamis di Atas Baris Filter Transaksi
- Di atas baris filter tabel riwayat transaksi (Tab 1), terdapat 3 widget ringkasan:
  - **📥 MASUK**: Menghitung total transaksi pemasukan (`amount > 0`) dari data hasil filter.
  - **📤 KELUAR**: Menghitung total transaksi pengeluaran (`amount < 0`) dari data hasil filter.
  - **⚖️ SELISIH**: Menghitung selisih bersih ($\text{MASUK} - \text{KELUAR}$).
- Bekerja interaktif terhadap seluruh filter: Rekening, Kategori, Folder, Bulan & Tahun, serta pencarian teks.

---

### 6. Pembatasan Kategori Pie Chart Pengeluaran Bulanan
- Data diagram pie chart (**Pengeluaran per Kategori**) dikunci secara ketat hanya mengambil nilai pengeluaran dari 7 kategori pokok & operasional:
  $$\text{HOM}, \text{AYH}, \text{TAG}, \text{WJB}, \text{TLG}, \text{BANK}, \text{HUT}$$

---

### 7. Penyesuaian Penamaan Kategori pada Batas Anggaran
- `TAG` &rarr; **TAGIHAN**
- `HOM` &rarr; **HOME**
- `AYH` &rarr; **PRIVATE**
- `WJB` &rarr; **RUTIN**

---

### 8. Saldo Bulan Lalu Tidak Dimasukkan ke Saldo Akhir Status Rekening
- Pada tabel **1a. Status Rekening (Bulan Berjalan)**:
  $$\text{Saldo Akhir} = \text{Masuk (Bln Ini)} - \text{Keluar (Bln Ini)}$$
- Saldo akumulasi penuh dipantau di tabel **1b. Akumulasi Rekening Tahun Berjalan (YTD)**.

---

### 9. Penghapusan Kolom Bulan Lalu pada Tabel Transaksi Extra
- Pada tabel **Transaksi Extra**, kolom **Bulan Lalu** ditiadakan sehingga tabel tampil dengan 4 kolom ringkas:
  $$\text{Kategori} \quad | \quad \text{Pemasukan} \quad | \quad \text{Pengeluaran} \quad | \quad \text{Saldo}$$

---

### 10. Penyeragaman Warna Angka Positif (+) dan Negatif (-)
- **Angka Positif (+)** menggunakan warna hijau (`text-emerald-600`).
- **Angka Negatif / Pengeluaran (-)** menggunakan warna merah (`text-rose-600`).
- **Batas Anggaran (Budget)** tetap berwarna hitam netral (`text-slate-800`).

---

### 11. Menu Fleksibel Penempatan Kategori & Rekening pada 6 Tabel Laporan [Fitur Terbaru]
- **Tabel-Tabel yang Didukung**:
  1. **Status Rekening (Bulan Berjalan)**
  2. **Akumulasi Rekening Kas Utama (YTD)**
  3. **Pengeluaran Pokok & Wajib**
  4. **Transaksi Lainnya**
  5. **HUTANG**
  6. **Transaksi Extra**
- **Akses Pengaturan Cepat & Terpusat**:
  - **Tombol Langsung di Header Tabel (`Tab 2: Laporan`)**: Di samping judul tiap-tiap tabel, tersedia tombol `⚙️ Atur Rekening` atau `⚙️ Atur Kategori`.
  - **Panel Manajemen Terpusat (`Tab 3: Pengaturan & Master Data`)**: Ditambahkan kartu **🗂️ Pengaturan Penempatan Kategori & Rekening pada Tabel Laporan** dengan 6 tombol navigasi cepat ke masing-masing tabel.
- **Modal Konfigurasi Interaktif (`tableConfigModal`)**:
  - **Daftar Item Aktif**: Menampilkan seluruh kategori/rekening yang saat ini sedang aktif dihitung pada tabel tersebut.
  - **Keluarkan Item (`✕ Keluarkan`)**:
    - Untuk rekening: Mengeluarkan rekening dari perhitungan tabel terkait (Status Rekening atau YTD).
    - Untuk kategori: Mengeluarkan kategori dari tabel (otomatis dipindahkan ke kelompok *Transaksi Lainnya* atau disembunyikan).
  - **Pindahkan Kategori Antar-Tabel (`Pindah ke...`)**: Dropdown pemindahan langsung untuk menukar posisi kategori ke *Pengeluaran Pokok*, *Transaksi Lainnya*, *HUTANG*, *Transaksi Extra*, atau disembunyikan.
  - **Tambah Item Baru ke Tabel**: Dropdown pilihan untuk menambahkan kategori atau rekening yang belum berada di tabel tersebut.
- **Sinkronisasi & Persistensi**:
  - **Versi Supabase (`index.html`)**: Menyimpan preferensi kelompok kategori langsung ke kolom `group_type` pada tabel `categories` Supabase serta cadangan `localStorage`.
  - **Versi PHP / MySQL (`index.php` & `api.php`)**: Endpoint `update_category_group` memperbarui kolom `group_type` kategori di MySQL secara permanen. Preferensi rekening tabel Status dan YTD disimpan tersinkron di `localStorage`.

---

### 12. Penyesuaian Kolom ke-3 pada Menu "Tambah Kategori Baru" [Fitur Terbaru]
- Pada formulir **Tambah Kategori Baru** (Tab 3), pilihan pada kolom ke-3 (`newCategoryGroup`) telah diperbarui sesuai dengan 4 kelompok tabel laporan:
  1. **Pengeluaran Pokok** (masuk ke tabel *Pengeluaran Pokok & Wajib*)
  2. **Transaksi Lainnya** (masuk ke tabel *Transaksi Lainnya*)
  3. **HUTANG** (masuk ke tabel *HUTANG*)
  4. **Transaksi Extra** (masuk ke tabel *Transaksi Extra*)
- Saat kategori baru ditambahkan, sistem langsung menempatkannya ke dalam tabel laporan yang dipilih, menyinkronkannya ke basis data, dan memperbarui rekapitulasi laporan secara real-time.

---

### 13. Penghapusan Kolom "Saldo Bulan Lalu" pada Tabel Status Rekening [Pembaruan Terkini]
- Pada tabel **1a. Status Rekening (Bulan Berjalan)**:
  - Kolom **Saldo Bulan Lalu** telah dihapus dari `<thead>`, `<tbody>`, dan `<tfoot>`.
  - Tabel kini tampil lebih ringkas dan fokus dengan **4 kolom**:
    $$\text{Rekening} \quad | \quad \text{Masuk (Bln Ini)} \quad | \quad \text{Keluar (Bln Ini)} \quad | \quad \text{Saldo}$$
  - Logika kalkulasi tabel menghitung seluruh mutasi rekening bulan berjalan untuk rekening yang terdaftar di konfigurasi *Status Rekening*, dengan format selisih mutasi bulan berjalan ($\text{Masuk} - \text{Keluar}$) dan baris total footer yang selaras.

---

### 14. Widget Kecil Saldo Bulan Ini untuk 5 Rekening Utama (CASH, SPAY, JAGO, GAJI, nGAJI) [Pembaruan Terkini]
- Ditempatkan tepat **di bawah widget akumulasi pemasukan dan pengeluaran** bulanan dalam format **1 baris (5 kolom)**:
  1. 💵 **CASH**
  2. 📱 **SPAY** (mendukung kode `SPAY` / `SPY`)
  3. 🏦 **JAGO**
  4. 💼 **GAJI**
  5. 💳 **nGAJI** (mendukung kode `nGAJI` / `NGAJI` / `NGJ` / `NON-GAJI`)
- **Fitur & Perilaku Widget**:
  - Menampilkan **Saldo Bulan Berjalan** ($\text{Masuk Bulan Ini} - \text{Keluar Bulan Ini}$) dari masing-masing rekening kas utama.
  - Warna dan format Rupiah interaktif dan real-time:
    - **Hijau (`+`)**: jika saldo bulan berjalan positif / surplus.
    - **Merah (`-`)**: jika saldo bulan berjalan negatif / defisit.
    - **Netral (`Rp 0`)**: jika tidak ada mutasi atau seimbang.
  - Otomatis tersinkronisasi saat filter bulan/tahun diganti atau transaksi baru dicatat/diedit.

---

### 15. Penambahan Kolom NOTE dan Balon Percakapan Komik (Comic Speech Bubble) pada Kolom Folder [Pembaruan Terkini]
- **Formulir Input & Edit Transaksi**:
  - Baris input transaksi disesuaikan menjadi:
    - **Baris 2**: Deskripsi (2 kolom) dan Nominal Masuk/Keluar/Transfer (1 kolom).
    - **Baris 3**: Kode FILE (Folder) (1 kolom) dan **💬 NOTE (Catatan)** (2 kolom).
  - Kolom **NOTE** bersifat opsional dan dapat diisi dengan catatan keterangan tambahan.
  - Mendukung input transaksi baru, transfer antar-rekening, edit transaksi lama (`triggerEdit`), dan reset formulir.
- **Tampilan Balon Komik pada Kolom Folder**:
  - Header kolom pada tabel riwayat transaksi dinamai **Folder**.
  - Jika transaksi memiliki catatan (`note`):
    - Muncul ikon kecil balon percakapan **`💬`** di samping nama folder (atau berdiri sendiri jika tanpa folder).
    - Saat mouse diarahkan ke ikon (**hover / mouseover**) atau ditekan (**click**), muncul **balon percakapan komik (`comic speech bubble`)**:
      - Desain khas komik: border hitam tegas (`border-2 border-slate-900`), sudut membulat, bayangan tebal pop-art (`shadow-[0_10px_25px_rgba(0,0,0,0.22)]`), header bernada kartun `💭 NOTE TRANSAKSI`, dan teks catatan yang rapi.
      - Dilengkapi **ekor balon komik (triangle tail pointer)** yang secara dinamis dan presisi mengarah ke ikon 💬 yang sedang disorot.
      - Menggunakan positioning `fixed` di `<body>` dengan `getBoundingClientRect()`, sehingga **tidak terpotong (`anti-clipping`)** oleh batas `overflow-x-auto` kontainer tabel.
      - Dilengkapi proteksi flip arah cerdas: jika ikon berada terlalu dekat dengan batas atas layar, balon komik otomatis muncul di bagian bawah ikon dengan ekor yang membalik ke atas.
- **Pencarian Teks & Filter (`searchTerm`)**:
  - Kolom pencarian riwayat transaksi secara otomatis mencocokkan kata kunci ke dalam isi teks catatan (`NOTE`).
- **Dukungan Cetak PDF & Ekspor CSV**:
  - Pada cetak dokumen PDF, catatan ditampilkan secara rapi di kolom Folder: `📁 [FOLDER] (Catatan)`.
  - Pada ekspor Excel/CSV, ditambahkan kolom khusus `"Catatan"`.
  - Pada impor CSV, kolom `Catatan` atau `NOTE` otomatis dibaca dan disimpan.
- **Kompatibilitas Penuh (Dual-Stack)**:
  - **Versi PHP / MySQL (`index.php`, `api.php`, `database.sql`)**: Kolom `note TEXT NULL` di database MySQL dengan auto-migration saat API dijalankan.
  - **Versi Supabase (`index.html`)**: Mendukung kolom native `note` di Supabase serta graceful fallback otomatis berbasis format `[NOTE: ...]` pada `description` jika skema Supabase pengguna belum menjalankan migrasi DDL.

### 16. Pemilihan Bulan & Perhitungan Pengeluaran Bulan Lalu Murni pada Tabel Rekapitulasi Kode FILE [Pembaruan Terkini]
- **Tujuan**:
  - Menyajikan komparasi pengeluaran per Folder/FILE yang akurat dan berimbang antara **Bulan Terpilih ($M$)** dengan **Bulan Sebelumnya ($M-1$)**.
  - Mengizinkan pengguna memilih bulan yang ingin diinspeksi langsung dari tabel Rekapitulasi Kode FILE tanpa harus mengubah filter utama halaman jika tidak diinginkan.
- **Perubahan Utama**:
  1. **Penggantian Dropdown "Urutkan:" Menjadi Pilihan "📅 Bulan:"**:
     - Di header kartu Rekapitulasi Kode FILE, dropdown selector kini berisi pilihan **12 Bulan (Januari - Desember)**.
     - Saat memilih bulan di dropdown ini, tabel secara instan menghitung dan menampilkan pengeluaran folder untuk bulan terpilih tersebut dan bulan sebelumnya.
     - Ketika filter bulan di bagian atas dashboard diubah, tabel ini secara otomatis tersinkronisasi mengikuti bulan laporan aktif.
  2. **Perhitungan Murni Pengeluaran Bulan Lalu ($M-1$)**:
     - *Sebelumnya*: Kolom "Bulan Lalu" menghitung seluruh akumulasi saldo masa lalu ($< \text{Bulan Terpilih}$).
     - *Sekarang*: Kolom "Bulan Lalu" hanya menghitung transaksi pengeluaran (`tx.amount < 0`) yang terjadi tepat pada satu bulan sebelum bulan terpilih ($M-1$, dengan penanganan pergantian tahun misalnya Januari $M-1$ adalah Desember tahun sebelumnya).
     - Kolom "Bulan Ini" menghitung transaksi pengeluaran (`tx.amount < 0`) pada bulan terpilih ($M$).
  3. **Label Header Kolom Dinamis Berbasis Nama Bulan**:
     - Header kolom secara otomatis menampilkan nama bulan yang relevan:
       $$\text{Nama File} \quad | \quad \text{Bulan Lalu (Nama Bulan } M-1\text{)} \quad | \quad \text{Bulan Ini (Nama Bulan } M\text{)} \quad | \quad \text{Jumlah Semua}$$
       *(Contoh: "Bulan Lalu (Agustus)" dan "Bulan Ini (September)")*.
  4. **Pengurutan Baris Tetap Fleksibel via Klik Header Kolom (`<th>`)**:
     - Pengguna tetap dapat mengurutkan baris kapan saja dengan mengklik langsung judul kolom (*Nama File*, *Bulan Lalu*, *Bulan Ini*, *Jumlah Semua*).
     - Dilengkapi indikator arah aktif (`▲` untuk terkecil/A-Z, `▼` untuk terbanyak/Z-A, `⇅` untuk kolom yang tidak aktif).
     - Preferensi sorting disimpan di `localStorage` (`file_recap_sort_by`) dan default terurut berdasarkan pengeluaran bulan ini terbanyak (`current_desc`).
- **Tersinkronisasi Penuh di Seluruh Versi**:
  - `keuangan-pribadi/index.html` (Versi Supabase / Cloud)
  - `keuangan-pribadi/index-note.html`
  - `keuangan-pribadi-php/index.php` (Versi PHP / MySQL)

### 17. Penambahan Kolom SELISIH dengan Indikator Panah Kenaikan/Penurunan pada Rekapitulasi Kode FILE [Pembaruan Terkini]
- **Tujuan**:
  - Memberikan visualisasi cepat terhadap tren pengeluaran setiap Folder/FILE: apakah pengeluaran bulan ini mengalami penurunan (hemat) atau peningkatan (boros) dibandingkan dengan bulan lalu.
- **Fitur & Perilaku Kolom SELISIH**:
  - **Posisi Kolom**: Terletak tepat di antara kolom **Bulan Ini** dan kolom **Jumlah Semua**:
    $$\text{Nama File} \quad | \quad \text{Bulan Lalu} \quad | \quad \text{Bulan Ini} \quad | \quad \mathbf{\text{Selisih}} \quad | \quad \text{Jumlah Semua}$$
  - **Perhitungan Nilai Selisih**:
    - Dihitung sebagai selisih pengeluaran: $|\text{Bulan Ini} - \text{Bulan Lalu}|$.
    - Nilai nominal angka ditampilkan dengan **warna teks hitam** (`text-slate-800`).
  - **Indikator Ikon Segitiga Dinamis**:
    - 🟢 **Segitiga ke bawah HIJAU (`▼`)**: Jika pengeluaran bulan ini **lebih rendah** dari bulan lalu (pengeluaran berhasil ditekan/berkurang).
    - 🔴 **Segitiga ke atas MERAH (`▲`)**: Jika pengeluaran bulan ini **lebih tinggi** dari bulan lalu (pengeluaran membengkak/meningkat).
    - ⚪ **Tanda strip (`-`)**: Jika pengeluaran bulan ini tepat sama dengan bulan lalu.
  - **Baris Total Footer**:
    - Kolom footer menghitung total selisih pengeluaran keseluruhan dengan warna angka hitam tegas dan ikon segitiga `▼` / `▲` yang sesuai.
  - **Dukungan Pengurutan (Sorting)**:
    - Header kolom **Selisih** dapat diklik langsung untuk mengurutkan baris:
      - `diff_desc`: Pos dengan kenaikan pengeluaran terbesar di urutan paling atas.
      - `diff_asc`: Pos dengan penurunan pengeluaran terbesar di urutan paling atas.
      - Disertai indikator panah arah aktif (`▲`, `▼`, `⇅`).
- **Tersinkronisasi Penuh di Seluruh Versi**:
  - `keuangan-pribadi/index.html` (Versi Supabase / Cloud)
### 18. Perhitungan Kolom "Jumlah Semua" dari Januari hingga Bulan Terpilih [Pembaruan Terkini]
- **Tujuan**:
  - Menyajikan akumulasi pengeluaran riil setiap Folder/FILE sepanjang tahun berjalan (*Year-to-Date / YTD*), yaitu dari **bulan Januari** hingga **bulan terpilih**, bukan sekadar penjumlahan bulan lalu dan bulan ini saja.
- **Logika Perhitungan Baru**:
  - Untuk setiap transaksi pengeluaran (`tx.amount < 0`):
    - Transaksi diperiksa tanggalnya pada tahun terpilih (`selYear`): `tx.date.startsWith(`${selYear}-`)`.
    - Diambil nomor bulannya (`txM = parseInt(tx.date.slice(5, 7), 10)`).
    - Jika `txM >= 1` dan `txM <= selMonthNum`, maka mutasi pengeluaran tersebut (`Math.abs(amt)`) diakumulasikan ke dalam nilai `totalYtd` folder bersangkutan.
  - Nilai pada kolom **Jumlah Semua** dan baris footer **Total Pengeluaran Folder** menampilkan total akumulasi dari Januari s/d bulan terpilih tersebut.
  - Sorting berdasarkan kolom **Jumlah Semua** (`total_desc` / `total_asc`) otomatis mengurutkan berdasarkan akumulasi YTD Januari hingga bulan terpilih.
### 19. Pembaruan Keterangan Grafik Pie, Urutan Rekening, Persentase Pengeluaran Pokok & Opsi "SETAHUN" [Pembaruan Terkini]
- **1. Keterangan Warna Grafik Pie Dinamis (List & Persen)**:
  - Keterangan warna pie chart (Doughnut Chart) dipindahkan dari legend standar ke **format list interaktif** di bawah grafik (`#categoryChartLegendList`).
  - Dilengkapi label nama kategori, kotak warna swatch, nominal rupiah riil, dan **persentase (%) di ujung kanan**.
  - Diurutkan secara dinamis dari **pengeluaran terbesar di urutan paling atas** (descending order).
- **2. Standarisasi Urutan Nama Rekening**:
  - Urutan rekening pada tabel **1a. Status Rekening (Bulan Berjalan)** dan **1b. Akumulasi Rekening Kas Utama (YTD)** serta **5 Widget Saldo Dashboard** telah distandarisasi secara seragam:
    1. `CASH`
    2. `SPAY` (alias: `SPY`)
    3. `JAGO`
    4. `Non-GAJI` (alias: `NGJ`, `NGAJI`, `NON-GAJI`, `NON GAJI`)
    5. `GAJI`
- **3. Persentase Kategori Real & Urutan Tabel Pengeluaran Pokok (Tabel 2)**:
  - Kolom **Jumlah Real** pada masing-masing kategori pengeluaran pokok dilengkapi persentase proporsi dalam kurung, contoh: `Rp 2.500.000 (35.2%)`, serta `(100%)` pada baris total footer.
  - Urutan baris kategori pengeluaran pokok dikunci sesuai urutan:
    1. `HOME` (HOM)
    2. `PRIVATE` (AYH / PRV)
    3. `TAGIHAN` (TAG)
    4. `RUTIN` (WJB)
- **4. Penambahan Opsi "SETAHUN" (Akumulasi 1 Tahun) pada Pilihan Bulan**:
  - Dropdown filter bulan di tab laporan (`filterMonthNum`) dan dropdown rekapitulasi folder (`fileRecapMonthSelect`) dilengkapi pilihan **`SETAHUN (Akumulasi 1 Tahun)`**.
  - Saat dipilih:
    - Seluruh tabel laporan menghitung akumulasi tahun berjalan (Januari - Desember).
    - Batas anggaran bulanan dikalikan 12 ($\times 12$).
    - Kolom saldo lalu/awal mengacu pada saldo sebelum 1 Januari tahun terpilih.
    - Tabel Rekapitulasi Kode FILE membandingkan pengeluaran Tahun Berjalan vs Tahun Sebelumnya.
- **Tersinkronisasi Penuh di Seluruh Versi**:
  - `keuangan-pribadi/index.html` (Versi Supabase / Cloud)
  - `keuangan-pribadi/index-note.html`
  - `keuangan-pribadi-php/index.php` (Versi PHP / MySQL)

### 20. Pembaruan Menu Cetak PDF Ramping, Spasi Rapat Legend Pie, Nilai Bulan Lalu Kategori HUTANG, & Perbaikan Filter Rekap Tahunan [Pembaruan Terkini]
- **1. Menu Cetak PDF Ramping & Filter 1 Baris Penuh**:
  - Pilihan **Bulan**, **Tahun**, dan tombol **Cetak PDF (`📄 PDF`)** disatukan secara kompak dalam **1 baris horisontal tunggal** (`flex-nowrap gap-1.5 sm:gap-2 shrink-0`).
  - Menghilangkan *wrapping* / terbelah menjadi 2 baris sehingga jarak vertikal (*vertical space*) menjadi sangat ramping, rapi, dan hemat tempat.
  - Menggunakan ikon dokumen PDF yang tegas (`📄 PDF`) dengan aksen warna merah khas PDF (`bg-red-600 hover:bg-red-700 text-white`).
- **2. Spasi Legend Grafik Pie Dirapatkan (Hingga 50%)**:
  - Jarak vertikal antar baris item pada legend list grafik pie dirapatkan secara signifikan dari `py-1.5` (~32px) menjadi `py-0.5` (~18px), memangkas tinggi baris sebesar ~44% - 50%.
  - Kotak warna swatch diperkecil menjadi `w-2.5 h-2.5`, badge persentase di ujung kanan dibuat lebih kompak (`px-1.5 py-0 leading-tight text-[11px]`), dan margin atas kontainer dirapatkan ke `mt-1.5`.
- **3. Nilai Bulan Lalu pada Kategori HUTANG**:
  - Mengatasi kendala nilai Bulan Lalu yang sempat bernilai Rp 0 pada baris kategori `HUTANG` (`HUT`).
  - Memperbaiki pengelompokan kategori dengan self-healing group type sehingga kategori `HUT`, `HUTANG`, `ZIS`, dan `DC` selalu diakui dalam kelompok `hutang`.
  - Normalisasi otomatis kode `HUTANG` menjadi `HUT` pada pembacaan transaksi dan mapping kategori.
  - Nilai mutasi masa lalu sebelum periode berjalan (`isPastPeriod`) secara otomatis dan akurat terakumulasi ke dalam kolom **Bulan Lalu** pada kategori HUTANG, dan kolom **Selisih** dihitung berdasarkan saldo mutasi berjalan ($\text{Bulan Lalu} + \text{Masuk} - \text{Keluar}$).
- **4. Perbaikan Bug Filter Tahun Rekapitulasi Tahunan**:
  - Menghilangkan *issue* dropdown tahun terpental kembali ke 2026 setelah beberapa saat dan tabel tidak berubah saat memilih tahun lain.
  - Mengganti `onchange="loadTransactions()"` pada `<select id="filterYear">` dengan fungsi khusus `onFilterYearChange()` yang secara instan menghitung ulang `updateSummariesAndReports()` dan `renderCharts()` tanpa *roundtrip* fetch Supabase / reload berulang.
  - Memisahkan secara tegas cakupan tahun laporan utama (`yVal`) untuk mutasi rekening kas utama (YTD) dan tahun yang dipilih (`selectedYear`) untuk Tabel Rekapitulasi Tahunan (Tabel 5) serta grafik Arus Kas Tahunan (12 Bulan).
### 21. Pemadatan Jarak Spasi Antar-Baris (~40%) pada Tabel Riwayat Mutasi dan Semua Tabel Tab Laporan [Pembaruan Terkini]
- **Tujuan**:
  - Memadatkan tampilan vertikal seluruh tabel data utama sehingga informasi lebih banyak terlihat dalam satu layar tanpa perlu banyak scroll, lebih efisien, dan tetap nyaman dibaca (*compact & clean layout*).
- **1. Tabel Riwayat Mutasi (Tab Transaksi)**:
  - Header (`thead`) dan baris data (`tbody`): padding vertikal diturunkan dari `py-3` (12px atas/bawah) menjadi `py-1.5` (6px atas/bawah).
  - Tombol aksi (*Edit ✏️* dan *Hapus 🗑️*): padding tombol disesuaikan dari `p-1.5` menjadi `p-1` agar tinggi baris tidak tertahan oleh tombol.
  - Hasil: Ketinggian baris berkurang ~40% - 50%, menghasilkan daftar mutasi yang rapi, padat, dan proporsional.
- **2. Semua Tabel di Tab Laporan (Tab Laporan & Rekapitulasi)**:
  - Meliputi seluruh tabel laporan:
    1. **Tabel 1a. Status Rekening (Bulan Berjalan)**
    2. **Tabel 1b. Akumulasi Rekening Kas Utama (YTD)**
    3. **Tabel 2. Pengeluaran Pokok & Wajib**
    4. **Tabel 3. Transaksi Lainnya**
    5. **Tabel 3b. HUTANG (HUT, ZIS, DC)**
    6. **Tabel 3c. Transaksi Extra**
    7. **Tabel Rekapitulasi Kode FILE**
    8. **Tabel 4. PENGHASILAN**
    9. **Tabel 5. Rekapitulasi Tahunan**
  - Pada seluruh header (`th`), baris data (`td` di `tbody`), dan baris total footer (`td` di `tfoot`):
    - Padding vertikal dipangkas dari `py-2.5` (10px atas/bawah) menjadi `py-1.5` (6px atas/bawah).
    - Penurunan tepat sebesar:
      $$\frac{10\text{px} - 6\text{px}}{10\text{px}} \times 100\% = 40\%$$
- **Tersinkronisasi Penuh di Seluruh Versi**:
  - `keuangan-pribadi/index.html` (Versi Supabase / Cloud)
  - `keuangan-pribadi/index-note.html`
  - `keuangan-pribadi-php/index.php` (Versi PHP / MySQL)

### 22. Perbaikan Saldo BANK di Transaksi Extra (Absolut Rp 0) & Kebebasan Pemindahan Kategori HUTANG [Pembaruan Terkini]
- **1. Saldo Kategori BANK pada Tabel Transaksi Extra (Absolut Rp 0)**:
  - **Masalah Sebelumnya**: Ketika kategori `BANK` dipindahkan dari *Transaksi Lainnya* ke tabel *Transaksi Extra*, saldo kategori BANK menghitung selisih $\text{Pemasukan} - \text{Pengeluaran}$, dan selisihnya ikut menambahkan total saldo Transaksi Extra.
  - **Perbaikan**:
    - Nilai saldo untuk kategori `BANK` kini **selalu dikunci absolut Rp 0** pada tabel Transaksi Extra (`reportExtraBody`), sama seperti perilakunya di tabel Transaksi Lainnya.
    - Total saldo footer Transaksi Extra (`sumExtraSaldo`) mengecualikan saldo kategori BANK sehingga total saldo akhir tabel Transaksi Extra tetap murni mencerminkan transaksi extra non-bank.
    - Proteksi serupa juga dipasang pada tabel HUTANG (`reportHutangBody`) untuk memastikan jika kategori BANK dipindahkan ke tabel mana pun, saldonya tetap bernilai 0.
- **2. Kebebasan Pemindahan Kategori HUTANG ke Tabel Mana Pun**:
  - **Masalah Sebelumnya**: Kategori `HUTANG` (`HUT`) tidak bisa dipindahkan dari posisinya di tabel *HUTANG*, baik ketika dipindahkan ke *Transaksi Extra* maupun ke *Transaksi Lainnya*. Setiap kali dipindahkan, kategori tersebut kembali lagi ke tabel HUTANG karena adanya pengecekan *hardcode* dan *self-healing* yang menimpa preferensi pengguna.
  - **Akar Masalah & Perbaikan**:
    1. **Fungsi `getCategoryGroup` & `loadMasterData`**: Menghapus baris kode yang secara paksa mengembalikan grup ke `'hutang'` jika kategori bernilai `HUT`/`HUTANG`. Preferensi grup yang dipilih pengguna kini dihormati sepenuhnya.
    2. **Sinkronisasi Alias `HUT` dan `HUTANG`**: Pembaruan grup kategori kini menyinkronkan kedua kode alias (`HUT` dan `HUTANG`) sekaligus di `localStorage`, Supabase, dan API PHP MySQL (`api.php`) agar tidak terjadi inkonsistensi alias.
    3. **Penentuan Kode Grup (`updateSummariesAndReports`)**: Menghilangkan klausul *hardcode* `|| ['HUT', 'ZIS', 'DC'].includes(code)`. Sekarang kategori `HUT` hanya masuk ke tabel laporan yang sesuai dengan grup pilihannya (`getCategoryGroup(code)`).
    4. **Perhitungan Mutasi Transaksi**: Pada pengelompokan mutasi `isPastPeriod` (bulan lalu) dan `inReportPeriod` (bulan berjalan), transaksi kategori `HUT` kini mengalir secara dinamis ke `extraData` jika dipindah ke Transaksi Extra, ke `group2Data` jika dipindah ke Transaksi Lainnya, atau ke `group1Data` jika dipindah ke Pengeluaran Pokok.
    5. **Dukungan Modal Atur Kategori**: Modal Atur Kategori (`openTableConfigModal`) pada seluruh tabel kini mengenali seluruh kode kategori sistem (`HOM`, `AYH`, `TAG`, `WJB`, `HUT`, `ZIS`, `DC`, `INT`, `-`, `TLG`, `SVG`, `BANK`) sehingga perpindahan antar tabel melalui dropdown *"Pindah ke..."* maupun tombol *"+ Tambah Kategori"* bekerja mulus dua arah.
- **Tersinkronisasi Penuh di Seluruh Versi**:
  - `keuangan-pribadi/index.html` (Versi Supabase / Cloud)
  - `keuangan-pribadi/index-note.html`
  - `keuangan-pribadi-php/index.php` (Versi PHP / MySQL)
  - `keuangan-pribadi-php/api.php` (Backend API PHP)

### 23. Pembaruan 6 Fitur & Tampilan Laporan Keuangan [Pembaruan Terkini]
- **1. Judul Kolom Tabel Transaksi Extra & Transaksi Lainnya**:
  - Kolom **Pemasukan** telah diubah menjadi **Masuk**.
  - Kolom **Pengeluaran** telah diubah menjadi **Keluar**.
  - Judul kolom kini lebih singkat, padat, dan seragam dengan tabel HUTANG dan tabel rekap lainnya.
- **2. Penyederhanaan Form Tambah Kategori Baru**:
  - Menghilangkan dropdown pemilihan kelompok (`newCategoryGroup` / *"akan dimasukkan ke mana"*).
  - Form input kini langsung fokus pada kolom **Kode** dan **Nama/Keterangan**.
  - Kategori baru yang ditambahkan langsung otomatis tersedia di list pilihan kategori pada dropdown form input transaksi (`#category_id`). Penempatan kelompok laporan dapat diatur fleksibel kapan saja melalui tombol *⚙️ Atur Kategori*.
- **3. Saldo Bulan Lalu Kategori HUTANG di Tabel Transaksi Lainnya**:
  - Memperluas cakupan `carryForwardCodes` menjadi `['TLG', 'SVG', 'HUT', 'ZIS', 'DC']`.
  - Jika kategori `HUTANG` (`HUT`) dipindahkan ke tabel *Transaksi Lainnya*, nilai saldo **Bulan Lalu** kini tetap dihitung dari akumulasi transaksi masa lalu dan ditampilkan dalam format mata uang yang benar, bukan tanda strip (`-`).
  - Alokasi otomatis 3% dari kategori `IN` untuk kategori `ZIS` juga tetap aktif dan berjalan normal jika `ZIS` dipindahkan ke tabel Transaksi Lainnya.
- **4. Pie Chart Proporsi Pengeluaran Lebih Rapi & Dilengkapi Garis Penunjuk**:
  - Diameter pie chart diperkecil (`radius: '65%'`, `cutout: '55%'`) agar terdapat ruang yang cukup untuk label dan garis tunjuk.
  - Ditambahkan plugin kustom Chart.js (`pointerLines`) yang secara otomatis menggambar garis siku berujung label persentase (`${item.pct}%`).
  - Warna garis penunjuk dan teks persentase otomatis mengikuti warna potongan (*slice*) dan *legend* masing-masing kategori pengeluaran.
- **5. Pemadatan Widget Batas Anggaran & Informasi Persentase Kelebihan**:
  - Spasi kartu batas anggaran (*budgeting*) diperkecil dan diperpadat (`p-2.5`, `space-y-1.5`, progress bar `h-1.5`).
  - Ketika pengeluaran melebihi limit anggaran (*over budget*), kartu secara otomatis menampilkan persentase kelebihannya, contoh: `Kelebihan: Rp 250.000 (+12.5%)`.
- **6. Widget Saldo Tab Laporan (HUTANG, TALANGAN, ZIS, DC)**:
  - Di atas navigasi tab, disediakan dua set kontainer widget:
    - `#widgetContainerRekening`: Menampilkan 5 rekening utama (CASH, SPAY, JAGO, Non-GAJI, GAJI).
    - `#widgetContainerLaporan`: Menampilkan 4 saldo kategori utama (🏷️ HUTANG, 🤝 TALANGAN, 🌙 ZIS, 🛡️ DC).
  - Ketika pengguna membuka tab **Laporan & Anggaran** (`switchTab('report')`), widget rekening secara otomatis berganti menjadi widget saldo **HUTANG, TALANGAN, ZIS, dan DC**.
  - Ketika berpindah kembali ke tab **Mutasi & Input** atau **Master Data**, widget secara otomatis kembali menampilkan status 5 rekening utama.

---

## 📁 File yang Dimodifikasi

1. [`keuangan-pribadi/index.html`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi/index.html) *(Aplikasi web Supabase: 6 pembaruan fitur & tampilan)*
2. [`keuangan-pribadi/index-note.html`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi/index-note.html) *(Salinan identik index.html tersinkronisasi)*
3. [`keuangan-pribadi-php/index.php`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi-php/index.php) *(Tampilan PHP Native: 6 pembaruan fitur & tampilan)*
4. [`keuangan-pribadi-php/api.php`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi-php/api.php) *(Backend PHP)*
5. [`keuangan-pribadi-php/database.sql`](file:///C:/Users/MIMIN/.gemini/antigravity/scratch/keuangan-pribadi-php/database.sql) *(Skema DDL MySQL)*


