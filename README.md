# 📦 Flask API for EDI File Processing

This project provides a Flask-based API that takes an EDI file as input and processes it using AI (Gemini LLM) to extract structured shipment data. The data is matched with a PostgreSQL database, and relevant information is inserted into a freight audit table.

---

## 🚀 API Overview

The Flask API accepts EDI files via a POST endpoint and processes them through AI extraction, JSON transformation, database lookup, and insertion operations.

### Endpoint

- **URL:** `http://127.0.0.1:5000/process-edi`
- **Method:** `POST`
- **Body (form-data):**
  - `edi_file`: Upload the EDI file

---

## 🤖 AI Processing – Using Gemini LLM

After receiving the EDI file, the API uses Gemini LLM to extract structured fields like tracking number, shipment ID, carrier name, origin, and destination.

### 🔍 Sample Gemini LLM Output

```json
{
  "shipment_id": "123456789",
  "carrier": "UPS",
  "origin": "Los Angeles",
  "destination": "New York",
  "tracking_number": "TRK987654321"
}
```

---

## 🔧 Post-Processing Gemini Data

After getting the response from Gemini, we convert the AI-generated JSON into a cleaner, structured format for matching with the database.

### 🧠 Sample Python Code

```python
def format_gemini_output(gemini_json):
    return {
        "shipment": {
            "id": gemini_json.get("shipment_id"),
            "tracking_number": gemini_json.get("tracking_number"),
            "carrier": gemini_json.get("carrier"),
            "route": {
                "origin": gemini_json.get("origin"),
                "destination": gemini_json.get("destination")
            }
        }
    }
```

---

## 📬 API Usage and Response

You can test the API using Postman or curl.

### ✅ Example Request (via Postman)

- **Method:** POST  
- **URL:** `http://127.0.0.1:5000/process-edi`  
- **Body (form-data):**
  - Key: `edi_file`
  - Value: *(EDI file upload)*

### 📦 Sample JSON Response

```json
{
  "status": "success",
  "tracking_number": "TRK987654321",
  "matched_data": {
    "shipment_id": "123456789",
    "carrier": "UPS",
    "origin": "Los Angeles",
    "destination": "New York"
  },
  "message": "Data inserted into freight_audit table."
}
```

---

## 🗃️ Database Integration – Tracking Number Matching

The most important field extracted from the EDI file is the **tracking number**.

- We use the `psycopg2` Python library to connect to a PostgreSQL database.
- The extracted tracking number is matched with rows in the `Shipments` table.

### 🧩 Database Connection Example

```python
import psycopg2

def get_db_connection():
    return psycopg2.connect(
        dbname="your_db",
        user="your_user",
        password="your_password",
        host="localhost",
        port="5432"
    )
```

---

## 🔄 Common Data & Database Manipulation

Once the tracking number is matched in the database, the system performs the following:

1. Compares the original JSON extracted from the EDI file with the matched row in the `Shipments` table.
2. Finds the common fields between both.
3. Adds the common data to a new field called `common_data` in the final payload.
4. Inserts the final payload into the `freight_audit` table.
5. Returns a success message.

### 🧪 Sample Merge and Insert Code

```python
def get_matched_shipment(tracking_number):
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute("SELECT * FROM Shipments WHERE tracking_number = %s", (tracking_number,))
    row = cur.fetchone()
    cur.close()
    conn.close()
    return row

def find_common_data(original_json, db_row_dict):
    return {
        key: original_json[key]
        for key in original_json if key in db_row_dict and original_json[key] == db_row_dict[key]
    }

def insert_freight_audit_data(tracking_number, common_data):
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute("""
        INSERT INTO freight_audit (tracking_number, common_data)
        VALUES (%s, %s)
    """, (tracking_number, json.dumps(common_data)))
    conn.commit()
    cur.close()
    conn.close()
```

---

## ✅ Final Workflow Summary

1. API receives an EDI file.
2. Gemini LLM extracts relevant shipment fields.
3. JSON data is cleaned and transformed.
4. Tracking number is used to query the `Shipments` table.
5. Common fields are calculated and stored in the `freight_audit` table.
6. A success message is returned with the tracking number and matched data.

---

## 🔐 Notes

- Ensure you set your PostgreSQL credentials securely using environment variables or a `.env` file.
- Make sure Gemini API access is set up and properly authorized.
- Handle large EDI files and slow network requests using timeouts and async tasks if needed.

---

### 📧 Contact

For questions or contributions, feel free to open an issue or pull request.