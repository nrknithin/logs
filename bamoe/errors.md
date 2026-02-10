# Build Errors Log

Build date: 2026-02-05 Command: `pnpm -r --no-bail build:dev`

---

## Summary

**Total Failed Packages: 12**

| # | Package | Exit Code | Root Cause |
|---|---------|-----------|------------|
| 1 | `@kie-tools/runtime-tools-process-webapp-components` | 2 | TypeScript - Apollo Client type mismatch (graphql version conflict) |
| 2 | `@kie-tools/runtime-tools-swf-webapp-components` | 2 | TypeScript - Apollo Client type mismatch (graphql version conflict) |
| 3 | `@kie-tools/jbpm-quarkus-devui` | 1 | Maven - Missing dependency (cascading from dashbuilder) |
| 4 | `vscode-extension-dashbuilder-editor` | 2 | Webpack - Missing dependency (cascading failure) |
| 5 | `@kie-tools/dashbuilder-viewer-image` | 1 | Missing dashbuilder artifacts (cascading failure) |
| 6 | `@kie-tools/runtime-tools-management-console-webapp` | 1 | Webpack - Depends on runtime-tools-process-webapp-components |
| 7 | `@bamoe/canvas-dev-deployment-quarkus-blank-app` | 1 | Maven - Missing sonataflow-quarkus-devui-bom dependency |
| 8 | `@bamoe/maven-repository` | 1 | Maven - Missing dependencies |
| 9 | `@kie-tools/serverless-workflow-dev-ui-webapp` | 1 | Webpack - Depends on runtime-tools-swf-webapp-components |
| 10 | `@kie-tools/sonataflow-management-console-webapp` | 1 | Webpack - Depends on runtime-tools components |
| 11 | `@bamoe/maven-repository-image` | 9 | Missing maven-repository artifacts (cascading failure) |
| 12 | `@kie-tools/sonataflow-quarkus-devui` | 1 | Maven - Missing dependencies |
| 13 | `@kie-tools/serverless-logic-web-tools-swf-deployment-quarkus-app` | 1 | Maven - Missing sonataflow-quarkus-devui-bom:pom:0.0.0 |

---

## Root Cause Analysis

### 1. PRIMARY: Apollo Client / GraphQL Version Conflict

**Affected Packages:**
- `runtime-tools-process-webapp-components`
- `runtime-tools-swf-webapp-components`

**Error Type:** TypeScript TS2345

**Description:**
Type incompatibility between `apollo-client` resolved with different `graphql` versions:
- `graphql@16.11.0` vs `graphql@14.3.1`

**Files with errors:**
- `JobsManagementChannelApiImpl.tsx` (4 errors)
- `ProcessDefinitionsListChannelApiImpl.ts` (2 errors)
- `ProcessDetailsChannelApiImpl.ts` (16 errors)
- `ProcessListChannelApiImpl.ts` (9 errors)
- `WorkflowDefinitionListQueries.ts` (1 error)
- `WorkflowDetailsQueries.ts` (14 errors)
- `WorkflowListQueries.ts` (7 errors)

**Sample Error:**
```
error TS2345: Argument of type 'import(".../apollo-client@2.6.10_graphql@16.11.0/.../ApolloClient").default<any>'
is not assignable to parameter of type 'import(".../apollo-client@2.6.10_graphql@14.3.1/.../ApolloClient").default<any>'.
```

---

### 2. SECONDARY: Missing Maven Dependencies

**Affected Packages:**
- `serverless-logic-web-tools-swf-deployment-quarkus-app`
- `canvas-dev-deployment-quarkus-blank-app`
- `sonataflow-quarkus-devui`
- `jbpm-quarkus-devui`

**Error:**
```
Non-resolvable import POM: Could not find artifact
org.apache.kie.sonataflow:sonataflow-quarkus-devui-bom:pom:0.0.0 in central
```

**Description:**
The `sonataflow-quarkus-devui-bom` is not available in Maven Central and needs to be built/published first.

---

### 3. CASCADING FAILURES

These packages failed due to upstream dependencies failing:

| Package | Depends On |
|---------|------------|
| `runtime-tools-management-console-webapp` | `runtime-tools-process-webapp-components` |
| `serverless-workflow-dev-ui-webapp` | `runtime-tools-swf-webapp-components` |
| `sonataflow-management-console-webapp` | `runtime-tools-*-components` |
| `vscode-extension-dashbuilder-editor` | `dashbuilder` artifacts |
| `dashbuilder-viewer-image` | `dashbuilder` build output |
| `maven-repository-image` | `maven-repository` |

---

## Recommended Fixes

### Fix 1: GraphQL Version Alignment
Align the `graphql` dependency version across the workspace:
```bash
# Option A: Force resolution in root package.json
"pnpm": {
  "overrides": {
    "graphql": "16.11.0"
  }
}

# Option B: Update packages to use consistent graphql version
```

### Fix 2: Build sonataflow-quarkus-devui-bom First
The BOM needs to be built and installed to local Maven repository before dependent packages can build:
```bash
# Build and install the BOM first
cd packages/sonataflow-quarkus-devui
mvn clean install -DskipTests
```

---

## Successful Builds (Notable)

- ✅ `serverless-workflow-diagram-editor` (Maven GWT)
- ✅ `process-instance-migration-addon` (Maven)
- ✅ `extended-services-java` (Maven)
- ✅ `canvas` (Webpack)
- ✅ `developer-tools-for-vscode` (Webpack)
- ✅ `online-editor` (Webpack)
- ✅ `sonataflow-operator` (Go)
- ✅ All helm charts

---

## Full Build Log

See: `pnpm-build-dev-run.log`
