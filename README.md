# @proxima-nexus/openapi

[![npm version](https://img.shields.io/npm/v/@proxima-nexus/openapi.svg)](https://www.npmjs.com/package/@proxima-nexus/openapi)
[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](https://opensource.org/licenses/ISC)

Official OpenAPI 3.0 specification for the **Proxima Nexus Data Plane API**. This package provides machine-readable API documentation in both JSON and YAML formats, enabling automatic SDK generation, API testing, and client integration.

## 📦 Installation

```bash
npm install @proxima-nexus/openapi
```

or with yarn:

```bash
yarn add @proxima-nexus/openapi
```

or with pnpm:

```bash
pnpm add @proxima-nexus/openapi
```

## 📋 Contents

This package includes:

- **`openapi.json`** - OpenAPI 3.0 specification in JSON format
- **`openapi.yaml`** - OpenAPI 3.0 specification in YAML format

Both files are identical in content and are provided for convenience depending on your tooling preferences.

## 🚀 Usage

### Accessing the Specification Files

```typescript
import openapiJson from '@proxima-nexus/openapi/openapi.json';
import * as fs from 'fs';
import * as path from 'path';

// In Node.js
const openapiPath = path.join(
  require.resolve('@proxima-nexus/openapi'),
  '../openapi.yaml'
);
const spec = fs.readFileSync(openapiPath, 'utf-8');
```

```javascript
// In Node.js (CommonJS)
const openapiJson = require('@proxima-nexus/openapi/openapi.json');
const openapiYaml = require('@proxima-nexus/openapi/openapi.yaml');
```

```python
# In Python
import json
import pkg_resources

openapi_json_path = pkg_resources.resource_filename(
    'proxima_nexus_openapi', 'openapi.json'
)
with open(openapi_json_path, 'r') as f:
    spec = json.load(f)
```

### Generating SDKs

#### TypeScript/JavaScript

Using [openapi-generator](https://openapi-generator.tech/):

```bash
npx @openapitools/openapi-generator-cli generate \
  -i node_modules/@proxima-nexus/openapi/openapi.yaml \
  -g typescript-axios \
  -o ./generated-sdk
```

#### Python

```bash
npx @openapitools/openapi-generator-cli generate \
  -i node_modules/@proxima-nexus/openapi/openapi.yaml \
  -g python \
  -o ./generated-sdk
```

#### Java

```bash
npx @openapitools/openapi-generator-cli generate \
  -i node_modules/@proxima-nexus/openapi/openapi.yaml \
  -g java \
  -o ./generated-sdk
```

#### Go

```bash
npx @openapitools/openapi-generator-cli generate \
  -i node_modules/@proxima-nexus/openapi/openapi.yaml \
  -g go \
  -o ./generated-sdk
```

### Using with Swagger UI

```typescript
import SwaggerUI from 'swagger-ui-react';
import openapiJson from '@proxima-nexus/openapi/openapi.json';

function ApiDocs() {
  return <SwaggerUI spec={openapiJson} />;
}
```

### Using with Postman

1. Import `openapi.json` or `openapi.yaml` into Postman
2. Postman will automatically generate a collection with all API endpoints

### Validating API Requests

```typescript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';
import openapiJson from '@proxima-nexus/openapi/openapi.json';

const ajv = new Ajv();
addFormats(ajv);

// Validate request body against schema
const validate = ajv.compile(
  openapiJson.components.schemas.CreateUserDto
);

const isValid = validate(requestBody);
if (!isValid) {
  console.error(validate.errors);
}
```

## 📚 API Overview

The Proxima Nexus Data Plane API provides RESTful endpoints for managing:

### Resources

- **Users** (`/user`) - User profile management, search, and connections
- **Events** (`/event`) - Event creation, updates, and discovery
- **Groups** (`/group`) - Group management and membership

### Key Features

- **Geospatial Search** - Location-based queries with radius and bounding box support
- **Text Search** - Display name search with relevance ranking
- **Connections** - Relationship management between entities
- **Multi-tenancy** - Tenant isolation for all operations

### Authentication

All endpoints require API key authentication via the `X-Proxima-Nexus-Api-Key` header:

```http
GET /user/123 HTTP/1.1
Host: api.proxima-nexus.com
X-Proxima-Nexus-Api-Key: your-api-key-here
```

## 🔄 Versioning

This package follows [Semantic Versioning](https://semver.org/):

- **MAJOR** version increments when breaking changes are introduced to the API
- **MINOR** version increments when new endpoints or features are added (backward compatible)
- **PATCH** version increments for bug fixes and non-breaking changes

The package version matches the API version. Check the [CHANGELOG](./CHANGELOG.md) for detailed version history.

## 🔗 Related Packages

- **TypeScript SDK** - Coming soon
- **Python SDK** - Coming soon

## 🤝 Contributing

This package is automatically generated from the API implementation. To update the specification:

1. Make changes to the API implementation
2. The OpenAPI specification is automatically regenerated via CI/CD
3. A new version of this package is published

## 🔍 Examples

### Example: Generate TypeScript SDK

```bash
# Install the package
npm install @proxima-nexus/openapi

# Generate TypeScript SDK
npx @openapitools/openapi-generator-cli generate \
  -i node_modules/@proxima-nexus/openapi/openapi.yaml \
  -g typescript-axios \
  -o ./src/generated/api-client \
  --additional-properties=npmName=@my-org/proxima-nexus-client,npmVersion=1.0.0

# Use the generated client
import { Configuration, UsersApi } from './src/generated/api-client';

const config = new Configuration({
  apiKey: process.env.PROXIMA_NEXUS_API_KEY,
  basePath: 'https://api.proxima-nexus.com',
});

const usersApi = new UsersApi(config);
const users = await usersApi.userControllerSearch({
  displayName: 'John',
  limit: 10,
});
```

### Example: Validate Request Schema

```typescript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';
import openapiJson from '@proxima-nexus/openapi/openapi.json';

const ajv = new Ajv({ allErrors: true });
addFormats(ajv);

// Get schema for CreateUserDto
const createUserSchema = openapiJson.components.schemas.CreateUserDto;
const validate = ajv.compile(createUserSchema);

// Validate user input
const userInput = {
  displayName: 'John Doe',
  email: 'john@example.com',
  location: {
    latitude: 40.7128,
    longitude: -74.0060,
  },
};

if (validate(userInput)) {
  console.log('Valid input');
} else {
  console.error('Validation errors:', validate.errors);
}
```

### Example: Use with Redoc

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Proxima Nexus API Documentation</title>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://fonts.googleapis.com/css?family=Montserrat:300,400,700|Roboto:300,400,700" rel="stylesheet">
    <style>
      body {
        margin: 0;
        padding: 0;
      }
    </style>
  </head>
  <body>
    <redoc spec-url='./node_modules/@proxima-nexus/openapi/openapi.json'></redoc>
    <script src="https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"></script>
  </body>
</html>
```

---

**Note**: This package contains only the OpenAPI specification. It does not include the API implementation, which remains proprietary.
