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
# 👋 Table definitions and example rows
# 👋 Few-shot examples
# 👋 Dynamic few-shot examples
