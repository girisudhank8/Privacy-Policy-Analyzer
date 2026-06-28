# Privacy Policy Analyzer (Text-Based) 🔐🤖

An AI-powered, C++ application designed to make data privacy transparent and easily understandable. The tool parses privacy policies, highlights critical data clauses, categorizes privacy risks, and generates human-readable AI summaries using a locally integrated **Gemma 2B** Large Language Model.

---

## 💡 Project Architecture & Core Features

*   **🧠 AI-Powered Summarization:** Integrates with a local **Gemma 2B** instance via a dedicated `LLMManager` server connection to generate actionable 3-4 sentence summaries based on identified risks.
*   **📂 Object-Oriented C++ Design:** Engineered cleanly with dedicated classes including `TextAnalyzer`, `KeywordMatcher`, `DatabaseManager`, and `LLMManager`.
*   **💾 Persistent Storage:** Leverages a local **MariaDB/MySQL** database containing a predefined dictionary of 48+ legal terms classified under specific privacy domains. It also securely saves policy text, execution histories, and generated summaries.

### 🛡️ Analysis Domains Covered
1.  `data_collection`: Identifies tracking keywords like *collect*, *personal information*, *cookie*, *location*.
2.  `data_sharing`: Detects clauses involving *third-party*, *share*, *disclose*, *transfer*.
3.  `security`: Flags encryption and protection terms like *encrypt*, *secure*, *breach*.
4.  `user_rights`: Highlights privacy rights and agency terms like *opt-out*, *delete*, *access*, *consent*.
5.  `tracking`: Monitored tracking indicators like *user behavior*, *analytics*, *IP address*.

---

## ⚙️ Tech Stack & Requirements

- **Language:** C++ (C++11 or higher)
- **Database:** MariaDB / MySQL Server
- **Libraries:** `libmariadb` (MariaDB Client Library for C connectivity)
- **LLM Engine:** Google Gemma 2B (hosted via local API endpoint)
- **OS Environment:** Linux / WSL (Ubuntu, Kali, Debian, etc.)

---

## 🚀 Setup & Installation Instructions

Follow these steps to set up, build, and run the Privacy Policy Analyzer on your machine (e.g., WSL or Kali Linux):

### 1. Install System Dependencies
Update your package repository and install build utilities along with MariaDB developer client libraries:
```bash
sudo apt update
sudo apt install build-essential cmake libmariadb-dev libmariadb-dev-compat mariadb-server -y
```

### 2. Configure the MariaDB Database
Start the database service and configure the schema requirements:
```bash
# Start the local MariaDB server
sudo service mariadb start

# Access the MariaDB terminal as root
sudo mysql -u root -p
```
Inside the MariaDB prompt, create the database and seed tables:
```sql
CREATE DATABASE privacy_db;
USE privacy_db;

-- Table to store custom privacy keywords and their classification
CREATE TABLE privacy_keywords (
    id INT AUTO_INCREMENT PRIMARY KEY,
    keyword VARCHAR(255) NOT NULL,
    category VARCHAR(100) NOT NULL
);

-- Table to store uploaded/analyzed documents
CREATE TABLE stored_policies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    content TEXT NOT NULL,
    source VARCHAR(50) NOT NULL,
    filename VARCHAR(255),
    char_count INT,
    analysis_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

-- Table to link policy IDs with evaluation metadata and AI generation summaries
CREATE TABLE policy_analysis (
    id INT AUTO_INCREMENT PRIMARY KEY,
    policy_id INT,
    keyword_analysis TEXT NOT NULL,
    ai_summary TEXT,
    analysis_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
    FOREIGN KEY (policy_id) REFERENCES stored_policies(id)
);
```

### 3. Ensure the Gemma 2B Server is Running
Make sure your local server hosting the **Gemma 2B** model is up and listening on the API URL configured inside `LLMManager` (default timeout handling is bounded up to 120 seconds).

### 4. Build and Compile the Application
Compile the project binaries by linking against the local MariaDB client library:
```bash
g++ main.cpp DatabaseManager.cpp KeywordMatcher.cpp TextAnalyzer.cpp LLMManager.cpp -o privacy_analyzer -lmariadb
```

---

## 📖 How to Use the Application

Launch the program using:
```bash
./privacy_analyzer
```

Upon executing, a CLI dashboard console layout presents the main options sequence:

```text
--------- MENU ---------
1. Load text from file
2. Enter text manually
3. Analyze privacy policy
4. Generate AI summary
5. Store current policy in database
6. Store analysis results in database
7. View stored policies
8. View analysis history
9. Exit
------------------------
```

### Typical Workflow Sequence:
1.  **Option 1:** Type `1` and input your target file (e.g., `file.txt`).
2.  **Option 3:** Type `3` to perform raw phrase pattern token matches against the database dictionary to flag terms.
3.  **Option 4:** Select `4` to communicate with the local **Gemma 2B** LLM engine and output your 3-to-4 sentence natural language plain-text executive brief.
4.  **Options 5 & 6:** Store the respective text assets and generated analytical values directly into your persistent tables.
5.  **Options 7 & 8:** Browse historical entry records and inspect classification occurrences over past executions dynamically.
