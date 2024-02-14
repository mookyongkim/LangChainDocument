# 🐶 Quickstart
이 문서에선는 SQL 데이터베이스에 사용자의 질문에 답변하는 SQL Chain과 SQL Agent를 만드는 방법에 대해서 연구해봅니다.

# ⚾️Security note
SQL 데이터베이스에 대ㅎ해 Q&A 시스템을 구축하려면 모델에서 생성된 SQL 쿼리를 실행해야 합니다.  
이 작업에는 내재된 위험이 있습니다. 데이터베이스 연결 권한은 필요에 따라 가능한 한 좁은 범위로 설정해야 합니다. 이렇게 하면 구축의 위험을 완화할 수는 있지만 제거할 수는 없습니다.  

# ⚾️Architecture
- 사용자 질의를 SQL 쿼리로 변환: Model 사용자 질의를 SQL 쿼리로 변환
- 생성된 SQL 쿼리 실행
- 사용자 질문에 답변하기: SQL 쿼리 결과를 사용해서 사용자 질의에 대한 답변을 생성

![SQL 쿼리 생성 아키텍처](https://python.langchain.com/assets/images/sql_usecase-d432701261f05ab69b38576093718cf3.png)

# ⚾️Setup
```
!pip install -quiet langchain  
!pip install -quiet langchain-community  
!pip install -quiet langchain-openai  

```

```python
import os

os.environ["OPENAI_API_KEY"] = ""
os.environ["LANGCHAIN_API_KEY"] = ""
os.environ["LANGCHAIN_TRACING_V2"] = "true"

from langchain_community.utilities import SQLDatabase

# SQLAlchemy 라이브러리를 지원하는 모든 DB는 연결이 가능하다.
db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
db.run("SELECT * FROM Artist LIMIT 10;")

```

# ⚾️Chain
# 🎾Convert question to SQL query
# 🎾Execute SQL query
# 🎾Answer the question
# ⚾️Agents
# 🎾Initializing agent

