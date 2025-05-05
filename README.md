📊 Dataset Stats Dashboard
This project allows users to view, add, and delete image dataset class statistics through a simple web interface. The backend is built in Flask (Python) and connected to a PostgreSQL RDS database, while the frontend is a static HTML+JS page that fetches data from the API.

⚙️ Stack Overview
Component	Technology
Backend	Flask + psycopg2
Database	PostgreSQL (AWS RDS)
Frontend	HTML + JavaScript
Hosting	Flask on AWS EC2
Static Files	Amazon S3 Website
CORS Support	flask-cors

🚀 API Endpoints
GET /stats — Fetch all dataset stats

POST /stats — Add a new class stat

DELETE /stats/<class_name> — Delete a class by name

🗃️ PostgreSQL Table Schema
sql
Copy
Edit
CREATE TABLE dataset_stats (
  class TEXT PRIMARY KEY,
  image_count INT,
  avg_width INT,
  avg_height INT,
  formats TEXT,
  corrupt_files INT
);
🧪 Example Data Entry
class	image_count	avg_width	avg_height	formats	corrupt_files
cat	500	224	224	JPG	0

🔧 Setup
Backend (on EC2)

bash
Copy
Edit
sudo apt update
sudo apt install python3-pip
pip install flask flask-cors psycopg2-binary
python3 app.py
Database

Create the dataset_stats table in PostgreSQL RDS

Allow public access (port 5432)

Use DBeaver or psql to connect and verify

Frontend

Save the HTML file as index.html

Upload to your S3 bucket

Enable static website hosting

Visit the HTTP endpoint (not HTTPS if Flask is not secured)
