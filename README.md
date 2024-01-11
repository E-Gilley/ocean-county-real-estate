# <a href="https://public.tableau.com/views/OceanCountyRealEstateMarketOverview/OceanCountyDashboard?:language=en-US&:display_count=n&:origin=viz_share_link" target="_blank">Ocean County Real Estate Market Analysis</a>


[![A dashboard showing real estate information for Ocean County New Jersey](images/Dashboard.png?raw=true)](https://public.tableau.com/views/OceanCountyRealEstateMarketOverview/OceanCountyDashboard?:language=en-US&:display_count=n&:origin=viz_share_link){:target="_blank"}

## Purpose

This project walks through downloading property data from Redfin with the goal of using the information to create a Tableau dashboard.

Please click on the image above or the heading of the project to get direct access to the dashboard.

## The Problem

Two real estate investors focus mainly on Ocean County New Jersey. The investors spend a lot of time analyzing properties at the micro-level. They came to me for a way to "see the forrest through the trees".

They wanted an easy way to check on the Ocean County real estate market at a macro level.

All of the information below walks through how the data was extracted from RedFin and imported to Tableau using a Google Colab notebook and Python.

## <font color="white">Imports</font>

```Python
from google.colab import drive, files  # Specific to Google Colab, used for file management
from datetime import datetime  # For working with date and time
import pandas as pd  # Import Pandas library for data manipulation and analysis
import time  # For timing the execution of code
import warnings  # To handle warning messages
import requests  # Library for making HTTP requests
import io  # For working with streams and binary data

# Set Pandas options to display all columns in DataFrame
pd.set_option('display.max_columns', None)

# Filter out warnings to avoid clutter in output
warnings.filterwarnings('ignore')
```
## <font color="white">Data</font>

### Data Source

The data is sourced from [Redfin's zipcode-level data](https://www.redfin.com/news/data-center/).

Ideally, I'd download all the data and be able to use one notebook to filter data for every state. Unfortunately, it appears creating a Notebook for each state you want to analyze is the most feasible option.

In this case, I filtered for New Jersey and limited our chunk size to not use up all the RAM.

```Python
state_code = 'NJ'  # Define the state code to filter by

chunk_size = 100000  # Adjust the chunk size according to your system's memory capacity

url = 'https://redfin-public-data.s3.us-west-2.amazonaws.com/redfin_market_tracker/zip_code_market_tracker.tsv000.gz'

startTime = time.time()  # Record the start time for measuring execution time

response = requests.get(url, stream=True)  # Send an HTTP GET request to the URL, streaming the response content

if response.status_code == 200:  # Check if the response status is successful
    # Read the streamed content in chunks, using specified chunk size, and handle potential problematic lines
    df_chunks = pd.read_csv(io.BytesIO(response.content), compression='gzip', sep='\t', chunksize=chunk_size, on_bad_lines='skip')

    filtered_chunks = []  # Initialize an empty list to store filtered chunks of the DataFrame
    for chunk in df_chunks:  # Loop through each chunk of the DataFrame
        # Filter each chunk based on the 'state_code' column
        filtered_chunk = chunk[chunk['state_code'] == state_code]
        filtered_chunks.append(filtered_chunk)  # Append each filtered chunk to the list

    df_filtered = pd.concat(filtered_chunks)  # Concatenate all filtered chunks into a single DataFrame
    executionTime = (time.time() - startTime)  # Calculate the execution time
    print('Execution time in minutes: ' + str(round(executionTime / 60, 2)))  # Display the execution time in minutes
    print('Num of rows after filtering by state:', len(df_filtered))  # Display the total number of rows in the filtered DataFrame
    print('Num of cols:', len(df_filtered.columns))  # Display the total number of columns in the filtered DataFrame
    display(df_filtered.head())  # Show the first few rows of the filtered DataFrame
else:
    print("Failed to retrieve data")  # Display a message if data retrieval fails
```


