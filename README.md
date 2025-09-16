## 🧪 Laboratorio: Caso Clínico – Atención de una Neumonía
🎯 Objetivos

Comprender cómo se representan en FHIR distintos aspectos de la atención clínica.

Practicar la creación, consulta y relación de múltiples recursos FHIR.

Reforzar la visión de FHIR como un ecosistema, no solo recursos aislados.

1. Contexto del caso clínico

Ana Pérez, paciente de 45 años, acude al hospital con síntomas de fiebre, tos y dificultad respiratoria. El médico diagnostica neumonía adquirida en la comunidad y prescribe antibióticos. Además, se solicita una radiografía de tórax.

2. Recursos FHIR involucrados

Para representar esta situación, trabajaremos con los siguientes recursos:

Patient → información de Ana Pérez.

Practitioner → médico que atiende el caso.

Encounter → consulta en urgencias.

Condition → diagnóstico de neumonía.

Observation → fiebre y saturación de oxígeno.

MedicationRequest → prescripción de antibiótico.

ServiceRequest → solicitud de radiografía.

3. Bundle de persistencia inicial

👉 Copia este archivo como neumonia-bundle.json

```bash

{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "fullUrl": "urn:uuid:patient-ana",
      "resource": {
        "resourceType": "Patient",
        "id": "patient-ana",
        "name": [
          { "family": "Pérez", "given": ["Ana"] }
        ],
        "gender": "female",
        "birthDate": "1980-02-12"
      },
      "request": { "method": "POST", "url": "Patient" }
    },
    {
      "fullUrl": "urn:uuid:practitioner-julio",
      "resource": {
        "resourceType": "Practitioner",
        "id": "practitioner-julio",
        "name": [
          { "family": "García", "given": ["Julio"] }
        ]
      },
      "request": { "method": "POST", "url": "Practitioner" }
    },
    {
      "fullUrl": "urn:uuid:encounter-urgencias",
      "resource": {
        "resourceType": "Encounter",
        "id": "encounter-urgencias",
        "status": "in-progress",
        "class": { "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode", "code": "AMB" },
        "subject": { "reference": "urn:uuid:patient-ana" },
        "participant": [
          { "individual": { "reference": "urn:uuid:practitioner-julio" } }
        ]
      },
      "request": { "method": "POST", "url": "Encounter" }
    },
    {
      "fullUrl": "urn:uuid:condition-neumonia",
      "resource": {
        "resourceType": "Condition",
        "id": "condition-neumonia",
        "subject": { "reference": "urn:uuid:patient-ana" },
        "encounter": { "reference": "urn:uuid:encounter-urgencias" },
        "code": {
          "coding": [
            { "system": "http://snomed.info/sct", "code": "233604007", "display": "Pneumonia" }
          ]
        },
        "clinicalStatus": { "coding": [{ "system": "http://terminology.hl7.org/CodeSystem/condition-clinical", "code": "active" }] }
      },
      "request": { "method": "POST", "url": "Condition" }
    },
    {
      "fullUrl": "urn:uuid:observation-fiebre",
      "resource": {
        "resourceType": "Observation",
        "id": "observation-fiebre",
        "status": "final",
        "code": {
          "coding": [
            { "system": "http://loinc.org", "code": "8310-5", "display": "Body temperature" }
          ]
        },
        "valueQuantity": { "value": 38.5, "unit": "°C" },
        "subject": { "reference": "urn:uuid:patient-ana" }
      },
      "request": { "method": "POST", "url": "Observation" }
    },
    {
      "fullUrl": "urn:uuid:observation-satO2",
      "resource": {
        "resourceType": "Observation",
        "id": "observation-satO2",
        "status": "final",
        "code": {
          "coding": [
            { "system": "http://loinc.org", "code": "59408-5", "display": "Oxygen saturation in Arterial blood by Pulse oximetry" }
          ]
        },
        "valueQuantity": { "value": 89, "unit": "%" },
        "subject": { "reference": "urn:uuid:patient-ana" }
      },
      "request": { "method": "POST", "url": "Observation" }
    },
    {
      "fullUrl": "urn:uuid:medicationRequest-amoxicilina",
      "resource": {
        "resourceType": "MedicationRequest",
        "id": "medicationRequest-amoxicilina",
        "status": "active",
        "intent": "order",
        "medicationCodeableConcept": {
          "coding": [
            { "system": "http://www.nlm.nih.gov/research/umls/rxnorm", "code": "723", "display": "Amoxicillin 500mg" }
          ]
        },
        "subject": { "reference": "urn:uuid:patient-ana" },
        "encounter": { "reference": "urn:uuid:encounter-urgencias" },
        "authoredOn": "2025-09-11"
      },
      "request": { "method": "POST", "url": "MedicationRequest" }
    },
    {
      "fullUrl": "urn:uuid:serviceRequest-radiografia",
      "resource": {
        "resourceType": "ServiceRequest",
        "id": "serviceRequest-radiografia",
        "status": "active",
        "intent": "order",
        "code": {
          "coding": [
            { "system": "http://snomed.info/sct", "code": "168731009", "display": "Chest X-ray"
            }
          ]
        },
        "subject": { "reference": "urn:uuid:patient-ana" },
        "encounter": { "reference": "urn:uuid:encounter-urgencias" }
      },
      "request": { "method": "POST", "url": "ServiceRequest" }
    }
  ]
}
```

4. Ejercicios
A. Persistencia

Cargar el bundle con:

curl -X POST -H "Content-Type: application/fhir+json" -d @neumonia-bundle.json http://localhost:8080/fhir/Bundle

B. Consultas guiadas

Buscar todas las condiciones de Ana Pérez:

GET http://localhost:8080/fhir/Condition?subject=Patient/[ID_PACIENTE]


Obtener las observaciones de tipo fiebre:

GET http://localhost:8080/fhir/Observation?code=http://loinc.org|8310-5&subject=Patient/[ID_PACIENTE]


Listar las órdenes médicas (MedicationRequest y ServiceRequest):

GET http://localhost:8080/fhir/MedicationRequest?subject=Patient/[ID_PACIENTE]
GET http://localhost:8080/fhir/ServiceRequest?subject=Patient/[ID_PACIENTE]


    
