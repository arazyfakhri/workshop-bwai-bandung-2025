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
- Membuat bucket baru di GCS dengan nama bucket **bwai_bandung2025_yourname_100525** & location : Multi Region US

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
- Download all dataset files (images and csv files) in folders [lab-workshop/gsp1246](https://github.com/saipulrx/workshop-bwai-bandung-2025/tree/main/lab-workshop/gsp1246) to your local computer
- Upload them to your GCS bucket **bwai_bandung2025_yourname_100525** 
![gcs_buckets_dataset](https://github.com/saipulrx/workshop-bwai-bandung-2025/blob/main/assets/gcs_buckets.png) 

#### 2.2 Grant IAM Storage Object Admin role to the connection's service account

Granting IAM permissions to the resource connection's service account before you start working in BigQuery will ensure you do not encounter access denied errors when running queries.

- Return to the root of the bucket.
- Click PERMISSIONS.
- Click GRANT ACCESS.
- In the New principals field, enter the service account ID you copied earlier.
- In the Select a role field, enter Storage Object, and then select Storage Object Admin role.
- Click Save.

The result is the service account now includes the Storage Object Admin role.

### Step 3. Create the dataset, and tables in BigQuery

In this task, you create a dataset for the project, the table for customer reviews, and the image object table.

#### 3.1 Create the dataset
- In the console, select the Navigation menu (Navigation menu icon), and then select BigQuery.
- In the Explorer panel, for qwiklabs-gcp-03-4acd241016e5, select View actions (More menu icon), and then select Create dataset.
You create a dataset to store database objects, including tables and models.
- In the Create dataset pane, enter the following information:

| Field      | Value       |
| -----------|-------------|
| Dataset ID | gemini_demo |
| Location Type | Select **Multi-region** |
| Multi-region | US |

Leave the other fields at their defaults.

- Click Create Dataset.

The result is the gemini_demo dataset is created and listed underneath your project in the BigQuery Explorer.

#### 3.2 Create the table for the customer reviews
To create the customer reviews table you will use a SQL query.

- Click the + to Create a new SQL Query.

- In the query editor, paste the query below.
```
LOAD DATA OVERWRITE gemini_demo.customer_reviews
(customer_review_id INT64, customer_id INT64, location_id INT64, review_datetime DATETIME, review_text STRING, social_media_source STRING, social_media_handle STRING)
FROM FILES (
  format = 'CSV',
  uris = ['gs://bwai_bandung2025_yourname_100525/gsp1246/customer_reviews.csv']);
```