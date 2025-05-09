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

## Prerequisite
- Have Google Cloud Credits
- Create new Project in Google Cloud
- Create new bucket in GCS with name **bwai_bandung2025_yourname_100525** & location : Multi Region US

## Step by Steps hands on Workshop

### Task 1 : Create the cloud resource connection and grant IAM role
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

### Task 2 Upload all datasets files to GCS buckets(images and csv files) and grant IAM role to service account
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

### Task 3. Create the dataset, and tables in BigQuery

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
This query uses the LOAD DATA statement to load the customer_reviews.csv file from Cloud Storage to a BigQuery table with the given column names and data types.
- Click **Run**

The result is the query is processed and the customer_reviews table created with the customer_review_id, customer_id, location_id, review_datetime, review_text, social_media_source, and social_media_handle for each review in the dataset.
- In the Explorer, click on the customer_reviews table and review the schema and details. Feel free to query the table to review records.

#### 3.3 Create the object table for the review images
To create the object table you will use a SQL Query.

1. Click the + to Create new SQL query.

2. In the query editor, paste the query below.
```
CREATE OR REPLACE EXTERNAL TABLE
  `gemini_demo.review_images`
WITH CONNECTION `us.gemini_conn`
OPTIONS (
  object_metadata = 'SIMPLE',
  uris = ['gs://bwai_bandung2025_yourname_100525/gsp1246/images/*']
  );
```
3. Run the Query.

The result is the review_images object table is added to the gemini_demo dataset and loaded with the uri (the cloud storage location) of each audio review in the sample dataset.

4. In the Explorer, click on the review_images table and review the schema and details. Feel free to query the table to review specific records.

### Task 4. Create the Gemini models in BigQuery

Now that the tables are created, you can begin to work with them. In this task, you create a model for Gemini 2.0 Flash in BigQuery.

#### 4.1 Create the Gemini 2.0 Flash model
1. Click the + to Create a new SQL Query.

2. In the query editor, paste the query below and run it.
```
CREATE OR REPLACE MODEL `gemini_demo.gemini_2_0_flash`
REMOTE WITH CONNECTION `us.gemini_conn`
OPTIONS (endpoint = 'gemini-2.0-flash')
```
The result is the gemini_2_0_flash model is created and you see it added to the gemini_demo dataset, in the models section.
3. In the Explorer, click on the gemini_2_0_flash model and review the details and schema.

### Task 5. Prompt Gemini to analyze customer reviews for keywords and sentiment
In this task, you will use Gemini model to analyze each customer review for keywords and sentiment, either positive or negative.

#### 5.1 Analyze the customer reviews for keywords
1. Click the + to Create a new SQL Query.

2. In the query editor, paste the query below, and run it.
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_keywords` AS (
SELECT ml_generate_text_llm_result, social_media_source, review_text, customer_id, location_id, review_datetime
FROM
ML.GENERATE_TEXT(
MODEL `gemini_demo.gemini_2_0_flash`,
(
   SELECT social_media_source, customer_id, location_id, review_text, review_datetime, CONCAT(
      'For each review, provide keywords from the review. Answer in JSON format with one key: keywords. Keywords should be a list.',
      review_text) AS prompt
   FROM `gemini_demo.customer_reviews`
),
STRUCT(
   0.2 AS temperature, TRUE AS flatten_json_output)));
```
This query takes customer reviews from the customer_reviews table, constructs prompts for the gemini_2_0_flash model to identify keywords within each review. The results are then stored in a new table customer_reviews_keywords.

Please wait. The model takes approximately 30 seconds to process the customer review records.

When the model is finished, the result is the customer_reviews_keywords table is created.

3. In the Explorer, click on the customer_reviews_keywords table and review the schema and details.

4. Click the + to Create a new SQL Query.

5. In the query editor, paste and run the query below.
```
SELECT * FROM `gemini_demo.customer_reviews_keywords`
```
The result is rows are displayed from the customer_reviews_keywords table with the ml_generate_text_llm_result column containing the keywords analysis, social_media_source, review_text, customer_id, location_id and review_datetime columns included.

#### 5.2 Analyze the customer reviews for positive and negative sentiment
1. Click the + to Create a new SQL Query.

2. In the query editor, paste the query below, and run it.
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_analysis` AS (
SELECT ml_generate_text_llm_result, social_media_source, review_text, customer_id, location_id, review_datetime
FROM
ML.GENERATE_TEXT(
MODEL `gemini_demo.gemini_2_0_flash`,
(
   SELECT social_media_source, customer_id, location_id, review_text, review_datetime, CONCAT(
      'Classify the sentiment of the following text as positive or negative.',
      review_text, "In your response don't include the sentiment explanation. Remove all extraneous information from your response, it should be a boolean response either positive or negative.") AS prompt
   FROM `gemini_demo.customer_reviews`
),
STRUCT(
   0.2 AS temperature, TRUE AS flatten_json_output)));
```
This query takes customer reviews from the customer_reviews table, constructs prompts for the gemini_2_0_flash model to classify the sentiment of each review. The results are then stored in a new table customer_reviews_analysis, so that you may use it later for further analysis.

Please wait. The model takes approximately 20 seconds to process the customer review records.

When the model is finished, the result is the customer_reviews_analysis table is created.

3. In the Explorer, click on the customer_reviews_analysis table and review the schema and details.

4. Click the + to Create a new SQL Query.

5. In the query editor, paste and run the query below.
```
SELECT * FROM `gemini_demo.customer_reviews_analysis`
ORDER BY review_datetime
```
The result is rows customer_reviews_analysis table with the ml_generate_text_llm_result column containing the sentiment analysis, with the social_media_source, review_text, customer_id, location_id and review_datetime columns included.

Take a look at some of the records. You may notice some of the results for positive and negative may not be formatted correctly, with extraneous characters like periods, or extra space. You can sanitize the records by using the view below.

#### 5.3 Create a view to sanitize the records
1. Click the + to Create a new SQL Query.

2. In the query editor, paste and run the query below.
```
CREATE OR REPLACE VIEW gemini_demo.cleaned_data_view AS
SELECT
REPLACE(REPLACE(REPLACE(LOWER(ml_generate_text_llm_result), '.', ''), ' ', ''), '\n', '') AS sentiment,
REGEXP_REPLACE(REGEXP_REPLACE(REGEXP_REPLACE(social_media_source, r'Google(\+|\sReviews|\sLocal|\sMy\sBusiness|\sreviews|\sMaps)?',
      'Google'), 'YELP', 'Yelp'), r'SocialMedia1?', 'Social Media') AS social_media_source,
review_text,
customer_id,
location_id,
review_datetime
FROM
gemini_demo.customer_reviews_analysis;
```
The query creates the view, cleaned_data_view and includes the sentiment results, the review text, the customer id and the location id. It then takes the sentiment result (positive or negative) and ensures that all letters are made lower case, and extreanous characters like extra spaces or periods are removed. The resulting view will make it easier to do further analysis in later steps within this lab.

3. You can query the view with the query below, to see the rows created.
```
SELECT * FROM `gemini_demo.cleaned_data_view`
ORDER BY review_datetime
```
This query is designed to fetch all data from the cleaned_data_view view and then arrange it in ascending order based on the date and time of the reviews.

#### 5.4 Create a report of positive and negative review counts
1. You can use BigQuery to create a bar chart report of the counts of positive and negative reviews. Start with the query below.
```
SELECT sentiment, COUNT(*) AS count
FROM `gemini_demo.cleaned_data_view`
WHERE sentiment IN ('positive', 'negative')
GROUP BY sentiment; 
```
The result is counts for positive and negative reviews are displayed.

2. To create the bar chart report of these counts, click CHART in the Query results section of BigQuery. BigQuery will automatically set the chart configuration, with chart type of Bar, and the sentiment column (the predicted sentitment as positive or negative) and the bar will display the count.

#### 5.5 Create a count of positive and negative reviews by social media source
1. You can use BigQuery to list the count of positive and negative reviews per social media source using the query below.
```
SELECT sentiment, social_media_source, COUNT(*) AS count
FROM `gemini_demo.cleaned_data_view`
WHERE sentiment IN ('positive') OR sentiment IN ('negative')
GROUP BY sentiment, social_media_source
ORDER BY sentiment, count;    
```

### Task 6. Respond to customer reviews
You can also use Gemini to respond to customer reviews. In this task you will learn how to create a marketing response using zero-shot and a customer service response using few-shot, against specific reviews in the customer_reviews table.

#### Marketing response
The customer with customer_id 5576 responded with:
> The location was clean and inviting. I also like that there is a variety of seating because sometimes I want to cuddle up with my coffee and read and other times I prefer a tall chair and table so I can work on projects.

This is clearly a positive review, how can you use Gemini 2.0 Flash to respond to this customer and incentivize them for the positive review?

1. You can use Gemini 2.0 Flash with these queries to accomplish this. In the query editor, paste the query below and run it.
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_marketing` AS (
SELECT ml_generate_text_llm_result, social_media_source, review_text, customer_id, location_id, review_datetime
FROM
ML.GENERATE_TEXT(
MODEL `gemini_demo.gemini_2_0_flash`,
(
   SELECT social_media_source, customer_id, location_id, review_text, review_datetime, CONCAT(
      'You are a marketing representative. How could we incentivise this customer with this positive review? Provide a single response, and should be simple and concise, do not include emojis. Answer in JSON format with one key: marketing. Marketing should be a string.', review_text) AS prompt
   FROM `gemini_demo.customer_reviews`
   WHERE customer_id = 5576
),
STRUCT(
   0.2 AS temperature, TRUE AS flatten_json_output)));
```
This query is designed to analyze customer reviews from the customer_reviews table, specifically those from customer ID 5576. When you run the query, it uses Gemini to generate marketing suggestions based on the review text and then stores the results in a new table called customer_reviews_marketing. This table will contain the original review data along with the generated marketing suggestions, allowing you to easily analyze and act upon them.

2. You can view the details of the customer_reviews_marketing table by running the SQL query below.
```
SELECT * FROM `gemini_demo.customer_reviews_marketing`
```
Notice that the ml_generate_text_llm_result column contains the response.

3. You can make this easier to read, and take action on the response by using the SQL query below:
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_marketing_formatted` AS (
SELECT
   review_text,
   JSON_QUERY(RTRIM(LTRIM(results.ml_generate_text_llm_result, " ```json"), "```"), "$.marketing") AS marketing,
   social_media_source, customer_id, location_id, review_datetime
FROM
   `gemini_demo.customer_reviews_marketing` results )
```
4. You can view the details of the table by running the SQL query below.
```
SELECT * FROM `gemini_demo.customer_reviews_marketing_formatted`
```
Notice the marketing column. An appliction can be written to take the response in the marketing column and attach the 10 percent off coupon file as a notifcation for the customer's account in the data beans app or an email can be generated with these to the customer as well.

#### Customer service response
The customer with customer_id 8844 responded with:
> I had a very disappointing experience at this coffee truck. The service was terrible - the staff were rude and inattentive, and we had to wait a long time for our food and drinks. The food itself was mediocre at best - the coffee was weak and the pastries were stale. The shop itself was also very cramped and noisy, making it difficult to relax and enjoy our time there. To top it all off, the prices were very high, making the whole experience even worse. I would definitely not recommend this place to anyone.

This is clearly a negative review, how can you use Gemini to respond to this customer and notify the coffee shop of their experience, in an effort to take action?

1. You can use Gemini with these queries to accomplish this. In the query editor, paste the query below and run it.
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_cs_response` AS (
SELECT ml_generate_text_llm_result, social_media_source, review_text, customer_id, location_id, review_datetime
FROM
ML.GENERATE_TEXT(
MODEL `gemini_demo.gemini_2_0_flash`,
(
   SELECT social_media_source, customer_id, location_id, review_text, review_datetime, CONCAT(
      'How would you respond to this customer review? If the customer says the coffee is weak or burnt, respond stating "thank you for the review we will provide your response to the location that you did not like the coffee and it could be improved." Or if the review states the service is bad, respond to the customer stating, "the location they visited has been notified and we are taking action to improve our service at that location." From the customer reviews provide actions that the location can take to improve. The response and the actions should be simple, and to the point. Do not include any extraneous or special characters in your response. Answer in JSON format with two keys: Response, and Actions. Response should be a string. Actions should be a string.', review_text) AS prompt
   FROM `gemini_demo.customer_reviews`
   WHERE customer_id = 8844
),
STRUCT(
   0.2 AS temperature, TRUE AS flatten_json_output)));
```
This query is designed to automate customer service responses by using Gemini to analyze customer reviews and generate appropriate responses and action plans. It's a powerful example of how Google Cloud can be used to enhance customer service and improve business operations. When the query is run, the result is the customer_reviews_cs_response table is created.

2. You can view the details of the table by running the SQL query below.
```
SELECT * FROM `gemini_demo.customer_reviews_cs_response`
```
Notice that the ml_generate_text_llm_result column contains the response and the actions as two keys.

3. You can make this easier to read, by using the SQL query below two separate the response and the actions into two columns in a new table called customer_reviews_cs_response_formatted:
```
CREATE OR REPLACE TABLE
`gemini_demo.customer_reviews_cs_response_formatted` AS (
SELECT
   review_text,
   JSON_QUERY(RTRIM(LTRIM(results.ml_generate_text_llm_result, " ```json"), "```"), "$.Response") AS Response,
   JSON_QUERY(RTRIM(LTRIM(results.ml_generate_text_llm_result, " ```json"), "```"), "$.Actions") AS Actions,
   social_media_source, customer_id, location_id, review_datetime
FROM
   `gemini_demo.customer_reviews_cs_response` results )
```
4. You can view the details of the table by running the SQL query below.
```
SELECT * FROM `gemini_demo.customer_reviews_cs_response_formatted`
```
Notice the response and actions fields are now created. You can build separate applications to respond to the customer, and to the location so that it can take actions to improve and the customer will be notified their feedback was received.

### Task 7. Prompt Gemini to provide keywords and summaries for each image
In this task, you will use Gemini to analyze images generating keywords and summaries.

#### Analyze the images with Gemini 2.0 Flash model
1. Click the + to Create a new SQL Query.

2. In the query editor, paste the query below, and run it.
```
CREATE OR REPLACE TABLE
`gemini_demo.review_images_results` AS (
SELECT
    uri,
    ml_generate_text_llm_result
FROM
    ML.GENERATE_TEXT( MODEL `gemini_demo.gemini_2_0_flash`,
    TABLE `gemini_demo.review_images`,
    STRUCT( 0.2 AS temperature,
        'For each image, provide a summary of what is happening in the image and keywords from the summary. Answer in JSON format with two keys: summary, keywords. Summary should be a string, keywords should be a list.' AS PROMPT,
        TRUE AS FLATTEN_JSON_OUTPUT)));
```
Please wait. The model takes approximately 3 minutes to complete.

When the model has finished processing the image, the result is the review_images_results table is created.

3. In the Explorer, click on the review_image_results table and review the schema and details.

4. Click the + to Create a new SQL Query.

5. In the query editor, paste and run the query below.
```
SELECT * FROM `gemini_demo.review_images_results`
```
The result is rows for each review image are displayed with the uri (the CloudStorage location of the review image) and a JSON result including the summary and keywords the Gemini Vision model.

You can retrieve these results in a more human readable way, by using the next query.

6. Click the + to Create a new SQL Query.

7. In the query editor, paste and run the query below.
```
CREATE OR REPLACE TABLE
  `gemini_demo.review_images_results_formatted` AS (
  SELECT
    uri,
    JSON_QUERY(RTRIM(LTRIM(results.ml_generate_text_llm_result, " ```json"), "```"), "$.summary") AS summary,
    JSON_QUERY(RTRIM(LTRIM(results.ml_generate_text_llm_result, " ```json"), "```"), "$.keywords") AS keywords
  FROM
    `gemini_demo.review_images_results` results )
```
The result is the review_images_results_formatted table is created.

8. You can query the table with the query below, to see the rows created.
```
SELECT * FROM `gemini_demo.review_images_results_formatted`
```
Notice how the uri column results remain the same, but the JSON is now converted to the summary and keywords columns for each row.

### Congratulations!
You successfully created cloud resource connection in BigQuery. You also created created a dataset, tables, and models to prompt Gemini to analyze sentimenet on customer reviews, with a report of positive and negative review counts. You then used zero-shot and few-shot prompts with Gemini to respond to these reviews. Finally, you used the Gemini model to analyze images and generate summaries and keywords.

