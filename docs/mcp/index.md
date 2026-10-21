# Overview

`@nest-openapi/mcp` turns your OpenAPI 3.x operations into MCP tools for NestJS applications.

- **Spec-driven** — Tools are generated from OpenAPI operations.
- **API contract ↔ MCP contract** — Request/input and success response/output OpenAPI schemas are mapped into MCP tool schemas.
- **Secure by default** — Tool exposure is opt-in by default (`x-mcp: true` per operation).
- **NestJS-native** — HTTP transport integration plus programmatic server APIs.

### Install

```bash
npm i @nest-openapi/mcp
```

### Quick Start

```ts
// app.module.ts
import { Module } from "@nestjs/common";
import { OpenAPIMcpModule } from "@nest-openapi/mcp";
import * as openApiSpec from "./openapi.json";

@Module({
  imports: [
    OpenAPIMcpModule.forRoot({
      specSource: { type: "object", spec: openApiSpec },
      http: { path: "/mcp" },
      executor: { baseUrl: "http://127.0.0.1:3000" },
    }),
  ],
})
export class AppModule {}
```

That’s it. Your MCP endpoint is available at `/mcp`.

### Tool Exposure with `x-mcp`

By default, only operations explicitly marked with `x-mcp: true` are exposed as tools.

```yaml
paths:
  /users/{id}:
    get:
      operationId: getUser
      x-mcp: true
```

You can change this behavior using [`tools.defaultInclude`](/mcp/options#tools).

### Tool Responses and Output Schemas

Successful tool calls return both text `content` and JSON `structuredContent`.
Text content keeps the original response payload for compatibility.

When every declared success response has an `application/json` schema, the tool's
`outputSchema` covers all 2xx responses, including `2XX` ranges. Different success
schemas are combined with `anyOf`; identical mapped schemas share one contract.
Local references and nullable schemas are supported. Combined schemas and schemas
without `type: object` use an object schema with a required `result` property,
and their structured content always uses that wrapper. A single schema with
`type: object` returns the response object directly in `structuredContent`.
If any success response has no body, lacks a schema, has an unresolved response
reference, or advertises a media type other than `application/json`, no output schema is
registered, so valid success responses are not rejected by an incomplete contract.
Without an output schema, objects are returned directly; arrays, primitives, and
`null` are wrapped as `{ result: payload }`.

For example, an array response `[ { "id": "7" } ]` is returned as
`structuredContent: { result: [{ id: "7" }] }`, while its text content remains
`[{"id":"7"}]`.

Upstream HTTP failures and execution errors return `isError: true` with text
content rather than structured success data.

## Next Steps

- [Advanced setup](./advanced-setup.md) — Transport/session strategy and server extension patterns
- [Options](./options.md) — Full configuration reference
- [Manual Usage](./manual.md) — Programmatic MCP server usage
