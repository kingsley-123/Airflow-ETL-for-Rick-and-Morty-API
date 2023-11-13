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

# Data Model
The data model consists of a single table named "movie" with the following schema:
- name (varchar): Character name.
- gender (varchar): Character gender.
- species (varchar): Character species.
- status (varchar): Character status.
- origin (varchar): Character origin.
- location (varchar): Character location.
- no_of_episode (int): Number of episodes the character appeared in.
- url (text): Character URL.

  
![Untitled](https://github.com/kingsley-123/Rick-Morty-End-to-End-Data-Engineering-Project/assets/63650573/413a487f-c47b-490c-b85d-4e9ceaed8fbe)


# ETL Pipeline
The ETL pipeline consists of the following tasks:
- drop_table: Drops the existing "movie" table.
- create_table: Creates a new "movie" table with the specified schema.
- if_api_exit: Checks if the Rick and Morty API is accessible.
- extract_data: Makes an HTTP request to the API to extract character data.
- load_data: Processes the extracted data and loads it into the "movie" table in PostgreSQL.

![Screenshot (31)](https://github.com/kingsley-123/Rick-Morty-End-to-End-Data-Engineering-Project/assets/63650573/e259f019-b32b-4806-b600-bf7842d5ef87)



# Development Setup
To set up and run the DAG locally, follow these steps:
- Build or pull the Docker image for the environment.
- Configure Airflow connections for PostgreSQL (rick_morty_db) and HTTP (rick_morty) using the Airflow UI.
- Define the required Airflow variables: db_user, db_pass, redshift_conn_string, and movie_s3_config.
- Execute the DAG by initiating the Airflow webserver and scheduler.


# Import necessary modules and classes from Airflow and other libraries
- from airflow import DAG
- from datetime import datetime, timedelta
- from airflow.providers.postgres.operators.postgres import PostgresOperator
- from airflow.providers.http.sensors.http import HttpSensor
- from airflow.providers.http.operators.http import SimpleHttpOperator
- import json
- from airflow.operators.python import PythonOperator
- import pandas as pd
- import requests
- from sqlalchemy import create_engine

# Define a function to process data extracted from the Rick and Morty API
def process_data(ti):
    # Initialize an empty checklist to store data
    checklist = []
    
    # Retrieve the API response from the 'extract_data' task using XCom
    response = ti.xcom_pull(task_ids='extract_data')  

    # Extract the total number of pages from the API response
    total_pages = response['info']['pages']

    # Loop through each page and extract relevant information
    for page_num in range(1, total_pages + 1):
        # Make a request for the current page and convert the response to JSON
        page_response = requests.get(f'{response["info"]["next"]}?page={page_num}').json()

        # Extract information for each character on the page
        for item in page_response['results']:
            # Define a dictionary to store selected attributes of each character
            cols = {
                'name': item['name'],
                'gender': item['gender'],
                'species': item['species'],
                'status': item['status'],
                'origin': item['origin']['name'],
                'location': item['location']['name'],
                'no_of_episode': len(item['episode']),
                'url': item['url']
            }
            # Append the dictionary to the checklist
            checklist.append(cols)

    # Create a pandas DataFrame from the checklist
    df = pd.DataFrame(checklist)
    
    # Create a PostgreSQL database engine
    engine = create_engine('postgresql+psycopg2://airflow:airflow@postgres:5432/rick_morty')
    
    # Write the DataFrame to the 'movie' table in the database
    df.to_sql('movie', con=engine, if_exists='replace', index=False)

# Define default arguments for the DAG
default_args = {
    'retries': 5,
    'retry_delay': timedelta(minutes=10)
}

# Define the DAG with a DAG ID, schedule interval, start date, and default arguments
with DAG(
    dag_id='rick_morty',
    schedule_interval='@daily',
    start_date=datetime(2023, 11, 4),
    default_args=default_args
) as dag:
    # Define tasks in the DAG
    
    # Task to drop the 'movie' table in the PostgreSQL database
    task_1 = PostgresOperator(
        task_id='drop_table',
        postgres_conn_id='rick_morty_db',
        sql='''drop table movie'''
    )
    
    # Task to create the 'movie' table in the PostgreSQL database
    task_2 = PostgresOperator(
        task_id='create_table',
        postgres_conn_id='rick_morty_db',
        sql='''create table movie(
              name varchar(250),
              gender varchar(250),
              species varchar(250),
              status varchar(250),
              origin varchar(250),
              location varchar(250),
              no_of_episode int,
              url text
              )'''
    )
  
    # Task to check if the API is accessible
    task_3 = HttpSensor(
        task_id='if_api_exit',
        http_conn_id='rick_morty',
        endpoint='/character'
    )

    # Task to extract data from the Rick and Morty API
    task_4 = SimpleHttpOperator(
        task_id='extract_data',
        http_conn_id='rick_morty',
        endpoint='/character',
        method='GET',
        response_filter=lambda response: json.loads(response.text),
        log_response=True
    )

    # Task to process and load the extracted data into the PostgreSQL database
    task_5 = PythonOperator(
        task_id='load_data',
        python_callable=process_data
    )

    # Define the task dependencies
    task_1 >> task_2 >> task_3 >> task_4 >> task_5

# Potential Improvements
- Error Handling: Implement comprehensive error-handling mechanisms for tasks that involve external dependencies, such as API requests.
- Logging and Monitoring: Enhance logging and monitoring to facilitate better visibility into DAG execution and potential issues.
