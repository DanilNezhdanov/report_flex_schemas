# Navixy Report Page Schema

**Draft 2020-12 JSON Schema** for describing extensible, scrollable report pages with rows of KPI tiles, tables, annotations — and charts (bar/pie).

**Schema Version:** 1.1.0 (additive release; includes Charts row)

## What this is

A pragmatic, forward-compatible contract apps can rely on to render report pages and run per-visual SQL with optional verification rules.

- **Page** = stacked rows
- **Row types:** tiles (1–3 numeric KPIs), table, annotation, charts (bar/pie)
- **Each visual** carries its own SQL (named params) and optional verify block to check shape/values

## Stability & Backward Compatibility

We follow SemVer for the schema itself (`meta.schema_version`):

- **MAJOR:** Breaking changes (e.g., rename/remove required fields). New major = new path (e.g., `/schema/v2/report-page.schema.json`). We maintain v1 for at least 12 months after v2 GA.
- **MINOR:** Additive changes (new optional fields/row types/visuals). v1.1.0 → this release (adds charts row).
- **PATCH:** Clarifications/bug fixes (no structural change).

**Deprecations:** Marked in docs and `$comments`; remain valid for ≥ two minor versions before removal in the next MAJOR.

## Files & Layout

```
/schema/report-page.schema.json          # the JSON Schema (v1 latest)
/schema/v1/report-page.schema.json       # pin by major
/examples/report-page.example.json
/tests/report-page.tests.json
/types/navixy-report-page.d.ts
LICENSE
```

**Stable raw URL pattern (example):**
```
https://raw.githubusercontent.com/<org>/<repo>/main/schema/v1/report-page.schema.json
```

## Quick Start

### Validate with AJV (Node.js)
```bash
npx ajv validate -s schema/report-page.schema.json -d examples/report-page.example.json
npx ajv test -t tests/report-page.tests.json
```

### Validate with Python (jsonschema)
```python
from jsonschema import validate
import json

schema = json.load(open("schema/report-page.schema.json"))
data = json.load(open("examples/report-page.example.json"))
validate(instance=data, schema=schema)
```

### TypeScript Usage
```typescript
import type { NavixyReportPage } from './types/navixy-report-page';

const report: NavixyReportPage = {
  title: "My Report",
  meta: {
    schema_version: "1.1.0",
    last_updated: "2025-01-27T00:00:00Z",
    updated_by: { id: "user123", name: "Analyst" }
  },
  rows: [
    {
      type: "tiles",
      visuals: [
        {
          kind: "tile",
          label: "Total Users",
          query: { sql: "SELECT COUNT(*) AS value FROM users" },
          verify: {
            row_count: { eq: 1 },
            columns: [{ name: "value", type: "integer" }]
          }
        }
      ]
    }
  ]
};
```

## Schema Structure

### Core Components

- **`title`** (required): Page title
- **`meta`** (required): Schema version, timestamps, author info
- **`rows`** (required): Array of row objects (tiles, table, annotation, charts)
- **`datasources`** (optional): Named SQL connections
- **`parameters`** (optional): Global parameters for SQL queries
- **`theme`** (optional): Page-level design defaults

### Row Types

#### 1. Tiles Row (`type: "tiles"`)
- **Purpose:** Display 1-3 numeric KPIs
- **Visuals:** Array of `tileVisual` objects
- **Each tile:** Requires `query.sql` returning single row with numeric `value` column

#### 2. Table Row (`type: "table"`)
- **Purpose:** Display tabular data with pagination
- **Visuals:** Single `tableVisual` object
- **Features:** Column formatting, sorting, pagination controls

#### 3. Annotation Row (`type: "annotation"`)
- **Purpose:** Display text content, notes, or documentation
- **Visuals:** Single `annotationVisual` object
- **Features:** Markdown support, custom styling

#### 4. Charts Row (`type: "charts"`) - *v1.1.0*
- **Purpose:** Display bar and pie charts
- **Visuals:** Array of `barVisual` or `pieVisual` objects (1-3 charts)
- **Features:** Color palettes, legends, tooltips, sorting

### Query & Verification

Each visual can include:

- **`query`:** SQL with named parameters (`:param_name`)
- **`verify`:** Optional validation rules for:
  - Row count constraints
  - Column shape/type validation
  - Value range checks

## Authoring Checklist

### Safe Defaults
- ✅ Fill `title` + `meta` (`schema_version`, `last_updated`, `updated_by`)
- ✅ Each visual has a `query.sql` (use named params) and, ideally, `verify`
- ✅ Tiles return 1 row / 1 numeric column (`value`); enforce with SQL or verify
- ✅ Tables: declare `options.columns` for formatting + `verify.columns` for shape
- ✅ ≤ 3 visuals in a tiles row; exactly 1 visual in table/annotation rows
- ✅ Prefer `connection.env` for DSNs (avoid secrets in JSON)
- ✅ Set `row_limit`/`timeout_ms` for heavy queries

### Example Query Patterns

**KPI Tile:**
```sql
SELECT COUNT(*) AS value FROM users WHERE created_at >= :start_date
```

**Bar Chart:**
```sql
SELECT status AS category, COUNT(*)::int AS value 
FROM orders 
WHERE tenant_id = :tenant_id 
GROUP BY status
```

**Table with Pagination:**
```sql
SELECT id, name, email, created_at 
FROM users 
WHERE tenant_id = :tenant_id 
ORDER BY created_at DESC 
LIMIT :row_limit
```

## Security & Operational Notes

- **SQL Security:** SQL is read-only in runners; use named params to prevent injection
- **Performance:** Cache stable tiles (e.g., daily aggregates); paginate tables
- **Access Control:** Lives outside the schema; don't embed secrets
- **Connection Management:** Use environment variables for database connections

## Contributing

### Process
- **MINOR/PATCH:** Propose changes via PR. Additions must be optional and non-breaking.
- **MAJOR:** Open an ADR (Architecture Decision Record) and plan a deprecation window.
- **Testing:** Keep tests updated (`/tests`) and examples reflecting the latest schema.

### Development Setup
```bash
# Clone the repository
git clone https://github.com/DanilNezhdanov/report_flex_schemas.git
cd report_flex_schemas

# Validate schema and examples
npx ajv validate -s schema/report-page.schema.json -d examples/report-page.example.json
npx ajv test -t tests/report-page.tests.json

# Run TypeScript type checking
npx tsc --noEmit types/navixy-report-page.d.ts
```

## License

MIT — see `$comment` in the schema and LICENSE file.

## Changelog

### v1.1.0 (2025-01-27)
- ✅ Added Charts row type with bar and pie visual support
- ✅ Added color palette support for charts
- ✅ Added chart-specific options (orientation, donut mode, legends, etc.)
- ✅ Maintained backward compatibility with v1.0.0

### v1.0.0 (Initial Release)
- ✅ Core schema with tiles, table, and annotation rows
- ✅ SQL query support with named parameters
- ✅ Verification rules for data validation
- ✅ Theme and layout support
- ✅ TypeScript type definitions
