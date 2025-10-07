===========================================
 MAPREDUCE WEATHER ANALYTIC PIPELINE
===========================================

Anggota: Abidzar Naufal Ghifari
         Raditya Sajid
         Faza Nuha Ulwani
         Alfaridzi thariq Iskandar

-------------------------------------------
1. Deskripsi Umum
-------------------------------------------
Pipeline ini menganalisis data cuaca global (NOAA GHCN) tahun 2023–2024
menggunakan Hadoop MapReduce Streaming berbasis Python.

Proses dilakukan dalam dua tahap:
  - JOB 1: Analisis deskriptif (jumlah, rata-rata, min, max, count)
  - JOB 2: Analisis lanjutan (Top 10 stasiun dengan curah hujan tertinggi tahun 2024)

-------------------------------------------
2. Struktur Data Input
-------------------------------------------
File: 2023.csv.gz, 2024.csv.gz
Format kolom:
ID, DATE, ELEMENT, VALUE, FLAG, QUALITY, SOURCE

Contoh nilai ELEMENT:
  - PRCP = Curah hujan (0.1 mm)
  - TAVG = Suhu rata-rata (0.1 °C)
  - TMAX = Suhu maksimum
  - TMIN = Suhu minimum

-------------------------------------------
3. Perintah HDFS & Hadoop yang Digunakan
-------------------------------------------

# Membuat folder input di HDFS
hdfs dfs -mkdir -p /input/weather

# Mengunggah file data ke HDFS
hdfs dfs -put input/2023.csv.gz /input/weather/
hdfs dfs -put input/2024.csv.gz /input/weather/

-------------------------------------------
 JOB 1: Analisis Deskriptif Cuaca
-------------------------------------------
hadoop jar "C:\hadoop\share\hadoop\tools\lib\hadoop-streaming-3.2.4.jar" ^
-files "mapper_stats.py,reducer_stats.py" ^
-mapper "python mapper_stats.py" ^
-reducer "python reducer_stats.py" ^
-input /input/weather ^
-output /output/weather_stats

# Mengecek hasil
hdfs dfs -cat /output/weather_stats/part-00000 | more
hdfs dfs -get /output/weather_stats/part-00000 output/stats_2023_2024.csv

-------------------------------------------
 JOB 2: Top 10 Curah Hujan Tertinggi 2024
-------------------------------------------
hadoop jar "C:\hadoop\share\hadoop\tools\lib\hadoop-streaming-3.2.4.jar" ^
-files "mapper_topk_2024_prcp.py,reducer_topk.py" ^
-mapper "python mapper_topk_2024_prcp.py" ^
-reducer "python reducer_topk.py" ^
-input /output/weather_stats ^
-output /output/topk_prcp_2024

# Mengecek hasil
hdfs dfs -cat /output/topk_prcp_2024/part-00000
hdfs dfs -get /output/topk_prcp_2024/part-00000 output/topk_prcp_2024.csv

-------------------------------------------
4. Hasil Akhir
-------------------------------------------
- File output:
  • stats_2023_2024_header.csv  → Statistik deskriptif tahunan
  • topk_prcp_2024_header.csv   → Top 10 stasiun PRCP tertinggi tahun 2024

-------------------------------------------
5. Catatan Tambahan
-------------------------------------------
Nilai PRCP dan suhu menggunakan satuan 0.1 mm / 0.1 °C sesuai format NOAA.
Data hasil Job 2 menunjukkan 10 stasiun dengan total PRCP tertinggi.
File .txt di folder 'scripts/' berisi source code dari MapReduce.
