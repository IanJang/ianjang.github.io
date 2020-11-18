# EC2 디스크 공간, 메모리 사용량 CloudWatch 지표 쌓기


## 들어가며

AWS CloudWatch를 이용하여 AWS Cloud Infrastructure에 대한 다양한 지표를 대시보드 형태로 구성하여 모니터링을 할 수 있습니다. 다만 EC2 내부의 디스크 공간(Disk Space)과 메모리 사용량(Memory Usage)을 모니터링하기 위해서는 별도 작업이 필요합니다.

---

## EC2 내부에 지표 수집용 스크립트 설치

  디스크 공간(Disk Space), 메모리 사용량(Memory Usage) 지표를 수집하고 싶은 EC2에 SSH 접속하여 아래 스크립트를 수행합시다.

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

  이후 아래 권한들이 부여된 IAM 계정을 생성하고, 해당 계정의 AccessKey, SecretAccessKey를 awscreds.conf 파일에 입력합시다.

- 필요권한
    - cloudwatch:PutMetricData
    - cloudwatch:GetMetricStatistics
    - cloudwatch:ListMetrics

---

## 지표 수집 동작 검증

  아래 스크립트를 수행하여 결과를 확인합니다. "Verification completed successfully. No actual metrics sent to CloudWatch." 메시지가 출력되면 성공입니다.

```bash
$ ./mon-put-instance-data.pl --mem-util --disk-space-util --disk-path=/ --verify --verbose
```

---

## Crontab 이용 주기적으로 지표 수집하기

  지표 수집 동작 검증이 완료되었지 실제로 지표가 수집되고 있는 것은 아닙니다. 실제로 지표를 수집하려면 crontab을 이용하여 주기적으로 스크립트가 수행되도록 작업이 필요합니다. 아래 스크립트를 참고하여 추가 작업을 진행합니다.

```bash
$ sudo crontab -e

# 아래 내용 입력
*/5 * * * * ${ScriptPath}/mon-put-instance-data.pl --mem-used-incl-cache-buff --mem-util --disk-space-util --disk-path=/ --from-cron
```

---

## CloudWatch에서 지표 수집 여부 확인

  모든 작업이 끝났다면 CloudWatch에 접속 후 아래 경로로 이동 후 지표가 잘 쌓이고 있는지 확인합시다.

- CloudWatch 접속 > 좌측 지표 메뉴 선택 > 하단 [사용자 지정 네임스페이스] 확인 > System/Linux 선택

---

## CloudWatch 경보를 SNS, Lambda를 이용하여 Slack에 알리기

  특정 EC2에 대해서 디스크 용량, 메모리 사용률이 위험수치에 도달했을 때 Slack에 알림이 되도록 작업 할 수 있습니다. 자세한 내용은 아래 링크를 참고합시다.

- [https://brunch.co.kr/@alden/56](https://brunch.co.kr/@alden/56)

---

## 맺으며 
  디스크 공간이 가득 차는 문제는 애플리케이션이 점진적으로 데이터가 쌓이는 구조로 되어 있다면 필연적으로 발생합니다. 이럴 때 데이터를 주기적으로 별도 공간에 백업 후 삭제하거나 하는 방안을 마련해야 합니다. 메모리 사용률은 트래픽 증가, 메모리 누수 등의 이유로 위험 수치에 도달할 수 있습니다. 본 글의 내용이 이러한 문제를 미연에 감지하고 예방하는 데 도움이 되길 바랍니다.   
  

---

## Reference

- [https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/mon-scripts.html](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/mon-scripts.html)

