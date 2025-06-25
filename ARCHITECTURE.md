# Explainable AI

## Overview

Based on provided EDA and requirements. i came to conclusion to use this architecture diagram

<img src="assets/insignia-test.drawio.png" alt="insignia-architecture"/>

this is a provided high level architecture for solving requirements described in

<img src="assets/requirements.png"/>

## Architecture

### 1. Event Driven

Based on example, here's the point i notes:

-  the app is a chatbot app
-  we can assume the data comes once a year in excel form. however, there's no guarantee that this always happens
-  as we may use ML in defining and identifying issues or anomalies, we need to train the model which can cause significant amount of time
-  we also need to log every critical operation as well (login, chat results, data request, pipeline job request and results, ML training results, etc etc)
-  user may want to identifying current data and ensure data driven decision made on previous data history without updating datasource.

this requires a fast and persistency across many type of services, hence we need an event driven architecture to solve this problem.

### 2. Component Interaction

#### A. Chatbot app

Chatbot app is a user interface server that handling authentication, session logging and user interaction to the main chatbot flow.

This is an n8n based flow that defines main agent of the chatbot for user interaction.

This is a server based n8n framework that handles caching, memory using redis and interaction with vector store based memory based on MCP server result.

This flow also should be able to redirect the data job into data pipeline using RabbitMQ.

#### B. RabbitMQ Cluster

This is a big cluster of RabbitMQ instance where it handles most of communication and logging operation between services.

RabbitMQ has mostly handle all the communication and has log features that can be used.

#### C. Data Pipeline

This is a pipeline either to only parse data from source, validating data, or if needed also cleansing data from the provided job.

Data pipeline has to be able pipping the provided data to PostgreSQL clusters for Machine Learning server and MCP server to consume.

#### D. MCP Server

MCP server is a collection of tools for AI agent in n8n to interact with. it is a proxy and adapter for ML integration and data visualization.

It retrieves model from RabbitMQ based on newest training data.

It also can interact directly with piped data in PostgreSQL clusters to be able dynamically handle user needs.

Data visualization will be integrated to MCP server. this may use google sheets API to interact and visualizing data.

#### E. Machine Learning Server

This server sole purpose is to train and generate new model for MCP to consume based on newest data to ensure the model is accounted for newest dataset.

ML server will be notified by Data Pipeline using RabbitMQ to train newest set of data. it will retrieve data from PostgreSQL clusters on the range window defined and trained a outlier detection model (LOF, Isolation Forest, Etc) using the data.

ML model can also be used by MCP server in regards to predict outlier before data pipeline job is trigerred.

## Root Cause Analysis (RCA)

<img src="assets/fishbone-diagram.png"/>

Root cause analysis in this instance functions as main feature to supplement identification in irregularity and detect anomalies in data integrity.

It will be used to seek out and identify root cause for an irregularity and determine action to be taken.

### 1. NLP / LLM Integration

NLP or LLM can be used to integrate data retrieval and helping user to determine irregularity/anomalies and helping user to implement RCA and exploring possibilities to seek actions from data.

### 2. Visualization Tools

Visualization tools is needed in this instance to enchance user data understanding and can be integrated to NLP/LLM to be used.

## Technical Stack

### 1. Chatbot app

it can be using any framework to handle user interaction to chatbot. in this particular case, we use:

-  NodeJS
-  n8n
-  PostgreSQL (User oriented)
-  MilvusDB (Vector Memory)
-  Redis (Caching)
-  RabbitMQ producer and consumer
-  LLM Model (Claude or Deepseek for thinking model)
-  JWT (optional)
-  Google Auth (optional)

it acts as first layer for user to interact with chatbot. It handles user interaction with chatbot agent and communication with MCP server. it also creates job for data pipeline.

### 2. MCP server

it acts as context-provider to LLM. it uses:

-  PostgreSQL
-  Python
-  Simple data processing tools (pandas, numpy, etc)
-  Google API (google sheets API)

this server provides context to LLM and acts as multipurpose tools that handles main interaction with provided context or it can be scallably divided per context as a tools for LLM. it communicates using RabbitMQ and gRPC to ML server for inferencing prediction.

### 3. Data Pipeline

it handles data parsing, validation and cleansing for big data. it uses:

-  Spark
-  Pyspark
-  RabbitMQ producer and consumer
-  PostgreSQL

### 4. Machine Learning server (ML server)

handling model training and inferencing. communicates directly to MCP server to serve inference result for current state of data.

it will be notified by data pipeline through RabbitMQ for new data. it will use:

-  Python
-  FastAPI
-  ML tools (SKlearn, Tensorflow, etc)
-  RabbitMQ producer and consumer
-  PostgreSQL
