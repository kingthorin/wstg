# API Testing TODO (temporary)

Items moved out of Business Logic (4.10) that belong in 4.12. Delete this file once merged into the relevant test cases.

## From 4.10.1 Business Logic Data Validation

- Server-side validation of JSON and GraphQL variable types, ranges and allowed sets, beyond what the schema declares.
- Schema-driven testing against an OpenAPI specification (for example [Schemathesis](https://schemathesis.readthedocs.io), [RESTler](https://github.com/microsoft/restler-fuzzer)).
- Unexpected or additional properties in request bodies (cross-reference [Mass Assignment](../07-Injection/20-Mass_Assignment.md)).
- OWASP API Security Top 10 2023: [API6 Unrestricted Access to Sensitive Business Flows](https://owasp.org/API-Security/editions/2023/en/0xa6-unrestricted-access-to-sensitive-business-flows/).

## From 4.10.0 Introduction

- Postman/Bruno style collections for replaying multi-step API flows.

## From 4.10.2 Ability to Forge Requests

- Hidden or undocumented endpoints and parameters in API definitions (parameter discovery, e.g. Arjun, Param Miner).
