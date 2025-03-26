A FastAPI-based API that enables users to upload, query, and chat with CSV files using Retrieval-Augmented Generation (RAG).

Setup and Installation
Prerequisites
Make sure you have the following installed:
Python 3.8+
pip (Python package manager)
MongoDB (Running locally or via cloud, e.g., MongoDB Atlas)

Installation Steps
Step 1: Clone or Extract the Build File

If using GitHub:
git clone https://github.com/your-repo/rag-csv-analyzer.git
cd rag-csv-analyzer

If using a build file (build.zip):
unzip build.zip
cd rag-csv-analyzer

Step 2: Install Dependencies
Run the following command to install required packages:
pip install -r requirements.txt

Step 3: Set Up Environment Variables
Create a .env file in the project folder and add:
MONGO_URI=mongodb://localhost:27017/
HF_API_URL=https://api-inference.huggingface.co/models/mistralai/Mistral-7B-Instruct
HF_API_TOKEN=your_huggingface_api_token  # Replace with actual API token

Step 4: Start MongoDB
If MongoDB is installed locally, start it:
mongod --dbpath /path/to/mongodb/data

Usage
Once the server is running, open Swagger UI to test the API:
http://127.0.0.1:8000/docs

After running
![image](https://github.com/user-attachments/assets/2d8f727b-791a-4bfd-855a-18a712d4f1c9)
![image](https://github.com/user-attachments/assets/10419ae4-fc0b-4ff8-b978-acdf460bfacc)
![image](https://github.com/user-attachments/assets/113ab8f0-fda7-44fe-bef1-87512f49d36f)
![image](https://github.com/user-attachments/assets/c3ae7cd4-67d9-4ccd-914f-4af811bf615c)

