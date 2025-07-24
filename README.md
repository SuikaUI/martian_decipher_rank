## Martian Language Decipher & Ranking

Notebook ini didedikasikan untuk memecahkan "Bahasa Mars" dari sebuah kompetisi, yang ternyata adalah anagram Bahasa Indonesia dengan pola enkripsi spesifik, lalu mencocokkannya dengan koleksi dokumen Bahasa Inggris.

### Tujuan Proyek
1.  **Dekripsi Pola**: Mengidentifikasi dan mengimplementasikan logika untuk memecahkan anagram Bahasa Mars.
2.  **Persiapan Data**: Mengubah file `queries.txt` dan `unk500.txt` yang terenkripsi menjadi format Bahasa Indonesia yang dapat dibaca.
3.  **Penyimpanan Hasil**: Menyimpan teks yang telah didekripsi untuk melatih model peringkat yang akan mencocokkannya dengan `eng_collection.txt`.

### Sumber Data
* `queries.txt`: 50 dokumen "Bahasa Mars" untuk diterjemahkan.
* `eng_collection.txt`: 1459 dokumen Bahasa Inggris sebagai kandidat terjemahan.
* `eng500.txt` & `unk500.txt`: Korpus paralel untuk pelatihan model.
* `list_1.0.0.txt`: Daftar kata KBBI sebagai kunci dekripsi anagram.

### Metodologi
Proyek ini mengadopsi pendekatan dua langkah:

1.  **Dekripsi Anagram (Offline)**:
    * Mengidentifikasi pola enkripsi: `{huruf_terakhir_asli}zk0{anagram_kata}xv{panjang_kata}{huruf_pertama_asli}`.
    * Membangun fungsi `decode_anagram` awal yang melakukan parsing, memuat KBBI, dan melakukan iterasi & pengecekan untuk memvalidasi anagram.
    * Mengembangkan dekoder yang dioptimalkan dengan membangun *lookup tables* KBBI untuk pencarian $O(1)$ dan menghasilkan variasi kata dengan imbuhan/reduplikasi.
    * Menggunakan *regular expression* untuk menemukan pola terenkripsi dalam kalimat dan menggantinya dengan hasil dekripsi.
    * Menyimpan hasil dekripsi ke `output_hasil_unk.txt` (korpus paralel) dan `output_hasil_query.txt` (kueri).

2.  **Pencarian Lintas-Bahasa & Pelatihan Kebal-Noise (Bi-Encoder Fine-Tuning)**:
    * Masalah ini adalah tantangan *Cross-Lingual Information Retrieval* (CLIR).
    * Strategi utama adalah dekripsi awal yang menghasilkan teks Bahasa Indonesia dengan *noise* sisa kode 'Mars'.
    * **Fine-Tuning Bi-Encoder**: Menggunakan model Transformer multilingual **LaBSE** (`Language-Agnostic BERT Sentence Embedding`) yang dirancang untuk ruang *embedding* multilingual bersama pada 109+ bahasa.
    * **Triplet Loss & Hard Negative Mining**: Melatih model dengan `TripletLoss` menggunakan pasangan *(anchor, positive, negative)*. *Anchor* adalah kueri ber-noise Bahasa Indonesia, *positive* adalah terjemahan Inggris yang benar, dan *negative* adalah dokumen Inggris yang mirip tetapi salah. *Hard Negative Mining* dipilih untuk membuat model lebih *robust* terhadap *noise*.
    * **Indexing dengan FAISS**: Mengonversi semua dokumen Inggris ke vektor menggunakan model yang telah di-*fine-tune*, lalu menormalisasi vektor (L2 normalization) dan membangun indeks FAISS (`IndexFlatIP`) untuk pencarian kemiripan yang efisien.
    * **Ranking & Submission**: Melakukan pencarian untuk setiap kueri menggunakan indeks FAISS dan memformat hasilnya untuk *submission*.

### Hasil Kunci
Model berhasil mencapai **Mean DCG 1.0000** pada validasi lokal, yang konsisten dengan skor **1.0** pada *public leaderboard* kompetisi. Keberhasilan ini dikaitkan dengan:
1.  Strategi dua langkah yang efektif: dekripsi awal diikuti dengan CLIR.
2.  Pemilihan model dasar yang kuat (`LaBSE`).
3.  Penggunaan *fine-tuning* dengan *Hard Negative Mining* yang membuat model kebal terhadap *noise*.


### Cara Menjalankan
1.  Kloning repositori ini.
2.  Pastikan dataset berada di lokasi yang sesuai (atau akan diunduh secara otomatis oleh notebook).
3.  Instal dependensi yang diperlukan seperti yang ditunjukkan di notebook.
4.  Buka dan jalankan notebook  di lingkungan Jupyter (misalnya, JupyterLab, Google Colab, atau VS Code).
5.  Ikuti instruksi di setiap bagian notebook untuk eksplorasi data, pemodelan, dan analisis hasil.