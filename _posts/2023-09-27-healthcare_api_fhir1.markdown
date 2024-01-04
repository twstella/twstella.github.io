---
layout: post
title: "Google Cloud Healthcare API-FHIR CRUD(1)"
date: 2023-09-27 22:46:23 +0900
category: Healthcare_API

---
# FHIR 리소스 CRUD(1)

<h3>FHIR 리소스 생성하기(Create)</h3>
<h4>-&nbsp;<a href='{{ "public/file/Patient.json" | relative_url }}' download>Patient.json</a> 파일 다운로드</h4>
```console
> curl -O https://cloud.google.com/healthcare-api/docs/resources/Patient.json
```
```json
{
  "name": [
    {
      "use": "official",
      "family": "Smith",
      "given": [
        "Darcy"
      ]
    }
  ],
  "gender": "female",
  "birthDate": "1970-01-01",
  "resourceType": "Patient",
  "identifier":[{
    "value":"104",
    "system":"my_code_system"
  }]
}
```
<div class="explain">
<p><span class="file">Patient.json</span></p>
</div>
<h4>-&nbsp;Patient.json을 Google Cloud FHIR 저장소에 업로드</h4>
```console
> curl -X POST  -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \   
-H "Content-Type: application/fhir+json; charset=utf-8" --data @file_name.json \
"https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store/fhir/Patient" 
```
<div class="explain">
<p><b style="color:red">projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store</b>는 사용하는 FHIR 데이터 저장소 Path를 입력<br>
<span class="example">ex) projects/rapid-arbor-00000/locations/asia-northeast3/datasets/Fhir_test/fhirStores/FHIR_TEST_1</span><br>
<b style="color:red">--data @file_name.json</b>에 파일 이름을 입력<br>
</p>
</div>
<img class="picture" src='{{ "public/img/create.png" | relative_url }}' alt='relative'><br>
<div class="explain">
<p> Patient의 id가 부여되고 부여된 <span style="color:red">Patient ID</span>가 응답에 포함됨
</p>
</div>
<h4>-&nbsp;Encounter.json을 Google Cloud FHIR 저장소에 업로드</h4>
```json
{
  "status": "finished",
  "class": {
    "system": "http://hl7.org/fhir/v3/ActCode",
    "code": "IMP",
    "display": "inpatient encounter"
  },
  "reasonCode": [
    {
      "text": "The patient had an abnormal heart rate. She was concerned about this."
    }
  ],
  "subject": {
    "reference": "Patient/PATIENT_ID"
  },
  "resourceType": "Encounter"
}
```
<div class="explain">
<p><span class="file">Encounter.json</span><br>
Patient 리소스를 생성할 때 부여된 <span style="color:red">Patient ID</span>를 명시<br>
</p>
</div>
```console
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/fhir+json" \
    -d @Encounter.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir/Encounter"
```
<div class="explain">
<p>
<span style="color:red">Encounter ID</span>도 부여됨
</p>
</div>
<h4>-&nbsp;Observation.json을 Google Cloud FHIR 저장소에 업로드</h4>
```json
{
  "resourceType": "Observation",
  "status": "final",
  "subject": {
    "reference": "Patient/PATIENT_ID"
  },
  "effectiveDateTime": "2020-01-01T00:00:00+00:00",
  "identifier": [
    {
      "system": "my-code-system",
      "value": "ABC-12345"
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "8867-4",
        "display": "Heart rate"
      }
    ]
  },
  "valueQuantity": {
    "value": 80,
    "unit": "bpm"
  },
  "encounter": {
    "reference": "Encounter/ENCOUNTER_ID"
  }
}
```
<div class="explain">
<p><span class="file">Observation.json</span><br>
Patient 리소스와 Encounter 리소스를 생성할 때 부여된 <span style="color:red">Patient ID, Encounter ID</span>를 명시<br>
</p>
</div>
```console
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/fhir+json" \
    -d @Observation.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir/Observation"
```
<div class="explain">
<p>
<span style="color:red">Observation ID</span>도 부여됨
</p>
</div>
<h4>-&nbsp;조건부 Patient 리소스 생성</h4>
```console
> curl -X POST \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/fhir+json" \
  -H "If-None-Exist: FHIR_SEARCH_QUERY" \
  -d @Patient.json \
  "https://healthcare.googleapis.com/v1/projects/rapid-arbor-395501/locations/asia-northeast3/datasets/Fhir_test/fhirStores/FHIR_TEST_1/fhir/Patient"
```
<div class="explain">
<p>
<b style="color:red">FHIR_SEARCH_QUERY</b>에 조건을 입력<br>
<span class="example">ex)If-None-Exist:identifier=my-code-system|104</span><br>
조건을 만족하는 리소스가 이미 존재하면 새로운 FHIR 리소스가 생성되지 않음<br>
Cloud Healthcare API v1에서는 <b>identifier</b> 검색 매개변수를 배타적으로 사용하기 때문에 identifier로 조건을 줘야하는 것으로 보임
</p>
</div>

<h3>FHIR 리소스 검색하기(Read)</h3>
<h4>-&nbsp;family name이 Smith인 Patient 리소스 검색</h4>
```console
> curl -X GET \ 
  -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \  
  "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store/fhir/Patient?family:exact=Smith" 
```
<div class="explain">
<p>family name이 Smith인 Patient 리소스가 Json 형태로 응답으로 리턴됨</p>
</div>
<img class="picture" src='{{ "public/img/read.png" | relative_url }}' alt='relative'><br>
