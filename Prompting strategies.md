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
