# Rick-Morty-End-to-End-Data-Engineering-Project

# Purpose
This Airflow DAG, named "rick_morty," is designed to extract, process, and load data from the Rick and Morty API into a PostgreSQL database named "rick_morty." The purpose is to collect information about characters from the API, transform the data, and store it in a structured format for further analysis or usage.

# Architecture
 ![Rick_Morty_Architecture drawio](https://github.com/kingsley-123/Rick-Morty-End-to-End-Data-Engineering-Project/assets/63650573/7930580a-c97e-4801-ae78-77a32ef7092a)

# The DAG Tasks:
- drop_table: Drops the existing "movie" table from the PostgreSQL database.
- create_table: Creates a new "movie" table in the PostgreSQL database with a defined schema.
- if_api_exit: Uses an HTTP sensor to check if the Rick and Morty API is accessible.
- extract_data: Extracts character data from the Rick and Morty API using an HTTP request.
- load_data: Processes the extracted data and loads it into the "movie" table in the PostgreSQL database.

# Choice of Technologies
The technologies used in this DAG include:
- Apache Airflow: Chosen for orchestrating the ETL pipeline, providing a modular and scheduled approach.
- PostgreSQL: Selected as the database for storing the processed data.
- HTTP Sensor and SimpleHttpOperator: Leveraged for checking API availability and making HTTP requests to the Rick and Morty API.
- Pandas and SQLAlchemy: Utilized for data processing and loading into the PostgreSQL database.
