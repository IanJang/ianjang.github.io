# Python Flask with SQLAlchemy

## 들어가며
pymysql을 이용해서 flask에서 Database를 다뤄보았습니다. 뭔가 부족한 감이 있었습니다. 자원을 체계적으로 관리하기 힘들어 보였습니다. 그래서 대안을 찾아보았고 SQLAlchemy과 Flask-SQLAlchemy에 대한 사례들이 있었습니다. 본문에 관련 내용을 정리해 보았습니다. 

---

## SQLAlchemy vs. Flask-SQLAlchemy
우선 모듈이 지속적으로 관리가 되고 있는지가 궁금했습니다. Github에서 각각의 저장소를 살펴봤습니다.
먼저 Flask-SQLAlchemy는 2.1 버전이 최신 stable 버전이며, 2015년 10월 23일에 Release 되었습니다.
반면 SQLAlchemy는 22일 전인 2017년 1월 18일에 1.1.5 버전이 release 되었으며, Release history를 보면 꾸준히 버전이 Update 되고 있음을 확인할 수 있었습니다. 그래서 SQLAlchemy를 쓰기로 결정했습니다.
- Reference
  - [Flask-SQLAlchemy Github](https://github.com/mitsuhiko/flask-sqlalchemy)
  - [SQLAlchemy Github](https://github.com/zzzeek/sqlalchemy)

---

## SQLAlchemy 시작하기
먼저 SQLAlchemy를 설치합시다

```bash
$ pip install sqlalchemy
```

최신 버전인 1.1.5 버전이 설치되었습니다.

```bash
Collecting sqlalchemy
  Downloading SQLAlchemy-1.1.5.tar.gz (5.1MB)
    100% |████████████████████████████████| 5.1MB 155kB/s
Installing collected packages: sqlalchemy
  Running setup.py install for sqlalchemy ... done
Successfully installed sqlalchemy-1.1.5
```

본격적으로 이용 방법을 알아봅시다. 아래는 API 문서링크입니다.
[SQLAlchemy official API Doc](http://docs.sqlalchemy.org/en/latest)

먼저 database.py를 작성합시다.

```python
# database.py

from sqlalchemy import create_engine
from sqlalchemy.orm import scoped_session, sessionmaker
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine(
    "mysql://test:test@localhost/test_database",
    isolation_level="READ_UNCOMMITTED"
)

db_session = scoped_session(
    sessionmaker(
        autocommit=False,
        autoflush=False,
        bind=engine
    )
)

Base = declarative_base()
Base.query = db_session.query_property()

def init_db():
    import yourapplication.models
    Base.metadata.create_all(bind=engine)
```

create_engine()의 두 파라미터를 통해서 DB Connection 정보와 Transation Isolation level을 설정합니다.

---

### Transaction Isolation Level
- READ COMMITTED
- READ UNCOMMITTED
- REPEATABLE READ
- SERIALIZABLE
- AUTOCOMMIT

위 다섯가지 종류를 지원합니다. Transaction Isolation Level에 대해서는 [데이터베이스 Isolation Level](http://hundredin.net/2012/07/26/isolation-level/)을 참고합시다.

---

다음으로 scoped_session을 이용하여 session을 생성합니다. 이를 이용하면 내부적으로 Thread 처리를 해주기 때문에 편리합니다.

declarative_base()을 호출하여 생성한 Base는 Table과 mapper를 만들기 위한 역할 합니다.

---

이어서 모델을 만듭시다. 특정 사이트를 크롤링해서 링크를 정도를 걸기 위한 목적으로 아래와 같은 모델을 구성해보았습니다.

```python
# models.py

from sqlalchemy import Column, Integer, String
from database import Base

class Summary(Base):
    __tablename__ = 'suit_summary'
    summary_id = Column(Integer, primary_key=True)
    site_cd = Column(String(20))
    category = Column(String(20))
    content_no = Column(Integer,)
    subject = Column(String(100))
    writer = Column(String(20))
    write_date = Column(String(20))
    hit_cnt = Column(Integer,)
    content_link = Column(String(100))
    create_date = Column(String(20))

    def __init__(self, site_cd=None, category=None, content_no=None, subject=None, writer=None, write_date=None, hit_cnt=None, content_link=None, create_date=None):
        self.site_cd = site_cd 
        self.category = category 
        self.content_no = content_no 
        self.subject = subject 
        self.writer = writer 
        self.write_date = write_date 
        self.hit_cnt = hit_cnt 
        self.content_link = content_link 
        self.create_date = create_date 

    def __repr__(self):
        return '<Summary %r>' % (self.subject)
```

DB Table과 적절히 매핑되기 위해 필드를 구성하고, __init__와 __repr__함수를 작성합시다.
__init__(self, ...)는 생성자, __repr__(self)는 print를 했을 시 출력형태를 정의하는 함수 입니다. 

---


## 이슈 및 해결

### ModuleNotFoundError: No module named 'MySQLdb'

앱을 실행시켰더니 아래와같은 에러가 나왔습니다.

```python
Traceback (most recent call last):
  File "D:\workspace\SuitFlaskApp\suit\database.py", line 12, in <module>
    isolation_level="READ_UNCOMMITTED"
  File "C:\Python36\lib\site-packages\sqlalchemy\engine\__init__.py", line 387, in create_engine
    return strategy.create(*args, **kwargs)
  File "C:\Python36\lib\site-packages\sqlalchemy\engine\strategies.py", line 80, in create
    dbapi = dialect_cls.dbapi(**dbapi_args)
  File "C:\Python36\lib\site-packages\sqlalchemy\dialects\mysql\mysqldb.py", line 110, in dbapi
    return __import__('MySQLdb')
ModuleNotFoundError: No module named 'MySQLdb'
```

에러가 나는 부분은 

```python
engine = create_engine(
    "mysql://test:test@localhost/test_database",
    isolation_level="READ_UNCOMMITTED"
)
```

MySQLdb 모듈이 없다고 합니다. pip를 이용하여 설치합시다.

```bash
$ pip install MySQLdb
```

그 결과 

```bash
Could not find a version that satisfies the requirement MySQLdb (from versions: )
No matching distribution found for MySQLdb
```

심플하게 해결될 문제일 줄 알았으나 해당 모듈이없다고 합니다. 구글링해보니 http://stackoverflow.com/questions/454854/no-module-named-mysqldb 에서
python 3.x 에는 MySQLdb가 없다고 합니다. 저는 python3.6 기반의 환경을 세팅한 상태였습니다. mysqlclient를 이용하면 된다는 글이 있어 시도해 보았습니다.

```bash
$ pip install mysqlclient
```

그 후에 실행해보니 여전히 안됩니다. 다시 구글링.

http://stackoverflow.com/questions/41824551/mysqldb-with-python-3-6
>try to use sqlalchemy that is fully suported for many ODBC driver. Insted of using mysqldb, mysqlconnector is better in python3.x. I migrated a project from sqlite to mysql server and work fine in python3.6 sqlalchemy offer a solution for connecting with another ODBC.

SQLalchemy는 많은 ODBC 드라이버를 지원하지만 MySQL만 지원하지 않는다고 합니다. 조금 더 찾아보았습니다.

http://stackoverflow.com/questions/27766794/switching-from-sqlite-to-mysql-with-flask-sqlalchemy
>It happened to me that the default driver used by SQLAlchemy (mqsqldb), doesn't get compiled for me in my virtual environments. So I have opted for a MySQL driver with full python implementation pymysql. Once you install it using pip install pymysql, the SQLALCHEMY_DATABASE_URI will change to:
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://username:password@localhost/db_name'

pymysql을 이용하면 된다고합니다. pymysql을 설치하고 이를 이용하도록 코드를 수정했습니다.

```bash
$ pip install pymysql
```

```python
engine = create_engine(
    "mysql://test:test@localhost/test_database",
    isolation_level="READ_UNCOMMITTED"
)
```

실행 결과 잘 동작합니다.

---

### Sqlalchemy with pymysql 한글깨짐

한글이 깨져 나올때는 utf8 인코딩 설정을 해줘야 합니다.

create_engine()을 호출할때 파라미터를 수정해주면 간단히 해결됩니다.

아래처럼 dbhost 파라미터에 *charset=utf8*을 추가합시다.

```python
engine = create_engine(
    "mysql://test:test@localhost/test_database&charset=utf8",
    isolation_level="READ_UNCOMMITTED"
)
```

---

## Reference 
- [SQLAlchemy Tutorial](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html)
- [[Python] SQLAlchemy 사용하기](http://yujuwon.tistory.com/entry/SQLAlchemy-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
- [SQLAlchemy 시작하기 – Part 1](http://www.haruair.com/blog/1682)
- [SQLAlchemy 시작하기 - Part 2](http://www.haruair.com/blog/1695)
- [SQLAlchemy in Flask](http://flask.pocoo.org/docs/0.12/patterns/sqlalchemy/)
- [Flask에서 SQLAlchemy 사용하기](http://flask-docs-kr.readthedocs.io/ko/latest/ko/patterns/sqlalchemy.html)\

