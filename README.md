# RAG Chatbot with API Integration using Amazon Bedrock-3

This project demonstrates the creation and deployment of a Retrieve and Generate (RAG) chatbot using Amazon Bedrock, AWS S3, and FastAPI. The project also includes the setup of an API that can be accessed by external applications, such as mobile apps or web apps, to interact with the chatbot seamlessly. This showcases a robust understanding of backend development, cloud architecture, and deploying AI models into production.
![Architecture](https://github.com/Amairrrr/RAG-Chatbot-API-Using-FastAPI-and-Bedrock/blob/4a6a7f325d77b674e73f084b74292e8a57657d76/images/Architecture.png)
## Overview

This project enables the development of a chatbot powered by Amazon Bedrock. The chatbot can answer queries based on data stored in Amazon S3, and it can be interacted with using a simple API built with FastAPI. The API is designed for easy integration with other applications and services, making it a versatile solution for real-time information retrieval.

### Key Technologies

- **Amazon Bedrock**: A fully managed AI service for building and deploying generative AI models.
- **Amazon S3**: Storage service used to store the knowledge base documents (PDFs, DOCs).
- **AWS CLI**: Command line tool used for interacting with AWS services locally.
- **FastAPI**: Web framework for building the chatbot's API.
- **Boto3**: Python SDK to interact with AWS services.

## Steps

### 1. **Create Knowledge Base on Amazon S3**

To set up an S3 bucket to store the training materials (documents such as PDFs and Word Docs). These documents form the knowledge base for the chatbot.

- **Bucket Name**: `johnlewis-rag-api`
- **Region**: `us-east-2` (Ohio)

Then create a knowledge base in Amazon Bedrock that uses these documents as its data source.

### 2. **Configure Amazon Bedrock**

In Amazon Bedrock:
- **Knowledge Base Name**: `johnlewis-intra-rag`
- **Embedding Model**: `Titan Text Embeddings V2`
- **Vector Store**: Amazon OpenSearch Serverless
- **S3 Bucket**: `johnlewis-rag-api` linked to the knowledge base for easy syncing.

### 3. **Choose AI Model for the Chatbot**

Select two models for our chatbot:
1. **Titan Text Embeddings V2**: Used for embedding documents and queries.
2. **Llama 3.3 70B Instruct**: Used for converting search results from the knowledge base into human-readable chat responses.

### 4. **Sync Knowledge Base with S3**

Synchronize the knowledge base with the S3 bucket to load the documents into our chatbot’s memory.

### 5. **Run AWS CLI Locally**

Use AWS CLI to interact with Bedrock directly from our local machine. This allows us to test the Bedrock model by sending queries through the terminal.

```bash
aws bedrock-agent-runtime retrieve-and-generate \
  --input '{"text": "Who is Liliana Demote?"}' \
  --retrieve-and-generate-configuration '{
    "knowledgeBaseConfiguration": {
      "knowledgeBaseId": "<your_knowledge_base_id>",
      "modelARN": "<your_model_arn>"
    },
    "type": "KNOWLEDGE_BASE"
  }' \
  --query 'output.text' \
  --output text
```

## 6. Build the API with FastAPI

The FastAPI framework is used to build the web API, which allows other applications to interact with the chatbot by sending HTTP requests. The API provides an endpoint for querying the chatbot.

### Key Components

- **FastAPI**: Web framework for creating the API.
- **Boto3**: AWS SDK for Python, used to connect to AWS services.
- **dotenv**: For loading environment variables from a `.env` file.

### Core FastAPI Code ([main.py](main.py))

```python
from fastapi import FastAPI, HTTPException, Query
import boto3
import os
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Retrieve AWS configuration from environment variables
AWS_REGION = os.getenv("AWS_REGION", "us-east-2")
KNOWLEDGE_BASE_ID = os.getenv("KNOWLEDGE_BASE_ID")
MODEL_ARN = os.getenv("MODEL_ARN")

app = FastAPI()

# Initialize Boto3 client for Bedrock Agent Runtime
def get_bedrock_client():
    return boto3.client("bedrock-agent-runtime", region_name=AWS_REGION)

@app.get("/")
async def root():
    return {"message": "Welcome to your RAG chatbot API!"}

@app.get("/bedrock/query")
async def query_bedrock(text: str = Query(..., description="Input text for the model")):
    client = get_bedrock_client()
    try:
        response = client.retrieve_and_generate(
            input={"text": text},
            retrieveAndGenerateConfiguration={
                "knowledgeBaseConfiguration": {
                    "knowledgeBaseId": KNOWLEDGE_BASE_ID,
                    "modelArn": MODEL_ARN
                },
                "type": "KNOWLEDGE_BASE"
            }
        )
        return {"response": response["output"]["text"]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## 7. Test the API Locally

To test the API locally, follow these steps:

### Clone the GitHub Repository:
```sh
git clone https://github.com/johnlewis/johnlewis-rag-api.git
cd johnlewis-rag-api
```

### Create a Virtual Environment:
```sh
python3 -m venv venv
source venv/bin/activate
```

### Install Dependencies:
```sh
pip3 install -r requirements.txt
```

### Run the API:
```sh
python3 -m uvicorn main:app --reload
```

### Test the API:
- Visit [http://127.0.0.1:8000](http://127.0.0.1:8000) to test the root endpoint.
- Use `/bedrock/query` to ask questions.

## 8. Create Environment Variables File

To securely store sensitive information such as the AWS region, knowledge base ID, and model ARN, create a `.env` file:

```ini
AWS_REGION=us-east-2
KNOWLEDGE_BASE_ID=<your_knowledge_base_id>
MODEL_ARN=<your_model_arn>
```

## 9. General Knowledge Query

An additional endpoint allows the API to call the AI model directly for general knowledge queries, without relying on the knowledge base. This makes the chatbot more versatile, providing both specific knowledge from the knowledge base and general information from the model.

## 10. Deploy the API

This project can be deployed to any cloud platform like AWS EC2, Lambda, or any hosting platform supporting FastAPI. The deployment requires setting up environment variables on the server and running the app with Uvicorn.


## Conclusion

This project demonstrates how to build and deploy a powerful RAG chatbot with Amazon Bedrock and S3. By building an API using FastAPI, this chatbot can be easily integrated with any application, opening up possibilities for creating chat interfaces, mobile apps, or other innovative solutions.

Feel free to reach out to me for further questions or contributions!


