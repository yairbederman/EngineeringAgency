# Output Schemas Reference

> **Token Efficiency**: Phase files should reference these schemas instead of inlining them.
> **Usage**: `See Schema "[name]" in _output-schemas.md`

---

## api-contracts.json

```json
{
  "[ClientOrController]": {
    "file": "[path]",
    "basePath": "[base URL]",
    "endpoints": [
      {
        "method": "GET|POST|PUT|DELETE|PATCH",
        "path": "/api/v1/...",
        "handler": "[function name]",
        "requestType": "[type name]",
        "requestFields": {
          "[field]": {
            "type": "[type]",
            "validation": { "required": true, "maxLength": 100 }
          }
        },
        "responseType": "[type name]",
        "responseFields": { "[field]": "[type]" },
        "queryParams": { "[param]": "[type]" }
      }
    ]
  },
  "_coverage": {
    "apiDirectories": 0,
    "apiFilesScanned": 0,
    "endpointsDocumented": 0,
    "skipped": [{ "path": "[dir]", "reason": "[reason]" }]
  }
}
```

---

## entity-contracts.json

```json
{
  "[EntityName]": {
    "file": "[path]",
    "type": "class|interface|type|enum",
    "fields": {
      "[field]": {
        "type": "[resolved type]",
        "optional": false
      }
    }
  },
  "_coverage": { "entitiesFound": 0, "fieldsResolved": 0 },
  "_unresolved": []
}
```

---

## service-topology.json

```json
{
  "generatedAt": "[ISO-8601]",
  "services": [
    {
      "name": "[service-name]",
      "type": "Frontend|Backend|Data Service|Shared Library",
      "role": "[from project-inventory]",
      "exposedEndpoints": 0,
      "callsServices": ["[verified names]"],
      "calledBy": ["[computed]"]
    }
  ],
  "dependencies": [
    {
      "from": "[calling]",
      "to": "[called]",
      "type": "http|grpc|message-queue|library",
      "codeEvidence": "[file]:[line] - [what was found]",
      "verified": true
    }
  ],
  "layers": {
    "presentation": [],
    "api": [],
    "integration": [],
    "data": [],
    "shared": []
  },
  "_verificationSummary": {
    "totalClaimed": 0,
    "verified": 0,
    "excluded": 0
  },
  "_warnings": []
}
```

---

## function-registry.json

```json
{
  "[ServiceName]": {
    "file": "[path]",
    "methods": [
      {
        "name": "[method]",
        "params": [{ "name": "[param]", "type": "[type]" }],
        "returnType": "[type]",
        "calls": ["[other services/methods]"],
        "externalCalls": ["[HTTP/external calls]"]
      }
    ]
  },
  "_coverage": { "servicesDocumented": 0, "methodsDocumented": 0 }
}
```

---

## project-inventory.json

```json
{
  "generatedAt": "[ISO-8601]",
  "projects": [
    {
      "name": "[project-name]",
      "path": "[absolute path]",
      "type": "Frontend|Backend|Library",
      "role": "[from configuration]",
      "status": "ready|missing-instructions|error",
      "aiInstructionsPath": "[path to .ai-instructions]"
    }
  ],
  "_summary": { "total": 0, "ready": 0, "missing": 0 }
}
```
