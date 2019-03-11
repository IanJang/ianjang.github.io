## AWS Media Convert

#### Pricing
* 기준:
    - Basic Plan
    - AVC 코덱
    - 아시아 태평양(서울)
- On-Demand 요금

|   |fps<=30|30<fps<=60|60<fps<=120|
|---|-------|----------|-----------|
|SD |0.0085 USD|0.0106 USD|0.0127 USD|
|HD |0.0170 USD|**0.0212 USD**|0.0255 USD|
|UHD|0.0340 USD|0.0425 USD|0.0510 USD|

- RTS(ReservedTranscodeSlots) 요금 12개월 약정: 500 USD

- 추가 요금
    - Dolby Audio 분당 0.0050 USD
    - 오디오 정규화 분당 0.0020 USD

- 베이직 티어와 프로페셔널 티어의 해상도는 아래와 같이 정의됨
    - SD: 수직해상도가 720 미만인 출력
        - 480p: 720x480 60fps
    - HD: 수직해상도가 720 이상이면서 1080 이하 출력
        - 720p: 1280x720 60fps
        - 1080i: 1920x1080 30fps(비월주사(인터레이즈.i))
        - FHD(1080p): 1920x1080 60fps
    - UHD: 수직해상도가 1080보다 크면서 최대 2160인 출력
        - 4K: 4096x2160 60 fps

- 비용 계산
    - 720p($0.0212/min) * 23,585 Minutes ≒ $500
    - 23,585 Minutes ≒ 393 Hours
    - 월 393시간 미만의 영상을 인코딩 한다면, On-Demand
    - 그 이상이라면 RTS를 1개 이상 두고, 초과 시 1개 더 늘리는 식..

- References
    - [[AWS Document]AWS Elemental MediaConvert 요금](https://aws.amazon.com/ko/mediaconvert/pricing/)
    - [[블로그]빠르크의 3분강좌-해상도(frame resolution)](https://parkpictures.tistory.com/91)

------

#### Output Groups
##### Apple HLS Group Setting
- Segment
    - Segment control
        - Segmented files: 기본
        - Single File: 하나의 Segment만 생성
            - segment를 나눈 것과 성능상의 큰 차이는 없음
            - master manifest 파일에는 ts 정보가 아닌 byte 정보가 담김
    - Segment length(sec)
        - live 는 보통 2초
        - Segment가 길수록 파일사이즈가 커지고, 최초 ts 다운로드 시 오래걸림
    - VOD는 굳이 자를 필요는 없음 (추천: 4초이상 안쓰는게 좋음)

- Codec specification
    - [RFC4281(default)](https://www.rfc-editor.org/rfc/rfc4281.txt)
    - [RFC6381](https://www.rfc-editor.org/rfc/rfc6281.txt)

###### Output Settings(H.264)
- Preset
    - Name modifier 세팅. 생성되는 파일의 post_fix
- Add I-Frame only manifest
    - 이용시, 플레이어의 비디오 미리 보기에서 콘텐츠 스크러빙을 더 쉽게 지원할 수 있음
    - References
        - [[AWS Document] AWS Elemental MediaLive, I-프레임 전용 HLS 매니페스트 및 JPEG 출력 기능 추가](https://aws.amazon.com/ko/about-aws/whats-new/2019/01/aws-elemental-medialive-add-i-frame-only-hls-manifest-and-jpeg-outputs/)
- Transfort stream settings
    - ?

##### Encoding Settings
- AFD(Active Format Description)
    - [[위키피디아]액티브 포맷 디스크립션](https://ko.wikipedia.org/wiki/%EC%95%A1%ED%8B%B0%EB%B8%8C_%ED%8F%AC%EB%A7%B7_%EB%94%94%EC%8A%A4%ED%81%AC%EB%A6%BD%EC%85%98)
- SEI(Supplemental Enhancement Information)
    - H.264의 임의액세스(영상 비트열 중간부터 재생가능하게 해주는 기능)를 위한 정보 프레임
    - References
        - [[AWS Document]출력 타임 코드 설정 조정](https://docs.aws.amazon.com/ko_kr/mediaconvert/latest/ug/timecode-output.html)

- Framerate
    - 기본적으로 소스를 따르게 설정

- Video Mode
    - Rate control mode
        - CBR: 비트레이트가 일정
        - VBR: 정해진 값에서 올라갔다 내려갔다
        - QVBR: 쉬운영상이 나오면 낮은 비트레이트 어려운 영상이 나오면 높은 비트레이트
    - Quality tuning Level / Max bitrate(bits/s) / QVBR Quality level

|해상도|너비|높이|QVBR 품질수준|최대 비트레이트|
|--|--|--|--|--|
|1,080p|1,920|1,080|9|6,000,000|
|720p|1,280|720|8|4,000,000|
|720p|1,280|720|7|2,000,000|
|480p|640|480|7|1,000,000|
|360p|480|360|7|700,000|
|240p|352|240|7|350,000|

  - References

      - [[AWS Document] QVBR 속도 제어 모드 사용](https://docs.aws.amazon.com/ko_kr/mediaconvert/latest/ug/cbr-vbr-qvbr.html)

- HRD Buffer

- GOP(Group of Pictures)
    - I frame의 간격을 말함
    - B frame 2개일 경우: IBBPBBPBBPBBPBBPBBPBBP...I
    - I: Key frame / B: 예측 frame / P: 참조 frame
    - I frame이 용량이 제일 큼 / B프레임이 용량이 제일 작음
    - 쉬운 영상(역동적이지 않은)일수록 B frame을 많이 쓰는 것이 유리
    - 낮은 대역폭에 대해서는 B frame을 많이 쓰는 것이 유리
    > 애니메이션은 Easy한 소스, bitrate를 낮춰도 크게 영상에 문제가 생기거나 하지 않음.
    720p라면 일반적으로 2Mbps쓰지만 1.5Mbps로도 충분하고, QVBR Target 7 쓰면 CDN비용 40%이상 Save 할것으로 예상. GOP size는 60 / B frames between reference frames는 2~3 정도 쓰는것이 좋다.
    - References
        - [[위키피디아]IPB Frame - Video compression picture types](https://en.wikipedia.org/wiki/Video_compression_picture_types#Bi-directional_predicted_frames.2Fslices_.28B-frames.2Fslices.29)
- Scene Change Detection

    - I프레임을 중간에 삽입해줌
- Profile
    - BaseLine(CBP/BP)
        - 저비용 어플리케이션(영상회의/모바일 어플리케이션 등)
        - 영상손실, 인코더 기능을 특정 Basic한 것만으로 제한하는 방식
        - 양방향 예측(B-Frame) 지원X
    - Main(MP)
        - 디지털 TV 방송 표준
        - 양방향 예측(B-Frame) 지원
    - High(HiP)
        - 고해상도 TV어플리케이션ㅡ Bluray Disc, DVB HDTV 방송 서비스
        - 양방향 예측(B-Frame) 지원
    - References:
        - [H.264/MPEG-4 AVC Profiles](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Profiles)
        - [H.264/MPEG-4 AVC Profiles KOR](https://ko.wikipedia.org/wiki/H.264/MPEG-4_AVC)
- Level
    - 비트율, 프레임율, 화면해상도
    - 서비스하려는 단말기 중에서 가장 낮은 레벨이 무엇인지 확인해야 한다.
- Entropy Encoding
    - 비디오 재생시에 CAVLC과 CABAC의 성능 차이는 없는 반면에 화면의 질은 CABAC이 낫다.
    - CABAC: 상황 적응 이진 산술 코딩. 기본 확률 모델에 대한 고효율 자동 조정을 제공. 산술코더, 가장 높은 압축 효율 비율. CAVLC보다 많은 하드웨어 리소스가 필요
    - CAVLC: 상황 적응 가변 길이 코딩. 호프만과 같은 압축 알고리즘 사용. 적절 품질, 빠른 데이터 압축
    - References
        - [http://www.rumpus.co.kr/04.Technology/TransCoderTab04.asp](http://www.rumpus.co.kr/04.Technology/TransCoderTab04.asp)
- Slice
    - 화면을 독립적으로 분할하여 인코딩 가능
    - 여러 CPU를 동시에 사용할 수 있다.
    - 각 슬라이스 간의 중복되는 부분을 알 수 없음. 인코딩 품질이 떨어질 수 있다

##### Audio encoding settings
- profile
    - HE-AAC: 디지털 오디오에서 쓰이는 손실 데이터 압축 방식
        - LC: 복잡성이 낮고, 스트리밍 오디오와 같은 낮은 비트레이트 애플리케이션에 최적화됨
        - HEV1: 스펙트럼 대역 복제(Spectral band replication, SBR)를 사용하여 주파수 영역(frequency domain)에서 압축 효율을 향상시킨다.
        - HEV2: 스펙트럼 대역 복제와 파라메트릭 스테레오(PS)가 한 쌍을 이루어 스테레오 신호의 압축 효율을 향상시킨다. 박수와 같은 압축하기 어렵다고 알려진 음원에서는 음질의 열화가 발생한다고 알려져 있다.
    - References
        - [[위키피디아]HE-AAC](https://ko.wikipedia.org/wiki/HE-AAC)

- Bitrate control mode
    - CBR
    - VBR

------

## Medea 서비스 제공 Origin
- MediaStore
    - 심플하게 라이브 스트리밍 구성할 경우 사용
- MediaPackage
    - 보다 복잡한 구성시 사용
- S3
   - VOD 용
   - Live용으로는 사용 불가능 / MetaData 업데이트가 안되서

## CDN
- Data Size -> 비용과 연결됨
- Profile Ordering
    - 대부분 낮은것 부터 올라가도록 세팅함
    - 다만, 충분히 네트웍 속도가 빠른 상태에서는 굳이 내려받지 않아도되는 해상도의 세그먼트 까지 다운받게 되어, 비용 낭비가 생김
    - 추천사항은 720p > 480p > 1080p 와 같이 적당히 중간것 부터 제공하도록

--------------

### 배경지식

- OTT(Over The Top)
    - 인터넷을 통해서 미디어 서비스를 받을 수 있는 모든 Device
- HLS
    - 전세게적으로 90% 정도 사용
    - 향후에는 대부분 CMAF로 갈거임 (HLS + DASH)
    - 애플에서 만든 Adaptive Bitrate Streaming
    - Master Manifest(index.m3u8)  > 사용자별로 다르게 구성할 수 있음..
        - Child Manifest(index_720p.m3u8)
        - Child Manifest(index_480p.m3u8)
        - Child Manifest(index_360p.m3u8)
    - Live 용 HLS 컨버팅시 파일 생성순서 TS > Child manifest > Master Manifest

- References
    - [[Tistory]개똥이야기-H.264소개](https://unipro.tistory.com/101)


### 문의사항
1. Apple HLS group settings > Advanced > Codec specification: RFC 4281 vs. RFC 6381
2.
