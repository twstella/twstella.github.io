---
layout: post
title: "Google Cloud Healthcare API-Bundle로 리소스 관리하기"
date: 2023-09-28 16:07:00 +0900
category: Healthcare_API
---
# Bundle로 리소스 관리하기
<h3>FHIR Bundle</h3>
<div class="explain">
FHIR Bundle을 사용해서 리소스에 대한 생성, 업데이트, 삭제 등의 작업을 한꺼번에 수행 할 수 있다.<br>
번들의 유형에 따라 번들의 작업 수행 방법이 결정된다.
<ul>
<li><b>batch</b>: 작업을 여러 독립적인 요청으로 수행</li>
<li><b>transaction</b>: 작업을 서로 의존되는 여러 요청으로 실행</li>
</ul>
번들 유형이 <b>batch</b>인 경우 작업이 실패하면 Cloud Healthcare API가 번들에서 나머지 작업을 수행</br>
번들의 유형이 <b>transaction</b>이면 작업이 실패하면 Cloud Healthcare API가 작업 실행을 중지하고 트랜잭션을 롤백함
</div>
<h4>-&nbsp;Bundle로 한번의 요청으로 리소스를 생성하고 삭제</h4>
```json
{
  "resourceType": "Bundle",
  "id": "bundle-transaction",
  "meta": {
    "lastUpdated": "2018-03-11T11:22:16Z"
  },
  "type": "transaction",
  "entry": [
    {
      "resource": {
        "resourceType": "Patient",
        "name": [
          {
            "family": "Smith",
            "given": [
              "Darcy"
            ]
          }
        ],
        "gender": "female",
        "address": [
          {
            "line": [
              "123 Main St."
            ],
            "city": "Anycity",
            "state": "CA",
            "postalCode": "12345"
          }
        ]
      },
      "request": {
        "method": "POST",
        "url": "Patient"
      }
    },
    {
      "request": {
        "method": "DELETE",
        "url": "Patient/1234567890"
      }
    }
  ]
}
```
<div class="explain">
<p><span class="file">Bundle.json</span>
</p>
</div>
```console
curl -X POST \
    -H "Content-Type: application/fhir+json; charset=utf-8" \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    --data @Bundle.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir"
```

<h4>-&nbsp;Bundle로 Patch</h4>
```json
{
  "type": "transaction",
  "resourceType": "Bundle",
  "entry": [
    {
      "request": {
        "method": "PATCH",
        "url": "Patient/PATIENT_ID"
      },
      "resource": {
        "resourceType": "Binary",
        "contentType": "application/json-patch+json",
        "data": "WyB7ICJvcCI6ICJyZXBsYWNlIiwgInBhdGgiOiAiL2JpcnRoRGF0ZSIsICJ2YWx1ZSI6ICIxOTkwLTAxLTAxIiB9IF0K"
      }
    }
  ]
}
```
<div class="explain">
<p><span class="file">request.json</span>
data 필드는 
</p>
</div>
```json
[
  {
    "op": "replace",
    "path": "/birthdate",
    "value": "1990-01-01"
  }
]
```
<div class="explain">
<p>
를 base64로 인코딩한 값<br>
인터넷에서 <a href="https://www.convertstring.com/ko/EncodeDecode/Base64Encode">온라인 base64 인코더</a>를 사용해서 인코딩 가능
</p>
</div>
```console
> curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/fhir+json" \
    -d @request.json \
    "https://healthcare.googleapis.com/v1beta1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir"
```
<h4>-&nbsp;Bundle로 리소스 생성 시 id 참조</h4>
<div class="explain">
<p>
번들의 유형이 <b>transaction</b>일 경우에는 번들 실행 중 생성된 리소스에 대한 참조를 포함할 수 있다. 번들에 있는 다른 리소스의 <b>fullUrl</b>값으로 리소스간의 참조 관계를 파악하고 값이 일치하면 새로 부여된 ID로 다시 작성된다.
<b>fullUrl</b>는 다음과 같은 형태를 띈다.
<ul>
<li>urn:uuid:UUID</li>
<li>urn:oid:OID</li>
<li>모든 URL</li>
</ul>
단, RESOURCE_TYPE/RESOURCE_ID는 사용하지 않는 것이 좋다.
</p>
</div>
```json
{
    "resourceType": "Bundle",
    "type": "transaction",
    "entry":[
      {
        "request": {
          "method":"POST",
          "url":"Patient"
        },
        "fullUrl": "urn:oid:05efabf0-4be2-4561-91ce-51548425acb9",
        "resource": {
          "resourceType":"Patient",
          "gender":"male",
          "name": [
            {
              "use": "official",
              "family": "Han567",
              "given": [
                "Soheon"
              ]
            }
          ],
          "identifier" :[{
            "system":"my_code_system",
            "value":"107"
          }]
        }
      },
      {
        "request": {
          "method":"POST",
          "url":"Observation"
        },
        "resource": {
          "resourceType":"Observation",
          "subject": {
            "reference": "urn:oid:05efabf0-4be2-4561-91ce-51548425acb9"
          },
          "status":"preliminary",
          "code": {
            "text":"heart rate"
          },
          "identifier":[{
            "system":"my-code-system",
            "value":"ABC66766"
          }]
        }
      }
    ]
}
```
<div class="explain">
<p><span class="file">transaction.json</span>
</p>
</div>
```console
curl -X POST \
    -H "Content-Type: application/fhir+json; charset=utf-8" \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    --data @transaction.json \
    "https://healthcare.googleapis.com/v1/projects/PROJECT_ID/locations/LOCATION/datasets/DATASET_ID/fhirStores/FHIR_STORE_ID/fhir"
```
<div class="explain">
<p>id가 새로 부여되고 부여된 id는 요청의 결과에 포함된다.
</p>
</div>
<h3>추가적인 자료</h3>
<a href="https://cloud.google.com/healthcare-api/docs/how-tos/fhir-bundles?hl=ko&cloudshell=false">https://cloud.google.com/healthcare-api/docs/how-tos/fhir-bundles?hl=ko&cloudshell=false</a>