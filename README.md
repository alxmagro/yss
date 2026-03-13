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
| `$item`    | array                   | Schema applied to every item                       |
| `$at`      | array                   | Schema applied per position index                  |
| `$contains`| array                   | One or more items must match a schema              |
| `$unique`  | array                   | No duplicate items allowed                         |
| `$size`    | string, array, object   | Exact size or `[min, max]` range (null = unbound)  |
| `$gt`      | integer, number         | Value must be > n                                  |
| `$gte`     | integer, number         | Value must be >= n                                 |
| `$lt`      | integer, number         | Value must be < n                                  |
| `$lte`     | integer, number         | Value must be <= n                                 |
| `$format`  | string                  | Named alias or `/regex/` the value must match      |
| `$enum`    | any                     | List of allowed values                             |
| `$const`   | any                     | Exact value match                                  |
| `$any_of`  | any                     | Value must match at least one of the listed schemas|
| `$one_of`  | any                     | Value must match exactly one of the listed schemas |

<br>

### $type

`$type` is the most basic rule — it defines what kind of value a field accepts.
It is always evaluated first, and if it fails, no other rules are checked for that field.
The simplest form is a bare type name:

```yaml
name: string
```

This validates that `name` is a string:

```json
"Hello World"
```

To accept more than one type, separate them with `|`:

```yaml
name: string | null
```

**Any**

`$type` is not required. Omitting it is perfectly valid, all other rules will still
run normally. Alternatively, you can declare a field as `any`, which explicitly skips type checking
while keeping the field visible in the schema for documentation purposes:

```yaml
metadata: any
```

**Objects**

Object schemas are declared by nesting field definitions. The type `object` is inferred
automatically — no `$type: object` needed.

```yaml
product:
  name: string
  price: number
  stock: integer
  available: boolean
```

<br>

### $required

By default, every field in an object is optional, if it's absent from the payload,
no error is raised. Use `$required` to list the fields that must be present:

```yaml
user:
  $required: [name, email]
  name: string
  email:
    $type: string
    $format: email
  phone: string      # optional
```

<br>

### $strict

Objects are open by default — fields not declared in the schema are simply ignored.
Add `$strict: true` to reject any unexpected fields. This cascades to all nested objects
unless overridden.

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

### $item

Array schemas are declared with `$type: array`. The rule `$item` is useful to define the rules
applied to every element in the array:

```yaml
emails:
  $type: array
  $item: string
```

When the item has no extra rules, you can write it inline using the generic type notation:

```yaml
emails: array<string>
```

<br>

### $at

If you need different rules per position, use `$at` instead. Each key is an index and the value
is the schema for that position. Positions not declared are not validated, so the array can be
longer without errors:

```yaml
point:
  $type: array
  $at:
    0: integer
    1: integer
    2: string   # description
```

<br>

### $contains

Use `$contains` to require that at least one item in the array matches a given schema.
The simplest form uses an inline rule directly:

```yaml
tags:
  $type: array
  $contains: string, == featured
```

For object items, pass the matching schema as a nested block:

```yaml
members:
  $type: array
  $contains:
    role: string, == owner
```

Both forms default to requiring **at least one** matching item. To control the exact count,
add `$quantity` alongside `$item`:

```yaml
# exactly 2 matches
tags:
  $type: array
  $contains:
    $quantity: 2
    $item: string, == featured

# at least 2 matches
party:
  $type: array
  $contains:
    $quantity: [2, ~]
    $item:
      race: string, == hobbit

# at most 1 match
crew:
  $type: array
  $contains:
    $quantity: [~, 1]
    $item:
      side: string, == dark

# between 2 and 4 matches
council:
  $type: array
  $contains:
    $quantity: [2, 4]
    $item:
      rank: string, == master
```

Setting `$quantity: 0` means the array must **not** contain any matching item:

```yaml
flags:
  $type: array
  $contains:
    $quantity: 0
    $item: string, == banned
```

<br>

### $unique

To ensure no two elements are the same, set `$unique: true`:

```yaml
tags:
  $type: array
  $unique: true
  $item: string
```

You can also write it using inline syntax by adding `unique` as a modifier after a comma:

```yaml
tags: array<string>, unique
```

<br>

### $size

Use `$size` to constrain how many elements an array can have. It accepts an exact number or a
`[min, max]` range, where `~` (null) means unbound:

```yaml
emails:
  $type: array
  $size: [1, 10]
  $item: string
```

Combined with `$at`, you can enforce a fixed-length array — a tuple — where each position has its
own type:

```yaml
rgb:
  $type: array
  $size: 3
  $at:
    0: integer   # red
    1: integer   # green
    2: integer   # blue
```

`$size` also applies to strings as character count, and objects as number of keys.

To write it in inline syntax, add `size` as a modifier after a comma:

```yaml
tags: array<string>, size [1, ~]
```

<br>

### $gt, $gte, $lt, $lte

These rules constrain the value of a numeric field. They apply to both `integer` and `number` types:

```yaml
age:
  $type: integer
  $gte: 18
  $lte: 120

price:
  $type: number
  $gt: 0
```

You can also write them inline:

```yaml
age:   integer, >= 18, <= 120
price: number, > 0
```

<br>

### $format

Validates that a string matches a named format. YSS ships with built-in aliases for common formats like emails, dates, UUIDs, and URLs:

```yaml
email:
  $type: string
  $format: email

slug:
  $type: string
  $format: /^[a-z0-9]+(?:-[a-z0-9]+)*$/
```

To write it inline, use the `~` modifier. Note that inline only accepts named aliases, not raw regex:

```yaml
email: string, ~ email
```

**Built-in aliases**

| Category       | Aliases                                          |
|----------------|--------------------------------------------------|
| Dates & times  | `date-time`, `date`, `time`, `duration`          |
| Email          | `email`, `idn-email`                             |
| Hostname       | `hostname`, `idn-hostname`                       |
| IP addresses   | `ipv4`, `ipv6`                                   |
| URIs           | `uri`, `uri-reference`, `iri`                    |
| Other          | `uuid`, `json-pointer`, `regex`                  |

**$patterns**

You can define your own named formats at the root of the schema under `$patterns`.
Each entry is a named `/regex/` that can be used anywhere `$format` is accepted:

```yaml
$patterns:
  zip-code: /^\d{5}-?\d{3}$/
  phone:    /^\+?[0-9]{10,11}$/

address:
  zip:
    $type: string
    $format: zip-code
  contact: string, ~ phone
```

<br>

### $enum, $const

Use `$enum` to restrict a field to a list of allowed values:

```yaml
status:
  $type: string
  $enum: [active, inactive, banned]
```

If you need a field to match one specific value exactly, `$const` is the right tool:

```yaml
version:
  $const: 1
```

Both can be written inline:

```yaml
status:  string, enum [active, inactive, banned]
version: any, == 1
```

<br>

### $any_of

When a field can take more than one valid shape, `$any_of` lets you declare each possibility as a
separate branch. The value is valid if it matches at least one of them:

```yaml
user:
  name: string
  email: string
  country: string

  $any_of:
    - country: string, == US
    - shipping:
        $required: [method, cost]
        method: string, enum [standard, express, pickup]
        cost: number, >= 0
        tracking:
          code: string
          carrier: string
          status: string, enum [pending, shipped, delivered, returned]
```

Fields declared alongside `$any_of` are merged into every branch. If the same field appears both
outside and inside a branch, the branch definition takes precedence — the more specific rule wins.

<br>

### $one_of

`$one_of` works exactly like `$any_of`, but requires the value to match **exactly one** branch.
If more than one branch matches, validation fails:

```yaml
shipment:
  $one_of:
    - $required: [delivery]
      delivery:
        $required: [address]
        address: string
        carrier: string
        status: string, enum [pending, shipped, delivered, returned]
    - $required: [pickup]
      pickup:
        $required: [location]
        location: string
        slot: string
```

A value with both `delivery` and `pickup` present would be rejected — only one shape is allowed at a time.

<br>

### Inline syntax

Throughout this guide you've seen rules written inline alongside the type — a shorthand that keeps
simple schemas compact. Here's a summary of every rule that supports it:

| Inline modifier     | Equivalent rule       |
|---------------------|-----------------------|
| `Type1 \| Type2`   | `$type: [Type1, Type2]` |
| `array<Type>`       | `$type: array` + `$item: Type` |
| `>= n`              | `$gte: n`             |
| `> n`               | `$gt: n`              |
| `<= n`              | `$lte: n`             |
| `< n`               | `$lt: n`              |
| `size n`            | `$size: n`            |
| `size [min, max]`   | `$size: [min, max]`   |
| `~ alias`           | `$format: alias`      |
| `enum [a, b]`       | `$enum: [a, b]`       |
| `== val`            | `$const: val`         |
| `unique`            | `$unique: true`       |

<br>

## Schema reuse with YAML anchors

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

## Modular schemas

Schemas can be split across files. Each imported file is compiled in isolation.

### $imports

Declared at the root. Each key is a namespace and the value is a relative path:

```yaml
$imports:
  user:   ./user.yaml
  shared: ./shared/types.yaml
```

<br>

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

<br>

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

## Implementation Guide

To implement a YSS parser or validator in any language, see [SPECIFICATION.md](SPECIFICATION.md).

<br>

## License

[MIT](http://opensource.org/licenses/MIT)

Copyright (c) 2026-present, Alexandre Magro
