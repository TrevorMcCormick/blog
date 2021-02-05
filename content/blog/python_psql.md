---
title: "🐍 How To Query PostgreSQL with Python"
date: 2021-02-04T11:20:22-04:00
draft: true
tags: [Python, AWS, RDS, How-To, Databases]
---

I wanted to document how I've been turning extracts from Postgres into Pandas dataframes. I haven't spent any time optimizing this code, but it works, and it is pretty easy to read. The only requirement to run these functions would be a valid Postgres database. I'll walk through:
* how to create a public PostgreSQL instance in RDS using free-tier
* 

## RDS
The only thing you have to do in the AWS console is create a Postgres DB instance and make sure it is open to the public (just for this example). Here is how to do that:

* Go to [Databases in RDS](https://us-east-2.console.aws.amazon.com/rds/home?region=us-east-2#databases:), and choose the region you want to create a database instance
* Create a database, selecting "Standard Create", and the PostgreSQL configuration. Make sure to use free-tier. You can name the database anything you want, and choose a username and password. The most important step here is in "Connectivity": make sure to fill in the bubble for "Yes" to Public Access. If you're using this database for any real-life work, then you'll want to fill in "No". You'll have to do some work to configure security groups and look at your architecture to only allow connections you want to approve.
* Once the database has been created, you'll be able to find the database endpoint in the "Connectivity & security" section. You'll use that to create a json file with your credentials. It should look something like this, with the values specific for your instance:
```json
{
"user":"postgres",
"password":"password",
"database":"postgres",
"host":"xxxx.xxxxxxx.us-east-2.rds.amazonaws.com"
}
```

## Python
Now that we have a Postgres DB instance, we'll need to try to connect to it. To go forward with this exercise, you'll need **pip**, and you'll need to execute the following in your terminal (I use VS code): `pip install psycopg2-binary pandas sqlalchemy`

In this example, I've provided some example functions that you can use to get started. Here is a quick summary of the sections, with the actual python code at the bottom of the post.

* **Import**: so you can skip a bunch of database driver steps
* **Client**: to connect to the psql instance for queries.
* **Load**: to load data into psql. I only put one function in this class for an example, so you can create and load a table in one step.
* **Query**: to query data in a table within your DB instance
* **Meta**: to inspect the DB instance


## Google Colab

I typically use Jupyter Notebooks in something like [Google Colab](https://colab.research.google.com/) when running code, because it's easy to run code blocks and debug interactively. In the Google Colab environment, you need to upload two files: your credentials json file, and your dataset. In this case, I've downloaded [the iris dataset](https://gist.githubusercontent.com/curran/a08a1080b88344b0c8a7/raw/0e7a9b0a5d22642a06d3d5b9bcbad9890c8ee534/iris.csv) and I will upload it to my DB instance as the table **iris**.

Here is a screengrab of me going through these steps, and [here is the link to the Python Notebook](https://colab.research.google.com/drive/1HmU9yFTJ30LzLf9ql8ahCcIuSb8RDh89?usp=sharing) that you can upload to your own Colab environment.

{{< youtube cJdXgguTEeY>}}


Your Python code will look something like this. Hope you have enjoyed the walkthrough!

```python
# Import 
import json
import pandas as pd
import psycopg2
from sqlalchemy import create_engine
#  Client
class Client:
    # Create a global psql connection object
    def __init__(self, user, password, database, host):
        self.user = user
        self.password = password
        self.database = database
        self.host = host
        global connection
        connection = psycopg2.connect(user = self.user,
                       password = self.password,
                       database = self.database,
                       host = self.host)
    def __repr__(self):
        return "User {0} connected to {1}.".format(self.user, self.database)
    @classmethod
    # Easy login with json
    def from_json(cls, file_path):
        with open(file_path, mode="r") as json_file:
            # Store credentials to the global namespace
            global credentials
            credentials = json.load(json_file)
        # Use credentials to log in
        return cls(credentials["user"],
                   credentials["password"],
                   credentials["database"],
                   credentials["host"])
class Load:
    @classmethod
    # One-step create table from pandas df
    def create_and_load(cls, data, tablename): 
        try:
            # Create sqlalchemy engine for pandas df.to_sql function
            engine = create_engine('postgresql+psycopg2://{0}:{1}@{2}/{3}'.format(credentials["user"], credentials["password"], credentials["host"], credentials["database"]))
        except Exception as ex:
            print('Exception:')
            print(ex)
        try:
            # Read csv and do not include index column
            df = pd.read_csv(data,index_col=False)
            # Do not store into postgres with an index column
            df.to_sql("{0}".format(tablename), con=engine, index=False)
            print("Created table: {0}".format(tablename))
        except Exception as ex:
            print('Exception:')
            print(ex)
        connection.commit()
    @classmethod
    # One-step drop table
    def drop(cls, table):
        cursor = connection.cursor()
        cursor.execute("""
        DROP TABLE IF EXISTS {0}  
        """.format(table)
        )
        connection.commit()
        cursor.close()
        print("Dropped table: {0}".format(table))
class Query:
    @classmethod
    # Retrieve column names in given table
    def get_columns(cls, table):
        cursor = connection.cursor()
        cursor.execute("""
        SELECT * FROM {0} LIMIT 0    
        """.format(table)
        )
        colnames = [desc[0] for desc in cursor.description]
        cursor.close()
        return(colnames)
    @classmethod
    # Return subset of table
    def get_sample(cls, table, num_records):
        cursor = connection.cursor()
        cursor.execute("""
        SELECT * FROM {0} LIMIT {1}    
        """.format(table, num_records)
        )
        result = cursor.fetchall()
        cursor.close()
        columns = Query.get_columns(table)
        df = pd.DataFrame(result, columns=columns)
        return(df)
    @classmethod
    # Get all records
    def fetchall(cls, table):
        cursor = connection.cursor()
        cursor.execute("""
        SELECT * FROM {0}
        """.format(table))     
        result = cursor.fetchall()
        colnames = [desc[0] for desc in cursor.description]
        cursor.close()
        df = pd.DataFrame(result, columns=colnames)
        return(df)
    @classmethod
    # Basic query
    def query(cls, query):
        cursor = connection.cursor()
        cursor.execute("""
        {0}
        """.format(query))     
        result = cursor.fetchall()
        colnames = [desc[0] for desc in cursor.description]
        cursor.close()
        df = pd.DataFrame(result, columns=colnames)
        return(df)
class Meta:
    @classmethod
    # See all tables in DB instance
    def get_tables(cls):
        cursor = connection.cursor()
        cursor.execute("""
        SELECT table_name FROM information_schema.tables
        WHERE table_schema = 'public'
        """)
        result = cursor.fetchall()
        cursor.close()
        if len(result) < 1:
            result = "There are no tables in this database"
        return(result)
```