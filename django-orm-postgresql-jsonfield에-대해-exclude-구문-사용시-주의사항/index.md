# Django ORM PostgreSQL JSONField에 대해 exclude 구문 사용시 주의사항


## 들어가며 

  PostgreSQL JSON Field에 대하여 ORM 구문 작성 시 exclude를 사용한다면 주의하길 바랍니다. JSON Field에 대한 질의 결과가 일반적인(?) 예상과는 조금 다른 결과가 나옵니다. 그 때문에 이 같은 내용을 모르고 사용한다면 혼란에 빠질 수 있습니다.  실제로 본인 포함한 여러 개발자가 동일한 케이스로 혼란을 겪은 경험이 있습니다. 본 글의 내용을 접한 분들은 동일한 혼란을 겪지 않을 수 있길 바랍니다.  
  
(본 글은 Django 3.0.7, psycopg2-binary 2.8.5, PostgreSQL 12.4 기준으로 테스트 후 작성되었습니다.)  

---

## JSON Field 필터링 테스트

아래 코드처럼 Book 모델이 구성되어 있고. 총 8개의 샘플 데이터가 있다고 가정합니다.

```python
# models.py
class Book(models.Model):
    # (...생략...)
    data = JSONField(null=True, blank=True)
```

```python
# test_jsonfield_filter.py

class TestJsonFieldFiltering(TestCase):
    def setUp(self):
        Book.objects.create(id=1, data=None)
        Book.objects.create(id=2, data={})
        Book.objects.create(id=3, data={'title': 'django'})
        Book.objects.create(id=4, data={'title': 'python'})
        Book.objects.create(id=5, data={'title': 'django', 'is_published': True})
        Book.objects.create(id=6, data={'title': 'python', 'is_published': True})
        Book.objects.create(id=7, data={'title': 'django', 'is_published': False})
        Book.objects.create(id=8, data={'title': 'python', 'is_published': False})
```

### filter 테스트

먼저 filter를 테스트해 봅시다. 

```python
def test_filter(self):
    queryset = Book.objects.filter(data__is_published=True)
    row_id_set = set(queryset.values_list('id', flat=True))
    self.assertEqual(row_id_set, {5, 6})
    
    queryset = Book.objects.filter(data__is_published=False)
    row_id_set = set(queryset.values_list('id', flat=True))
    self.assertEqual(row_id_set, {7, 8})
```

data__is_published=True로 필터 시 5, 6번 data가 조회되었고, data__is_published=False로 필터 시 7, 8번 데이터가 조회되었습니다. 의도한 대로 결과가 잘 조회됩니다.

### exclude 테스트

다음으로 exclude에 대해 테스트를 해 봅시다.

```python
def test_exclude(self):
    queryset = Book.objects.exclude(data__is_published=True)
    row_id_set = set(queryset.values_list('id', flat=True))
    self.assertEqual(row_id_set, {1, 2, 3, 4, 7, 8})

# AssertionError: Items in the second set but not the first: 2 3 4
```

Assertion Error가 발생했습니다. 의도 대로라면 is_published가 True로 세팅된 5, 6번을 제외한 1, 2, 3, 4, 7, 8번의 Book이 조회되어야 하는데, 결과는 1, 7, 8만 조회되었습니다. 이유가 무엇일까요?

---

## JSON Field 필터링 시 SQL 질의문(Query) 확인

앞서 테스트한 ORM 구문에 대해서 실제로 어떤 Query가 Database에 요청되는지 확인해 봅시다. 먼저 filter query부터 확인해 봅시다.

### filter query

```python
queryset = Book.objects.filter(data__is_published=True)
print(queryset.query)
```

```sql
SELECT (..생략..)
FROM   "myapp_book" 
WHERE  "myapp_book"."data" -> is_published = 'true'
```

특이 사항은 없어 보입니다. 다음은 exclude query를 확인해 봅시다.

### exclude query

```python
queryset = DummyModel.objects.exclude(data__is_published=True)
print(queryset.query)
```

```sql
SELECT (..생략..)
FROM   "myapp_book" 
WHERE  NOT "myapp_book"."data" -> is_published = 'true' 
AND    "myapp_book"."data" IS NOT NULL 
```

filter query와 달리 의도하지 않은 구문이 포함된 것이 보입니다.  "myapp_book"."data" IS NOT NULL 이것 때문인지 결과가 달라진 것은 아닐지 확인해 봤습니다. (결론부터 말하자면 해당 조건과는 무관했습니다) 일단 조건절 모양이 "NOT (A AND B)" 형태여서 눈에 잘 안 들어오니, 드모르간 법칙을 이용해 조건절을 보기 쉽게 "NOT A OR NOT B" 형태로 바꿔봤습니다.

```sql
SELECT (..생략..)
FROM   "myapp_book" 
WHERE  "myapp_book"."data" -> is_published != 'true' 
OR     "myapp_book"."data" IS NULL
```

풀어놓고 다시 보니 Query 구문 자체는 문제가 없어 보입니다. "myapp_book"."data" -> is_published != 'true'  조건만으로도 1, 2, 3, 4, 7, 8이 선택돼야 할 것처럼 보입니다.  아래 다이어그램처럼 말이죠….

![다이어그램](https://ianjang.github.io/img/jsonfield-filter-result-diagram.png)

Query 구문 확인으로도 여전히 의문은 풀리지 않았습니다. 그래서 이번엔 DB에 직접 Query를 날려 보며 결과를 확인해 봤습니다.

---

### DB Query 수행 결과 확인

위에서 마지막으로 도출했던 Query 구문의 조건절을 분리하여 각각 Query를 수행해 보았습니다.

```sql
# QUERY #1
postgres_db=# SELECT "id", "data" FROM "myapp_book" where "myapp_book"."data" IS NULL;
 id | data 
----+------
  1 | 
(1 row)

# QUERY #2
postgres_db=# SELECT "id", "data" FROM "myapp_book" where "myapp_book"."data" -> 'is_published' != 'true';
 id |                    data                    
----+--------------------------------------------
  7 | {"title": "django", "is_published": false}
  8 | {"title": "python", "is_published": false}
(2 rows)
```

원인을 찾았습니다. QUERY #2의 결과가 예상했던 1, 2, 3, 4, 7, 8이 아닌 달리 7, 8만 조회되네요. JSONField의 특정 key값에 대해서 조회 조건을 사용하면 해당 key값이 없는 row는 제외하고 결과가 조회됨을 알 수 있습니다.

몇 가지 더 테스트 해봤습니다.

```sql
postgres_db=# SELECT "id", "data" FROM "myapp_book" WHERE "myapp_book"."data" -> 'is_published' is null;
 id |        data         
----+---------------------
  1 | 
  2 | {}
  3 | {"title": "django"}
  4 | {"title": "python"}

postgres_db=# SELECT "id", "data" FROM "myapp_book" WHERE "myapp_book"."data" -> 'is_published' != 'true' or "myapp_book"."data" -> 'is_published' is null;
id |                    data                    
----+--------------------------------------------
  1 | 
  2 | {}
  3 | {"title": "django"}
  4 | {"title": "python"}
  7 | {"title": "django", "is_published": false}
  8 | {"title": "python", "is_published": false}
(6 rows)
```

- JSON Field의 특정 key 값에 대해서 "is null" 조건으로 조회하면 None, {}, key가 없는 것 세 가지가 다 조회가 됩니다.
- JSON Field의 특정 key 값이 존재하는 row도 있고, 존재하지 않는 row도 있다면, 해당 key 값에 대해 "is null" 조건을 함께 사용해야 의도한 결과를 도출 할 수 있음을 알 수 있습니다.

---

## 기타 이슈 및 해결

### ERROR: column "is_published" does not exist

- 현상: queryset.query 결과를 그대로 psql에서 수행시켰을 때 해당 에러가 나왔습니다.
- 해결: "myapp_book"."data" -> is_published에서 is_published를 single quote로 감싸서 query를 수행했더니 결과가 정상적으로 나왔습니다.

```sql
SELECT "id", "data" FROM "myapp_book" WHERE "myapp_book"."data" -> is_published != 'true'; # (X) ERROR:  column "is_published" does not exist
SELECT "id", "data" FROM "myapp_book" WHERE "myapp_book"."data" -> "is_published" != 'true'; # (X) ERROR:  column "is_published" does not exist
SELECT "id", "data" FROM "myapp_book" WHERE "myapp_book"."data" -> 'is_published' != 'true'; # (O) 정상 동작
```

---

## 맺으며
- JSON Field에 대해 ORM 구문 작성 시 조회대상 Key 값 존재여부를 체크하는 습관을 들입시다. 
- exclude보다는 filter를 사용합시다. exclude를 이용해서도 원하는 결과를 도출할 수 있지만, "NOT(A AND B)" 형태보다는 "A OR B" 형태가 훨씬 가독성이 높기 때문에 filter를 이용하는 편이 좋다고 생각합니다.

```python
# is_published=True 조건 조회 시, 아래 두 ORM 구문 모두 예상한 결과가 나옵니다.
Book.objects.filter(data__is_published=True) # O
Book.objects.filter(Q(data__is_published=True) & Q(data__is_published__isnull=False)) # O

# is_published=False 조건 조회 시, 아래 두 ORM 구문 모두 예상한 결과가 나옵니다. 
Book.objects.filter(data__is_published=False) # O
Book.objects.filter(Q(data__is_published=False) & Q(data__is_published__isnull=False)) # O

# is_published=True 만 제외하고 조회하려면 주의가 필요합니다. (예상 결과: 1, 2, 3, 4, 7, 8)
Book.objects.exclude(data__is_published=True) # 예상과 다르 결과가 나옵니다.
Book.objects.exclude(Q(data__is_published=True) & Q(data__is_published__isnull=False) # 예상한 결과가 나옵니다.
Book.objects.filter(Q(data__is_published=False) | Q(data__is_published__isnull=True) # 예상한 결과가 나오며 가장 추천하는 방법입니다.
```

