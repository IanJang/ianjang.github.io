# AWS Lambda, S3를 이용한 이미지 변환 자동화(feat. ImageMagick)


## 들어가며

  이미지 리소스를 S3에 업로드 하면 자동으로 압축이 되게 할 순 없을까? AWS Lambda를 이용하면 이 같은 자동화된 기능을 구현하여 사용할 수 있습니다. 여러 가지 사례를 참고하여 직접 자동화 작업을 진행하고, 그 내용을 정리해 보았습니다.   

---

## ImageMagick

  먼저 이미지 압축을 위해 사용한 ImageMagick을 소개합니다. ImageMagick은 이미지 변환용으로 널리 쓰이는 오픈소스 라이브러리입니다.
- [위키백과:이미지매직](https://ko.wikipedia.org/wiki/%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A7%A4%EC%A7%81)
  
### ImageMagick 설치

  저는 먼저 우분투환경에서 ImageMagick을 설치하여 사용해 보았습니다. 아래 순서로 진행하였습니다.

```bash
# PNG library 설치
$ sudo apt-get install libpng-dev

# ImageMagick 설치
$ git clone https://github.com/ImageMagick/ImageMagick.git
$ cd ImageMagick
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig /usr/local/lib
$ /usr/local/bin/convert logo: logo.gif
$ make check
```

- Reference: [https://imagemagick.org/script/install-source.php](https://imagemagick.org/script/install-source.php)

---

### Command Line Options
  
  설치한 ImageMagick을 CLI로 사용하려고 보니 정말 다양한 옵션값이 있었습니다. 하지만 이 중 Quality 옵션만으로도 간단한 이미지 압축이 가능합니다. (단, 압축이 어려운 복잡한 이미지일 경우 용량이 되려 커지는 경우도 있습니다) 모든 Option 정보는 아래 링크를 참고합시다.
- [https://www.imagemagick.org/script/command-line-options.php](https://www.imagemagick.org/script/command-line-options.php)

---

### 이미지 포맷 별 Quality Option 값의 의미
  
  이미지 포맷에 따라 quality가 의미하는 바가 다르다는 점에 유의해야 합니다. JPEG/MPEG 포맷의 경우 quality 값이 낮을수록 높은 압축률을 의미하고, MNG/PNG 포맷의 경우 quality 값이 높을 수록 높은 압축률을 의미합니다. 그뿐만 아니라 MNG/PNG 포맷의 경우는 quality 값에 의해 필터 타입도 결정됩니다.
    
- JPEG/MPEG: 1(압축률최대) ~ 100(압축률최저)
- MNG/PNG
  - zlib compression level: (quality / 10)
  - filter type: (quality % 10)
  
- Reference: [https://www.imagemagick.org/script/command-line-options.php#quality](https://www.imagemagick.org/script/command-line-options.php#quality)

---

### ImageMagick 명령어

  아래는 image.png를 quality 80 옵션으로 변환한 image_compressed.png를 생성하는 명령어입니다.

```bash
# Image Compression
$ magick convert image.png -quality 80 image_compressed.png
```

- Reference: [https://imagemagick.org/script/command-line-processing.php](https://imagemagick.org/script/command-line-processing.php)

---

## Wand(ImageMagick for python)

  지금까지 우분투 환경에서 ImageMagick을 설치하고 사용해 보았습니다. 이제 Python 코드 내에서 ImageMagick을 사용해 봅시다. 여러 가지 패키지가 있지만, 그중에서 Wand를 추천합니다. (PythonMagick, PythonMagickWand도 있지만, 이들은 마지막 Release 버전이 10년도 넘어서 사용하기 꺼려집니다)
  
```bash
$ pip install wand
```

  아래는 wand를 이용하여 PNG 이미지를 압축하는 Python 코드입니다. 좀 더 정확히 말하자면 origin/image1.png 파일을 "zlib_compression_level=8, filter_type=0" 옵션으로 압축한 파일을 compress/image1.png에 저장하는 코드입니다.

```python
from wand.image import Image

with Image(filename='origin/image1.png') as img:
    img.compression_quality = 80
    img.save(filename='compress/image1.png')
```

---

## Lambda 함수 작성

  이제 위에서 작성한 python 코드를 가지고 Lambda 함수를 작성해 봅시다. 

### 환경변수 설정

  S3 제어를 위해 access_key 및 secret_key 사용이 필요했고, 이를 lambda 함수의 환경변수로 세팅하고 코드에 사용했습니다.

```python
ACCESS_KEY = os.environ.get('ACCESS_KEY')
SECRET_KEY = os.environ.get('SECRET_KEY')
```

  AWS Lambda > 함수 선택 > 환경변수에 설정하고, 위 코드처럼 가져다 쓰면 됩니다.

---

### External Library를 포함하여 Lambda함수 생성하는 방법

  AWS CLI 명령어를 이용하면 간편하게 External Library를 포함하여 Lambda 함수를 갱신할 수 있습니다. wand python 패키지를 lambda 함수에 포함해 보았습니다.

```bash
# 패키지 설치용 폴더 생성
$ mkdir package
$ cd package

# 해당 경로에 패키지 설치
$ pip install wand --target .

# zip파일로 압축
$ zip -r9 ../function.zip .
$ cd ..

# lambda 함수 소스코드를 압축파일에 추가
$ zip -g function.zip function.py

# 압축 파일을 통하여 Lambda 함수 갱신
$ aws lambda update-function-code --function-name ${람다함수명} --zip-file fileb://function.zip
```

- Refecence
    - [https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)
    - [[AWS docs] 자습서: Amazon S3과 함께 AWS Lambda 사용](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/with-s3-example.html)

---

### Lambda 함수 S3 event trigger 테스트 방법

  아래 S3 ObjectCreated Event 샘플을 이용하여 테스트 이벤트를 구성하였습니다. 아래 코드 사용 시 파일에서 bucket과 object key 부분만 수정해서 사용하면 됩니다. 그 외 다른 Service의 Event가 필요하다면 Reference를 참고하여 테스트 이벤트를 구성하면 됩니다.

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "ap-northeast-2",
      "eventTime": "2019-04-05T07:49:12.805Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "AWS:생략"
      },
      "requestParameters": {
        "sourceIPAddress": "###.###.##.##"
      },
      "responseElements": {
        "x-amz-request-id": "생략",
        "x-amz-id-2": "생략"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "생략",
        "bucket": {
          "name": "s3.bucket.name",
          "ownerIdentity": {
            "principalId": "생략"
          },
          "arn": "arn:aws:s3:::s3.bucket.name"
        },
        "object": {
          "key": "object.key",
          "size": 1024,
          "eTag": "생략",
          "sequencer": "생략"
        }
      }
    }
  ]
}

```

- Reference: [https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/eventsources.html](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/eventsources.html)

---

### 완성된 코드

  최종적으로 완성된 코드는 아래 링크를 참고 부탁드립니다.
- [https://github.com/IanJang/lambda-wand-image-convert](https://github.com/IanJang/lambda-wand-image-convert)

---

## 이슈 및 해결

### magick: no decode delegate for this image format `PNG'

  이와 같은 에러 메시지를 만난다면 libpng library가 없는 경우 입니다. sudo apt-get install libpng-dev 명령으로 라이브러리를 설치하면 해결됩니다.
- Reference: [http://www.libpng.org/pub/png/libpng.html](http://www.libpng.org/pub/png/libpng.html)

### Lambda Event가 무한히 Trigger 되는 문제

- 현상
  - S3에서 파일을 다운로드  로컬에서 변형 > S3에 올리는 Lambda 함수를 구성하고, 이 함수의 Event Trigger로 s3 ObjectCreated Event를 연결했습니다. 이후 Lambda 함수가 s3 Event에 의해 Trigger 되어 잘 수행되는지 테스트를 진행했는데, Lambda 함수가 무한히 호출되는 현상이 발생했습니다.
- 원인
  - 사용자가 S3에 파일을 원본 이미지를 업로드 하는 것도 S3 ObjectCreated Event를 발생시키지만, Lambda 함수 수행 후 변환된 이미지를 S3에 업로드 하는 것 또한 ObjectCreated Event를 발생시켰기 때문에 무한루프에 빠진 것이 원인이었습니다.
- 해결
  - S3 CreatedObject Event Trigger에 Object Prefix 조건을 걸면 됩니다. 특정 경로에 파일 업로드 시만 Event가 Trigger 되도록 제한할 수 있습니다. Lambda 함수 내부에서 S3에 파일을 다시 올려야 한다면 Event Trigger를 걸어놓은 경로는 피하는 게 좋습니다.
  - 원본 이미지 업로드용 S3 Bucket과 변환 이미지 업로드용 S3 Bucket을 별도 구성해서 원본 이미지 업로드용 Bucket에만 Event Trigger를 거는 방법도 있습니다.

---

### [ERROR] ClientError: An error occurred (404) when calling the HeadObject operation: Not Found Traceback (most recent call last)

- 현상
  - 파일명이 test (1).png인 파일을 s3에 업로드 했더니 Lambda 함수 수행이 실패했습니다. 로그에는 제목과 같은 에러가 있었고 자세히 살펴보니 S3 ObjectCreated event에 포함된 object key 값이 test+%281%29.png과 같이 깨져있었습니다. 깨진 key 값 때문에 S3 파일 다운로드를 object를 찾지 못하고 에러가 발생한 것입니다.
- 원인
  - S3 event에 담아서 전달되는 object key 값은 URL 인코딩이 됩니다. 이를 디코딩 없이 사용한 것이 문제의 원인이었습니다.
- 해결
  - Object key를 디코딩하여 사용하도록 함수를 개선하였습니다.

```python
from urllib.parse import unquote_plus
# (...생략...)
decoded_key = unquote_plus(key)
```

- Reference: [https://forums.aws.amazon.com/thread.jspa?threadID=215813](https://forums.aws.amazon.com/thread.jspa?threadID=215813)

---

### Task timed out after 3.00 seconds

- 현상
  - 앞서 만든 이미지 변환 Lambda 함수를 테스트했더니 용량이 큰 이미지 파일에 대해서는 제대로 동작하지 않고 위 제목과 같은 에러가 발생했습니다.
- 원인
  - 친절한 에러 메세지덕에 timeout 문제임을 알 수 있었습니다.
```
# CloudWatch 에러로그
REPORT RequestId: ae6e0aae-9ca4-4f1a-ac65-b23cf75bb4fd	Duration: 3003.50 ms	Billed Duration: 3000 ms Memory Size: 128 MB	Max Memory Used: 65 MB	
2019-04-05T01:15:03.289Z ae6e0aae-9ca4-4f1a-ac65-b23cf75bb4fd Task timed out after 3.00 seconds
```
- 해결
  - Lambda 함수 메모리를 128MB -> 256MB로 늘여서 해결하였습니다.
  - Lambda 함수 할당 메모리양을 증가시키면 그에 비례한 성능의 CPU가 Lambda 함수에 할당됩니다. 이로 인해 함수 수행속도가 빨라지지만, 비용도 그만큼 증가합니다. 비용과 성능의 적정 타협점을 찾는 게 중요할 것 같습니다.

---

## 맺으며

  AWS Lambda를 이용하여 서버리스 아키텍처를 아주 살짝 경험해 봤습니다. 별도 서버 구축 없이 기능을 구현하여 사용자에게 제공하는 경험은 신선했습니다. AWS Lambda는 적재적소에 잘만 사용하면 더 빠르고 효율적으로 고객의 요구사항을 해결할 수도 있는 유용한 서비스인 것 같습니다. 

