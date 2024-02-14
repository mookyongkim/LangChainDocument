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

```python
context = db.get_context()
print(list(context))
print(context["table_info"])

```

<pre>
<span style="font-family: Consolas">
<span style="color: #000000">[</span><span style="color: #ff00ff">'table_info'</span><span style="color: #000000">, </span><span style="color: #ff00ff">'table_names'</span><span style="color: #000000">]</span>

<span style="color: #000000">CREATE TABLE albums (</span>
	<span style="color: #ff00ff">&quot;AlbumId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Title&quot; </span><span style="color: #000000">NVARCHAR(160) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;ArtistId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;AlbumId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;ArtistId&quot;</span><span style="color: #000000">) REFERENCES artists (</span><span style="color: #ff00ff">&quot;ArtistId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">albums table:</span>
<span style="color: #000000">AlbumId	Title	ArtistId</span>
<span style="color: #000000">1	For Those About To Rock We Salute You	1</span>
<span style="color: #000000">2	Balls to the Wall	2</span>
<span style="color: #000000">3	Restless </span><span style="color: #0000ff">and </span><span style="color: #000000">Wild	2</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE artists (</span>
	<span style="color: #ff00ff">&quot;ArtistId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Name&quot; </span><span style="color: #000000">NVARCHAR(120), </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;ArtistId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">artists table:</span>
<span style="color: #000000">ArtistId	Name</span>
<span style="color: #000000">1	AC</span><span style="color: #008080">/</span><span style="color: #000000">DC</span>
<span style="color: #000000">2	Accept</span>
<span style="color: #000000">3	Aerosmith</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE customers (</span>
	<span style="color: #ff00ff">&quot;CustomerId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;FirstName&quot; </span><span style="color: #000000">NVARCHAR(40) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;LastName&quot; </span><span style="color: #000000">NVARCHAR(20) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Company&quot; </span><span style="color: #000000">NVARCHAR(80), </span>
	<span style="color: #ff00ff">&quot;Address&quot; </span><span style="color: #000000">NVARCHAR(70), </span>
	<span style="color: #ff00ff">&quot;City&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;State&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;Country&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;PostalCode&quot; </span><span style="color: #000000">NVARCHAR(10), </span>
	<span style="color: #ff00ff">&quot;Phone&quot; </span><span style="color: #000000">NVARCHAR(24), </span>
	<span style="color: #ff00ff">&quot;Fax&quot; </span><span style="color: #000000">NVARCHAR(24), </span>
	<span style="color: #ff00ff">&quot;Email&quot; </span><span style="color: #000000">NVARCHAR(60) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;SupportRepId&quot; </span><span style="color: #000000">INTEGER, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;CustomerId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;SupportRepId&quot;</span><span style="color: #000000">) REFERENCES employees (</span><span style="color: #ff00ff">&quot;EmployeeId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">customers table:</span>
<span style="color: #000000">CustomerId	FirstName	LastName	Company	Address	City	State	Country	PostalCode	Phone	Fax	Email	SupportRepId</span>
<span style="color: #000000">1	Lu&iacute;s	Gon&ccedil;alves	Embraer </span><span style="color: #008080">- </span><span style="color: #000000">Empresa Brasileira de Aeron&aacute;utica S.A.	Av. Brigadeiro Faria Lima, 2170	S&atilde;o Jos&eacute; dos Campos	SP	Brazil	12227</span><span style="color: #008080">-</span><span style="color: #000000">000	</span><span style="color: #008080">+</span><span style="color: #000000">55 (12) 3923</span><span style="color: #008080">-</span><span style="color: #000000">5555	</span><span style="color: #008080">+</span><span style="color: #000000">55 (12) 3923</span><span style="color: #008080">-</span><span style="color: #000000">5566	luisg@embraer.com.br	3</span>
<span style="color: #000000">2	Leonie	K&ouml;hler	</span><span style="color: #0000ff">None	</span><span style="color: #000000">Theodor</span><span style="color: #008080">-</span><span style="color: #000000">Heuss</span><span style="color: #008080">-</span><span style="color: #000000">Stra&szlig;e 34	Stuttgart	</span><span style="color: #0000ff">None	</span><span style="color: #000000">Germany	70174	</span><span style="color: #008080">+</span><span style="color: #000000">49 0711 2842222	</span><span style="color: #0000ff">None	</span><span style="color: #000000">leonekohler@surfeu.de	5</span>
<span style="color: #000000">3	Fran&ccedil;ois	Tremblay	</span><span style="color: #0000ff">None	</span><span style="color: #000000">1498 rue B&eacute;langer	Montr&eacute;al	QC	Canada	H2G 1A7	</span><span style="color: #008080">+</span><span style="color: #000000">1 (514) 721</span><span style="color: #008080">-</span><span style="color: #000000">4711	</span><span style="color: #0000ff">None	</span><span style="color: #000000">ftremblay@gmail.com	3</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE employees (</span>
	<span style="color: #ff00ff">&quot;EmployeeId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;LastName&quot; </span><span style="color: #000000">NVARCHAR(20) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;FirstName&quot; </span><span style="color: #000000">NVARCHAR(20) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Title&quot; </span><span style="color: #000000">NVARCHAR(30), </span>
	<span style="color: #ff00ff">&quot;ReportsTo&quot; </span><span style="color: #000000">INTEGER, </span>
	<span style="color: #ff00ff">&quot;BirthDate&quot; </span><span style="color: #000000">DATETIME, </span>
	<span style="color: #ff00ff">&quot;HireDate&quot; </span><span style="color: #000000">DATETIME, </span>
	<span style="color: #ff00ff">&quot;Address&quot; </span><span style="color: #000000">NVARCHAR(70), </span>
	<span style="color: #ff00ff">&quot;City&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;State&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;Country&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;PostalCode&quot; </span><span style="color: #000000">NVARCHAR(10), </span>
	<span style="color: #ff00ff">&quot;Phone&quot; </span><span style="color: #000000">NVARCHAR(24), </span>
	<span style="color: #ff00ff">&quot;Fax&quot; </span><span style="color: #000000">NVARCHAR(24), </span>
	<span style="color: #ff00ff">&quot;Email&quot; </span><span style="color: #000000">NVARCHAR(60), </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;EmployeeId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;ReportsTo&quot;</span><span style="color: #000000">) REFERENCES employees (</span><span style="color: #ff00ff">&quot;EmployeeId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">employees table:</span>
<span style="color: #000000">EmployeeId	LastName	FirstName	Title	ReportsTo	BirthDate	HireDate	Address	City	State	Country	PostalCode	Phone	Fax	Email</span>
<span style="color: #000000">1	Adams	Andrew	General Manager	</span><span style="color: #0000ff">None	</span><span style="color: #000000">1962</span><span style="color: #008080">-</span><span style="color: #000000">02</span><span style="color: #008080">-</span><span style="color: #000000">18 00:00:00	2002</span><span style="color: #008080">-</span><span style="color: #000000">08</span><span style="color: #008080">-</span><span style="color: #000000">14 00:00:00	11120 Jasper Ave NW	Edmonton	AB	Canada	T5K 2N1	</span><span style="color: #008080">+</span><span style="color: #000000">1 (780) 428</span><span style="color: #008080">-</span><span style="color: #000000">9482	</span><span style="color: #008080">+</span><span style="color: #000000">1 (780) 428</span><span style="color: #008080">-</span><span style="color: #000000">3457	andrew@chinookcorp.com</span>
<span style="color: #000000">2	Edwards	Nancy	Sales Manager	1	1958</span><span style="color: #008080">-</span><span style="color: #000000">12</span><span style="color: #008080">-</span><span style="color: #000000">08 00:00:00	2002</span><span style="color: #008080">-</span><span style="color: #000000">05</span><span style="color: #008080">-</span><span style="color: #000000">01 00:00:00	825 8 Ave SW	Calgary	AB	Canada	T2P 2T3	</span><span style="color: #008080">+</span><span style="color: #000000">1 (403) 262</span><span style="color: #008080">-</span><span style="color: #000000">3443	</span><span style="color: #008080">+</span><span style="color: #000000">1 (403) 262</span><span style="color: #008080">-</span><span style="color: #000000">3322	nancy@chinookcorp.com</span>
<span style="color: #000000">3	Peacock	Jane	Sales Support Agent	2	1973</span><span style="color: #008080">-</span><span style="color: #000000">08</span><span style="color: #008080">-</span><span style="color: #000000">29 00:00:00	2002</span><span style="color: #008080">-</span><span style="color: #000000">04</span><span style="color: #008080">-</span><span style="color: #000000">01 00:00:00	1111 6 Ave SW	Calgary	AB	Canada	T2P 5M5	</span><span style="color: #008080">+</span><span style="color: #000000">1 (403) 262</span><span style="color: #008080">-</span><span style="color: #000000">3443	</span><span style="color: #008080">+</span><span style="color: #000000">1 (403) 262</span><span style="color: #008080">-</span><span style="color: #000000">6712	jane@chinookcorp.com</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE genres (</span>
	<span style="color: #ff00ff">&quot;GenreId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Name&quot; </span><span style="color: #000000">NVARCHAR(120), </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;GenreId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">genres table:</span>
<span style="color: #000000">GenreId	Name</span>
<span style="color: #000000">1	Rock</span>
<span style="color: #000000">2	Jazz</span>
<span style="color: #000000">3	Metal</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE invoice_items (</span>
	<span style="color: #ff00ff">&quot;InvoiceLineId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;InvoiceId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;TrackId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;UnitPrice&quot; </span><span style="color: #000000">NUMERIC(10, 2) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Quantity&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;InvoiceLineId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">) REFERENCES tracks (</span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;InvoiceId&quot;</span><span style="color: #000000">) REFERENCES invoices (</span><span style="color: #ff00ff">&quot;InvoiceId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">invoice_items table:</span>
<span style="color: #000000">InvoiceLineId	InvoiceId	TrackId	UnitPrice	Quantity</span>
<span style="color: #000000">1	1	2	0.99	1</span>
<span style="color: #000000">2	1	4	0.99	1</span>
<span style="color: #000000">3	2	6	0.99	1</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE invoices (</span>
	<span style="color: #ff00ff">&quot;InvoiceId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;CustomerId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;InvoiceDate&quot; </span><span style="color: #000000">DATETIME NOT NULL, </span>
	<span style="color: #ff00ff">&quot;BillingAddress&quot; </span><span style="color: #000000">NVARCHAR(70), </span>
	<span style="color: #ff00ff">&quot;BillingCity&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;BillingState&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;BillingCountry&quot; </span><span style="color: #000000">NVARCHAR(40), </span>
	<span style="color: #ff00ff">&quot;BillingPostalCode&quot; </span><span style="color: #000000">NVARCHAR(10), </span>
	<span style="color: #ff00ff">&quot;Total&quot; </span><span style="color: #000000">NUMERIC(10, 2) NOT NULL, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;InvoiceId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;CustomerId&quot;</span><span style="color: #000000">) REFERENCES customers (</span><span style="color: #ff00ff">&quot;CustomerId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">invoices table:</span>
<span style="color: #000000">InvoiceId	CustomerId	InvoiceDate	BillingAddress	BillingCity	BillingState	BillingCountry	BillingPostalCode	Total</span>
<span style="color: #000000">1	2	2009</span><span style="color: #008080">-</span><span style="color: #000000">01</span><span style="color: #008080">-</span><span style="color: #000000">01 00:00:00	Theodor</span><span style="color: #008080">-</span><span style="color: #000000">Heuss</span><span style="color: #008080">-</span><span style="color: #000000">Stra&szlig;e 34	Stuttgart	</span><span style="color: #0000ff">None	</span><span style="color: #000000">Germany	70174	1.98</span>
<span style="color: #000000">2	4	2009</span><span style="color: #008080">-</span><span style="color: #000000">01</span><span style="color: #008080">-</span><span style="color: #000000">02 00:00:00	Ullev&aring;lsveien 14	Oslo	</span><span style="color: #0000ff">None	</span><span style="color: #000000">Norway	0171	3.96</span>
<span style="color: #000000">3	8	2009</span><span style="color: #008080">-</span><span style="color: #000000">01</span><span style="color: #008080">-</span><span style="color: #000000">03 00:00:00	Gr&eacute;trystraat 63	Brussels	</span><span style="color: #0000ff">None	</span><span style="color: #000000">Belgium	1000	5.94</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE media_types (</span>
	<span style="color: #ff00ff">&quot;MediaTypeId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Name&quot; </span><span style="color: #000000">NVARCHAR(120), </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;MediaTypeId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">media_types table:</span>
<span style="color: #000000">MediaTypeId	Name</span>
<span style="color: #000000">1	MPEG audio </span><span style="color: #008080">file</span>
<span style="color: #000000">2	Protected AAC audio </span><span style="color: #008080">file</span>
<span style="color: #000000">3	Protected MPEG</span><span style="color: #008080">-</span><span style="color: #000000">4 video </span><span style="color: #008080">file</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE playlist_track (</span>
	<span style="color: #ff00ff">&quot;PlaylistId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;TrackId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;PlaylistId&quot;</span><span style="color: #000000">, </span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">) REFERENCES tracks (</span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;PlaylistId&quot;</span><span style="color: #000000">) REFERENCES playlists (</span><span style="color: #ff00ff">&quot;PlaylistId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">playlist_track table:</span>
<span style="color: #000000">PlaylistId	TrackId</span>
<span style="color: #000000">1	3402</span>
<span style="color: #000000">1	3389</span>
<span style="color: #000000">1	3390</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE playlists (</span>
	<span style="color: #ff00ff">&quot;PlaylistId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Name&quot; </span><span style="color: #000000">NVARCHAR(120), </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;PlaylistId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">playlists table:</span>
<span style="color: #000000">PlaylistId	Name</span>
<span style="color: #000000">1	Music</span>
<span style="color: #000000">2	Movies</span>
<span style="color: #000000">3	TV Shows</span>
<span style="color: #008080">*/</span>


<span style="color: #000000">CREATE TABLE tracks (</span>
	<span style="color: #ff00ff">&quot;TrackId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Name&quot; </span><span style="color: #000000">NVARCHAR(200) NOT NULL, </span>
	<span style="color: #ff00ff">&quot;AlbumId&quot; </span><span style="color: #000000">INTEGER, </span>
	<span style="color: #ff00ff">&quot;MediaTypeId&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;GenreId&quot; </span><span style="color: #000000">INTEGER, </span>
	<span style="color: #ff00ff">&quot;Composer&quot; </span><span style="color: #000000">NVARCHAR(220), </span>
	<span style="color: #ff00ff">&quot;Milliseconds&quot; </span><span style="color: #000000">INTEGER NOT NULL, </span>
	<span style="color: #ff00ff">&quot;Bytes&quot; </span><span style="color: #000000">INTEGER, </span>
	<span style="color: #ff00ff">&quot;UnitPrice&quot; </span><span style="color: #000000">NUMERIC(10, 2) NOT NULL, </span>
	<span style="color: #000000">PRIMARY KEY (</span><span style="color: #ff00ff">&quot;TrackId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;MediaTypeId&quot;</span><span style="color: #000000">) REFERENCES media_types (</span><span style="color: #ff00ff">&quot;MediaTypeId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;GenreId&quot;</span><span style="color: #000000">) REFERENCES genres (</span><span style="color: #ff00ff">&quot;GenreId&quot;</span><span style="color: #000000">), </span>
	<span style="color: #000000">FOREIGN KEY(</span><span style="color: #ff00ff">&quot;AlbumId&quot;</span><span style="color: #000000">) REFERENCES albums (</span><span style="color: #ff00ff">&quot;AlbumId&quot;</span><span style="color: #000000">)</span>
<span style="color: #000000">)</span>

<span style="color: #008080">/*</span>
<span style="color: #000000">3 rows </span><span style="color: #0000ff">from </span><span style="color: #000000">tracks table:</span>
<span style="color: #000000">TrackId	Name	AlbumId	MediaTypeId	GenreId	Composer	Milliseconds	Bytes	UnitPrice</span>
<span style="color: #000000">1	For Those About To Rock (We Salute You)	1	1	1	Angus Young, Malcolm Young, Brian Johnson	343719	11170334	0.99</span>
<span style="color: #000000">2	Balls to the Wall	2	2	1	</span><span style="color: #0000ff">None	</span><span style="color: #000000">342562	5510424	0.99</span>
<span style="color: #000000">3	Fast As a Shark	3	2	1	F. Baltes, S. Kaufman, U. Dirkscneider </span><span style="color: #008080">&amp; </span><span style="color: #000000">W. Hoffman	230619	3990994	0.99</span>
<span style="color: #008080">*/</span>
</span>
</pre>

```python
prompt_with_context = prompt_with_context = chain.get_prompts()[0].partial(table_info=context["table_info"])
print(prompt_with_context.pretty_repr()[:1500])

```
테이블 정보가 컨텍스트 윈도우 사이즈를 초과하는 경우, 사용자 질의와 관련된 테이블 정보만 프롬프트에 삽입하는 방안을 고민해야한다.



# 👋 Few-shot examples
프롬프트에 질문을 유효한 SQL 쿼리로 변환되는 예제를 포함하면 특히 복잡한 쿼리의 경우 모델 성능이 향상되는 경우가 많습니다.

```python
examples = [
    {"input": "List all artists.", "query": "SELECT * FROM Artist;"},
    {
        "input": "Find all albums for the artist 'AC/DC'.",
        "query": "SELECT * FROM Album WHERE ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'AC/DC');",
    },
    {
        "input": "List all tracks in the 'Rock' genre.",
        "query": "SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');",
    },
    {
        "input": "Find the total duration of all tracks.",
        "query": "SELECT SUM(Milliseconds) FROM Track;",
    },
    {
        "input": "List all customers from Canada.",
        "query": "SELECT * FROM Customer WHERE Country = 'Canada';",
    },
    {
        "input": "How many tracks are there in the album with ID 5?",
        "query": "SELECT COUNT(*) FROM Track WHERE AlbumId = 5;",
    },
    {
        "input": "Find the total number of invoices.",
        "query": "SELECT COUNT(*) FROM Invoice;",
    },
    {
        "input": "List all tracks that are longer than 5 minutes.",
        "query": "SELECT * FROM Track WHERE Milliseconds > 300000;",
    },
    {
        "input": "Who are the top 5 customers by total purchase?",
        "query": "SELECT CustomerId, SUM(Total) AS TotalPurchase FROM Invoice GROUP BY CustomerId ORDER BY TotalPurchase DESC LIMIT 5;",
    },
    {
        "input": "Which albums are from the year 2000?",
        "query": "SELECT * FROM Album WHERE strftime('%Y', ReleaseDate) = '2000';",
    },
    {
        "input": "How many employees are there",
        "query": 'SELECT COUNT(*) FROM "Employee"',
    },
]

from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

example_prompt = PromptTemplate.from_template("User Input: {input}\nSQL query: {query}" )

prompt = FewShotPromptTemplate(
    examples=examples[:5],
    example_prompt=example_prompt,
    prefix="You are a SQLite expert. Given an input question, create a syntactically correct SQLite query to run. Unless otherwise specificed, do not return more than {top_k} rows.\n\nHere is the relevant table info: {table_info}\n\nBelow are a number of examples of questions and their corresponding SQL queries.",
    suffix="User input: {input}\nSQL query: ",
    input_variables=["input","top_k","table_info"]
)

print(prompt.format(input="How many artists are there?", top_k=3, table_info="foo"))

```

## prompt 내용
```
You are a SQLite expert. Given an input question, create a syntactically correct SQLite query to run. Unless otherwise specificed, do not return more than 3 rows.

Here is the relevant table info: foo

Below are a number of examples of questions and their corresponding SQL queries.

User Input: List all artists.
SQL query: SELECT * FROM Artist;

User Input: Find all albums for the artist 'AC/DC'.
SQL query: SELECT * FROM Album WHERE ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'AC/DC');

User Input: List all tracks in the 'Rock' genre.
SQL query: SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');

User Input: Find the total duration of all tracks.
SQL query: SELECT SUM(Milliseconds) FROM Track;

User Input: List all customers from Canada.
SQL query: SELECT * FROM Customer WHERE Country = 'Canada';

User input: How many artists are there?
SQL query: 

```

```
FewShotPromptTemplate(
    input_variables=['input', 'table_info', 'top_k'],
    examples=[
        {'input': 'List all artists.', 'query': 'SELECT * FROM Artist;'},
        {'input': "Find all albums for the artist 'AC/DC'.", 'query': "SELECT * FROM Album WHERE ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'AC/DC');"},
        {'input': "List all tracks in the 'Rock' genre.", 'query': "SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');"},
        {'input': 'Find the total duration of all tracks.', 'query': 'SELECT SUM(Milliseconds) FROM Track;'},
        {'input': 'List all customers from Canada.', 'query': "SELECT * FROM Customer WHERE Country = 'Canada';"}
    ],
    example_prompt=PromptTemplate(
        input_variables=['input', 'query'],
        template='User Input: {input}\nSQL query: {query}'
    ),
    suffix='User input: {input}\nSQL query: ',
    prefix='You are a SQLite expert. Given an input question, create a syntactically correct SQLite query to run. Unless otherwise specificed, do not return more than {top_k} rows.\n\nHere is the relevant table info: {table_info}\n\nBelow are a number of examples of questions and their corresponding SQL queries.'
)

```


# 👋 Dynamic few-shot examples
