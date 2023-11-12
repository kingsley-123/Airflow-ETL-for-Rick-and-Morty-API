# Rick-Morty-End-to-End-Data-Engineering-Project

# Purpose
This Airflow DAG, named "rick_morty," is designed to extract, process, and load data from the Rick and Morty API into a PostgreSQL database named "rick_morty." The purpose is to collect information about characters from the API, transform the data, and store it in a structured format for further analysis or usage.

# Architecture
The DAG follows a simple sequence of tasks:
-- drop_table: Drops the existing "movie" table from the PostgreSQL database.
-- create_table: Creates a new "movie" table in the PostgreSQL database with a defined schema.
-- if_api_exit: Uses an HTTP sensor to check if the Rick and Morty API is accessible.
-- extract_data: Extracts character data from the Rick and Morty API using an HTTP request.
-- load_data: Processes the extracted data and loads it into the "movie" table in the PostgreSQL database.

### Hi there 👋
- 🔭 I’m currently working on a Document Streaming project using Apache Spark, Kafka, Flink, MongoDB, and PowerBI
- 📫 How to reach me: kingskonk@gmail.com
- ⚡ Fun fact: If data were dollars, we'd all be billionaires! By 2025, we're looking at 175 zettabytes of information. 
    That's more zeros than my bank account!" 💰📊😄
