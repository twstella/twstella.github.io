---
layout: post
title: "Google Cloud Health API-시작하기"
date: 2023-09-26 22:46:23 +0900
category: Health_API
---
# 시작하기 전에
<ol>
<li>Google Cloud 콘솔에서 프로젝트 생성/선택
<img class="picture"  src='{{ "public/img/console.png" | relative_url }}' alt='relative'/></li>
<li>
Google Healthcare API 사용 설정
<img class="picture"  src='{{ "public/img/api_valid.png" | relative_url }}' alt='relative'/><br>
<div class="explain">API 및 서비스 > API 및 서비스 사용 설정 > Cloud Healthcare API</div>
</li>
<li>Cloud Shell 실행 후 결과 확인</li>
</ol>
```
> gcloud auth application-default print-access-token
``` 

# 데이터셋 생성
<ol>
<li>Google Cloud 콘솔에서 Healthcare 검색
<img class="picture"  src='{{ "public/img/console_health.png" | relative_url }}' alt='relative'/>
</li>
<li>데이터 세트 만들기 클릭<br>
<img class="picture"  src='{{ "public/img/healthcare_browser.png" | relative_url }}' alt='relative'/><br>
<div class="explain">정보 입력 후 만들기</div>
</li>
<li>
데이터 스토어 만들기<br>
<img class="picture"  src='{{ "public/img/make_dataset1.png" | relative_url }}' alt='relative'/><br>
<div class="explain">2에서 만든 데이터 셋 클릭 > 데이터 스토어 만들기</div><br>
<img class="picture"  src='{{ "public/img/make_dataset2.png" | relative_url }}' alt='relative'/><br>
<div class="explain">
정보 입력 후 만들기<br>
브라우저 > 데이터 셋 선택 > 데이터 스토어 클릭 > 데이터 저장소 세부 정보나<br>
FHIR 뷰어 > FHIR 스토어 선택해서 Path 확인 가능
</div>
</li>
</ol>

# gcloud 사용하기
<div class="explain">
<p><a href="https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe?hl=ko
">gcloud CLI</a> 다운로드 및 실행
</p>
</div>
<table>
<tr>
<td><img  src='{{ "public/img/gcloud1.png" | relative_url }}' alt='relative'></td>
<td><img  src='{{ "public/img/gcloud2.png" | relative_url }}' alt='relative'></td>
<td><img  src='{{ "public/img/gcloud3.png" | relative_url }}' alt='relative'></td>
</tr>
<tr>
<td><img  src='{{ "public/img/gcloud4.png" | relative_url }}' alt='relative'></td>
<td><img  src='{{ "public/img/gcloud6.png" | relative_url }}' alt='relative'></td>
<td><img  src='{{ "public/img/gcloud7.png" | relative_url }}' alt='relative'></td>
</tr>
</table>
<div class="explain">
시간이 걸리는 작업이라 인내심을 가져야함
</div>
<img  src='{{ "public/img/gcloud8.png" | relative_url }}' alt='relative'>
<div class="explain">
cmd창에 위와 같이 뜸<br>
Y를 입력하면 인터넷 브라우저 창에 아래와 같은 창이 뜸<br>
허용을 클릭
</div>
<img  src='{{ "public/img/gcloud9.png" | relative_url }}' alt='relative'>
<img  src='{{ "public/img/gcloud10.png" | relative_url }}' alt='relative'>
<div class="explain">
cmd창에 위와 같이 뜸<br>
작업할 프로젝트를 선택하면 됨
Healcare API를 활성하고 싶으면 다음을 cmd창에 입력
</div>
```console
> gcloud services enable healthcare.googleapis.com 
```
<div class="explain">
Healthcare API를 사용하기 위해서는 다음과 같이 권한/역할을 부여해줘야함.
</div>
```console
> gcloud projects add-iam-policy-binding PROJECT_ID --member="user:EMAIL_ADDRESS" 
 --role=roles/healthcare.datasetAdmin 
> gcloud projects add-iam-policy-binding PROJECT_ID --member="user:EMAIL_ADDRESS" 
 --role=roles/healthcare.fhirStoreAdmin
```
<div class="explain">
아래의 명령어를 입력했을 때 FHIR 리소스를 가져와서 저장이되면 성공한 것임
</div>
```console
> gcloud healthcare fhir-stores import gcs my-fhir-store 
  --project=PROJECT_ID 
  --dataset=my-dataset 
  --location=my-fhir-country 
  --gcs-uri=gs://gcp-public-data--synthea-fhir-data-10-patients/fhir_r4_ndjson/*.ndjson 
  --content-structure=RESOURCE 
```
<img  src='{{ "public/img/gcloud11.png" | relative_url }}' alt='relative'>
<div class="explain">
아래의 명령어를 입력했을 때 나온 결과를 $(gcloud auth application-default print-access-token) 대신 사용하면<br> 로컬에서 curl 명령어로 통해 리소스 관리 가능
</div>
```console
> gcloud auth print-access-token
```