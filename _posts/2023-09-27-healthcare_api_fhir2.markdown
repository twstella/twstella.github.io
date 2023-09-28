---
layout: post
title: "Google Cloud Health API-FHIR CRUD(2)"
date: 2023-09-27 22:46:23 +0900
category: Health_API
---
# FHIR 리소스 CRUD(2)

<h3>FHIR 리소스 업데이트(Update)</h3>
<div class="explain">
Update는 리소스의 전체가 업데이트가 된다는 점에서 Patch와 차이가 있다.<br>
업데이트 내용을 담고 있는 리소스와 업데이트하는 리소스들은 동일한 id를 가지고 있어야 한다.
</div>
<h4>-&nbsp;Observation 리소스 업데이트</h4>
```json
{
  "resourceType": "Observation",
  "id": "OBSERVATION_ID",
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
    "value": 120,
    "unit": "bpm"
  },
  "encounter": {
    "reference": "Encounter/ENCOUNTER_ID"
  }
}
```
<div class="explain">
<p><span class="file">request.json</span>
</p>
</div>
```console
> curl -X PUT \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @request.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir/Observation/OBSERVATION_ID"
```
<h4>-&nbsp;조건부 Patient 리소스 업데이트</h4>
<div class="explain">
<p>조건부 업데이트는 한 번에 하나의 FHIR 리소스에만 적용 가능<br>
조건을 일치하는 리소스가 1개 초과로 존재하면 <b>412 Precondition Failed</b>오류가 반환됨<br>
<b>identifier</b>를 배타적으로 사용하여 조건과 일치하는 FHIR 리소스 확인
</p>
</div>
```json
{
    "name": [
      {
        "use": "official",
        "family": "Smith123",
        "given": [
          "Darcy0000"
        ]
      }
    ],
    "gender": "female",
    "birthDate": "1970-01-01",
    "id": "PATIENT_ID",
    "identifier":[{
      "value":"104",
      "system":"my_code_system"
    }]
    ,
    "resourceType": "Patient"
}
```
<div class="explain">
<p>
<span class="file">Patient_new.json</span>
</p>
</div>
```console
> curl -X PUT \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @Patient_new.json \
"https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir/Patient?identifier=my_code_system|104"

```
<div class="explain">
<p>
<b>identifier</b>로 조건을 줌<br>
</p>
</div>

<h3>FHIR 리소스 패치(Patch)</h3>