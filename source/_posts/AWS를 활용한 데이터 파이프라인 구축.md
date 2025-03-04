---
title: AWS 서비스들로 구현하는 데이터 파이프라인
catalog: true
date: 2025-02-26 18:15:06
subtitle: 클라우드 시대 데이터 파이프라인 구축, 우리는 이렇게 AWS를 활용했습니다.
header-img: "/img/article_header/data_pipeline/data_pipeline_background.jpg"
tags:
    - AWS
    - DevOps
    - Infra
catagories:
    - AWS

---

안녕하세요, 하이스트레인저 백엔드 엔지니어 이상훈입니다.

최근 서비스 개발에서 **데이터 파이프라인**은 없어서는 안 될 필수 요소로 자리 잡았습니다. 다른 개발자 여러분께서는 데이터 파이프라인을 어떻게 구축하고 계신가요?

저희 팀은 데이터 파이프라인을 도입하면서 여러 가지 고민이 있었는데요. 이번 글에서는 데이터 파이프라인을 도입하게 된 배경부터 구축 과정에서 마주한 고민 사항들, 그리고 해결을 위해 선택한 솔루션들에 대해 공유하고자 합니다.
특히 AWS를 활용하여 어떻게 효율적이고 확장 가능한 데이터 파이프라인을 구축했는지, 저희의 경험과 인사이트를 공유합니다. 데이터 파이프라인 구축을 고민하시는 분들께 도움이 되기를 바랍니다.
<br>

---

# 데이터 파이프라인이란 무엇인가요?
데이터 파이프라인은 다양한 소스에서 발생한 데이터를 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**수집**</span>하고, 필요에 따라 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**가공**</span>하여 최종 목적지로 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**전달**</span>하는 일련의 과정입니다. 이를 통해 데이터는 원천에서부터 활용 지점까지 원활하게 흐르고, 비즈니스 인사이트 도출이나 서비스 개선에 활용될 수 있습니다.
<br>

## 일상생활의 데이터 파이프라인 #1 : 실시간 교통 정보
<div style="text-align: center;">
  <img src="/img/article/data_pipeline/navigation.jpg" alt="실시간 교통 정보 제공" style="width: 80%; height: auto; border-radius: 10px;">
</div>

- **데이터 수집**: 도로에 설치된 센서나 차량의 GPS로부터 실시간 교통 데이터가 수집됩니다.
- **데이터 처리**: 수집된 데이터를 즉시 처리하여 도로의 혼잡도나 사고 정보를 파악합니다.
- **데이터 전달**: 내비게이션 앱이나 교통 안내 표시판에 실시간으로 업데이트하여 운전자들에게 제공합니다.
<br>

## 일상생활의 데이터 파이프라인 #2 : 날씨 예보 및 기상 정보
<div style="text-align: center;">
  <img src="/img/article/data_pipeline/weather_forecast.jpg" alt="날씨 예보 및 기상 정보 제공" style="width: 80%; height: auto; border-radius: 10px;">
</div>

- **데이터 수집**: 위성, 기상 관측소, 레이더 등에서 온도, 습도, 기압 등의 기상 데이터를 수집합니다.
- **데이터 처리**: 방대한 데이터를 실시간으로 처리하여 날씨 변화를 예측합니다.
- **데이터 전달**: 정확한 기상 정보를 애플리케이션이나 웹사이트를 통해 대중에게 제공합니다.

데이터 파이프라인은 이렇게 우리 생활 속에서 다양한 방식으로 활용되며, 데이터의 가치를 극대화하는 데 핵심적인 역할을 하고 있습니다.
<br>

---

# INSIGHT FLOW 소개
데이터 파이프라인을 도입하게 된 서비스는 [INSIGHT FLOW](https://www.insightflow-ai.com/)입니다.
<br>

<div style="text-align: center;">
  <img src="/img/article/data_pipeline/InsightFlow_LOGO.png" alt="Insight Flow Logo" style="width: 60%; height: auto; border-radius: 10px;">
</div>

INSIGHT FLOW는 관객의 반응을 분석하여 콘텐츠의 경쟁력을 높이는 데 도움을 주는 서비스예요.
간단히 말해, 이 서비스는 관객이 콘텐츠를 볼 때 얼마나 집중하고 어떤 감정을 느끼며 얼마나 만족하는지를 측정합니다. 이를 위해 맥박이나 뇌파 같은 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**바이오 신호**</span>와 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**적외선 카메라**</span>를 사용하여 관객의 반응을 수집하고, AI 기술로 이를 분석합니다.

이렇게 얻은 데이터를 통해 콘텐츠 제작자나 마케팅 담당자는 관객이 실제로 어떻게 반응하는지를 알 수 있어, 더 좋은 콘텐츠를 만들고 전략을 세울 수 있어요.
<br>

## 기존 데이터 전달 방식의 문제점

초기에는 측정 인원이 많지 않아, 태블릿 로컬에 저장된 바이오 데이터와 카메라 데이터를 일일이 PC에 연결하여 추출했습니다. 그런 다음 이 데이터를 사내 NAS 서버로 업로드하고, 머신 러닝 PC에서 해당 데이터를 다시 다운로드하여 분석하는 방식으로 운영되었습니다.

하지만 시간이 지나면서 측정자가 늘어나고, 관리해야 할 데이터의 규모도 GB 단위에서 TB 단위로 커지면서 다음과 같이 여러 문제가 발생했습니다.

1. <span style="background-color: #ff9e9e; padding: 2px; border-radius: 4px">**시간 소요 증가**</span>: 데이터를 수동으로 옮기다 보니 시간이 많이 소요되었습니다.
2. <span style="background-color: #ff9e9e; padding: 2px; border-radius: 4px">**휴먼 에러 발생**</span>: 사람이 직접 데이터를 이동시키면서 중복되거나 누락된 데이터가 발생하였습니다.
3. <span style="background-color: #ff9e9e; padding: 2px; border-radius: 4px">**데이터 검증의 어려움**</span>: 데이터 검증이 제대로 이루어지지 않아 분석 과정에서 혼란이 생겼습니다.
4. <span style="background-color: #ff9e9e; padding: 2px; border-radius: 4px">**분석 결과 조회 지연**</span>: 분석 결과를 확인하기까지 시간이 많이 걸렸습니다.

이러한 문제들은 서비스의 효율성과 신뢰성을 저해하였습니다. 무엇보다 데이터를 다루는 서비스에서는 데이터의 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**일관성**</span>과 <span style="background-color: #A7F1A8; padding: 2px; border-radius: 4px">**무결성**</span>이 보장되어야 합니다. 이러한 필요성에 따라, 저희는 **데이터 파이프라인 도입**을 결정하게 되었습니다.
<br>

---

# AWS를 선택한 이유

데이터 파이프라인을 구축하기 위해 저희는 여러 가지 솔루션을 검토하였습니다. 그중에서도 **AWS**(Amazon Web Services)를 선택하게 된 이유는 다음과 같습니다.
<br>

## 다른 솔루션과의 비교

### 자체 인프라 구축 vs. AWS
- **자체 인프라의 한계**  
  초기 투자 비용이 많이 들고, 확장성이 떨어지며, 유지 보수에 많은 자원이 필요합니다. 또한 빠르게 변화하는 비즈니스 환경에 유연하게 대응하기 어렵습니다.

### 다른 클라우드 서비스(Azure, GCP) vs. AWS
- **서비스 범위 및 성숙도**  
  AWS는 가장 오랜 기간 클라우드 서비스를 제공해 왔으며, 서비스의 다양성과 안정성이 뛰어납니다.
- **커뮤니티와 지원**  
  AWS는 사용자 커뮤니티가 활발하고 자료가 풍부하여 기술적인 문제를 해결하기 쉽습니다.
<br>

## 데이터 파이프라인의 구성요소를 제공

데이터 파이프라인은 다음의 **5가지 필수 구성요소**로 이루어집니다. (<span style="color: gray">[GeeksForGeeks: Data Pipeline](https://www.geeksforgeeks.org/data-pipeline-design-patterns-system-design/)</span>)  
AWS는 이러한 각 구성요소에 적합한 서비스를 제공하여 통합된 환경에서 파이프라인을 구축할 수 있게 합니다.

1. **데이터 소스 (Data Sources)**  
   다양한 소스에서 데이터가 생성됩니다. AWS는 이러한 다양한 데이터 소스와의 통합을 지원합니다.  
   <span style="color:gray; font-size:0.9em;">관련 AWS 서비스: Amazon RDS, Amazon DynamoDB, Amazon S3 등</span>

2. **수집 (Ingestion)**  
   데이터를 파이프라인으로 추출하고 가져오는 단계입니다.  
   <span style="color:gray; font-size:0.9em;">관련 AWS 서비스: Amazon Kinesis Data Streams, AWS Data Migration Service, AWS IoT Core</span>

3. **데이터 처리 (Data Processing/Transformation)**  
   수집된 데이터를 정제하고 변환하여 사용 가능한 형태로 만듭니다.  
   <span style="color:gray; font-size:0.9em;">관련 AWS 서비스: AWS Glue, AWS Lambda, Amazon EMR</span>

4. **저장 (Storage)**  
   처리된 데이터를 저장하여 추가 활용이 가능하도록 합니다.  
   <span style="color:gray; font-size:0.9em;">관련 AWS 서비스: Amazon S3, Amazon Redshift, Amazon RDS, Amazon DynamoDB</span>

5. **워크플로 오케스트레이션 (Workflow Orchestration)**  
   파이프라인의 작업 순서와 종속성을 관리하며, 모니터링과 오류 처리를 통해 안정성을 보장합니다.  
   <span style="color:gray; font-size:0.9em;">관련 AWS 서비스: AWS Step Functions, AWS EventBridge</span>
<br>

## 비용 효율성

AWS를 선택한 또 다른 중요한 이유는 **비용 효율성**입니다.
- **초기 투자 비용 절감**  
  자체 인프라를 구축하려면 서버, 네트워킹 장비, 스토리지 등 막대한 초기 비용이 필요합니다. 반면, AWS는 필요한 자원을 클라우드에서 즉시 활용할 수 있어 초기 투자 비용을 크게 절감할 수 있습니다.
- **유연한 비용 구조**  
  AWS는 사용한 만큼만 비용을 지불하는 On-Demand 모델을 제공합니다. 이를 통해 불필요한 비용 지출을 막을 수 있고, 예산 계획도 유연하게 수립할 수 있습니다.
<br>

---

# 구축
데이터 파이프라인 구축은 먼저 전체적인 **오케스트레이션**을 설정한 후, **데이터 수집 → 데이터 전처리 → 데이터 분석 → 저장**의 진행 단계에 따라 순차적으로 진행하였습니다.

전체 플로우 다이어그램은 다음과 같습니다.

<div style="position: relative; overflow: visible;">
  <img src="/img/article/data_pipeline/data_pipeline_diagram.png" alt="Data Pipeline Diagram" style="position: relative; left: 50%; transform: translateX(-45%); width: 120rem; max-width: none; border-radius: 10px;">
</div>
<br>

## 오케스트레이션
<br>
<div style="display: flex; justify-content: center; align-items: center;">
  <img src="/img/article/data_pipeline/event_bridge.png" alt="Event Bridge" style="width: 20%; border-radius: 10px; margin: 0 1%;">
  <img src="/img/article/data_pipeline/step_function.png" alt="Step Function" style="width: 20%; border-radius: 10px; margin: 0 1%;">
</div>

데이터 파이프라인은 사실 오케스트레이션 설정 없이도, 예를 들어 S3 업로드가 AWS Glue나 AWS Lambda를 트리거하는 방식으로 구축할 수 있어요. 그러나 이러한 방식은 추후 유지보수 시 시간이 많이 소요되고, 각 설정을 하나하나 기억해야 하는 부담이 있습니다.

저희는 데이터 파이프라인의 흐름을 중앙에서 효율적으로 관리하기 위해 **Event Bridge**와 **Step Functions**를 활용한 오케스트레이션 방식을 선택하였습니다. Event Bridge를 사용하면 전체 이벤트 중에서 특정 이벤트만 필터링하여 원하는 AWS 서비스로 전달할 수 있고, 이를 통해 데이터의 전체 흐름을 Step Functions로 관리하면 파이프라인의 복잡성을 줄이고 유지보수를 간소화할 수 있습니다.
<br>

<img src="/img/article/data_pipeline/orchestration.gif" alt="Step Function Monitoring" style="width: 80%; border-radius: 10px;">
<div style="text-align: center;">
  <em>&lt;Step Function Monitoring&gt;</em>
</div>
<br>

---

## 데이터 수집
<br>
<div style="display: flex; justify-content: center; align-items: center;">
  <img src="/img/article/data_pipeline/lambda.png" alt="Lambda" style="width: 20%; border-radius: 10px; margin: 0 1%;">
  <img src="/img/article/data_pipeline/firehose.png" alt="Firehose" style="width: 20%; border-radius: 10px; margin: 0 1%;">
  <img src="/img/article/data_pipeline/s3.png" alt="S3" style="width: 20%; border-radius: 10px; margin: 0 1%;">
</div>

저희는 앱으로부터 총 **4가지 종류의 데이터**를 수집합니다:

1. **측정자의 서명** (`.png`)
2. **사용자의 개인 정보** (`.json`)
3. **카메라 영상** (`.mp4`)
4. **바이오 신호** (`byte`)

서명, 개인 정보, 영상 데이터는 Lambda를 활용하여 앱에 일시적인 S3 Presigned URL을 발급함으로써 간단하게 수집할 수 있었어요. 앱은 이 URL을 통해 해당 데이터를 직접 S3 버킷에 업로드합니다.

그러나 바이오 신호의 수집에는 고민이 있었습니다. 측정이 종료된 후에야 바이오 신호 데이터를 받게 되면, 측정 중에 발생한 이상을 즉시 파악할 수 없었습니다. 더욱이 우리는 **실시간**으로 사용자의 바이오 신호를 모니터링하고자 했습니다.

이러한 문제를 해결하기 위해 Amazon Kinesis Data Firehose를 도입하였습니다. [Firehose](https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/what-is-this-service.html)는 준실시간으로 데이터를 수집하고 전송할 수 있는 서비스예요. 데이터를 저장하기 위한 별도의 전처리 과정도 필요 없어서 빠르게 구축하기에 적합하다고 판단했습니다. 이를 통해 앱으로부터 일정 간격으로 바이오 신호 데이터를 전송받아 즉시 저장하고 확인할 수 있게 되었습니다.

다만, Firehose를 통해 데이터를 S3에 저장할 때 저장 디렉토리 구조를 변경하고 싶다면 Lambda를 거쳐야 해요. 이 과정에서 최소 60초의 지연 시간이 발생하기 때문에, 완전한 실시간 처리를 원한다면 적합하지 않을 수 있습니다.

<br>

---

## 데이터 전처리
<br>
<div style="display: flex; justify-content: center; align-items: center;">
  <img src="/img/article/data_pipeline/glue.png" alt="Glue" style="width: 20%; border-radius: 10px;">
</div>

데이터 전처리 과정에서 어떤 AWS 서비스를 사용할지 고민이 있었어요.

AWS에는 복잡한 설정 없이 간편하게 전처리할 수 있는 Lambda라는 선택지와, 복잡하고 대용량의 전처리도 지원하는 Glue라는 선택지도 있었습니다. 처음에는 Lambda를 이용해본 경험이 있어서 손쉽게 개발할 수 있고, 데이터의 크기도 그리 크지 않기 때문에 Lambda를 선택하려 했습니다. 그러나 마음 한편에서는 데이터의 **일관성**과 **무결성** 관점에서 무엇이 더 좋을까 하는 궁금증이 있었습니다.

Glue는 사용하기에 러닝 커브도 있고 익숙한 서비스는 아니었지만, Glue에는 **Data Catalog**라는 서비스가 존재해요. Data Catalog는 데이터 세트에 대한 메타데이터(Schema)를 저장하는 저장소입니다. 이를 이용하면 전처리된 데이터의 형식을 일관성 있게 유지할 수 있고, 추후 분석 시에도 Athena와 연계하여 SQL을 이용해 일관되게 데이터를 가져올 수 있는 시너지 효과가 있어서 Glue를 선택하였습니다.

Crawler를 이용해 원천 데이터 세트에 대한 메타데이터를 자동으로 생성하여 Data Catalog에 저장할 수도 있었지만, 우리는 좀 더 정확하고 일관된 데이터를 위해 직접 Data Catalog를 생성하였습니다.

<div style="position: relative; overflow: visible;">
  <img src="/img/article/data_pipeline/meta_data_schema.png" alt="Meta Data Schema" style="position: relative; left: 50%; transform: translateX(-45%); width: 110rem; max-width: none; border-radius: 10px;">
</div>
<!-- <div style="text-align: center;">
  <img src="/img/article/data_pipeline/meta_data_schema.png" alt="Meta Data Schema" style="width: 100%; height: auto; border-radius: 10px;">
</div> -->
<div style="text-align: center;">
  <em>&lt;Meta Data Schema&gt;</em>
</div>
<div style="position: relative; overflow: visible;">
  <img src="/img/article/data_pipeline/sensor_data_schema.png" alt="Sensor Data Schema" style="position: relative; left: 50%; transform: translateX(-45%); width: 110rem; max-width: none; border-radius: 10px;">
</div>
<!-- <div style="text-align: center;">
  <img src="/img/article/data_pipeline/sensor_data_schema.png" alt="Sensor Data Schema" style="width: 100%; height: auto; border-radius: 10px;">
</div> -->
<div style="text-align: center;">
  <em>&lt;Sensor Data Schema&gt;</em>
</div>

전처리된 데이터는 빅데이터 분야에서 많이 사용하는 **Parquet** 형식으로 저장하였습니다.

<br>

---

## 데이터 분석

<br>
<div style="display: flex; justify-content: center; align-items: center;">
  <img src="/img/article/data_pipeline/athena.png" alt="Athena" style="width: 20%; border-radius: 10px; margin: 0 1%;">
  <img src="/img/article/data_pipeline/lambda.png" alt="Lambda" style="width: 20%; border-radius: 10px; margin: 0 1%;">
</div>

데이터 분석을 위해 우리는 **Lambda**, **EC2**, 그리고 사내에 세팅된 **MLPC**까지 총 세 가지 선택지가 있었습니다. 각 옵션마다 장단점이 있었기 때문에 다음과 같은 기준을 세워 우리에게 적합한 서비스를 결정하였습니다.

1. **비용 효율성**
   - AWS를 선택한 이유 중 하나는 온디맨드 방식으로 사용한 만큼만 비용을 지불할 수 있다는 점이었습니다. 하지만 지속적으로 높은 비용이 발생한다면 우리에게 적합하지 않습니다.

2. **유지보수 및 모니터링의 용이성**
   - 데이터 파이프라인은 추후 확장성이 필요하며, 어느 단계에서 문제가 발생하는지 빠르게 파악할 수 있어야 합니다.

3. **복잡한 분석 코드의 실행 가능성**
   - 서비스가 고도화될수록 분석 코드도 복잡해질 거예요. 우리는 이러한 코드를 무리 없이 실행할 수 있는 서비스가 필요합니다.

비용 효율성에서는 MLPC는 이미 세팅되어 있어 유지비용이 거의 들지 않고, Lambda도 월 10달러 미만으로 비용이 비슷했습니다. 반면에 EC2는 사용하지 않아도 유지 기간 동안 계속 비용이 발생했습니다. 유지보수 및 모니터링의 용이성에서는 Lambda와 EC2는 AWS 서비스이므로 CloudWatch 연계가 수월하여 관리가 편리했지만, MLPC는 AWS 외부 시스템이어서 추가적인 조치가 필요했습니다. 복잡한 분석 코드의 실행 가능성에서는 Lambda는 메모리와 실행 시간에 제한이 있어 부적합했지만, MLPC와 EC2는 이러한 제약이 없어 복잡한 코드를 실행하기에 적합했습니다.

이러한 평가를 토대로 우리는 먼저 Lambda를 이용해 구축하기로 결정하였습니다. 그러나 Lambda는 최대 실행 시간이 15분으로 제한되어 있다는 단점이 있어, 추후 분석 코드가 고도화되면 EC2로 전환할 계획이에요.

<br>

---

## 데이터 저장
<br>
<div style="display: flex; justify-content: center; align-items: center;">
  <img src="/img/article/data_pipeline/s3.png" alt="S3" style="width: 20%; border-radius: 10px;">
</div>

데이터는 혼용을 막기 위해 원천 데이터, 전처리된 데이터, 분석 완료된 데이터 3가지 버킷으로 나누어 저장하였습니다. 

---

# 성과
구축한 데이터 파이프라인을 통해 다음과 같은 성과를 얻을 수 있었습니다:

1. **데이터 전송 시간 80% 단축**
   - 한 명분의 데이터 전송 시간이 평균 **5분 이상**에서 **1분 이내**로 감소하여 **80% 이상** 단축되었습니다.

2. **휴먼 에러 발생률 100% 감소**
   - 자동화를 통해 수동 작업에서 발생하던 **데이터 누락 및 중복 오류를 완전히 제거**하였습니다.

3. **데이터 관리 효율성 50% 향상**
   - 데이터별 라벨링을 통해 **데이터 검색 및 관리 시간이 절반으로 단축**되었습니다.

4. **분석 결과 조회 시간 80% 단축**
   - 분석 결과 확인 시간이 평균 **15분**에서 **3분 이내**로 감소하여 **준실시간 조회**가 가능해졌습니다.

---

# 마치며

이번에 온전히 AWS의 서비스들로만 구성하여 데이터 파이프라인을 구축해보았습니다. 단순히 데이터 전송의 자동화만을 목표로 했다면 더 빠른 시간 안에 구축할 수 있었겠지만, 데이터의 일관성과 무결성, 모니터링, 추후 확장성 등 다양한 관점을 고려하다 보니 많은 공부를 병행해야 했던 프로젝트였습니다. 아직 "완전한 실시간 데이터 전송은 가능할까?", "데이터 형식이 달라졌을 때에도 동작하는 확장성 있는 아키텍처는 무엇일까?" 하는 마음속 과제들이 남아 있지만, AWS의 다양한 서비스를 활용해 보며 클라우드 활용 능력이 성장할 수 있는 기회가 되었던 것 같습니다.

이 글이 AWS를 활용하여 데이터 파이프라인 구축을 계획 중이신 엔지니어 분들께 도움이 되길 바랍니다. 읽어주셔서 감사합니다.