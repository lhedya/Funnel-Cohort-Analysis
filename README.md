# 📊 Funnel Analysis & Cohort Analysis
### Data Science Bootcamp — Strategic Growth Insights

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-1.x-150458?logo=pandas&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-Express-3F4F75?logo=plotly&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)
![Google Colab](https://img.shields.io/badge/Google-Colab-F9AB00?logo=googlecolab&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-22C55E)

---

## 📌 Deskripsi Proyek

Proyek ini merupakan hasil analisis data e-commerce menggunakan dua pendekatan utama:

- **Funnel Analysis** — menganalisis alur konversi pengguna dari home → product → cart → purchase → cancel, mengidentifikasi titik drop-off dan hambatan dalam proses.
- **Cohort Analysis** — mengelompokkan pelanggan berdasarkan bulan akuisisi (first purchase) dan menelusuri pola retensi mereka dari waktu ke waktu selama tahun 2023.

Seluruh proses pengolahan data dikerjakan menggunakan Python di Google Colaboratory melalui notebook **`Strategic_Growth_Insights.ipynb`**, dan hasil visualisasinya ditampilkan sebagai **Cohort Dashboard di Microsoft Power BI**.

---

## 🎯 Objectives

1. Memahami perbedaan dan use case **Funnel Analysis** vs **Cohort Analysis**
2. Membuat dataset cohort menggunakan Python (CohortGroup, CohortPeriod, RetentionRate)
3. Membuat **Dashboard Cohort di Power BI** berupa retention rate heatmap
4. Mengidentifikasi **di periode berapa customer mengalami churn rate tertinggi**

---

## 🗂️ Struktur Proyek

```
📁 funnel-cohort-analysis/
│
├── 📄 README.md                          ← Dokumentasi proyek ini
├── 📊 Cohort.xlsx                        ← Dataset cohort (output Python)
├── 📓 cohort_analysis.ipynb              ← Google Colab notebook
├── 📊 Presentasi_Funnel_Cohort_Analysis.pptx ← Slide presentasi (PowerPoint)
│
├── 📁 Dashboard/
│   ├── cohort analysis.pbix                ← Dashboard Power BI
│
├── 📁 Notebook/
│   ├── Strategic Growth Insights.ipynb     ← Google Colab
│
└── 📁 data/
    └── Datasets Funnel Cohort.xlsx  ← Dataset asli


```

---

## 🔍 1. Perbedaan Funnel Analysis vs Cohort Analysis

Dari notebook `Strategic_Growth_Insights.ipynb`, Section 1:

| Aspek | Funnel Analysis | Cohort Analysis |
|-------|----------------|----------------|
| **Definisi** | Menganalisis tahapan perjalanan customer dari awal (visit) hingga akhir (purchase/churn) | Mengelompokkan pengguna berdasarkan karakteristik/peristiwa yang sama dan memeriksa perilaku mereka dari waktu ke waktu |
| **Tujuan** | Mengidentifikasi hambatan & mengoptimalkan proses konversi | Memahami perilaku dan tren pengguna jangka panjang |
| **Fokus** | Alur konversi step-by-step | Retensi pelanggan dari waktu ke waktu |
| **Output** | Conversion rate per tahap | Retention rate matrix/heatmap per cohort |
| **Use Case** | Optimasi UX, checkout, landing page | Churn analysis, LTV, loyalty |

---

## 🐍 2. Alur Pengerjaan di Notebook Python

### 2.1 Cohort Analysis — Kode Utama

```python
# 1. Import & Load Data
import pandas as pd
import numpy as np

data_all_sheets = pd.read_excel(
    '/content/Assignment Datasets Funnel Cohort.xlsx',
    sheet_name=['Events', 'Sales']
)

# Ensure datetime format
data_all_sheets['Sales']['order_date'] = pd.to_datetime(data_all_sheets['Sales']['order_date'])
data_all_sheets['Events']['created_at'] = pd.to_datetime(data_all_sheets['Events']['created_at'])

# 2. Determine Cohort Group (first purchase date per customer)
data_all_sheets['Sales']['CohortGroup'] = data_all_sheets['Sales'] \
    .groupby('user_id')['order_date'] \
    .transform('min')
data_all_sheets['Sales']['CohortGroup'] = data_all_sheets['Sales']['CohortGroup'].dt.to_period('M')

# 3. Calculate Cohort Period (months since first purchase)
data_all_sheets['Sales']['CohortGroup'] = data_all_sheets['Sales']['CohortGroup'].dt.to_timestamp()
data_all_sheets['Sales']['CohortPeriod'] = (
    (data_all_sheets['Sales']['order_date'] - data_all_sheets['Sales']['CohortGroup'])
    .dt.days / 30.4375
).astype(int)

# 4. Aggregate & Pivot
cohort_counts = data_all_sheets['Sales'] \
    .groupby(['CohortGroup', 'CohortPeriod'])['user_id'] \
    .nunique().reset_index()
cohort_counts.rename(columns={'user_id': 'TotalUsers'}, inplace=True)

cohort_pivot = cohort_counts.pivot_table(
    index='CohortGroup', columns='CohortPeriod', values='TotalUsers'
)

# 5. Calculate Retention Rate
cohort_size = cohort_pivot.iloc[:, 0]
retention_matrix = cohort_pivot.divide(cohort_size, axis=0)

# 6. Convert to Long Format & Export
retention_long = retention_matrix.reset_index().melt(
    id_vars=['CohortGroup'],
    var_name='CohortPeriod',
    value_name='RetentionRate'
)
retention_long.to_excel('retention_long.xlsx', index=False)
```

### 2.2 Funnel Analysis — Kode Utama

```python
# Define the funnel stages
funnel_steps = ['home', 'product', 'cart', 'purchase', 'cancel']

# Filter events & sort
funnel_df = data_events[data_events['event_type'].isin(funnel_steps)].copy()
funnel_df.dropna(subset=['user_id'], inplace=True)
funnel_df.sort_values(by=['user_id', 'created_at'], inplace=True)

# Get first event for each user at each stage
funnel_agg = funnel_df.groupby(['user_id', 'event_type'])['created_at'].min().reset_index()
funnel_pivot = funnel_agg.pivot_table(
    index='user_id', columns='event_type', values='created_at'
)

# Calculate funnel metrics
funnel_metrics = pd.DataFrame({
    'Step': funnel_conversions.index,
    'UserCount': funnel_conversions.values
})
funnel_metrics['Step'] = pd.Categorical(funnel_metrics['Step'], categories=funnel_steps, ordered=True)
funnel_metrics.sort_values('Step', inplace=True)
funnel_metrics['ConversionRate'] = funnel_metrics['UserCount'].pct_change().fillna(1.0)

# Visualize with Plotly
import plotly.express as px
fig = px.funnel(funnel_metrics, x='UserCount', y='Step', title='Funnel Analysis')
fig.show()

# Save
funnel_metrics.to_excel('funnel_metrics.xlsx', index=False)
```

---

## 🔽 3. Hasil Funnel Analysis

| Step | User Count | Conversion Rate |
|------|-----------|----------------|
| 🏠 home | 21,296 | 100.00% |
| 🛍️ product | 28,921 | 35.80% |
| 🛒 cart | 28,921 | 0.00% |
| ✅ purchase | 28,941 | 0.07% |
| ❌ cancel | 21,768 | **-24.78% ⚠️** |

> **Red Flag:** Cancel rate -24.78% mengindikasikan lebih dari 75% pembelian dibatalkan. Ini memerlukan investigasi mendalam pada proses checkout.

---

## 📈 4. Cohort Retention Matrix

Dataset output (Cohort.xlsx) diimpor ke Power BI dan divisualisasikan sebagai heatmap:

| Month | P0 | P1 | P2 | P3 | P4 | P5 | P6 | P7 | P8 | P9 | P10 | P11 | Total |
|-------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|------|-------|
| January | 100% | 4.9% | 2.7% | 2.3% | 2.3% | 2.3% | 2.4% | 2.0% | 2.4% | 1.7% | 2.0% | 2.4% | 127.5% |
| February | 100% | 2.6% | 2.6% | 2.4% | 2.4% | 2.4% | 1.8% | 3.0% | 2.6% | 2.7% | 2.8% | 2.3% | 125.2% |
| March | 100% | 5.3% | 2.5% | 1.7% | 2.9% | 2.6% | 2.6% | 2.6% | 1.9% | 2.6% | 0.2% | – | 124.9% |
| April | 100% | 4.5% | 1.9% | 2.4% | 2.9% | 2.8% | 2.4% | 2.2% | 2.3% | 0.2% | – | – | 121.6% |
| May | 100% | 4.4% | 2.2% | 3.3% | 2.1% | 2.4% | 2.4% | 2.5% | 0.1% | – | – | – | 119.4% |
| June | 100% | 4.0% | 2.5% | 2.8% | 2.7% | 2.3% | 3.1% | – | – | – | – | – | 117.4% |
| July | 100% | 4.5% | 2.5% | 1.6% | 3.0% | 2.5% | 0.1% | – | – | – | – | – | 114.2% |
| August | 100% | 3.8% | 2.8% | 3.1% | 3.4% | – | – | – | – | – | – | – | 113.6% |
| September | 100% | 4.4% | 2.9% | 2.7% | – | – | – | – | – | – | – | – | 110.0% |
| October | 100% | 5.2% | 3.7% | – | – | – | – | – | – | – | – | – | 108.9% |
| November | 100% | 4.3% | – | – | – | – | – | – | – | – | – | – | 104.3% |
| December | 100% | – | – | – | – | – | – | – | – | – | – | – | 100.0% |

---

## 🚨 5. Insight Utama: Churn Rate Tertinggi

### ➡️ Jawaban: **Churn tertinggi terjadi pada PERIODE 1** (bulan pertama setelah akuisisi)

| Periode | Avg Retention | Avg Churn |
|---------|-------------|-----------|
| Period 0 | 100.00% | — |
| **Period 1** | **4.42%** | **🔴 95.58% — TERTINGGI** |
| Period 2 | 2.65% | 1.77% |
| Period 3–11 | ~2.0–2.5% | <1% (stabil) |

### Rincian per Cohort:

| Cohort | Retention P1 | Churn P1 |
|--------|------------|---------|
| January | 4.95% | 95.05% |
| **February** | 2.64% | **97.36% (TERTINGGI)** |
| March | 5.27% | 94.73% |
| April | 4.47% | 95.53% |
| October | 5.25% | 94.75% |

> 📌 Cohort **Februari 2023** memiliki churn tertinggi (97.36%). Cohort **Maret & Oktober** menunjukkan retensi awal terbaik.

---

## 💡 6. Rekomendasi Strategis

### 🚀 1. Perkuat Onboarding (30 Hari Pertama)
- Welcome email sequence hari ke-1, 3, 7, 14, 30
- Insentif pembelian kedua (voucher, free ongkir, poin)
- Tutorial & panduan produk yang informatif

### 💎 2. Program Loyalitas untuk Loyal Segment
- Sistem reward & poin untuk pelanggan berulang
- Membership dengan benefit eksklusif
- Personalisasi rekomendasi berbasis riwayat belanja

### 🔧 3. Perbaiki Cancel Rate
- Audit UX dan friction point di proses checkout
- Perbanyak pilihan metode pembayaran
- A/B test halaman checkout

### 📅 4. Strategi Khusus Cohort Februari
- Analisis mendalam kampanye & traffic source bulan Februari
- Re-engagement campaign untuk pelanggan churned

---

## 🛠️ Cara Menjalankan

### Prerequisites
```bash
pip install pandas numpy openpyxl plotly
```

### Langkah-langkah
1. Buka `Strategic_Growth_Insights.ipynb` di [Google Colab](https://colab.research.google.com)
2. Upload dataset `Assignment Datasets Funnel Cohort.xlsx`
3. Jalankan semua cell secara berurutan
4. Download output:
   - `retention_long.xlsx` → rename ke `Cohort.xlsx`
   - `funnel_metrics.xlsx`
5. Import `Cohort.xlsx` ke Power BI
6. Buat Matrix visual dengan CohortGroup sebagai baris, CohortPeriod sebagai kolom, RetentionRate sebagai nilai

### Struktur Dataset Output (Cohort.xlsx)
```
CohortGroup    | CohortPeriod | RetentionRate
2023-01-01     | 0            | 1.0000
2023-01-01     | 1            | 0.0495
2023-01-01     | 2            | 0.0275
...
2023-12-01     | 0            | 1.0000
```

---

## 📝 Kesimpulan

1. **Churn rate tertinggi terjadi pada Period 1** dengan rata-rata 95.58% — momen paling kritis dalam siklus hidup pelanggan
2. Setelah Period 1, retensi **stabil di kisaran 2–3%** — segmen loyal customer yang potensial
3. **Cancel rate 24.78%** di funnel adalah red flag yang memerlukan perbaikan segera
4. **Cohort Februari** memiliki churn tertinggi (97.36%), sementara **Maret & Oktober** lebih baik
5. Fokus strategis pada **30 hari pertama pelanggan** untuk mengurangi churn secara signifikan

---

## 🛠️ Tools & Teknologi

| Tool | Versi | Fungsi |
|------|-------|--------|
| Python | 3.x | Bahasa pemrograman utama |
| Pandas | latest | Manipulasi dan analisis data |
| NumPy | latest | Komputasi numerik |
| Plotly Express | latest | Visualisasi funnel chart interaktif |
| Google Colab | – | Cloud notebook environment |
| Microsoft Power BI | Desktop | Dashboard cohort visualization |

---

## 👤 Author

**Data Analyst** — Data Science Bootcamp, Dibimbing  
Periode: Strategic Growth Insights | April 2026

---

*"Data is the new oil, but insight is the refinery." — Analisis ini dibuat untuk keperluan pembelajaran dan pengembangan kompetensi data analyst.*
