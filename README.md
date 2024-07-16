
# Natural Language Interface for MySQL Database Using Python and LangChain

This project demonstrates how to create a natural language interface for querying a MySQL database using Python and LangChain. The project allows users to input questions in natural language, which are then converted to SQL queries and executed on a MySQL database. The results are returned in natural language.

## Table of Contents
- [Introduction](#introduction)
- [Features](#features)
- [Setup](#setup)
- [Usage](#usage)
- [Credits](#credits)
- [License](#license)

## Introduction
This project showcases the use of LangChain to bridge the gap between natural language and SQL queries. By leveraging a large language model (LLM), the system can understand user questions and generate the appropriate SQL queries to retrieve data from a MySQL database.

## Features
- Convert natural language questions into SQL queries.
- Execute SQL queries on a MySQL database.
- Return query results in natural language.

<img width="678" alt="image" src="https://github.com/user-attachments/assets/4842828b-be34-4e49-8acd-65d338d5d7d3">


## Setup
### Prerequisites
- Python 3.7+
- MySQL database

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/abdoolamunir/Chat-with-mySQL
   ```

2. Install the required Python packages:
   ```bash
   pip install langchain mysql-connector-python openai python-dotenv
   ```

3. You can create a `.env` file in the root directory of your project and add your credentials, for a proof of concept i opted not to:
   ```
   OPENAI_API_KEY=your-openai-api-key
   DB_URI=mysql+mysqlconnector://username:password@localhost:3306/Chinook
   ```

## Usage
1. Define the SQL Chain:
   ```python
   from langchain_core.prompts import ChatPromptTemplate
   template = """
   Based on the table schema below, write a SQL Query that will answer the user's question:
   {schema}

   Question: {question}
   SQL Query:
   """
   prompt = ChatPromptTemplate.from_template(template)
   ```

2. Test the prompt:
   ```python
   prompt.format(schema="your schema", question="How many users are there?")
   ```

3. Create a SQLDatabase instance and test the connection:
   ```python
   from langchain_community.utilities import SQLDatabase
   db = SQLDatabase.from_uri(db_uri)
   db.run("SELECT * FROM your_table LIMIT 5")
   ```

4. Define functions to get schema and run queries:
   ```python
   def get_schema(_):
       return db.get_table_info()

   def run_query(query):
       return db.run(query)
   ```

5. Create the LangChain pipeline and test:
   ```python
   from langchain_core.runnables import RunnablePassthrough
   from langchain_openai import ChatOpenAI

   llm = ChatOpenAI()
   sql_chain = (
       RunnablePassthrough.assign(schema=get_schema)
       | prompt
       | llm.bind(stop="\nSQL Result:")
       | StrOutputParser()
   )
   sql_chain.invoke({"question": "How many artists are there?"})
   ```

6. Create and test the full chain:
   ```python
   template = """
   Based on the schema below, question, sql query, and sql response, write a natural language response:
   {schema}

   Question: {question}
   SQL Query: {query}
   SQL Response: {response}
   """
   prompt = ChatPromptTemplate.from_template(template)
   full_chain = (
       RunnablePassthrough.assign(query=sql_chain).assign(
           schema=get_schema,
           response=lambda vars: run_query(vars["query"]),
       )
       | prompt
       | llm
       | StrOutputParser()
   )
   full_chain.invoke({"question": "How many artists are there?"})
   ```



## Credits
This project was inspired by and uses code from Alejandro AO's [tutorial](https://alejandro-ao.com/chat-with-mysql-using-python-and-langchain/). Special thanks to Alejandro for his detailed guide.

## License
This project is licensed under the MIT License.
```
