# Jekyll3로 블로그 만들기


## 들어가며

  Jekyll을 이용하여 블로그를 구성해봅시다.

## 사전준비
### Ruby 설치
- Jekyll 2 사용 시 v1.9.3 이상 / Jekyll 3 사용 시 v2 이상
- 저는 2.3.3 버전을 설치했습니다. "설치경로\bin" 을 환경변수에 추가합시다.
- Download Link: [https://rubyinstaller.org/downloads](https://rubyinstaller.org/downloads)

### RubyGems 설치
- Ruby 1.9 이상의 버전을 설치하면 RubyGems가 동봉되어 있습니다. (2.3.3 버전을 설치했으므로 패스)
* Download Link: [https://www.python.org/downloads](https://www.python.org/downloads)

---

### Jekyll 설치하기 

```bash
$ gem install jekyll
```

---

## Jekyll 템플릿 적용하기
  검색해보면 다양한 jekyll 무료 템플릿이 존재합니다. 저는 ([lanyon_plus](https://github.com/dyndna/lanyon-plus))을 찾아 적용했습니다. 템플릿 적용에 대해서는 아래 포스트를 참고 바랍니다. 
- [Jekyll-template-적용하기](https://ianjang.github.io/jekyll-template-적용하기/)

---

## 서버 구동

  Template 설치가 끝나면 해당 폴더 아래로 이동하여 아래 명령어를 수행합시다. 

```bash
$ jekyll serve 
```

  이제 브라우저에서 http://127.0.0.1:4000/ (혹은 5000 포트) 에 접속하여 블로그가 잘 보이는지 확인해 봅시다.

---

## Reference
  - [https://jekyllrb-ko.github.io/](https://jekyllrb-ko.github.io/)

