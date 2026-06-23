# 🍱 Analisis Sentimen Komentar TikTok — Program Makan Bergizi Gratis (MBG)

> Mengukur opini publik terhadap program pemerintah melalui komentar TikTok menggunakan hybrid model: Rule-based + Groq LLaMA 3.3 70B + IndoBERT.

---

## 📌 Latar Belakang

Program Makan Bergizi Gratis (MBG) merupakan salah satu kebijakan strategis pemerintah yang memicu perdebatan publik di media sosial. Penelitian ini menganalisis sentimen komentar pada akun **@tvoneofficial** terkait video penghentian sementara program MBG, untuk memahami pola opini publik di TikTok secara kuantitatif.

---

## 📊 Hasil Utama

| Sentimen | Jumlah | Persentase |
|----------|--------|------------|
| 🔴 Negatif | 316 | 50,4% |
| ⚫ Netral  | 251 | 40,0% |
| 🟢 Positif |  60 |  9,6% |

**Total komentar dianalisis:** 627 dari 703 komentar mentah (setelah filtering)

**Kata dominan per sentimen:**
- **Negatif:** tutup, mahal, korupsi, stop, hentikan
- **Positif:** bagus, dukung, lanjut, bermanfaat, semangat
- **Netral:** sementara, presiden, sekolah, kenapa, bagaimana

---

## 🗂️ Struktur Repositori

```
├── Analisis_Sentimen_MBG.ipynb        # Pipeline utama: scraping → preprocessing → labeling → evaluasi → visualisasi
├── Analisis_Sentimen_MBG.docx         # Laporan lengkap (metode, temuan, kesimpulan)
├── Komentar TikTok MBG.csv            # Data mentah hasil scraping (703 komentar)
├── Komentar TikTok MBG Labelled.csv   # Data setelah labeling Groq LLaMA
├── Sentimen_MBG_Labeled.csv           # Data final dengan label hybrid
├── labeling manual.csv                # 100 sampel untuk evaluasi manual
└── README.md
```

---

## ⚙️ Pipeline Analisis

### 1. Scraping Data
- **Tool:** Apify TikTok Comment Scraper (`BDec00yAmCm1QbMEI`)
- **Target:** Komentar video @tvoneofficial tentang penghentian sementara MBG
- **Periode:** Maret–Mei 2026
- **Hasil:** 703 komentar mentah

### 2. Preprocessing & Filtering
Komentar yang dihapus sebelum analisis:
- Sticker kosong dan pure emoji
- Spam "aamiin" dan spam like (`yang di-like`, `follow back`, dsb.)
- Komentar duplikat
- Komentar kurang dari 2 kata (tidak bermakna secara analitik)

**Hasil:** 627 komentar bersih (89,2% retention rate)

### 3. Labeling Sentimen — Hybrid Model

Pipeline labeling tiga lapis:

```
Komentar bersih
      │
      ▼
[Rule-based] ──── kata eksplisit positif/negatif terdeteksi? ──→ label langsung
      │ tidak terdeteksi
      ▼
[Groq LLaMA 3.3 70B] ──── klasifikasi konteks bahasa Indonesia informal
      │
      ▼
[IndoBERT override] ──── jika Groq = neutral & confidence IndoBERT ≥ 0.80
      │                   → gunakan label IndoBERT
      ▼
[Label Final / Hybrid]
```

**Tiga model yang digunakan:**

| Model | Peran |
|-------|-------|
| Rule-based (keyword matching) | Menangkap komentar dengan polaritas eksplisit (tutup, stop, dukung, lanjutkan) |
| Groq LLaMA 3.3 70B | Mengklasifikasikan komentar ambigu, bahasa gaul, dan sindiran |
| IndoBERT (`w11wo/indonesian-roberta-base-sentiment-classifier`) | Mengoreksi label netral dari Groq dengan confidence score ≥ 0,80 |

### 4. Evaluasi Model
- Pengambilan 100 sampel stratified (proporsi per kelas)
- Labeling manual sebagai ground truth
- Metrics: accuracy, precision, recall, F1-score (sklearn `classification_report`)
- Confusion matrix dengan visualisasi seaborn

### 5. Visualisasi
- Bar chart distribusi sentimen
- Pie chart proporsi sentimen
- Bar chart engagement (rata-rata & total like per sentimen)
- WordCloud per kategori sentimen (Greens / Reds / Greys)

---

## 🛠️ Teknologi

| Kategori | Library / Tools |
|----------|-----------------|
| Scraping | `apify-client` |
| Data Processing | `pandas`, `re` |
| LLM Labeling | `groq` (LLaMA 3.3 70B) |
| NLP Model | `transformers` — `w11wo/indonesian-roberta-base-sentiment-classifier` |
| Evaluasi | `scikit-learn` |
| Visualisasi | `matplotlib`, `seaborn`, `wordcloud` |
| Environment | Google Colab |

---

## 🚀 Cara Menjalankan

1. Buka notebook di Google Colab:

   [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1Sz5E8---e3xfTV-qdMKn-Zir_rc2eFFI?usp=sharing)

2. Install dependensi:
   ```bash
   pip install apify-client groq transformers torch scikit-learn wordcloud
   ```

3. Set API keys (ganti dengan key milikmu):
   ```python
   # Apify — untuk scraping
   APIFY_TOKEN = "your_apify_token"

   # Groq — untuk LLM labeling
   GROQ_API_KEY = "your_groq_api_key"
   ```

4. Jalankan cell secara berurutan dari atas ke bawah.

> **Catatan:** Jika tidak ingin scraping ulang, langsung load `Komentar TikTok MBG.csv` dan mulai dari tahap preprocessing.

---

## 📋 Definisi Kategori Sentimen

| Sentimen | Definisi Operasional |
|----------|---------------------|
| **Positif** | Dukungan, apresiasi, pujian, harapan positif, atau persetujuan terhadap program MBG |
| **Negatif** | Penolakan, kritik, sindiran, ketidakpuasan, atau permintaan penghentian program MBG |
| **Netral** | Pertanyaan, permintaan informasi, laporan kondisi, atau pernyataan tanpa keberpihakan jelas |

---

## 📝 Temuan Kunci

1. **Sentimen negatif mendominasi (50,4%)** — didorong oleh konteks video yang membahas penghentian sementara program, konten kontroversi secara natural menarik respons negatif lebih besar.

2. **Engagement komentar negatif paling tinggi** — fitur *like* pada komentar negatif memperkuat visibilitasnya di kolom komentar TikTok, menciptakan efek echo chamber opini negatif.

3. **Tiga isu utama dalam komentar negatif:** tuntutan penghentian permanen program, kekhawatiran kenaikan harga bahan pokok, dan perbandingan dengan program pendidikan gratis.

4. **Komentar netral cukup signifikan (40,0%)** — mencerminkan sebagian publik masih dalam fase pencarian informasi, bukan sikap yang sudah terbentuk.

---

2026

---

## 📚 Referensi

- Koto, F., Rahimi, A., Lau, J. H., & Baldwin, T. (2020). *IndoLEM and IndoBERT: Benchmark Dataset and Pre-trained Language Model for Indonesian NLP*. https://arxiv.org/abs/2011.00677
- Setiawan, B. (2025). *A Review of Sentiment Analysis Applications in Indonesia Between 2023–2024*. Journal of Information Engineering and Educational Technology, 8(2), 71–83.
- Setiawan, J. C., Lhaksmana, K. M., & Bunyamin. (2023). *Sentiment Analysis of Indonesian TikTok Review Using LSTM and IndoBERTweet Algorithm*. JIPI.
- Sumartha, D. K. (2025). *Implementasi IndoBERT untuk Analisis Sentimen Opini Publik pada Media Sosial*. Jurnal Informatika dan Teknik Elektro Terapan.
