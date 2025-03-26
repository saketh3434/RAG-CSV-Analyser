from fastapi import FastAPI, File, UploadFile, HTTPException, Query
from pydantic import BaseModel
from typing import List
import pandas as pd
import os
import uuid
from pymongo import MongoClient
import requests

# Initialize FastAPI app
app = FastAPI()

# MongoDB connection
client = MongoClient("mongodb://localhost:27017/")
db = client["rag_csv_db"]
collection = db["csv_files"]

UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

# Hugging Face API settings
HF_API_URL = "https://api-inference.huggingface.co/models/mistralai/Mistral-7B-Instruct"
HF_API_TOKEN = "hUGGING FACE TOKEN"  # Replace with your HF token

headers = {"Authorization": f"Bearer {HF_API_TOKEN}"}

# Model for Query Request
class QueryRequest(BaseModel):
    file_id: str
    query: str

@app.post("/upload")
async def upload_csv(file: UploadFile = File(...)):
    try:
        file_id = str(uuid.uuid4())
        file_path = os.path.join(UPLOAD_DIR, file.filename)
        
        with open(file_path, "wb") as buffer:
            buffer.write(await file.read())
        
        df = pd.read_csv(file_path)
        data = df.to_dict(orient="records")
        
        collection.insert_one({"file_id": file_id, "file_name": file.filename, "data": data})
        
        return {"file_id": file_id, "message": "Upload successful"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/files")
async def list_files():
    try:
        files = list(collection.find({}, {"_id": 0, "file_id": 1, "file_name": 1}))
        return {"files": files}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
@app.post("/query")
async def query_csv(request: QueryRequest):
    file = collection.find_one({"file_id": request.file_id}, {"_id": 0, "data": 1})
    if not file:
        raise HTTPException(status_code=404, detail="File not found")

    df = pd.DataFrame(file["data"])

    try:
        # Convert all columns to lowercase for case-insensitive searching
        df.columns = df.columns.str.lower()

        # Check if query is asking about the CSV structure
        if "field" in request.query.lower() or "columns" in request.query.lower():
            return {"response": f"The CSV contains these fields: {', '.join(df.columns)}"}

        # Match the query against CSV contents
        mask = df.apply(lambda row: row.astype(str).str.contains(request.query, case=False, na=False).any(), axis=1)
        relevant_data = df[mask]

        if relevant_data.empty:
            return {"response": "No relevant data found in CSV."}

        # Convert relevant rows to string for LLM input
        csv_context = relevant_data.to_string(index=False)

        # Query Hugging Face API
        payload = {"inputs": f"Given this CSV data:\n{csv_context}\nAnswer this question: {request.query}"}
        response = requests.post(HF_API_URL, headers=headers, json=payload)

        if response.status_code != 200:
            raise HTTPException(status_code=500, detail="Error querying LLM API")

        result = response.json()
        return {"response": result[0]["generated_text"]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.delete("/file/{file_id}")
async def delete_file(file_id: str):
    result = collection.delete_one({"file_id": file_id})
    if result.deleted_count == 0:
        raise HTTPException(status_code=404, detail="File not found")
    return {"message": "File deleted successfully"}
