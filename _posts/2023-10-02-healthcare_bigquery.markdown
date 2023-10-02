---
layout: post
title: "Google Cloud Healthcare API- BigQuery"
date: 2023-10-02 15:36:23 +0900
categories:  Health_API
---
# Google Cloud BigQuery로 Healthcare data 분석하기
<ol>
<li>Google Cloud BigQuery API 사용 설정<br>
<div class="explain">API 및 서비스 > API 및 서비스 사용 설정 > BigQuery API</div>
</li>
<li>BigQuery 권한 허용</li>
</ol>
```console
> gcloud projects add-iam-policy-binding PROJECT_ID \   
    --member=serviceAccount:service-PROJECT_NUMBER@gcp-sa-healthcare.iam.gserviceaccount.com \  
    --role=roles/bigquery.dataEditor 
```
<ol start="3">
<li>Google Cloud Console에서 BigQuery 검색 후 클릭</li>
<li>BigQuery 데이터셋 만들기<br><img class="picture"  src='{{ "public/img/bigquery1.png" | relative_url }}' alt='relative'/><br>
<div class="explain">BigQuery > SQL 작업공간 > 프로젝트 선택 > 데이터 세트 만들기</div></li>
<li> 정보 입력 후 만들기<br>
<img class="picture"  src='{{ "public/img/bigquery2.png" | relative_url }}' alt='relative'/><br>
</li>
<li>Healthcare API FHIR 스토리지에서 BigQuery로 데이터 내보내기<br>
<img class="picture"  src='{{ "public/img/bigquery3.png" | relative_url }}' alt='relative'/><br>
<div class="explain">Healthcare API > 브라우저 > 데이터 저장소 선택 > 작업 > 내보내기</div>
<img class="picture"  src='{{ "public/img/bigquery5.png" | relative_url }}' alt='relative'/><br>
<div class="explain">대상 위치를 BigQuery 테이블로 선택 > 데이터 세트 선택 > 만들기</div>
</li>
<li>BigQuery 창에서 데이터 확인<br>
<img class="picture"  src='{{ "public/img/bigquery6.png" | relative_url }}' alt='relative'/>
</li>
<li>BigQuery 창에서 쿼리 생성 및 실행 <br>
<img class="picture" src='{{"public/img/bigquery7.png" | relative_url }}' alt='relative'/>
<div class="explain">쿼리를 실행하면 아래에 결과가 출력됨</div>
</li>
</ol>
```sql
select * from `PROJECT_NAME.DATA_SET_NAME.TABLE_NAME`;
```
<span class="example">select * from rapid-arbor-000000.fhir_example.Patient</span>
<div class="explain">'' 대신 ``이 사용됨</div>
```sql
SELECT address[SAFE_OFFSET(0)].city,birthDate, name[SAFE_OFFSET(0)].family, name[SAFE_OFFSET(0)].given 
FROM `rapid-arbor-395501.fhir_example.Patient`;
```
<div class="explain">Array 타입인 열에 대해서는 특별한 처리가 필요한 것으로 보임</div>
