# 👋 Prompting strategies
```
SQL 쿼리 생성 성능 향상 방법
Prompt에서 데이터베이스 정보를 가져오는 방법
```

# 👋 샘플 데이터 준비
```ssh
!wget https://www.sqlitetutorial.net/wp-content/uploads/2018/03/chinook.zip
!unzip -l chinook.zip
!unzip chinook.zip
```
# 👋 Setup
```ssh
!pip install --quiet langchain langchain-community langchain-experimental langchain-openai
```
## 👀 코드
```python
import os
os.environ["OPENAI_API_KEY"] = "..."
os.environ["LANGCHAIN_API_KEY"] = "..."
os.environ["LANGCHAIN_TRACING_V2"] = "true"

from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///chinook.db", sample_rows_in_table_info=3)
print(db.dialect)
print(db.get_usable_table_names())
db.run("select * from artists limit 10;")
```
## 👀 결과
<pre>
<span style="font-family: Consolas">
<span style="color: #000000">sqlite</span>
<span style="color: #000000">[</span><span style="color: #ff00ff">'albums'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'artists'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'customers'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'employees'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'genres'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'invoice_items'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'invoices'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'media_types'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'playlist_track'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'playlists'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'tracks'</span><span style="color: #000000">]</span>
<span style="color: #000000">[(1, </span><span style="color: #ff00ff">'AC/DC'</span><span style="color: #000000">), (2, </span><span style="color: #ff00ff">'Accept'</span><span style="color: #000000">), (3, </span><span style="color: #ff00ff">'Aerosmith'</span><span style="color: #000000">), (4, </span><span style="color: #ff00ff">'Alanis Morissette'</span><span style="color: #000000">), (5, </span><span style="color: #ff00ff">'Alice In Chains'</span><span style="color: #000000">), (6, </span><span style="color: #ff00ff">'Ant&ocirc;nio Carlos Jobim'</span><span style="color: #000000">), (7, </span><span style="color: #ff00ff">'Apocalyptica'</span><span style="color: #000000">), (8, </span><span style="color: #ff00ff">'Audioslave'</span><span style="color: #000000">), (9, </span><span style="color: #ff00ff">'BackBeat'</span><span style="color: #000000">), (10, </span><span style="color: #ff00ff">'Billy Cobham'</span><span style="color: #000000">)]</span>
</span>
</pre>

```
위의 코드는 
- LLM이 질의 할 데이터베이스와 연결하고,
- 사용할 테이블의 스키마 정보 및 테이블 목록을 생성
- SQL 질의문 실행
```

# 👋 Dialect-specific prompting
가장 간단하게 프롬프트를 설정하는 방법은 사용할 SQL 데이터베이스 특성에 맞게 프롬프트를 설정하는 것이다.
```python
from langchain.chains.sql_database.prompt import SQL_PROMPTS

list(SQL_PROMPTS)

```
<pre>
<span style="font-family: Consolas">
<span style="color: #000000">[</span><span style="color: #ff00ff">'crate'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'duckdb'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'googlesql'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'mssql'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'mysql'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'mariadb'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'oracle'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'postgresql'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'sqlite'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'clickhouse'</span><span style="color: #000000">,</span>
 <span style="color: #ff00ff">'prestodb'</span><span style="color: #000000">]</span>
</span>
</pre>


```python
from langchain.chains import create_sql_query_chain
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
chain = create_sql_query_chain(llm, db)
chat.get_prompts()[0].pretty_print()
```

<pre>
<span style="font-family: Consolas">
<span style="color: #000000">You are a SQLite expert. Given an </span><span style="color: #808000">input </span><span style="color: #000000">question, first create a syntactically correct SQLite query to run, then look at the results of the query </span><span style="color: #0000ff">and return </span><span style="color: #000000">the answer to the </span><span style="color: #808000">input </span><span style="color: #000000">question.</span>
<span style="color: #000000">Unless the user specifies </span><span style="color: #0000ff">in </span><span style="color: #000000">the question a specific number of examples to obtain, query </span><span style="color: #0000ff">for </span><span style="color: #000000">at most 5 results using the LIMIT clause </span><span style="color: #0000ff">as </span><span style="color: #000000">per SQLite. You can order the results to </span><span style="color: #0000ff">return </span><span style="color: #000000">the most informative data </span><span style="color: #0000ff">in </span><span style="color: #000000">the database.</span>
<span style="color: #000000">Never query </span><span style="color: #0000ff">for </span><span style="color: #808000">all </span><span style="color: #000000">columns </span><span style="color: #0000ff">from </span><span style="color: #000000">a table. You must query only the columns that are needed to answer the question. Wrap each column </span><span style="color: #008080">name </span><span style="color: #0000ff">in </span><span style="color: #000000">double quotes (</span><span style="color: #ff00ff">&quot;) to denote them as delimited identifiers.</span>
<span style="color: #000000">Pay attention to use only the column names you can see </span><span style="color: #0000ff">in </span><span style="color: #000000">the tables below. Be careful to </span><span style="color: #0000ff">not </span><span style="color: #000000">query </span><span style="color: #0000ff">for </span><span style="color: #000000">columns that do </span><span style="color: #0000ff">not </span><span style="color: #000000">exist. Also, pay attention to which column </span><span style="color: #0000ff">is in </span><span style="color: #000000">which table.</span>
<span style="color: #000000">Pay attention to use date(</span><span style="color: #ff00ff">'now'</span><span style="color: #000000">) function to </span><span style="color: #008080">get </span><span style="color: #000000">the current date, </span><span style="color: #0000ff">if </span><span style="color: #000000">the question involves </span><span style="color: #ff00ff">&quot;today&quot;</span><span style="color: #000000">.</span>

<span style="color: #000000">Use the following </span><span style="color: #008080">format</span><span style="color: #000000">:</span>

<span style="color: #000000">Question: Question here</span>
<span style="color: #000000">SQLQuery: SQL Query to run</span>
<span style="color: #000000">SQLResult: Result of the SQLQuery</span>
<span style="color: #000000">Answer: Final answer here</span>

<span style="color: #000000">Only use the following tables:</span>
<span style="color: #000000">{table_info}</span>

<span style="color: #000000">Question: {</span><span style="color: #808000">input</span><span style="color: #000000">}</span>
</span>
</pre>


# 👋 Table definitions and example rows
# 👋 Few-shot examples
# 👋 Dynamic few-shot examples
