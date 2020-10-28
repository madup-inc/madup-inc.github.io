---
layout: post
title:  "Amazon Athena를 통해 '데일리 광고 성과 리포트' 자동화하기" 
author: Wendy
categories: [ Tech ]
image: assets/images/thumbnail/amazon-athena.jpg
featured: true
excerpt: Amazon Athena를 도입하게 된 과정과 간단한 사용법에 대하여
---

### **애드테크 - 데이터 = 0**
​
앞선 애드테크에 대한 포스팅에서 알 수 있듯이, 애드테크의 근간은 '데이터'입니다. 때문에 애드테크 회사인 매드업은 현재 여러가지 광고 매체의 광고 성과 데이터를 수집하고 있습니다. 
​
이러한 데이터는 시시때때로 광고 운영 인사이트를 도출하기 위한 '애드혹 분석(Ad-hoc Analysis)'에도 활용되기도 하고, 스태틱하고 정기적으로 진행되는 '광고 성과 리포팅'에도 활용됩니다. 오늘은 후자와 관련하여 쿼리 기반 분석도구인 Amazon Athena를 AE들의(어쩌면 모든 마케터들에게) 숙명이라고 할 수 있는 '데일리 성과 리포트' 추출에 활용하고 있는 이야기를 해볼까 합니다.
​
앞서 말씀드렸다시피 매드업에서는 페이스북, 구글 등 여러 매체에서 제공하는 API를 통해 광고 데이터를 수집하고 있습니다. 매드업은 수집에 걸리는 시간과 데이터를 처리하는 복잡성을 단축하기 위해 JSON, CSV, TSV 등 API를 통해 전달되는 데이터 형식 그대로 AWS의 스토리지 서비스인 Amazon S3에 저장하고 있습니다. 
​
​
<img style="display:block;margin:0 auto;" src="../assets/images/amazon-athena/2.jpg">
​
### **그래서 데일리 리포트는?**
​
맨 처음에는 데이터 분석가들에게 가장 친숙한 파이썬 라이브러리 중 하나인 'Pandas'를 활용하여 리포트를 생성했습니다. Pandas의 용도가 데이터 조작과 분석인 만큼, 데이터를 전처리하거나 여러 데이터를 조인하여 리포트를 만드는 데도 전혀 문제가 없었습니다. 데이터 정제 요구사항을 취합하여 코드로 구현하였고, AWS Lambda를 통해 데일리로 리포트가 생성 되었습니다. 생성된 광고주별 데일리 성과 리포트는 AE들에게 메일로 전달되었죠. 이러한 과정을 통해 이전에는 매체 대시보드에서 데이터를 직접 다운로드 받아서, 엑셀로 로드하고 리포트를 만들던 반복적인 작업 시간의 상당 부분이 단축되었습니다. 
​
하지만 Pandas를 통한 리포트 생성은 개발적인 측면에서는 한계점이 있었습니다. 코드 기반의 리포트 생성 자체가 개발한 사람에게 의존도가 높다는 문제가 있기 때문입니다. 수정 요청이 잦은 업무임에도 불구하고 개발팀 내에는 Pandas를 활용해본 사람이 없어 담당자(나..?)만이 새로운 요청 사항을 반영할 수 있었습니다. 또한 개발한 사람의 역량에 따라 코드의 성능 자체가 좌지우지 될 수 있다는 문제도 있었죠. 
​
​
<img style="display:block;margin:0 auto;" src="../assets/images/amazon-athena/3.jpg">
​
### **Pandas를 벗어나자!**
​
이러한 단점을 극복하기 위해 저희는 한발을 더 내딛어야 했습니다! 그리고 Amazon Athena를 통해 리포트를 만들기로 결정 했습니다. 고려한 사항들은 다음과 같습니다.
​
​
​
1. 수집된 원본 데이터는 S3에 다양한 형식으로 저장되어있다. 
​
   => Data Lake의 특성을 활용할 수 있어야 한다.
​
2. 수집하는 데이터의 컬럼이 쉽게 변동될 수 있다. 
​
   => Data Lake의 특성을 활용할 수 있어야 한다.
​
3. 리포트를 수정해야 하는 일이 잦다. 
​
   => SQL처럼 보편적인 언어이면 좋겠다.
​
4. 요구사항에 따라 데이터를 처리하는 복잡도가 올라갈 수 있다. 
​
   => SQL처럼 누가 개발하더라도 성능이 어느정도 보장된 언어이면 좋겠다.
​
 
​
Amazon Athena는 표준 SQL을 사용해 Amazon S3에 저장된 데이터를 간편하게 분석할 수 있는 서비스입니다. S3에 있는 데이터의 스키마를 정의하고 테이블을 만들어준 후에 원하는 쿼리를 날리면 바로 결과가 떨어지는 아주 간편한 사용법으로 동작합니다. 특히 빅데이터 쿼리 엔진인 Presto 기반의 서비스로, 대용량의 데이터를 조인하고 복잡한 분석에도 굉장히 빠른 속도를 지닌 장점을 가지고 있습니다. Tableau 등 BI 툴에서도 Athena를 쉽게 연동 할 수 있어 S3 데이터를 쉽게 조회할 수 있습니다.
​
이제 저희는 S3에 저장된 데이터들을 Athena를 통해 리포트로 만들고 있습니다. 광고주별, 매체별로 필요한 쿼리를 다양하게 작성하고, 쿼리를 실행해주는 Lambda를 통해 리포트 데이터를 데일리로 추출하고 있는데요. SQL로는 구현하기 힘든 데이터 파싱이 필요한 경우에는 python을 통해 데이터를 처리하고 다시 S3에 저장하여 이를 활용하고 있습니다. Athena는 사용 방법이 심플하고, SQL을 안다면 누구나 접근하기 쉽다는 장점이 있습니다. 그럼 Athena를 시작하는 방법을 간단하게 알아볼까요?
​
​
<img style="display:block;margin:0 auto;" src="../assets/images/amazon-athena/4.jpg">
​
### **Amazon Athena 시작하기**
​
먼저, Athena를 사용하려면 Amazon S3에 데이터를 저장해야 합니다. 
​
Athena를 사용하시려면 S3에 저장할 때 부터 '파티셔닝' 기능을 염두하고 경로를 지정하여 저장하는 것이 좋습니다. 파티셔닝을 사용하면 쿼리의 WHERE 절에서 스캔할 데이터를 한정지을 수 있습니다. Athena는 쿼리를 수행할때 스캔한 데이터의 양에 따라 과금이 되기 때문에, 성능을 높이고 비용을 절감하기 위한 필수 기능이라고 할 수 있습니다. Athena는 데이터 파티셔닝에 Hive를 활용한다고 합니다.
​
데일리 리포트의 경우 일자를 기준으로 쿼리를 하는 경우가 잦기 때문에 year, month, day를 파티션 키로 지정해보도록 합시다. 아래와 같은 방식으로 S3 경로에 키를 지정하면 설정된 key가 열 이름이 되고, 지정한 key를 바탕으로 분할된 데이터에 쿼리를 사용할 수 있습니다.
​
s3://madup-bucket/advertiser_0001/google_ads/**year=2020/month=04/day=01**
​
만약 저장할 때부터 위와 같은 규칙을 사용하지 않았다면, `ALTER TABLE ADD PARTITION`을 사용하여 별도로 각 파티션을 추가해야합니다. 
​
​
​
그 다음, 스키마를 정의해줘야 하는데요.  JSON 형식의 데이터로 예시를 들어보겠습니다. S3에 다음과 같은 JSON 데이터가 들어있다고 해봅시다.
​
```json
{
​
  "customer_id":"madup_customer_0001",
  "campaign":"display_campaign_0001",
  "impressions":11680,
  "clicks":72,
  "cost":8145.456242,
  "date":"2020-04-01"
​
}
```
​
​
​
이러한 데이터를 advertiser_0001_google_ads라는 이름의 테이블로 만들고 싶다면, 아래와 같은 DDL을 사용해서 스키마를 지정해주면 됩니다. 지원하는 데이터 타입은 [이 페이지](https://docs.aws.amazon.com/ko_kr/athena/latest/ug/data-types.html)를 참고하세요.
​
```SQL
CREATE EXTERNAL TABLE IF NOT EXISTS madup_db.advertiser_0001_google_ads (
				 customer_id string,
  			 campaign string,
         impressions int,
         clicks int,
         cost double,
         date date 
) PARTITIONED BY (
         year int,
         month int,
         day int
) 
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' LOCATION 's3://madup-bucket/advertiser_0001/google_ads/' TBLPROPERTIES ('has_encrypted_data'='false')
```
​
여기서, 파티션에 대한 정보를 PARTITIONED BY 이후에 포함해줘야 LOCATION에 지정해놓은 경로 하위의 폴더를 쿼리할 수 있으니 주의하세요. 
​
만일 데이터를 저장할 때, S3 경로에 key를 명시해놓지 않았다면 테이블 생성 후에 아래와 같이 수동으로 파티션을 만들어 줘야 합니다.
​
```SQL
ALTER TABLE madup_db.advertiser_0001_google_ads ADD PARTITION (year='2020',month='04',day='01') LOCATION 's3://madup-bucket/advertiser_0001/google_ads/2020/04/01'
```
​
​
​
파티션 정보를 DDL에 포함하여 실행했다고 가정하고, 쿼리를 날려볼까요? 
​
```SQL
SELECT *
FROM madup_db.advertiser_0001_google_ads
WHERE year=2020 AND month=04 AND day=01
```
​
​
​
*아무런 결과도 나오지 않습니다...* DDL을 실행한 후 파티션에 데이터가 로드되지 않아서인데요. `MSCK REPAIR TABLE madup_db.advertiser_0001_google_ads` 라는 쿼리를 실행한 후 다시 시도해보시면, 쿼리가 잘 수행되는 것을 확인 할 수 있습니다. 
​
Athena의 쿼리 결과는 기본적으로 S3 버킷에 저장됩니다. 버킷의 위치를 지정해놓지 않으면 AWS가 자동으로 생성한 버킷에 저장되니 참고하세요. 
​
더 자세하고 심도깊은 사용법은 [공식 사용 설명서](https://docs.aws.amazon.com/ko_kr/athena/latest/ug/what-is.html)를 참고해보시면 좋을 것 같습니다.
​
​
​
이번 포스팅에서는 Athena를 도입하게 된 과정과 간단한 사용법에 대해 알아봤는데요. Amazon Athena는 S3를 데이터 레이크로 활용하고 있다면 상당히 유용한 분석도구라고 생각합니다. 다음에 기회가 된다면 더 깊숙한 Athena 사용 에피소드를 포스팅 해보도록 하겠습니다.
​
<br/>

[매드업이 잘하는 AD-TECH 읽으러 가기][ad-tech]
[매드업 채용 바로가기][madup]

[madup]: <https://www.notion.so/maduphr/f5cafd7a9ab645889a843dcb2bc8605e>
[ad-tech]: <https://tech.madup.com/madup-adtech-1/>
