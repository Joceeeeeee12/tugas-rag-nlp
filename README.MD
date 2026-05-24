# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari CHUNK_SIZE)*

> Jawab: _SIZE berfungsi untuk menentukan panjang tiap potongan teks dari PDF, nilai lebih besar membuat potongan berisi lebih banyak konteks (lebih lambat, risiko pencarian kurang presisi), sedangkan nilai lebih kecil membuat potongan lebih singkat (lebih cepat, tapi bisa memecah konteks sehingga jawaban kurang lengkap).

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (100–200)** | *Jawaban lebih ringkas/padat, terasa sedikit kehilangan detail karena konteks per chunk sangat pendek* |
| **Sedang (250–400)** ⭐ Recommended | *Ukuran ini direkomendasikan karena masih cukup detail untuk menjaga konteks, tetapi belum terlalu besar sampai membuat pencarian jadi kurang fokus.* |
| **Besar (500–1000)** | *Jawabannya mirip dengan CHUNK_SIZE 300, tapi proses pencarian lebih lambat karena chunk lebih besar* |

**Pertanyaan Refleksi:**

* Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?
  > Jawab: Chunk sedang lebih tepat untuk dokumen yang saya tes, karena hasilnya cukup lengkap tetapi tetap menjaga jawaban agar tidak terlalu panjang atau kehilangan konteks penting. Dari percobaan saya, chunk terlalu kecil membuat jawaban lebih mudah terpotong, sedangkan chunk terlalu besar membuat isi per potongan terlalu banyak dan waktu pencarian lebih lama.

* Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: Jika jawaban terasa "terpotong" atau kurang konteks, parameter yang perlu diubah adalah CHUNK_SIZE. Nilai CHUNK_SIZE bisa diperbesar agar setiap potongan teks memuat konteks yang lebih lengkap.

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TOP_K_INITIAL)*

> Jawab: TOP_K_INITIAL berfungsi untuk mengatur jumlah calon chunk yang akan dipertimbangkan lebih dulu oleh sistem, supaya proses reranking bisa memilih konteks yang paling relevan untuk dijadikan jawaban.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (5–10)** | *Pencarian lebih cepat karena kandidat yang diperiksa lebih sedikit. Namun, recall bisa turun karena ada kemungkinan chunk yang relevan tidak ikut terambil di tahap awal.* |
| **Sedang (15–25)** ⭐ Recommended | *Range ini direkomendasikan karena cukup banyak kandidat untuk memberi peluang menemukan konteks relevan, tapi belum terlalu berat sehingga proses masih efisien.* |
| **Besar (30–50)** | *Recall cenderung lebih tinggi karena sistem mempertimbangkan lebih banyak kandidat. Tetapi waktu proses jadi lebih lama karena reranker harus memeriksa lebih banyak chunk.* |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

* Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.
  > Jawab: Untuk dataset saya ini, TOP_K kecil (5-10) cukup karena informasinya tidak terlalu tersebar.Tetapi Untuk dokumen panjang dan kompleks, TOP_K besar lebih aman.

* Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL hanya berpengaruh besar kalau USE_RERANKER = True, karena kandidat awal yang diambil akan diranking ulang sebelum dipilih final. Kalau reranker dimatikan, maka TOP_K_INITIAL tidak terlalu berdampak, karena sistem langsung memakai hasil teratas tanpa proses penyaringan tambahan.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari FINAL_TOP_M)*

> Jawab: FINAL_TOP_M berfungsi untuk membatasi jumlah chunk (M) yang dipakai model agar jawaban tetap relevan, tidak terlalu penuh informasi, dan proses generasi lebih efisien.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Sedikit (2–3)** | *Jawaban lebih fokus, cepat dan tetap lengkap karena dokumen PDF-nya memang pendek. Untuk dokumen panjang, risiko konteks terlewat lebih tinggi.* |
| **Sedang (4–6)** ⭐ Recommended | *Range ini direkomendasikan karena masih cukup untuk membawa konteks penting, tetapi tidak terlalu banyak sampai membuat jawaban jadi berat atau membingungkan.* |
| **Banyak (7–10)** | *Konteks yang dikirim lebih lengkap, tetapi waktu proses lebih lama dan LLM bisa menerima terlalu banyak informasi yang kadang membuat jawaban kurang fokus.* |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

* Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.
  > Jawab: Tidak selalu. Dari eksperimen saya, konteks yang terlalu banyak memang bisa menambah informasi, tetapi juga membuat proses lebih lambat dan kadang jawaban menjadi kurang fokus. Jadi, lebih baik jumlah konteks yang seimbang.

* Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: Jika konteks yang dikirim saling bertentangan, LLM bisa bingung memilih informasi yang paling tepat. Akibatnya, jawaban bisa menjadi kurang konsisten atau mencampur beberapa informasi yang tidak selaras.

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari USE_RERANKER)*

> Jawab: USE_RERANKER berfungsi untuk meningkatkan relevansi konteks yang dipakai model, karena chunk yang awalnya hanya mirip secara umum akan dipilih ulang berdasarkan relevansi yang lebih akurat.

**Pengaruhnya:**

| Mode | Efek |
| --- | --- |
| `True` | *Presisi dan relevansi konteks biasanya lebih baik karena chunk hasil retrieval diseleksi ulang sebelum dikirim ke LLM. Latency bisa sedikit naik karena ada proses tambahan, tetapi kadang total waktu tetap efisien jika konteks akhir jadi lebih ringkas dan tepat.* |
| `False` | *Sistem hanya mengandalkan cosine similarity dari retrieval awal, jadi proses lebih cepat dan lebih sederhana. Namun, chunk yang dipilih bisa kurang presisi untuk pertanyaan yang ambigu atau teknis, sehingga relevansi konteks bisa sedikit turun.* |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

* Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.
  > Jawab: Reranker paling terasa manfaatnya pada query yang agak ambigu atau query teknis yang butuh konteks spesifik. Misalnya, ketika pertanyaan punya banyak istilah mirip di dokumen, reranker membantu memilih chunk yang benar-benar paling relevan.

* Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: Secara teori reranker bisa menambah tahap proses, tetapi pada eksperimen saya tambahan latency tidak terlihat jelas. Jadi, pengaruhnya tergantung pada ukuran dokumen, jumlah chunk, model yang dipakai, dan seberapa banyak konteks yang akhirnya dikirim ke LLM.

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari TEMPERATURE)*

> Jawab: Temperature berfungsi untuk mengatur tingkat kepastian, kreativitas, dan variasi jawaban dari model saat menyusun respons. Nilai rendah membuat jawaban lebih konsisten dan fokus, sedangkan nilai tinggi membuat jawaban lebih bervariasi dan kreatif

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **0.0–0.2** ⭐ Recommended untuk RAG | *Jawaban lebih konsisten, stabil, dan faktual. Cocok untuk RAG karena model tetap mengikuti konteks dokumen dan mengurangi risiko jawaban melenceng.* |
| **0.3–0.5** | *Jawaban masih cukup stabil, tetapi mulai ada sedikit variasi dalam cara penyampaian. Ini bisa dipakai kalau ingin jawaban tetap aman namun tidak terlalu kaku.* |
| **0.6–0.8** | *Jawaban lebih kreatif dan bervariasi, tetapi risiko keluar dari konteks dokumen mulai meningkat.* |
| **0.9–1.0** | *Jawaban bisa lebih acak dan berisiko halusinasi, karena model lebih bebas dalam memilih kata.* |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap *grounded* pada konteks yang diberikan.

**Pertanyaan Refleksi:**

* Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.
  > Jawab: Temperatur ideal untuk QA berbasis dokumen perusahaan adalah rendah, sekitar 0.1–0.3, karena jawaban harus tetap konsisten, faktual, dan mengikuti isi dokumen. Dengan temperature rendah, model lebih kecil kemungkinan membuat jawaban yang terlalu bebas atau tidak sesuai konteks.

* Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: Saat temperature dinaikkan ke 0.9, jawaban menjadi jauh lebih panjang, kreatif, dan berformat berbeda. Namun muncul informasi yang tidak ada dalam dokumen PDF.

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model | Ukuran | Pengaruh terhadap Kecepatan & Kualitas |
| --- | --- | --- |
| `llama-3.1-8b-instant` (Default) | 8B parameter | *Lebih cepat dan ringan, cocok untuk development atau percobaan awal. Kualitas jawabannya cukup baik untuk tugas dasar, tetapi detail dan ketelitian konteksnya biasanya masih di bawah model yang lebih besar.* |
| `openai/gpt-oss-120b` | 120B parameter | *Model besar dengan kualitas jawaban lebih kuat dan lebih stabil untuk pertanyaan kompleks. Namun biasanya lebih berat dan waktu proses bisa lebih lama, jadi cocok saat prioritasnya akurasi tinggi* |
| `llama-3.3-70b-versatile` | 70B parameter | *Lebih seimbang antara kualitas dan kecepatan. Cocok untuk eksperimen yang butuh jawaban lebih baik daripada model kecil, tetapi masih ingin proses yang relatif efisien.* |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

* Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.
  > Jawab: Menurut saya, peningkatan kualitas dari model besar cukup sebanding jika tujuan utamanya adalah akurasi dan jawaban yang lebih stabil. Namun, kalau tugasnya hanya eksperimen atau kebutuhan cepat, penambahan waktu dari model besar mungkin terasa kurang efisien dibanding manfaat tambahannya.

* Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: Model yang paling cocok untuk use-case dokumen saya adalah llama-3.3-70b-versatile, karena kualitas jawabannya sudah bagus tetapi masih cukup efisien untuk eksperimen RAG. openai/gpt-oss-120b juga sangat kuat, tetapi untuk tugas ini terasa lebih besar dari yang dibutuhkan, sehingga tidak selalu menjadi pilihan paling praktis.

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter | Nilai Default | Fungsi |
| --- | --- | --- |
| `CHUNK_OVERLAP` | 60 | *Fungsinya buat ngatur berapa kata dari chunk sebelumnya yang "ikut dibawa" ke chunk berikutnya. Gunanya supaya kalau ada kalimat yang kepotong di batas chunk, konteksnya tetap nyambung. Kalau terlalu kecil, informasi di peralihan antar chunk bisa hilang dan jawaban jadi kurang lengkap. Kalau terlalu besar, banyak teks yang muncul berulang-ulang di beberapa chunk sekaligus, bikin index lebih berat dan konteks yang dikirim ke LLM jadi redundant.* |
| `SCORE_THRESHOLD` | 0.30 | *Ini ambang batas minimum kemiripan (cosine similarity) antara pertanyaan dan chunk. Chunk yang skornya di bawah 0.30 langsung dibuang, dianggap tidak relevan. Kalau dinaikkan misalnya ke 0.5, hasil retrieval makin presisi tapi sistem jadi lebih "pemilih" dan bisa saja tidak menemukan konteks sama sekali untuk pertanyaan yang kurang spesifik. Kalau diturunkan, sistem lebih toleran dan lebih banyak chunk yang masuk, tapi risikonya chunk yang kurang relevan ikut terbawa ke LLM.* |

---

## 📝 Tips Eksperimen

* Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
* Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
* Catat kapan sistem salah & mengapa
* Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
* Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

* Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
* Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
