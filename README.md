
# Automatize Consultas SQL com Python, LangChain e MySQL
## Sumário

- [Introdução](#introdução)
- [Pré-requisitos](#pré-requisitos)
- [Configuração do Banco de Dados](#configuração-do-banco-de-dados)
    - [SQLite](#sqlite)
    - [MySQL](#mysql)
- [Criando a Cadeia LangChain](#criando-a-cadeia-langchain)
    - [Instalando Pacotes](#instalando-pacotes)
    - [Carregando o Banco de Dados](#carregando-o-banco-de-dados)
    - [Criando a Cadeia SQL](#criando-a-cadeia-sql)
    - [Criando a Cadeia Completa](#criando-a-cadeia-completa)
- [Conclusão](#conclusão)

## Introdução

Neste tutorial, você aprenderá a interagir com um banco de dados **MySQL** (ou SQLite) de maneira dinâmica, utilizando Python e LangChain. Exploraremos como o LangChain, aliado ao SQLAlchemy, pode facilitar essa conexão. Além disso, vamos desenvolver uma cadeia personalizada com o LangChain para permitir que consultas ao banco de dados sejam realizadas de forma intuitiva, usando linguagem natural.

![Diagrama sem nome.drawio (1).png](Diagrama_sem_nome.drawio_(1).png)

### Pré-requisitos

Antes de começar, certifique-se de ter os seguintes itens instalados:

- **Python 3.9 ou superior**
- **MySQL**
- **SQLite**

## Configuração do Banco de Dados

### SQLite

1. Instale o SQLite:
     - **Mac ou Linux**: O SQLite geralmente já vem instalado no sistema.
     - **Windows**: Baixe os binários pré-compilados na página de download do SQLite.

2. Crie um banco de dados:
     ```bash
     sqlite3 chinook.db
     ```

3. Carregue o banco de dados:
     ```bash
     .read Chinook.sql
     ```

4. Teste a configuração:
     ```bash
     SELECT * FROM albums;
     ```

### MySQL

1. Instale o MySQL:
     - [How to Install MySQL on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)
     - [How to Install MySQL on Windows by Amit](https://www.youtube.com/watch?v=uj4OYk5nKCg)

2. Configure o banco de dados:
     ```bash
     mysql -u root -p
     CREATE DATABASE chinook;
     USE chinook;
     SOURCE chinook.sql;
     ```

3. Teste a configuração:
     ```sql
     SELECT * FROM albums LIMIT 10;
     ```

## Criando a Cadeia LangChain

### Instalando Pacotes

Instale os pacotes necessários:
```bash
pip install langchain mysql-connector-python
```

### Carregando o Banco de Dados

Carregue o banco de dados com o seguinte código:
```python
from langchain_community.utilities import SQLDatabase

sqlite_uri = 'sqlite:///./Chinook.db'
mysql_uri = 'mysql+mysqlconnector://root:admin@localhost:3306/test_db'

db = SQLDatabase.from_uri(sqlite_uri)
```

### Criando a Cadeia SQL

Crie um template de prompt:
```python
from langchain_core.prompts import ChatPromptTemplate

template = """Based on the table schema below, write a SQL query that would answer the user's question:
{schema}

Question: {question}
SQL Query:"""
prompt = ChatPromptTemplate.from_template(template)
```

Gere o esquema do banco de dados:
```python
def get_schema(_):
        return db.get_table_info()
```

Configure a cadeia SQL:
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

sql_chain = (
        RunnablePassthrough.assign(schema=get_schema)
        | prompt
        | llm.bind(stop=["\nSQLResult:"])
        | StrOutputParser()
)
```

### Criando a Cadeia Completa

Crie um template de prompt para a resposta:
```python
template = """Based on the table schema below, question, sql query, and sql response, write a natural language response:
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}"""
prompt_response = ChatPromptTemplate.from_template(template)
```

Função para executar a consulta SQL:
```python
def run_query(query):
        return db.run(query)
```

Configure a cadeia completa:
```python
full_chain = (
        RunnablePassthrough.assign(query=sql_chain).assign(
                schema=get_schema,
                response=lambda vars: run_query(vars["query"]),
        )
        | prompt_response
        | model
)
```

Teste a cadeia completa:
```python
user_question = 'how many albums are there in the database?'
result = full_chain.invoke({"question": user_question})
print(result)
```

## Conclusão

Neste tutorial, aprendemos como interagir com um banco de dados MySQL (ou SQLite) usando Python e LangChain. Utilizamos o **wrapper** do LangChain para o **SQLAlchemy** para interagir com o banco de dados e também criamos uma cadeia personalizada usando o pacote **LangChain**. Essa cadeia nos permitiu conversar com o banco de dados utilizando linguagem natural.

Esperamos que este tutorial tenha sido útil. Se você tiver alguma dúvida ou comentário, por favor, nos avise. Agradecemos por acompanhar o tutorial!
