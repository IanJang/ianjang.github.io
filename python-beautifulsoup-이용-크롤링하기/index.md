# Python BeautifulSoup 이용 크롤링하기


## 들어가며
Python을 이용하여 크롤링을 해보았습니다. BeautifulSoup를 이용한 사례들이 많았고 참고하여 진행해 보았습니다. 

---

## BeautifulSoup 설치

```bash
$ pip install beautifulSoup4
```

---

## 크롤링 시작하기
클리앙의 모두의공원 카테고리의 글을 크롤링해보려 합니다.
레퍼런스 사이트내용을 참고해서 아래와 같이 진행해 보았습니다.
우선 해당 페이지의 모든 html text를 긁어와봅시다.

```python
import requests
from bs4 import BeautifulSoup

if __name__ == '__main__':
    url = "http://www.clien.net/cs2/bbs/board.php?bo_table=park"
    source_code = requests.get(url)
    plain_text = source_code.text
    print(plain_text)
```

콘솔창에 엄청나게 길게 출력된 내용을 볼 수 있습니다.

이어서 BeautifulSoup 라이브러리를 이용하여 html 문서를 파싱합니다.
```python

...생략...

if __name__ == '__main__':
    url = "http://www.clien.net/cs2/bbs/board.php?bo_table=park"
    source_code = requests.get(url)
    plain_text = source_code.text
    print(plain_text)
    soup = BeautifulSoup(plain_text, 'lxml')
```

혹시 아래와 같은 에러를 만난다면

```bash
bs4.FeatureNotFound: Couldn't find a tree builder with the features you requested: lxml. Do you need to install a parser library?
```

lxml 라이브러리가 없어 발생한 문제입니다. 설치합시다.

```bash
$ pip install lxml
```

계속 진행해봅시다.
크롬 인스펙터를 이용해서 구조를 파악하고,
아래와 같이 게시물 번호와 제목을 가져오기로 했습니다.

```python

# (...생략...)

if __name__ == '__main__':
    url = "http://www.clien.net/cs2/bbs/board.php?bo_table=park"
    source_code = requests.get(url)
    plain_text = source_code.text
    soup = BeautifulSoup(plain_text, 'lxml')
    for tr in soup.select('tr.mytr'):
        idx = 0
        for td in tr.select('td'):
            if idx==0:
                print("NUM : "+ td.string)
            elif idx==1:
                print("SUBJECT : "+ td.find('a').text)
            idx+=1
    

```

그 결과물은

```bash
NUM : 52750721
SUBJECT : êµ°ë ìë¹µ.. êµ¬ìë¨¹ì ë§ë¬ì£ 
```

한글이 깨져나옵니다. 오랜시간의 사투 결과 찾은 해결책은 `plain_text = source_code.text` 를 
`plain_text = source_code.content` 로 변경하는 것이었습니다.

```
NUM : 52750735
SUBJECT : 미드 에이젼트 오브 쉴드...
```

완료!

---

## Reference
- [파이썬(Python)-취미 프로그래밍, 취미 프로젝트의 시작.](http://hurderella.tistory.com/96)
- [파이썬(Python) - beautifulSoup 으로 html 파싱](http://hurderella.tistory.com/113)
- [[PYTHON 3] Tutorials 25. 웹 크롤러(like Google) 만들기 2 - How to build a web crawler](http://creativeworks.tistory.com/entry/PYTHON-3-Tutorials-25-%EC%9B%B9-%ED%81%AC%EB%A1%A4%EB%9F%AClike-Google-%EB%A7%8C%EB%93%A4%EA%B8%B0-2-How-to-build-a-web-crawler)

---

끝.
