# Workshop Google Cloud Roadshow with BWAI Bandung 2025

Repository ini adalah repository untuk hands on workshop **SQL Meets AI: Analyze Customer Reviews in BigQuery & Gemini** yang diadakan di event [Google Cloud Roadshow with BWAI Bandung 2025](https://roadshow.cloudbandung.id/).

Di workshop ini, kita akan belajar bareng cara menganalisa sentimen dari data review customers menggunakan BQML dan model LLM Gemini Flash 2.0

Setelah sesi ini, harapannya kamu bisa:
- Belajar cara membuat koneksi dan menghubungkan external data source(Vertex AI) di BigQuery.
- Belajar cara membuat dataset dan tabel di BigQuery.
- Belajar membuat Gemini Remote model di BigQuery.
- Prompt ke Gemini untuk menganalisa keyword dan sentimen(positif atau negatif) dari data text customer reviews
- Generate report menghitung jumlah review positif dan negatif
- Memberikan respon dari review customer
- Prompt ke Gemini untuk mengekstrak ringkasan dan keyword dari masing - masing gambar review customer

Event ini adalah bagian dari Google Cloud Roadshows x Build with AI, salah satu event pembelajaran terbesar yang digerakkan oleh komunitas dan diselenggarakan oleh Google Developer Groups (GDG) Bandung. Event ini dirancang untuk memberdayakan developer, pelajar, dan profesional teknologi dengan mengeksplorasi kemampuan Google Cloud Platform (GCP) dan Artificial Intelligence (AI).

Lab Workshop ini di ambil dari [GSP1246 - Analyze Customer Reviews with Gemini Using SQL](https://www.cloudskillsboost.google/paths/1281/course_templates/1133/labs/507933) dengan sedikit modifikasi untuk menyesuaikan dengan hands on langsung di GCP Console

## Video Recording Hands on Workshop

Untuk video hands on workshop ini bisa dilihat rekaman nya di : 

## Prerequisite
- Memiliki kredit GCP
- Membuat projek baru di GCP

## Step by Steps hands on Workshop

### Step 1 : Create the cloud resource connection and grant IAM role
#### 1.1 Create the cloud resource connection in BigQuery
In this step you create a Cloud resource connection in BigQuery, so you can work with Gemini and Gemini models. You will also grant the cloud resource connection's service account IAM permissions, through a role, to enable it access the Vertex AI services.
- In the Google Cloud console, on the **Navigation menu**, click **BigQuery**.
- Click **DONE** on the Welcome pop-up
- To create a connection, click **+ ADD**, and then click **Connections to external data sources**.

  Note: Alternatively, if you do not see the option for **+ Add** followed by **Connections to external data sources**, you can click **+ Add data**, and then use the search bar for data sources to search for **Vertex AI**. Click on the result for **Vertex AI**.
- In the Connection type list, select **Vertex AI remote models, remote functions and BigLake (Cloud Resource)**.
- In the Connection ID field, enter **gemini_conn** for your connection.
- For **Location type** select **Multi-region** and then, from dropdown select **US** multi-region.
- Use the defaults for the other settings.
- Click **Create connection**.
- Click **GO TO CONNECTION**.
- In the Connection info pane, copy the service account ID to a text file for use in the next task. You will also see that the connection is added under the External Connections section of your project in the BigQuery Explorer.

#### 1.2 Grant Vertex AI User role to the connection's service account
- In the console, on the **Navigation menu**, click **IAM & Admin**.
- Click **Grant Access**.
- In the **New principals** field, enter the service account ID that you copied earlier.
- In the Select a role field, enter **Vertex AI**, and then select **Vertex AI User** role.
- Click **Save**.

  The result is the service account now includes the Vertex AI User role

### Step 2 Upload all datasets files to GCS buckets(images and csv files) and grant IAM role to service account
In this task, you uploads all datasets files (csv and image files) to GCS Buckets , then you grant IAM permissions to the cloud resource connection's service account.
#### 2.1 Upload all datasets files to GCS buckets(images and csv files)
- 
