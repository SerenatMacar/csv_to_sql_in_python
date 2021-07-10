# csv_to_sql_in_python
This code transfers a csv file from your computer to SQL database by using Python. It converts text to columns(data frame). In other words, csv --> data frame --> SQL table. 


import pandas as pd
import pyodbc
from fast_to_sql import fast_to_sql as fts
from datetime import date


def import_csv_file():
    
    #IMPORT CSV 
    column_names = ["id","Name","Age","Hobbies","Goals","Values"]
    first_row =   'id;Name;Age;Hobbies;Goals;Values' #INSERT FIRST ROW OF THE CSV FILE
    df = pd.read_csv('INSERT_THE_NAME_OF_YOUR_CSV_FILE_HERE.csv', dtype={first_row:str})
    print("CSV IMPORTED")
    
    
    
    
    #SPLIT TEXT INTO COLUMNS
    split_cols = df['id;Name;Age;Hobbies;Goals;Values'].str.split(';',expand=True) # You should be inserting the columns of the data frame in the string as shown.
    split_cols.columns = [f'{i}' for i in column_names]  #Renaming columns if necessary 
    df = split_cols
    print("TEXT SPLIT INTO COLUMNS")
    
    
    
    
    #DATE COLUMN ADDED
    #This is to add a date column as a first column. If not necessary, you might skip this part.
    row, col = df.shape
    today = date.today()
    today = today.strftime("%B %d, %Y")
    date_update = [today for i in range(row)]
    df.insert(0,'Date Update',date_update)
    print("DATE COLUMN ADDED")
     
    
    
    
    
    #CONNECT TO THE SQL SERVER
    server = 'SERVER_NAME'           #server_name.database.windows.net You should be inserting your server name here. It might have a similar structure with the one stated.
    database = 'NAME_OF_DATABASE'    #Equalize database into your database name. It should be a string. Same applies for username and password.
    username = 'USERNAME'
    password = 'PASSWORD'
    
    conn = pyodbc.connect('DRIVER={ODBC Driver 17 for SQL Server};'
                                    'SERVER='+server+';'
                                    'DATABASE='+database+';'
                                    'UID='+username+';'
                                    'PWD=' + password)
    cursor = conn.cursor()
    
    print("CONNECTION SATISFIED")
    
    
    
    
    #CREATE TABLE AND TRANSFER DF INTO THE TABLE
    table_name = "NAME_YOUR_TABLE_HERE"
    create_statement = fts.fast_to_sql(df, "NAME_YOUR_TABLE_HERE", conn, if_exists="replace", custom={"id":"nvarchar(50)"}, temp=False) 
    #Inside the custom, you can add some more parameters if necessary. For example: custom={"id":"nvarchar(50)", "Age": "int"}
    print(f"TABLE CREATED AND DF TRANSPORTED. Table name: {table_name}.")
    # Commit upload actions and close connection
    conn.commit()
    conn.close() 
    
import_csv_file()
