# spanner-vertex-search
We will build a demo web application to perform apparel search based on user input text. The application allows users to search for apparel by entering a text description. This project demonstrates how to implement Vector Search with data from Spanner database, while demonstrating the implementation of batch data export from Spanner database to Vector Search database and indexes. We will also build a demo web application to perform apparel search based on user input text. The application allows users to search for apparel by entering a text description.

The data that contributes to the inventory of the apparel search is stored in Spanner. We will invoke the Vertex AI Embeddings API in the ML.PREDICT construct directly from Spanner data. There is a dataflow job that bulk uploads this data (inventory and embeddings) into the Vertex AI’s Vector Search database and refreshes the index. When a user enters an apparel description, the app generates the embeddings in realtime using the Text Embeddings api. This is then sent as input to the Vector Search API to find 10 relevant product descriptions from the index and displays the corresponding image. 

## The architecture of the Spanner-Vertex Vector Search application is shown in the following diagram:

![alt text](https://github.com/AbiramiSukumaran/spanner-vertex-search/blob/main/data%20files/arch.JPG?raw=true)

The application consists of three main components:
A web application that allows users to enter text descriptions of apparel.
A dataflow job that bulk uploads data (inventory and embeddings) into the Vertex AI Vector Search database.
A Vector Search API that is used to find relevant product descriptions from the index.

Make sure you complete the database steps below before you start executing the Python script in this project:

# Backend: Spanner data to Vector Search
In this use case, Spanner database houses the inventory of apparels with the corresponding images and description. Make sure you generate embeddings for the text description and store them in your Spanner database as ARRAY<float64>.

## Create the Spanner data
Create an instance named “spanner-vertex” and a database named “spanner-vertex-embeddings”. Create a table using the DDL:
CREATE TABLE
  apparels ( id NUMERIC,
    category STRING(100),
    sub_category STRING(50),
    uri STRING(200),
    content STRING(2000),
	image STRING(200),
    embedding ARRAY<FLOAT64>
    )
PRIMARY KEY
  (id);
	

## Insert data into the table using the INSERT SQL
Insert scripts are made available here.

## Create Text Embeddings model 
This is required so we can generate embeddings for the content in the input. Below is the DDL for the same:

CREATE MODEL text_embeddings INPUT(content STRING(MAX))
OUTPUT(
  embeddings
    STRUCT<
      statistics STRUCT<truncated BOOL, token_count FLOAT64>,
      values ARRAY<FLOAT64>>
)
REMOTE OPTIONS (
  endpoint = '//aiplatform.googleapis.com/projects/abis-345004/locations/us-central1/publishers/google/models/textembedding-gecko');

## Generate text embeddings for the source data
Create a table to store the embeddings and insert the embeddings generated.
CREATE TABLE apparels_embeddings (id string(100), embedding ARRAY<FLOAT64>) PRIMARY KEY (id);

INSERT INTO apparels_embeddings(id, embeddings) 
SELECT CAST(id as string), embeddings.values
FROM ML.PREDICT(
  MODEL text_embeddings,
  (SELECT id, content from apparels)
) ;

Now that the bulk content and embeddings are ready, let us create a Vector Search Index and Endpoint to store the embeddings that will help perform the Vector Search. 

## Create a Cloud Storage [Bucket]([url](https://cloud.google.com/storage/docs/creating-buckets))
This is required to store embeddings from Spanner in a GCS bucket in a json format that Vector Search expects as input. Create a bucket in the same region as your data in Spanner. Create a folder inside if required, but mainly create an empty file called empty.json in it.

## Create an index
https://colab.research.google.com/github/GoogleCloudPlatform/generative-ai/blob/main/embeddings/vector-search-quickstart.ipynb

## Create workflow to export embeddings from Spanner to Vector Search

## Deploy index to an endpoint
https://colab.research.google.com/github/GoogleCloudPlatform/generative-ai/blob/main/embeddings/vector-search-quickstart.ipynb

## Finally invoke the  Vector Search api to run similarity search for your input text
Refer to spanner-vector-search.ipynb file
# spanner-vertex-search
