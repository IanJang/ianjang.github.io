# EC2 디스크 공간, 메모리 사용량 CloudWatch 지표 쌓기


## 들어가며

  AWS CloudWatch를 이용하여 AWS Cloud Infrastructure에 대한 다양한 지표를 대시보드 형태로 구성하여 모니터링을 할 수 있다. 다만 EC2내부의 DiskSpace와 MemoryUsage에 대해서 모니터링을 하기 위해서는 별도 작업이 필요하다.

---

## EC2 내부에 지표 수집용 스크립트 설치

  DiskSpace, MemoryUsage에 대한 지표를 수집하기 원하는 EC2에 SSH 접속 후 아래 스크립트를 수행하자.

```bash
$ sudo apt-get update
$ sudo apt-get install unzip
$ sudo apt-get install libwww-perl libdatetime-perl

$ curl <https://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.2.zip> -O
$ unzip CloudWatchMonitoringScripts-1.2.2.zip
$ rm CloudWatchMonitoringScripts-1.2.2.zip
$ cd aws-scripts-mon

$ cp awscreds.template awscreds.conf
```

  이후 아래 권한이 부여된 IAM 계정을 생성하고, 해당 계정의 AccessKey, SecretAccessKey를 awscreds.conf 파일에 입력하자.

- 필요권한
    - cloudwatch:PutMetricData
    - cloudwatch:GetMetricStatistics
    - cloudwatch:ListMetrics

---

## 지표 수집 동작 검증

  아래 스크립트를 수행하여 결과를 확인한다. Verification completed successfully. No actual metrics sent to CloudWatch. 메시지가 출력되면 성공!

```bash
$ ./mon-put-instance-data.pl --mem-util --disk-space-util --disk-path=/ --verify --verbose
```

---

## Crontab 연동하여 주기적으로 지표 수집 시작

  동작 검증이 끝났지만 지표가 실제로 수집되고 있는 것은 아니다. 실제로 지표를 수집하려면 crontab을 이용하여 주기적으로 스크립트가 수행되도록 작업이 필요하다. 아래 스크립트를 참고하여 작업하자.

```bash
$ sudo crontab -e

# 아래 내용 입력
*/5 * * * * ${ScriptPath}/mon-put-instance-data.pl --mem-used-incl-cache-buff --mem-util --disk-space-util --disk-path=/ --from-cron
```

---

## CloudWatch에서 확인

  모든 작업이 끝났다면 CloudWatch에 접속 후 아래 경로대로 이동하여 지표가 잘 쌓이는지 확인하자.

- CloudWatch 접속 > 좌측 지표 메뉴 선택 > 하단 [사용자 지정 네임스페이스] 확인 > System/Linux 선택

---

## CloudWatch 경보 SNS Lambda 이용 Slack에 푸시

  CloudWatch 경보를 이용하여 디스크 용량, 메모리 사용률이 위험수치에 도달했을 때 Slack에 푸시가 올 수 있도록 작업 할 수 있다. 아래 링크 참고!

- [https://brunch.co.kr/@alden/56](https://brunch.co.kr/@alden/56)

---

## 끝내며

  나의 경우는 대부분의 디스크 용량 문제는 로그관리를 잘 하지 못해서 발생했다. 로그 외에도 점진 적으로 데이터가 쌓일 수 밖에 없는 어플리케이션 구조를 갖고 있다면, 해당 데이터를 주기적으로 백업하는 방안을 마련해야 할 것이다. 그 방안이 마련되기 전, 혹은 마련 후에도 디스크 용량 문제 발생 가능성을 미연에 감지하기 위해서 본 작업이 유용할 것이라고 생각된다.  

---

## Reference

- [https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/mon-scripts.html](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/mon-scripts.html)

