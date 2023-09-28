---
layout: post
title: "Google Cloud Health API-FHIR CRUD"
date: 2023-09-26 22:46:23 +0900
category: Health_API
---
# FHIR 리소스 CRUD

<h3>Patient 리소스 생성하기(Create)</h3>
<h4>-&nbsp;<a href='{{ "public/file/Patient.json" | relative_url }}' download>Patient.json</a> 파일 다운로드</h4>
```console
> curl -O https://cloud.google.com/healthcare-api/docs/resources/Patient.json
```
<h4>-&nbsp;Patient.json을 Google Cloud FHIR 저장소에 업로드</h4>
```console
> curl -X POST  -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \   
-H "Content-Type: application/fhir+json; charset=utf-8" --data @Patient.json \
"https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store/fhir/Patient" 
```
<div class="explain">
<p><b style="color:red">projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store</b>는 사용하는 FHIR 데이터 저장소 Path를 입력<br>
ex) projects/rapid-arbor-00000/locations/asia-northeast3/datasets/Fhir_test/fhirStores/FHIR_TEST_1
</p>
</div>
<img class="picture" src='{{ "public/img/create.png" | relative_url }}' alt='relative'><br>
<div class="explain">
<p> Patient의 id가 부여되고 부여된 <span style="color:red">Patient ID</span>가 응답에 포함됨
</p>
</div>
<h3>Patient 리소스 검색하기(Read)</h3>
<h4>-&nbsp;family name이 Smith인 리소스 검색</h4>
```console
> curl -X GET \ 
  -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \  
  "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/datasets/my-dataset/fhirStores/my-fhir-store/fhir/Patient?family:exact=Smith" 
```
<div class="explain">
<p>family name이 Smith인 Patient 리소스가 Json 형태로 응답으로 리턴됨</p>
</div>
<img class="picture" src='{{ "public/img/read.png" | relative_url }}' alt='relative'><br>

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
<p>Encounter.json<br>
Patient 리소스를 생성할 때 부여된 <span style="color:red">Patient ID</span>를 명시<br>
</p>
</div>
```console
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/fhir+json" \
    -d @request.json \
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
<p>Observation.json<br>
Patient 리소스와 Encounter 리소스를 생성할 때 부여된 <span style="color:red">Patient ID, Encounter ID</span>를 명시<br>
</p>
</div>
```console
curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/fhir+json" \
    -d @request.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir/Observation"
```
<div class="explain">
<p>
<span style="color:red">Observation ID</span>도 부여됨
</p>
</div>