# QUIZ STRUKTUR DATA
---------------------------------------------
Nama  : Bagaskara Refananda

NIM   : 2501010102

Kelas : D

---------------------------------------------
# Analisis Struktur Data: Array & Linked List

> Catatan akademik tentang karakteristik memori, efisiensi operasi, dan implementasi struktur data linear.

---

## Daftar Isi

1. [Karakteristik Memori dan Akses Data](#1-karakteristik-memori-dan-akses-data)
2. [Analisis Efisiensi Operasi Manipulasi](#2-analisis-efisiensi-operasi-manipulasi)
3. [Konsep Doubly Linked List](#3-konsep-doubly-linked-list)
4. [Mekanisme Circular Linked List](#4-mekanisme-circular-linked-list)
5. [Array Dinamis di Python](#5-array-dinamis-di-python)

---

## 1. Karakteristik Memori dan Akses Data

### Mengapa akses elemen pada Array O(1) sedangkan pada Singly Linked List O(n)?

**Array** dialokasikan secara *contiguous* (berurutan) di memori. Karena setiap elemen memiliki ukuran tetap, alamat elemen ke-`i` dapat dihitung langsung dengan rumus:

```
alamat[i] = alamat_awal + (i × ukuran_elemen)
```

Perhitungan ini adalah operasi aritmatika sederhana — CPU menyelesaikannya dalam satu instruksi tanpa perlu menelusuri data. Hasilnya: akses **O(1)** yang benar-benar konstan, tidak peduli ukuran array-nya satu atau satu juta elemen.

**Singly Linked List** bekerja sebaliknya. Node tersebar di heap memori secara acak — hanya pointer `next` yang menjadi satu-satunya "peta". Untuk mengakses elemen ke-`i`, program harus memulai dari `head` dan melompati pointer satu per satu hingga mencapai posisi yang dituju, menghasilkan waktu linear **O(n)**.

### Hubungan dengan Alamat Memori

| Struktur Data | Alokasi Memori | Mekanisme Akses | Kompleksitas |
|---|---|---|---|
| Array | Kontinu (berurutan) | Direct addressing | O(1) |
| Singly Linked List | Non-kontinu (tersebar) | Sequential traversal | O(n) |

### 💡 Informasi Tambahan — Cache Locality

Perbedaan ini semakin nyata di level hardware. CPU modern menggunakan *cache hierarchy* (L1, L2, L3). Karena array menyimpan data secara berurutan, ketika CPU mengambil satu elemen, elemen-elemen tetangganya otomatis ikut ter-*cache* (*spatial locality*).

Pada linked list, setiap node berada di alamat memori yang tersebar, sehingga setiap akses berpotensi menyebabkan ***cache miss*** — memaksa CPU mengambil data langsung dari RAM yang jauh lebih lambat.

```
Perbandingan latensi akses memori (perkiraan):
  L1 Cache  →  ~1 ns    (array sangat diuntungkan)
  L2 Cache  →  ~4 ns
  L3 Cache  →  ~10 ns
  RAM       →  ~100 ns  (linked list sering terpaksa ke sini)
```

> Inilah mengapa dalam praktiknya, array sering jauh lebih cepat dari yang dicerminkan oleh notasi Big-O saja. Notasi O hanya mengukur jumlah operasi, bukan biaya hardware dari setiap operasi tersebut.

---

## 2. Analisis Efisiensi Operasi Manipulasi

### Dalam kondisi apa Linked List lebih unggul dibanding Array untuk operasi insertion dan deletion?

**Linked List lebih unggul ketika:**

- Frekuensi penyisipan/penghapusan sangat tinggi pada posisi tengah atau awal list, sedangkan akses acak jarang dilakukan.
- Ukuran data tidak diketahui sebelumnya dan sering berubah secara dinamis, sehingga array statis akan boros memori atau sering perlu realokasi.

### Perbandingan Kompleksitas Operasi

| Operasi | Array | Linked List (dengan pointer) | Keterangan |
|---|---|---|---|
| Akses elemen ke-i | O(1) | O(n) | Array unggul |
| Insert/delete di awal | O(n) | O(1) | Linked List unggul |
| Insert/delete di tengah | O(n) | O(1)* | Linked List unggul* |
| Insert/delete di akhir | O(1) amortized | O(n) / O(1)** | Tergantung implementasi |
| Search (unsorted) | O(n) | O(n) | Sama |

> \* O(1) hanya jika sudah memiliki pointer ke node yang akan dimanipulasi. Jika belum, traversal O(n) tetap diperlukan.
> \*\* O(1) jika list menyimpan pointer ke `tail`.

### Alasan Teoritis

**Pada Array:** Insertion/deletion di posisi tengah/awal membutuhkan pergeseran elemen lain (O(n) untuk shift), dan jika kapasitas penuh perlu alokasi ulang (O(n) copy).

**Pada Linked List:** Operasi cukup mengubah pointer di node tetangga tanpa menggeser data apapun.

```
Insert di posisi tengah Array:
  [A][B][C][D][E]  →  geser D,E ke kanan  →  [A][B][X][C][D][E]
  Kompleksitas: O(n) karena semua elemen setelah posisi harus digeser

Insert di posisi tengah Linked List (dengan pointer):
  A → B → C → D      tambahkan X antara B dan C:
  B.next = X          (1 operasi)
  X.next = C          (1 operasi)
  Kompleksitas: O(1)
```

### 💡 Informasi Tambahan — Memory Fragmentation

Array statis juga rentan terhadap *external fragmentation*. Jika ukuran data terus bertambah dan array perlu di-realokasi, sistem harus menemukan blok memori kontinu yang cukup besar — yang bisa gagal meskipun total free memory masih banyak. Linked List tidak mengalami masalah ini karena setiap node dapat ditempatkan di mana saja.

---

## 3. Konsep Doubly Linked List

### Struktur Anatomi Node

Setiap node pada Doubly Linked List terdiri dari tiga bagian:

```
┌──────┬──────────┬──────┐
│ prev │   data   │ next │
└──────┴──────────┴──────┘
   ↑                  ↑
pointer ke        pointer ke
node sebelumnya   node berikutnya
```

### Dampak Pointer Tambahan terhadap Memori

Setiap node membutuhkan ruang ekstra untuk satu pointer `prev` dibandingkan Singly Linked List.

```
Singly Linked List node:  [next | data]       = 8 + data bytes
Doubly Linked List node:  [prev | data | next] = 16 + data bytes
                                                  ↑ overhead +8 bytes per node
```

Pada sistem 64-bit, ukuran pointer = 8 byte. Untuk list dengan **1 juta node**, overhead tambahan:
- `1.000.000 × 8 bytes = 8 MB` hanya untuk pointer `prev`.

### Dampak terhadap Fleksibilitas Penelusuran

| Kemampuan | Singly | Doubly |
|---|---|---|
| Traversal maju (head → tail) | ✅ | ✅ |
| Traversal mundur (tail → head) | ❌ | ✅ |
| Hapus node tanpa tahu predecessor | ❌ (perlu traversal) | ✅ (via `prev`) |
| Implementasi Deque (double-ended queue) | Sulit | Mudah |

### 💡 Informasi Tambahan — Kasus Penggunaan Nyata

Doubly Linked List digunakan secara luas di sistem nyata, antara lain:

- **Browser history** — tombol Back/Forward menavigasi dua arah antar halaman.
- **Fitur Undo/Redo** di text editor (Ctrl+Z / Ctrl+Y).
- **LRU Cache (Least Recently Used)** — algoritma eviction cache populer yang mengkombinasikan HashMap + Doubly Linked List untuk mencapai operasi O(1) pada get dan put. Digunakan di Redis, sistem operasi, dan database.
- **Struktur data Deque** di Python (`collections.deque`) diimplementasikan sebagai Doubly Linked List.

---

## 4. Mekanisme Circular Linked List

### Perbedaan Teoritis dengan Linked List Biasa

**Linked List biasa (non-circular):**
```
head → [A] → [B] → [C] → [D] → NULL
```

**Circular Linked List:**
```
head → [A] → [B] → [C] → [D] ──┐
         ↑________________________│
         (tail.next menunjuk ke head)
```

Node terakhir tidak menunjuk ke `null`, melainkan kembali ke `head`, membentuk siklus tak berujung. Tidak ada node dengan `next = null`.

### Contoh Skenario Penggunaan (Use Case)

**Round-robin scheduling** dalam sistem operasi atau multiplayer game:

```
Proses: P1 → P2 → P3 → P4 → (kembali ke P1)

Tanpa Circular: setelah P4 selesai, perlu reset pointer ke P1 secara manual
Dengan Circular: P4.next otomatis menunjuk ke P1, traversal berlanjut tanpa reset
```

### 💡 Informasi Tambahan — Variasi dan Perhatian Implementasi

Circular Linked List hadir dalam dua varian:

- **Circular Singly Linked List** — `tail.next = head`
- **Circular Doubly Linked List** — `tail.next = head` DAN `head.prev = tail`

**Perhatian penting saat implementasi:**
Karena tidak ada node `null`, kondisi berhenti loop tidak bisa mengecek `current != null`. Harus menggunakan penanda lain, misalnya:

```python
# Salah (infinite loop):
while current is not None:  # ← tidak pernah True pada circular list

# Benar:
while current != head:      # ← berhenti ketika kembali ke awal
```

Kegagalan menangani hal ini akan menyebabkan **infinite loop** yang merupakan bug klasik pada implementasi Circular Linked List.

---

## 5. Array Dinamis di Python

### Mekanisme di Balik Layar saat `list.append()` Kehabisan Kapasitas

Python `list` diimplementasikan sebagai *dynamic array* dengan strategi **over-allocation**. Langkah internal saat `append()`:

```
1. Cek: apakah size < allocated?
   ├── YA  → tempatkan elemen baru langsung, size += 1  → O(1)
   └── TIDAK (kapasitas penuh):
       ├── Alokasi array baru (ukuran lebih besar ~1.125×)
       ├── Salin semua elemen lama ke array baru  → O(n)
       ├── Bebaskan memori array lama
       └── Tambahkan elemen baru  → O(1)
```

### Pola Pertumbuhan Kapasitas CPython

Rumus pertumbuhan kapasitas pada CPython (bukan selalu kelipatan 2):

```
kapasitas_baru = kapasitas_lama + (kapasitas_lama >> 3) + (3 atau 6)
```

Ilustrasi nyata pola pertumbuhan:

| Jumlah elemen | Kapasitas dialokasikan |
|---|---|
| 0 | 0 |
| 1 | 4 |
| 5 | 8 |
| 9 | 16 |
| 17 | 25 |
| 26 | 35 |
| 36 | 46 |

### Analisis Amortized Complexity

Meskipun kadang terjadi operasi O(n) saat realokasi, frekuensinya sangat jarang karena faktor pertumbuhan > 1. Biaya rata-rata per operasi `append` tetap **O(1) amortized**.

```
Contoh 8 append pertama (kapasitas awal 0 → 4 → 8):

append ke-1: kapasitas penuh (0) → alokasi 4 → salin 0 elemen → O(1)
append ke-2: ada ruang          → langsung simpan            → O(1)
append ke-3: ada ruang          → langsung simpan            → O(1)
append ke-4: ada ruang          → langsung simpan            → O(1)
append ke-5: kapasitas penuh (4) → alokasi 8 → salin 4 elemen → O(4)
append ke-6: ada ruang          → langsung simpan            → O(1)
append ke-7: ada ruang          → langsung simpan            → O(1)
append ke-8: ada ruang          → langsung simpan            → O(1)

Total operasi salin: 0 + 4 = 4 untuk 8 append
Rata-rata: 4/8 = 0.5 operasi per append → O(1) amortized ✅
```

### 💡 Informasi Tambahan — Perbandingan dengan Bahasa Lain

Strategi over-allocation bukan hanya milik Python. Bahasa lain menggunakan pendekatan serupa namun dengan faktor pertumbuhan berbeda:

| Bahasa / Struktur | Faktor Pertumbuhan | Catatan |
|---|---|---|
| Python `list` | ~1.125× | Lebih konservatif, hemat memori |
| Java `ArrayList` | 1.5× | `newCapacity = oldCapacity + (oldCapacity >> 1)` |
| C++ `std::vector` | 2× | Standar de-facto, lebih agresif |
| Go `slice` | 2× (kecil), ~1.25× (besar) | Adaptif berdasarkan ukuran |
| C# `List<T>` | 2× | Sama dengan Java awalnya |

Faktor pertumbuhan yang lebih besar (2×) mengurangi frekuensi realokasi tetapi memboroskan lebih banyak memori. Python memilih nilai konservatif (~1.125×) karena Python sangat sering digunakan untuk list kecil hingga menengah di mana penghematan memori lebih menguntungkan.

**Cara mengecek kapasitas aktual di Python:**

```python
import sys

lst = []
for i in range(10):
    lst.append(i)
    print(f"len={len(lst)}, allocated≈{sys.getsizeof(lst)} bytes")
```

> Python menyembunyikan seluruh kompleksitas realokasi ini, sehingga programmer dapat menggunakan `list` seolah-olah tanpa batasan ukuran — abstraksi yang elegan di balik implementasi yang cerdas.

---

## Ringkasan Perbandingan

| Aspek | Array | Singly Linked List | Doubly Linked List | Circular Linked List |
|---|---|---|---|---|
| Akses elemen ke-i | **O(1)** | O(n) | O(n) | O(n) |
| Insert/delete awal | O(n) | **O(1)** | **O(1)** | **O(1)** |
| Insert/delete tengah | O(n) | O(1)* | **O(1)** | O(1)* |
| Traversal mundur | ❌ | ❌ | **✅** | Tergantung varian |
| Cache locality | **Tinggi** | Rendah | Rendah | Rendah |
| Overhead memori | Rendah | 1 pointer/node | 2 pointer/node | 1–2 pointer/node |
| Traversal tak terbatas | ❌ | ❌ | ❌ | **✅** |

> \* O(1) hanya jika sudah memiliki pointer ke posisi yang dituju.

---

*Dibuat untuk keperluan akademik — Struktur Data & Algoritma*
