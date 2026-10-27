# Panduan Dasar SELECT, FROM, WHERE & Relasi Tabel

Dokumen ini berisi penjelasan singkat mengenai apa itu `SELECT`, `FROM`, dan `WHERE`, serta bagaimana cara menghubungkan tabel yang saling berelasi menggunakan Implicit JOIN beserta contoh-contoh yang diambil dari skema basis data `dbPenjualanBarang.docx`.

---

## 1. Apa itu `SELECT`?

### Penjelasan Singkat:

`SELECT` adalah perintah SQL yang digunakan untuk menentukan kolom mana saja yang ingin ditampilkan ke layar hasil query. Perhitungan aritmetika juga bisa ditulis langsung di bagian ini.

### Contoh:

- **Menampilkan seluruh kolom:**

  ```sql
  SELECT * FROM tblBarang;
  ```

  *(tanda `*` artinya tampilkan semua kolom yang ada di tabel tanpa terkecuali)*
- **Menampilkan kolom tertentu saja:**

  ```sql
  SELECT kodebarang, namabarang FROM tblBarang;
  ```
- **Melakukan perhitungan matematika di `SELECT`:**

  ```sql
  SELECT (d.jumlah * d.harga) AS Subtotal
  ```

---

## 2. Apa itu `FROM`?

### Penjelasan Singkat:

`FROM` digunakan untuk menentukan dari tabel mana data diambil. Jika data yang dibutuhkan tersebar di beberapa tabel, seluruh tabel tersebut dicantumkan setelah klausa `FROM`.

### Contoh :

- **Mengambil data dari satu tabel:**

  ```sql
  SELECT * FROM tblPelanggan;
  ```
- **Mengambil data dari beberapa tabel sekaligus:**

  ```sql
  FROM tblBarang b, tblPelanggan p, tblTransaksi t, tblDetilTransaksi d
  ```

  *(b, p, t, d adalah alias tabel agar penulisan kolom pada query berikutnya lebih ringkas)*

---

## 3. Apa itu `WHERE`?

### Penjelasan Singkat:

`WHERE` bertindak sebagai penyaring data. Klausa ini digunakan untuk menentukan syarat atau kondisi baris mana saja yang valid untuk ditampilkan. Baris data yang tidak memenuhi syarat akan dilewati.

Selain menyaring nilai tertentu, pada sintaks SQL dasar `WHERE` juga berfungsi sebagai jembatan penghubung relasi antar tabel.

---

## 4. Cara Menggabungkan Tabel Berelasi dengan Implicit JOIN (`WHERE ... = ...`)

### Mengapa Perlu Dihubungkan?

Tabel pada basis data relasional biasanya dipisah menjadi entitas-entitas mandiri:

- `tblPelanggan` menyimpan data profil pelanggan (kodepelanggan, namapelanggan).
- `tblTransaksi` menyimpan berkas transaksi (nomortransaksi, kodepelanggan).

Jika hanya menulis:

```sql
SELECT * FROM tblPelanggan, tblTransaksi;
```

Hasil query akan menghasilkan perkalian silang (Cartesian / Cross Join), di mana setiap satu baris pelanggan akan dipasangkan secara acak ke seluruh baris transaksi.

### Solusi: Memasangkan Kunci Relasi di `WHERE`

Untuk mendapatkan data yang valid, pasangkan kolom Primary Key dan Foreign Key yang nilainya saling terkait:

```sql
WHERE tblTransaksi.kodepelanggan = tblPelanggan.kodepelanggan
```

Artinya: *"Tampilkan data transaksi hanya jika kode pelanggan di tabel transaksi sama persis dengan kode pelanggan di tabel pelanggan."*

### Contoh:

Pada penyusunan nota penjualan lengkap, data diambil dari 4 tabel sekaligus dengan 3 kondisi relasi pengikat:

```sql
SELECT 
    t.nomortransaksi,
    p.namapelanggan,
    b.namabarang,
    d.jumlah,
    d.harga
FROM tblBarang b, tblPelanggan p, tblTransaksi t, tblDetilTransaksi d
WHERE b.kodebarang = d.kodebarang
  AND d.nomortransaksi = t.nomortransaksi
  AND t.kodepelanggan = p.kodepelanggan;
```

> **Aturan Penggabungan Relasi:**
> Jika menggabungkan N tabel, dibutuhkan minimal N - 1 kondisi relasi di klausa WHERE agar tidak terjadi perkalian silang.

## 5. Beberapa Contoh Kasus `WHERE`

Klausa `WHERE` sangat fleksibel: bisa digunakan untuk memfilter nilai di satu tabel, menghubungkan tabel berelasi, atau bahkan menggabungkan beberapa kondisi sekaligus menggunakan operator logika seperti `AND`, `OR`, perbandingan angka (`>`, `<`, `>=`), hingga pencarian teks (`LIKE`).

### Kasus A: Filter Sederhana di Satu Tabel (Kondisi Angka atau Teks)

Mencari barang yang harganya di atas Rp 20.000:

```sql
SELECT * FROM tblBarang
WHERE hargadasar > 20000;
```

Mencari pelanggan yang tinggal di jalan tertentu:

```sql
SELECT * FROM tblPelanggan
WHERE alamat LIKE '%Merdeka%';
```

---

### Kasus B: Filter dengan Beberapa Syarat Sekaligus (`AND` / `OR`)

Mencari barang yang kemasannya "Botol" **DAN** harganya di bawah Rp 15.000:

```sql
SELECT * FROM tblBarang
WHERE kemasan = 'Botol' 
  AND hargadasar < 15000;
```

Mencari transaksi yang memiliki diskon **ATAU** pajak lebih dari 0:

```sql
SELECT * FROM tblTransaksi
WHERE diskon > 0 
   OR pajak > 0;
```

---

### Kasus C: Menggabungkan Relasi Tabel dengan Filter Spesifik

Menghubungkan tabel relasi, lalu menambahkan filter agar hanya menampilkan transaksi milik pelanggan "P001":

```sql
SELECT t.nomortransaksi, p.namapelanggan, t.tanggal
FROM tblTransaksi t, tblPelanggan p
WHERE t.kodepelanggan = p.kodepelanggan
  AND p.kodepelanggan = 'P001'; 
```

---

### Kasus D: Banyak Relasi + Banyak Filter Sekaligus

Menggabungkan 4 tabel, tetapi kita hanya ingin melihat transaksi yang terjadi di tahun 2026 **DAN** jumlah barang yang dibeli lebih dari 2:

```sql
SELECT 
    t.nomortransaksi,
    t.tanggal,
    p.namapelanggan,
    b.namabarang,
    d.jumlah
FROM tblBarang b, tblPelanggan p, tblTransaksi t, tblDetilTransaksi d
WHERE b.kodebarang = d.kodebarang
  AND d.nomortransaksi = t.nomortransaksi
  AND t.kodepelanggan = p.kodepelanggan	
  AND t.tanggal >= '2026-01-01'
  AND d.jumlah > 2;
```

---

### Kasus E: Menggabungkan Dua Tabel Tanpa Filter Tambahan

Jika hanya ingin melihat daftar harga transaksi di setiap nota barang tanpa memfilter apa pun:

```sql
SELECT d.nomortransaksi, b.namabarang, d.jumlah, d.harga
FROM tblBarang b, tblDetilTransaksi d
WHERE b.kodebarang = d.kodebarang;
```

---

## Ringkasan Alur Berpikir SQL

Urutan analisis saat menyusun query:

1. **FROM**: Tentukan tabel mana saja yang memuat kolom-kolom yang dibutuhkan.
2. **WHERE**: Tentukan relasi antar tabel tersebut dan tentukan syarat baris mana saja yang ingin disaring.
3. **SELECT**: Tentukan kolom atau perhitungan aritmetika yang ingin ditampilkan.
