# Crontab 이용하여 주기적으로 오래된 파일 제거하기


## 들어가며

  서버 애플리케이션이 특정 경로에 파일(로그, 이미지 등)을 쌓는 구조로 되어 있다면 언젠가 용량이 가득 차는 문제를 맞이할 수 있습니다. 이를 예방하기 위해선 특정 기간이 지난 파일을 지우거나, 압축하여 별도 저장소로 옮긴다거나 하는 과정이 필요합니다. 간단한 shell 스크립트를 작성하고 crontab에 등록하여 이러한 과정을 자동화 할 수 있습니다.

---

## Crontab 명령어

```bash
# crontab 추가 / 수정
$ crontab -e

# crontab 확인
$ crontab -l

# crontab 로그 확인
$ cat /var/log/syslog | grep CRON
```

---

## 오래된 파일을 제거하는 스크립트 작성하기

  아래 정리된 명령어를 참고합시다.

```bash
# * 수정한지 90일 이상 된 파일과 디렉토리 조회
$ find . -mtime +90 -ls

# 수정한지 90일 이상된 파일만 조회
$ find . -mtime +90 -type f -ls

# 수정한지 90일 이상된 파일만 삭제
# 정기적으로 90일지 지난 파일을 삭제할 때 유용 )
$ sudo find . -mtime +90 -type f -ls -exec rm -f {} \\;
```

  아래는 직접 완성한 스크립트입니다.

```bash
# remove_old_files.sh
path = "/home/ubuntu/work/data/"
days = 90
sudo find ${path} -mtime +${days} -type f -ls -exec rm -f {} \\;
```

- Reference
    - [https://zetawiki.com/wiki/리눅스_날짜_기준으로_파일_삭제하기](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EB%82%A0%EC%A7%9C_%EA%B8%B0%EC%A4%80%EC%9C%BC%EB%A1%9C_%ED%8C%8C%EC%9D%BC_%EC%82%AD%EC%A0%9C%ED%95%98%EA%B8%B0)
    - [https://zetawiki.com/wiki/리눅스_n일전_파일_삭제](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_n%EC%9D%BC%EC%A0%84_%ED%8C%8C%EC%9D%BC_%EC%82%AD%EC%A0%9C)
    - [https://dbrang.tistory.com/867](https://dbrang.tistory.com/867)

---

## Crontab에 작업 등록하기

  작성한 스크립트를 UTC기준 매일 15시 1분(KST기준 00시 1분)에 수행되도록 하였습니다.

```bash
$ crontab -e

# 아래 내용 입력 
1 15 * * * /home/ubuntu/myscripts/remove_old_files.sh >> /home/ubuntu/myscripts/remove_old_files.sh.log 2>&1
```

- Reference: [https://jdm.kr/blog/2](https://jdm.kr/blog/2)

---

## 이슈 및 해결
### Crontab default editor 변경

  contab -e 명령 수행 시 nano editor가 열러 작업하기 불편하다면 아래 방법으로 에디터를 수정 할 수 있습니다.

```bash
# 기본 에디터를 고르는 명령어 수행
$ select-editor

# nano로 셋팅된것을, vim으로 변경
Select an editor.  To change later, run 'select-editor'.
  1. /bin/ed
  2. /bin/nano        <---- easiest
  3. /usr/bin/vim.basic
  4. /usr/bin/vim.tiny

Choose 1-4 [2]: 3
```

- Reference: [http://www.42.mach7x.com/2017/02/01/changing-default-editor-for-crontab-from-vi-to-vim/](http://www.42.mach7x.com/2017/02/01/changing-default-editor-for-crontab-from-vi-to-vim/)

---

### root 권한으로 crontab 작업을 등록하고 싶을 때

  아래 명령을 이용하면 됩니다. 참고로 명령어에서 sudo를 빼면 현재 로그인된 계정에 대한 crontab 설정파일을 수정할 수 있습니다.

```bash
$ sudo crontab -e
```

- Reference: [https://askubuntu.com/questions/173924/how-to-run-a-cron-job-using-the-sudo-command](https://askubuntu.com/questions/173924/how-to-run-a-cron-job-using-the-sudo-command)

