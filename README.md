# 03 Config Data Lake by using Lake Formation and Glue

## [ 01 프로젝트 설명 ]
프로젝트 명 : Lake Formation을 이용한 권한 세분화 데이터 레이크 구축

프로젝트 인원 : 1명

프로젝트 기간 : 2023.08 ~ 

프로젝트 소개 : 
이 프로젝트는 가상의 회사가 현재 운영 중인 데이터베이스(DB), 서버 및 서비스를 운영중임을 전제로 하며 다양한 방법을 통해 수집한 데이터를 S3 버킷에 저장한 후, Lake Formation을 활용하여 세분화된 권한을 가진 데이터 레이크를 구축하는 데 초점을 두고 있습니다.
더불어, AWS Glue의 Crawler와 Catalog 기능을 활용하여 데이터를 관리하고 기초적인 ETL(Job)을 수행함으로써 데이터를 요구사항에 맞게  공할 수 있다는 것을 보여주며, Athena를 이용하여 간단한 쿼리 실행하는 프로젝트입니다.

***

## [ 02 클라이언트 상황 ]

* IDC 중 일부를 임대하여, 자사와 먼 거리에 6대 이상 운영중

* AWS RDS, S3 등의 서비스를 이미 운영하고 있음.

* 회사가 성장함에 따라 수집되는 데이터의 증가

* 기간 한정 할인 이벤트 및, 다양한 새로운 서비스 런칭을 계획하고 있음.

* 추 후 사내 데이터를 이용하여 대규모 머신러닝을 진행할 예정

***

## [ 03 요구사항 ]

* 온프레미스, 클라우드에 존재하는 데이터를 하나의 스토리지에 저장

* 데이터 관리 및 사내 요구사항에 부합하기 위해 각 ETL 과정의 결과물을 s3에 저장

* 확장성 및 탄력성 확보 및 데이터 사용에 대한 모니터링 요망

* 사내 각 팀에게 조건에 맞는 필요 데이터만 접근할 수 있도록 세분화된 엑세스 권한 부여 요망

***

## [ 04 다이어 그램 ] 
<img width="1401" alt="DataLake_diagram" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/4a2ee15c-0245-49ab-883b-f26d9326ac83">

* Ingset 영역에서 DMS, Data Sync, 다른 Kinesis 서비스 등을 사용 가능
* ETL 과정에서 Glue만 사용했지만 EMR도 고려 가능함. 또한 Glue ETL 작업을 Wrokflow를 통해 제어 및 On-demand 방식이 아닌 특정 조건에 따라 실행도 가능
* Analysis 영역에서 ML을 진행할 때 Sage Maker를 고려할 수 있으며, Athena의 결과를 QuickSight를 통해 시각화 가능


***

## [ 05 핵심 서비스 ]

### 01 Ingest

<img width="1417" alt="01_ingest" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/62b2c069-ae2d-4237-9b72-1ad758930ab5">

#### 1. Storage Gateway : Cloud 기반 스토리지와 On premise를 연결하여 데이터 동합을 제공하는 서비스.

#### 2. DB Snaphost : 특정 시점의 DB의 상태를 파일이나 이미지로 저장
* AWS Console에서 수동으로 snapshot을 bucket에 저장하거나 AWS CLI를 사용할 수 있음.

#### 3. Kinesis Firehose : 스트리밍 데이터를 Data Store나 분석 도구에 로드하기 위한 완전관리형 서비스. 
* 자동으로 Scaling되는 Serverless Service
* 다양한 데이터 포맷의 변환, 압축, 병합을 지원.
*  AWS Service(S3, Redshift, ElasticSearch)와 써드 파티, HTTP Endpoint에 데이터를 전송할 수 있다.

***

### 02 Data Lake

<img width="1409" alt="02_lake_formation" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/a7ee61ea-21ec-487e-a1c9-2e69f767b028">

#### 0. Data Lake : 분석 목적을 위해 모든 데이터를 한 곳에 모아주는 중앙 집중식 저장소

#### 1. Lake Formation : 데이터 분석 및 기계학습을 위한 데이터를 중앙에서 관리하고, 보호할 수 있는 완전 관리형 Serverless 서비스
* AWS Glue Catalog를 사용하여 중앙집중 방식으로 메타 데이터와 데이터 권한을 관리할 수 있음.
* 한 곳에 모든 데이터를 저장하지만, IAM과의 통합으로 User 수준의 보안 설정 및 Row, Column 기반의 세분화된 엑세스 컨트롤을 부여.
* AWS 서비스 이므로 CloudTrail, CloudWatch와 연동하여 모니터링 가능
* BluePrint를 이용하여 S3, RDS, NoSQL DB등 과의 손쉬운 연동 가능 

#### 2. Glue : 머신 러닝, 데이터 분석, 어플리케이션 개발을 위한 데이터를 탐색, ETL을 지원하는 Serverless 데이터 통합 서비스. 
* 중앙 메타데이터 레포지토리인 Data Catalog로 구성되어 있음. 
* Crawler를 통해 데이터 스토어를 스캔하고 스키마와 파티션 구조를 자동으로 추론함. 이 결과를 Data Catalog에 저장.
* ETL 엔진은 자동으로 Scala or Python Script를 생성하고, 사용자는 이를 수정하여 요구사항에 맞게 수정 가능

***

### 03 Analysis

<img width="1417" alt="03_analysis" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/e201886a-30ff-4026-bdac-62aa3a5d9aac">

#### 1. Athena : 저장된 데이터를 분석하기 위한 Serverless Query Service
* 표준 SQL 언어를 사용하여 파일을 Query.
* CSV, JSON, ORC, Parquet과 같은 파일 형식을 지원.

#### 2. OpenSearch : 로그 분석, 실시간 애플리케이션 모니터링, 데이터 검색 등을 쉽게 수행할 수 있는 관리형 서비스.
* Serverless가 아닌, Instance들의 Cluster임. 
* SQL 언어를 사용하지 않고, 자체 언어 존재
* DashBoard를 통해 검색 결과 시각화 

***

## [ 06 구현 과정 ]

### [ 06-1 Ingest ] 

#### 1. Storage Gateway 
* Storage Gateway를 이용하여 On Premise의 데이터를 S3로 적재하는 것은 [02_Hybrid_Data_integration_using_File_Gateway] 프로젝트에서 구현

[ Github Link ] https://github.com/heungbot/02_Hybrid_Data_integration_using_File_Gateway


#### 2. DB Snapshot
* AWS Console에서 진행하는 방법과 AWS CLI를 사용하여 Snaphost을 Bucket으로 내보내는 방법이 존재 

* 내보낼 S3 Bucket은 Snapshot과 동일한 Region에 위치해야 함

* RDS의 Snapshot을 S3 Bucket으로 내보내기 위해선 S3 bucket에 대한 권한이 필요함


#### 2-1. Snapshot Export을 위한 IAM Policy 생성


```
aws iam create-policy  --policy-name [POLICY_NAME] --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ExportPolicy",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject*",
                "s3:ListBucket",
                "s3:GetObject*",
                "s3:DeleteObject*",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::[BUCKET_NAME]",
                "arn:aws:s3:::[BUCKET_NAME]/*"
            ]
        }
    ]
}'
```
* 위 명령어에서 [BUCKET_NAME] 부분과 [POLICY_NAME]은 사용자가 정의


#### 2-2 Policy를 연결할 Role 생성 및 연결

```
aws iam create-role  --role-name [ROLE_NAME]  --assume-role-policy-document '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
            "Service": "export.rds.amazonaws.com"
          },
         "Action": "sts:AssumeRole"
       }
     ] 
   }'
```
* [ROLE_NAME]은 사용자가 정의

```
aws iam attach-role-policy  --policy-arn [POLICY_ARN]  --role-name [ROLE_NAME]
```
* [ROLE_NAME]은 위에서 사용자가 정의한 Role의 이름 입력
* [POLICY_ARN]은 2-1 명령어로 생성한 Policy의 ARN을 넣어줘야함.

-> AWS Console에서 확인 가능하며 아래의 CLI를 통해 확인 가능
  
```
aws iam list-policies --query 'Policies[?PolicyName==`[POLICY_NAME]`].Arn'
```

#### 2-3 DB Snapshot 추출
* 작업을 수행하기 위해서 위에서 생성한 IAM Role ARN과 전송 중 암호화를 위한 KMS KEY ARN을 인자로 넣어줘야 함

```
aws rds start-export-task \
    --export-task-identifier [SNAPSHOT_EXTRACT_TASK_IDENTIFIER] \
    --source-arn arn:aws:rds:[AWS_REGION]:[ACCOUNT_ID]:snapshot:[SNAPSHOT_NAME] \
    --s3-bucket-name [BUCKET_NAME] \
    --iam-role-arn [ROLE_ARN] \
    --kms-key-id [KMS_KEY_ARN] \
    -- S3-prefix [BUCKET_PREFIX/]
```
* [SNAPSHOT_EXTRACT_TASK_IDENTIFIER], [SNAPSHOT_NAME]은 사용자가 정의. 나머지 인자들은 실제 Value로 넣어줘야 함.
* [BUCKET_PERFIX] 옵션은 선택이며, 나머지 인자는 필수적
* Snapshot을 추출하는 데는 시간이 소요되므로 아래의 명령어를 통해 현 진행 상황을 파악할 수 있음.

```
aws rds describe-export-tasks
```

<img width="615" alt="스크린샷 2023-08-28 오후 9 00 33" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/f367b867-2172-49a1-b333-951db1836caf">

<img width="627" alt="스크린샷 2023-08-28 오후 8 58 56" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/a2177adf-5ae3-4da9-b8cc-92a3ea64de48">

* parquet format으로 S3 bucket에 적재 성공

***

#### 3. Kinesis Firehose

#### 3-1 Kinesis Firehose 생성

<img width="665" alt="kinesis_create" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/21a5f539-1168-450f-9fb4-dbcdb8760754">

* Kinesis을 테스트하기 위해 Direct PUT 방식을 선택(Kinesis Stream을 운영하고 있다면 Kinesis Stream 선택)
* firehose의 TEST Data를 전송받기 위해 Data Lake로 사용될 S3 Bucket을 설정하고, 적절한 Prefix 입력

<img width="1143" alt="firehose_status_real" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/3390fde1-6118-498b-9e46-90b72579edb3">

#### 3-2 DEMO Data 전송

<img width="673" alt="스크린샷 2023-08-29 오전 12 01 45" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/d003c5cc-4c4a-4a86-a568-a31efbae94da">

* Kinesis Demo Data를 설정한 S3 Bucket으로 전송
* 데이터 전송 후 몇 분(5분 내외) 정도 기다린 후 S3 Bucket 확인


<img width="766" alt="object_in_bucket" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/08a1fcfa-de52-43c6-b8c3-74f64e702ad5">

<img width="394" alt="object_info" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/c3a51756-29ad-4837-bed8-33c6366f5bba">

* 설정한 Prefix로 DEMO Data 전송된 것을 확인할 수 있음
* Kinesis firehose가 s3 bucket으로 데이터를 전송할 때, "yyyy/mm/dd/hh" (UTC) prefix를 추가함을 확인

*** 

### [ 06-2 Lake Formation ]

#### 1. Lake Formation 셋업
<img width="1178" alt="01_LakeFormation_AdminUser_Location" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/1971fafd-f49a-4629-a94f-dd790c280a6d">

* Lake Formation에 대한 Admin User를 등록해야 함. 이는 Root User가 아닌, IAM User를 등록

<img width="662" alt="02_Create_First_Database" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/a4fbb0df-40c2-4b48-97a6-945e81a893ad">

* Data Lake의 Location을 지정. 
* 모든 Raw Data가 담기는 Bucket의 특정 Prefix 입력

#### 2. Data Lake Crawler 생성

<img width="572" alt="03_Crawler_Config" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/ef233efb-7b56-46e3-b653-0867e6eae698">

* 현재 로그인된 User의 Bucket에 대해 Crawler를 실행시킬 것이므로 Bucket Path 입력
* Bucket Path의 내부 경로를 모두 크롤링 하기 위해 "Crawl all-sub-folders" 선택

<img width="986" alt="04_Crawler_Role" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/f0bcda10-49c3-4997-aea8-f5539384638b">

* Crawler에 대한 권한 생성 후 선택
* Lake Formation의 Credential을 사용하여 Crawling 진행하고자 하므로 옵션 선택

#### 3. Crawler의 권한 승인

<img width="721" alt="06_Crawler_Permission_Grant" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/aa0fa421-85d0-4d62-8876-f4099e72d3bc">

<img width="711" alt="05_Crawler_Role_Grant" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/31358336-871c-43f7-8608-25a510af0c04">

* Crawler의 Role에 대한 권한 승인
* LF-Tags를 사용하지 않으므로, 승인할 database와 table 선택
* table을 대상으로 어떤 액세스 권한을 줄 것인지 선택

<img width="1164" alt="07_Crawler_run" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/9304b7e4-d1d6-4d4e-a64f-3bf76944130a">

* 생성을 완료했다면, Crawler Run

#### 4. Crawler 실행 및 카탈로그 확인

<img width="872" alt="08_Crawler_Create_Table" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/106066c2-ef09-4ebd-84f6-9419e925271b">

* Crawler의 실행이 끝나면, Catalog에 Schema, Partiion, Index에 대한 정보가 저장됨을 확인

#### 5. Glue ETL Job

<img width="1160" alt="01_Ingest" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/6448e61a-9cc4-4464-9411-825c821ea108">

* ETL을 진행할 Source Data 선택 과정
* Crawler로 생성된 Catalog Table을 선택

 
<img width="1038" alt="02_transform" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/0267e62d-9b5f-40b5-b100-187d07d3926a">

* Schema 변환과정
* 특정 key의 이름을 변경하거나, Drop 시킬 수 있음.

 
<img width="1290" alt="03_target" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/c9b884e9-04c7-4ffd-b584-980c7093d77f">

* 변환 과정을 거친 Data를 적재할 곳을 선택
* S3 bucket을 선택했으며 저장할 형식은 "Parquet"으로 선정
* 실제 운영되는 데이터에 적용하는 것이었다면 압축을 진행

 
<img width="506" alt="04_etl_job_script" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/8e0eac13-576e-42a2-8336-744541c9a819">

*  변환 ETL 과정에 대한 Script. Visual ETL 보다 세부적인 조정 가능
* 이는 수정하거나 "Job Detail"을 통해 작성된 Script 사용 가능

 
<img width="689" alt="04_job_detail_1" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/dfa4323e-5e0e-4369-b1e6-88ed38a1da77">

* Glue Version과 Type, Worker Node의 사양을 정할 수 있음.
* Language는 Python, Scala를 지원함
* ETL Job에 대한 IAM Role을 부여해야 함. 이 role은 source가 되는 Glue의 권한과 target인 s3에 대한 권한을 가져야 함.

 
```
{
    "Version": "2022-08-28",
    "Statement": [
        {
            "Sid": "AllowCatalogTableETL",
            "Effect": "Allow",
            "Action": [
                "glue:GetTable",
                "glue:GetPartition",
                "glue:GetDatabase",
                "glue:GetDatabaseTableVersions",
                "glue:UpdateTable"
            ],
            "Resource": [
                "arn:aws:glue:ap-northeast-2:[ACCOUNT_ID]:catalog/*",
                "arn:aws:glue:ap-northeast-2:[ACCOUNT_ID]:database/*",
                "arn:aws:glue:ap-northeast-2:[ACCOUNT_ID]:table/*",
                "arn:aws:glue:ap-northeast-2:[ACCOUNT_ID]:partition/*"
            ]
        },
        {
            "Sid": "AllowTargetBucketAccess",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::[TARGET_BUCKET_NAME]/*"
            ]
        }
    ]
}
```
* 세분화된 권한을 위해 와일드 카드(*) 대신, 각각의 리소스에 대한 이름을 명시해 줘야함.

 
<img width="646" alt="04_job_detail_2" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/402facde-3849-4017-8f1c-1239fae8d5cb">

* 사용된 script가 저장될 경로 및 작업이 이루어질 Temp Directory 그리고 Spark Log가 적재될 Bucket PATH를 설정
* Script에서 사용된 라이브러리에 대한 zip 파일이 존재한다면, S3 내부의 File Path 설정
* 설정 완료 후 ETL Job Run

 
<img width="1157" alt="05_etl_job_monitoring" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/f494d327-42b3-4e80-bdb4-f5273fde4da8">

* ETL 작업이 성공적으로 수행됨

 
<img width="721" alt="06_Job_Monitoring_Result_Bucket" src="https://github.com/heungbot/03_Building_Data_Lake_Using_Lake_Formation/assets/97264115/7f62235c-d6c0-42f6-a1b9-5c933870ae7a">

* ETL Job의 Target Bucket 
* Parquet 타입으로 데이터 형식이 바뀜을 확인할 수 있음







