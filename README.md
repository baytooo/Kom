# 📊 Dataset Stats Dashboard

A simple web app to manage and display image dataset class statistics using Python Flask, PostgreSQL, and a static HTML interface.

---

## 🔧 Tech Stack

| Layer         | Technology                  |
|---------------|-----------------------------|
| Backend       | Flask (Python)              |
| Database      | PostgreSQL on AWS RDS       |
| Frontend      | HTML + JavaScript (Vanilla) |
| Hosting       | Flask on AWS EC2            |
| Static Files  | Amazon S3 Static Website    |
| CORS Support  | Flask-CORS                  |

---

## 🗃️ Database Schema

```sql
CREATE TABLE dataset_stats (
  class TEXT PRIMARY KEY,
  image_count INT,
  avg_width INT,
  avg_height INT,
  formats TEXT,
  corrupt_files INT
);
```

---

## 🚀 Getting Started


### 1. Install Requirements

```bash
sudo apt update
sudo apt install python3-pip
python3.12 -m venv venv
source venv/bin/activate
pip install flask flask-cors psycopg2-binary
```
### 2. Run the Flask App

```bash
python app.py
```


### 3. Python and HTML files
Creaate a file in EC2
```bash
nano app.py
```
```bash
from flask import Flask, jsonify, request
import psycopg2
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

DB_HOST = "your database endport"
DB_NAME = "postgres"
DB_USER = "master username"
DB_PASS = "password"

def get_conn():
    return psycopg2.connect(
        host=DB_HOST,
        database=DB_NAME,
        user=DB_USER,
        password=DB_PASS
    )

@app.route('/stats', methods=['GET'])
def get_stats():
    conn = get_conn()
    cursor = conn.cursor()
    cursor.execute("SELECT class, image_count, avg_width, avg_height, formats, corrupt_files FROM dataset_stats")
    rows = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify([
        {
            "class": row[0],
            "image_count": row[1],
            "avg_width": row[2],
            "avg_height": row[3],
            "formats": row[4],
            "corrupt_files": row[5]
        } for row in rows
    ])

@app.route('/stats', methods=['POST'])
def add_stat():
    data = request.json
    conn = get_conn()
    cursor = conn.cursor()
    cursor.execute("""
        INSERT INTO dataset_stats (class, image_count, avg_width, avg_height, formats, corrupt_files)
        VALUES (%s, %s, %s, %s, %s, %s)
    """, (
        data['class'], data['image_count'], data['avg_width'],
        data['avg_height'], data['formats'], data['corrupt_files']
    ))
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"message": "Entry added"}), 201

@app.route('/stats/<class_name>', methods=['DELETE'])
def delete_stat(class_name):
    conn = get_conn()
    cursor = conn.cursor()
    cursor.execute("DELETE FROM dataset_stats WHERE class = %s", (class_name,))
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"message": f"Deleted class: {class_name}"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

Put this to your S3
```bash
<!DOCTYPE html>
<html>
<head>
  <title>Dataset Stats</title>
</head>
<body>
  <h1>Dataset Statistics</h1>

  <form onsubmit="addStat(); return false;">
    <input id="class" placeholder="Class" required>
    <input id="image_count" type="number" placeholder="Image Count" required>
    <input id="avg_width" type="number" placeholder="Avg Width" required>
    <input id="avg_height" type="number" placeholder="Avg Height" required>
    <input id="formats" placeholder="Format" required>
    <input id="corrupt_files" type="number" placeholder="Corrupt Files" required>
    <button type="submit">Add</button>
  </form>

  <table border="1">
    <thead>
      <tr>
        <th>Class</th>
        <th>Image Count</th>
        <th>Avg Width</th>
        <th>Avg Height</th>
        <th>Format</th>
        <th>Corrupt Files</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody id="stats-table"></tbody>
  </table>

  <script>
    const API = "http://yourec2:8000/stats";

    function fetchStats() {
      fetch(API)
        .then(res => res.json())
        .then(data => {
          const table = document.getElementById("stats-table");
          table.innerHTML = "";
          data.forEach(row => {
            const tr = document.createElement("tr");
            tr.innerHTML = 
              <td>${row.class}</td>
              <td>${row.image_count}</td>
              <td>${row.avg_width}</td>
              <td>${row.avg_height}</td>
              <td>${row.formats}</td>
              <td>${row.corrupt_files}</td>
              <td><button onclick="deleteStat('${row.class}')">❌ Delete</button></td>
            ;
            table.appendChild(tr);
          });
        });
    }

    function addStat() {
      const data = {
        class: document.getElementById("class").value,
        image_count: parseInt(document.getElementById("image_count").value),
        avg_width: parseInt(document.getElementById("avg_width").value),
        avg_height: parseInt(document.getElementById("avg_height").value),
        formats: document.getElementById("formats").value,
        corrupt_files: parseInt(document.getElementById("corrupt_files").value)
      };

      fetch(API, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      }).then(() => {
        document.querySelector("form").reset();
        fetchStats();
      });
    }

    function deleteStat(className) {
      fetch(${API}/${className}, {
        method: "DELETE"
      }).then(() => fetchStats());
    }

    // Load on start
    fetchStats();
  </script>
</body>
</html>
```
### 3. PostgreSQL Setup

- Launch an AWS RDS PostgreSQL instance
- Run the schema SQL above using DBeaver or psql
- Allow access on port 5432 (edit security group)

