<p align="center">
  <img src="assets/logo.svg"/>
</p>

<p align="center">
  A <b>human-friendly</b> alternative to JSON Schema. Write validation schemas in clean YAML.
</p>

## Introduction

YSS is a schema language for validating JSON payloads, written in YAML.
It is designed to be simple to read, simple to write, and simple to implement in any language.

> Version 0.2.0

<br>

## Core principles

- Every field is **optional by default**. Use `$required` to mark required fields.
- Every object is **open by default**. Use `$strict` to reject extra fields.
- Fields and rules are additive — the more you declare, the stricter the validation.
- The schema should be readable by non-developers without explanation.

<br>

## Types

| Type      | Validates                              |
|-----------|----------------------------------------|
| `string`  | UTF-8 string                           |
| `integer` | Whole number (no decimal part)         |
| `number`  | Any number, including decimals         |
| `boolean` | `true` or `false`                      |
| `null`    | JSON null value                        |
| `any`     | Any value, no type check               |
| `object`  | Key-value pairs (inferred from fields) |
| `array`   | Ordered list                           |

<br>

## Rules

All rules are prefixed with `$`. They can be declared in block form or inline.

| Rule       | Applies to              | Description                                        |
|------------|-------------------------|----------------------------------------------------|
| `$type`    | any                     | The type of the field                              |
| `$required`| object                  | List of required field names                       |
| `$strict`  | object                  | Reject extra fields. Cascades to nested objects    |
| `$size`    | string, array, object   | Exact size or `[min, max]` range (null = unbound)  |
| `$gt`      | integer, number         | Value must be > n                                  |
| `$gte`     | integer, number         | Value must be >= n                                 |
| `$lt`      | integer, number         | Value must be < n                                  |
| `$lte`     | integer, number         | Value must be <= n                                 |
| `$format`  | string                  | Named alias or `/regex/` the value must match      |
| `$enum`    | any                     | List of allowed values                             |
| `$const`   | any                     | Exact value match                                  |
| `$unique`  | array                   | No duplicate items allowed                         |
| `$item`    | array                   | Schema applied to every item                       |
| `$at`      | array                   | Schema applied per position index                  |

<br>

## Object

Object schemas are declared by nesting field definitions. The type `object` is inferred
automatically — no `$type: object` needed.

```yaml
address:
  street: string
  city: string
  country: string
```

Fields are optional by default. Mark required ones with `$required`:

```yaml
user:
  $required: [name, email]
  name: string
  email:
    $type: string
    $format: email
  phone: string      # optional
```

### Strict mode

By default, objects allow extra fields not declared in the schema. To reject them,
add `$strict: true`. This cascades to all nested objects unless overridden.

```yaml
user:
  $strict: true
  name: string
  email: string
  address:
    street: string   # also strict, inherited
```

A nested object can opt out with `$strict: false`:

```yaml
user:
  $strict: true
  name: string
  meta:
    $strict: false   # open, overrides parent
    source: string
```

<br>

## Array

```yaml
emails:
  $type: array
  $size: [1, 20]
  $item:
    $type: string
    $format: email
```

### Unique items

```yaml
roles:
  $type: array
  $unique: true
  $item:
    $type: string
    $enum: [admin, editor, viewer]
```

### Positional validation (`$at`)

Validates specific positions by index. If the array is shorter than a declared index,
that position is skipped.

```yaml
point:
  $type: array
  $at:
    0: number   # x
    1: number   # y
    2: number   # z (optional, skipped if absent)
```

<br>

## Union (`$any_of`)

The value must match at least one of the listed branches.

```yaml
id:
  $any_of:
    - integer
    - string
```

Branches can include full rule blocks. Fields declared alongside `$any_of` are merged
into every branch:

```yaml
user:
  name: string
  email: string
  country: String

  $any_of:
    - country: String, == US
    - shipping:
        $required: [method, cost]
        method: string, enum [standard, express, pickup]
        cost: number, >= 0
        tracking:
          code: string
          carrier: string
          status: string, enum [pending, shipped, delivered, returned]
```

<br>

## Inline syntax

When a field has only `$type`, it can be written as a bare value:

```yaml
name: string
active: boolean
```

Constraints can be added inline as comma-separated modifiers after the type:

```yaml
field: type, modifier value, modifier value
```

Available modifiers:

| Modifier      | Equivalent     | Example                              |
|---------------|----------------|--------------------------------------|
| `~ alias`     | `$format`      | `string, ~ email`                    |
| `== value`    | `$const`       | `string, == admin`                   |
| `>= n`        | `$gte`         | `integer, >= 18`                     |
| `> n`         | `$gt`          | `number, > 0`                        |
| `<= n`        | `$lte`         | `integer, <= 100`                    |
| `< n`         | `$lt`          | `number, < 1000`                     |
| `size n`      | `$size` (exact)| `string, size 4`                     |
| `size [n, m]` | `$size` (range)| `string, size [~, 80]`               |
| `enum [a, b]` | `$enum`        | `string, enum [active, inactive]`    |
| `uniq`        | `$unique: true`| `array, uniq`                        |

### Union types

```yaml
id: string | integer
deleted_at: string | null
```

### Generic array syntax

```yaml
tags:  array<string>
ids:   array<integer>
```

### Combining modifiers

```yaml
name:   string, size [2, 80]
email:  string, ~ email
status: string, enum [active, inactive, banned]
score:  integer, >= 0, <= 100
```

<br>

## Schema reuse (YAML anchors)

Within the same file, use native YAML anchors to avoid repeating structures:

```yaml
$anchors:
  - &Address
    street: string
    city: string
    country: string

user:
  billing_address: *Address
  shipping_address: *Address
```

<br>

## Modular schemas (`$imports` / `$ref`)

Schemas can be split across files. Each imported file is compiled in isolation.

### $imports

Declared at the root. Each key is a namespace and the value is a relative path:

```yaml
$imports:
  user:   ./user.yaml
  shared: ./shared/types.yaml
```

### $ref

References an imported schema by namespace:

```yaml
profile:
  $ref: user
```

Use dot notation to navigate into a nested field:

```yaml
$imports:
  post: ./post.yaml

items:
  $type: array
  $item:
    $ref: post.data
```

### $patterns across files

`$patterns` from an imported file are available under the namespace prefix:

```yaml
# formats.yaml
$patterns:
  slug: /^[a-z0-9]+(?:-[a-z0-9]+)*$/
```

```yaml
# main.yaml
$imports:
  fmt: ./formats.yaml

slug:
  $type: string
  $format: fmt.slug
```

<br>

## Custom format aliases (`$patterns`)

Named patterns can be declared at the root under `$patterns`. Each pattern is a `/regex/`
string and can be used anywhere `$format` is accepted.

```yaml
$patterns:
  zip-code: /^\d{5}-?\d{3}$/
  phone: /^\+?[0-9]{10,11}$/

address:
  zip:
    $type: string
    $format: zip-code
  contact: string, ~ phone
```

<br>

## Built-in format aliases

### Dates & Times — RFC 3339

| Alias       | Example                |
|-------------|------------------------|
| `date-time` | `2024-01-01T00:00:00Z` |
| `date`      | `2024-01-01`           |
| `time`      | `14:30:00Z`            |
| `duration`  | `P3D`, `PT1H30M`       |

### Email

| Alias       | Description                         |
|-------------|-------------------------------------|
| `email`     | Standard email address (RFC 5321)   |
| `idn-email` | Email with international characters |

### Hostname

| Alias          | Example       |
|----------------|---------------|
| `hostname`     | `example.com` |
| `idn-hostname` | `münchen.de`  |

### IP Addresses

| Alias  | Example         |
|--------|-----------------|
| `ipv4` | `192.168.0.1`   |
| `ipv6` | `2001:db8::1`   |

### Resource Identifiers

| Alias           | Example                      |
|-----------------|------------------------------|
| `uri`           | `https://example.com/path`   |
| `uri-reference` | `/relative/path` or full URI |
| `iri`           | URI with international chars |

### UUID — RFC 4122

| Alias  | Example                                |
|--------|----------------------------------------|
| `uuid` | `550e8400-e29b-41d4-a716-446655440000` |

### JSON Pointer — RFC 6901

| Alias          | Example      |
|----------------|--------------|
| `json-pointer` | `/foo/bar/0` |

### Regex

| Alias   | Description                       |
|---------|-----------------------------------|
| `regex` | A valid regular expression string |

<br>

## Full example

```yaml
# order.yaml

$imports:
  fmt: ./formats.yaml

$required: [id, customer, items]

id: string, ~ uuid
created_at: string | null

customer:
  $strict: true
  $required: [id, name, email]
  id: integer, >= 1
  name: string, size [2, 80]
  email: string, ~ email
  phones:
    $type: array
    $size: [1, ~]
    $item: string, ~ fmt.phone

items:
  $type: array
  $size: [1, 50]
  $item:
    $required: [id, name, qty, price]
    id: integer, >= 1
    name: string, size [1, 100]
    qty: integer, >= 1, <= 9999
    price: number, > 0
    tags:
      $type: array
      $unique: true
      $item: string, enum [fragile, perishable, digital, oversized]
    dimensions:
      $type: array
      $at:
        0: number   # width
        1: number   # height
        2: number   # depth

shipping:
  $required: [method, cost]
  method: string, enum [standard, express, pickup]
  cost: number, >= 0
  tracking:
    code: string
    carrier: string
    status: string, enum [pending, shipped, delivered, returned]
```
