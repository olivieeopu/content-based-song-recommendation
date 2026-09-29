# content-based-song-recommendation
Sistem rekomendasi lagu berbasis kemiripan 10 fitur audio menggunakan StandardScaler dan cosine similarity untuk menghasilkan top-5 recommendations.

# Content-Based Song Recommendation

Proyek akademik untuk membangun sistem rekomendasi lagu berdasarkan kemiripan karakteristik audio menggunakan **StandardScaler** dan **cosine similarity**. Pengguna memasukkan judul lagu yang tersedia dalam dataset, kemudian sistem mencari lima lagu dengan representasi audio terdekat.

## Tujuan

- Mengeksplorasi missing values, duplikasi, dan distribusi fitur audio.
- Merepresentasikan lagu menggunakan fitur numerik yang telah distandardisasi.
- Menghasilkan top-5 recommendations berdasarkan kemiripan konten.
- Membandingkan karakteristik lagu input dengan lagu yang direkomendasikan.

Sistem ini tidak menggunakan riwayat mendengarkan, rating pengguna, atau collaborative filtering. Input teks berfungsi untuk mencari judul lagu dalam katalog, bukan memahami permintaan bebas seperti deskripsi suasana hati.

## Dataset

Notebook membaca `song_recomendation_A.csv`, yang disediakan untuk tugas akademik. Metadata mencakup judul lagu, artis, album, tanggal rilis, URI, genre, dan popularity, beserta fitur audio. Sumber publik asli dan tanggal pengambilan data belum dicantumkan dalam notebook.

Jumlah baris dan statistik dataset perlu ditampilkan kembali setelah menjalankan notebook karena output pemeriksaan awal tidak tersimpan pada file yang digunakan untuk dokumentasi ini.

## Data Preparation

1. Menghapus baris yang kehilangan Track Name, Artist Name(s), atau Album Name.
2. Mengimputasi Acousticness dan Tempo menggunakan median.
3. Mengisi Artist Genres yang kosong dengan Unknown, meskipun genre tidak digunakan dalam perhitungan similarity akhir.
4. Menghapus baris duplikat penuh.
5. Memeriksa distribusi sembilan fitur numerik menggunakan boxplot. Nilai ekstrem dipertahankan dalam kode; validitas rentang setiap fitur masih perlu diperiksa secara eksplisit.
6. Memisahkan metadata tampilan dari fitur similarity.

## Fitur Rekomendasi

Sepuluh fitur digunakan dalam perhitungan:

| Fitur | Aspek yang direpresentasikan |
|---|---|
| Danceability | Karakteristik yang berkaitan dengan kemudahan mengikuti irama untuk menari |
| Energy | Intensitas audio |
| Loudness | Tingkat kenyaringan |
| Speechiness | Karakteristik ujaran pada audio |
| Acousticness | Karakteristik akustik |
| Instrumentalness | Karakteristik instrumental |
| Liveness | Karakteristik yang terkait dengan rekaman live |
| Valence | Karakter positif pada audio |
| Tempo | Kecepatan dalam BPM |
| Mode | Mode musikal yang dikodekan sebagai angka |

Track Name dan Artist Name(s) digunakan untuk identifikasi/tampilan. Popularity hanya ditampilkan pada output, bukan dipakai untuk mengurutkan rekomendasi. Genre, album, Key, dan Time Signature tidak digunakan sebagai fitur similarity.

## Cara Kerja

```text
Dataset → Cleaning → 10 fitur audio → StandardScaler
→ Cosine similarity antar lagu
→ Pencarian judul input → Pengurutan kandidat → Top-5 recommendations
```

StandardScaler mengubah setiap fitur berdasarkan mean dan standard deviation katalog agar perbedaan skala, seperti Tempo dan Loudness, tidak langsung mendominasi perhitungan.

Cosine similarity mengukur kemiripan arah vektor fitur hasil standardisasi. Skor lebih tinggi menunjukkan representasi yang lebih searah; skor bukan probabilitas pengguna menyukai lagu. Karena fitur telah dicenter, nilai similarity dapat negatif.

## Fungsi Rekomendasi

Notebook mendefinisikan:

```python
recommend_songs(song_title, df, similarity_matrix, top_n=5)
```

Perilaku implementasi awal:

- Mencari judul secara exact match dan case-sensitive.
- Menggunakan kemunculan pertama jika judul muncul lebih dari sekali.
- Mengurutkan similarity dari tertinggi ke terendah.
- Menampilkan Track Name, Artist Name(s), dan Popularity.
- Mengembalikan pesan jika lagu tidak ditemukan.

## Perbaikan Indeks Sebelum Menjalankan Evaluasi

Kode asli menghapus baris tanpa mereset indeks, lalu mencampurkan label indeks DataFrame dengan posisi baris similarity matrix. Hal ini dapat menghasilkan pasangan lagu yang salah atau KeyError.

Setelah membentuk df_clean dan sebelum membuat X serta similarity matrix, tambahkan:

```python
df_clean = df_clean.reset_index(drop=True)
X = df_clean[feature_cols]
X_scaled = scaler.fit_transform(X)
similarity_matrix = cosine_similarity(X_scaled)
```

Gunakan fungsi berikut agar pengambilan baris berdasarkan posisi dan lagu input dikeluarkan secara eksplisit, bukan hanya melewati kandidat pertama:

```python
def recommend_songs(song_title, df, similarity_matrix, top_n=5):
    matches = np.flatnonzero(df['Track Name'].eq(song_title).to_numpy())
    if len(matches) == 0:
        return 'Song not found in the dataset.'

    idx = int(matches[0])
    scores = similarity_matrix[idx]
    ranked = np.argsort(-scores, kind='stable')
    selected = ranked[ranked != idx][:top_n]

    result = df.iloc[selected][
        ['Track Name', 'Artist Name(s)', 'Popularity']
    ].copy()
    result['Similarity'] = scores[selected]
    return result.reset_index(drop=True)
```

Fungsi ini tetap memilih judul pertama jika ada judul yang sama. Penggunaan URI atau kombinasi judul dan artis merupakan pengembangan berikutnya. Kode perbaikan ini belum dijalankan pada dataset dalam penyusunan dokumentasi.

## Contoh Input dan Evaluasi

Notebook menyiapkan tiga contoh utama:

```python
recommend_songs('All About That Bass', df_clean, similarity_matrix)
recommend_songs('Begin Again', df_clean, similarity_matrix)
recommend_songs('It Will Rain', df_clean, similarity_matrix)
```

Evaluasi yang dirancang membandingkan fitur lagu input dengan fitur kandidat dan rata-rata fitur rekomendasi. Ini adalah pemeriksaan konsistensi terhadap fitur yang digunakan sistem, bukan bukti kepuasan pengguna atau accuracy rekomendasi.

Output ketiga contoh pada bagian evaluasi belum tersimpan. Ada output interaktif untuk Style dan It Will Rain, tetapi perlu dihitung ulang setelah perbaikan indeks sebelum digunakan sebagai contoh final.

Saat membandingkan hasil, gunakan baris rekomendasi yang tepat. Pemilihan ulang hanya berdasarkan daftar judul dapat mengambil lebih dari satu lagu dengan judul sama dan mengubah rata-rata fitur.

## Visualisasi untuk README

Setelah perbaikan dan rerun, dokumentasi dapat dilengkapi dengan:

- Tabel lima rekomendasi beserta similarity score untuk masing-masing dari tiga input.
- Heatmap fitur yang telah distandardisasi untuk lagu input dan rekomendasinya.
- Contoh boxplot distribusi fitur audio.

Belum ada gambar output tersimpan dalam notebook ini, sehingga dokumentasi tidak menyertakan screenshot hasil yang belum terverifikasi.

## Keterbatasan dan Pengembangan

- Kemiripan audio tidak menjamin kesesuaian genre, bahasa, artis, atau selera pengguna.

## Tools

Python, pandas, NumPy, scikit-learn, Matplotlib, dan Seaborn.

