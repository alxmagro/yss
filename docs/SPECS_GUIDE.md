# YSS Implementation Guide

This repository provides a language-agnostic test suite for the **YSS (YAML Simplified Schema)**
specification. Any YSS implementation (validator/parser) should pass these tests to ensure
compatibility with the standard.

To run these specs, you must implement a **test runner** in your language of choice.

<br>

## Directory Structure

```
specs/
  codes.yaml              — error code constants referenced in specs via {{ CODE_* }} templates
  rules/
    scalars/              — scalar rule specs (type, size, format, ...)
    composites/           — composite rule specs (object, array, any_of, ...)
  parser/                 — parser/compiler error specs (invalid schema inputs)
  integration/
    imports/              — multi-file specs exercising $imports and $ref
      schema.yaml         — the YSS schema under test
      spec.yaml           — scenarios validated against schema.yaml
      *.yaml              — auxiliary schemas imported by schema.yaml
```

<br>

## Spec format

### Standard spec (`rules/` and `parser/`)

```yaml
feature: Short description

given:                            # YSS schema used to validate the scenarios below
  field_name: string
  other_field:
    $type: integer
    $gte: 0

scenarios:
  - name: valid data
    when:                         # payload to validate
      field_name: hello
    then: []                      # [] means the payload is valid

  - name: multiple valid payloads
    when:                         # list of payloads — all must produce the same then
      - field_name: hello
      - field_name: world
    then: []

  - name: invalid data
    when:
      field_name: 42
    then:
      - path: field_name
        code: "{{ CODE_TYPE }}"
        message: Unexpected type
        data:
          value: 42
          expected: string
```

`given` is optional at the top level when all scenarios define their own `given`.

#### Per-scenario `given`

A scenario can define its own `given` to override the top-level one (or provide it when there is none). This is useful for testing parser/compiler behavior where different schemas are needed per scenario:

```yaml
feature: Parser errors

scenarios:
  - name: non-object root throws
    given: "string"
    then_throws: YSS schema root must be an object

  - name: unexpected field value type throws
    given:
      age: 42
    then_throws: "Unexpected schema value: 42"
```

#### `then_throws`

Instead of `then`, a scenario can use `then_throws` to assert that building a validator from `given` raises a specific error message. The comparison is an exact string match against the error message.

```yaml
  - name: invalid schema input
    given:
      status:
        - string
        - integer
    then_throws: 'Array field definitions are not supported...'
```

### Integration spec (`integration/`)

Each subdirectory has a `schema.yaml` (the YSS schema under test, may use `$imports`)
and a `spec.yaml` containing only `scenarios` — no `given`. The scenarios are validated
against `schema.yaml` loaded from the same directory. Auxiliary schemas imported by
`schema.yaml` also live in the same directory.

```yaml
feature: Short description

scenarios:
  - name: valid data
    when:
      slug: my-project
    then: []
```

#### `throws` (top-level)

If loading the schema itself is expected to raise an error (e.g. an unresolvable `$ref`),
the spec can declare a top-level `throws` instead of `scenarios`:

```yaml
throws: '$ref: unknown import namespace "unknown"'
```

The runner should attempt to load `schema.yaml` and assert that it raises the given message.

<br>

## Error object format

Each error in `then` is compared against the actual validator output:

```yaml
- path: field.nested[0].key   # dot/bracket path to the error location
  code: "{{ CODE_TYPE }}"      # error code constant (resolved before comparison)
  message: Unexpected type     # human-readable message
  data:                        # optional — extra context (value, expected, etc.)
    value: 42
    expected: string
```

Comparison uses deep equality. The actual error must contain exactly the same keys and values as specified — partial matching is not used.

<br>

## Error code templates

`codes.yaml` contains named constants for all error codes:

```yaml
CODE_TYPE:    type
CODE_REQUIRED: required
# ...
```

Spec files reference these constants via `{{ CODE_TYPE }}` placeholders instead of literal strings. The runner is responsible for resolving them before comparing.

<br>

## Implementing a runner

### Step 1 — Load codes

```
codes = parse_yaml("specs/codes.yaml")
```

### Step 2 — Resolve templates

Replace `{{ CODE_* }}` placeholders in expected errors before comparing:

```
function resolve_templates(value):
  if string: replace "{{ KEY }}" with codes[KEY]
  if array:  map resolve_templates over items
  if object: map resolve_templates over values
  else:      return as-is
```

### Step 3 — Run rules and parser specs

For each `.yaml` file in `specs/rules/` and `specs/parser/`:

1. Parse the file
2. If the file has a top-level `given`, build a default validator from it
3. For each scenario:
   - If the scenario has its own `given`, build a new validator from it (overrides the default)
   - If the scenario has `then_throws`:
     - Attempt to build a validator from `given`
     - Assert it raises an error whose message equals `then_throws`
   - Otherwise:
     - Validate `when` payload(s) using the active validator
     - Compare actual errors against the expected errors in `then`
   - Report PASS / FAIL

### Step 4 — Run integration specs

For each `spec.yaml` file in `specs/integration/*/`:

1. Attempt to load `schema.yaml` from the same directory (with `$imports` support)
2. If `spec.yaml` has a top-level `throws`:
   - Assert that loading `schema.yaml` raised an error whose message equals `throws`
   - Report PASS / FAIL and move on
3. Otherwise, for each scenario in `spec.yaml`:
   - Validate `when` payload(s)
   - Compare actual errors against the expected errors in `then`
   - Report PASS / FAIL

### Step 5 — Report

```
N passed, N failed, N skipped
```

Exit with a non-zero code if any test failed.

<br>

## Reference implementation

**See** [yss-js spec-runner](https://github.com/alxmagro/yss-js/blob/main/scripts/run-specs.js)
