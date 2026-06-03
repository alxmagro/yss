<p align="center">
  <img src="assets/logo.svg"/>
</p>

<p align="center">
  A <b>human-friendly</b> alternative to JSON Schema. Less noise, more clarity.
</p>

## Tools

| Language   | Package                                                              |
|------------|----------------------------------------------------------------------|
| JavaScript | [yss-validator-js](https://npmjs.com/package/yss-validator-js)      |

<br>

## Introduction

YSS *(YAML Schema Syntax)* is a schema language for validating JSON payloads, written in YAML.
It is designed to be simple to read, simple to write, and easy to reason about.

Schemas mirror the structure of the JSON they validate. Validation rules are expressed through $-prefixed keywords.

### Quick example

Schema:

```yaml
user:
  $required: [name, age]
  name: string
  age: integer, >= 18
```

Valid payload:

```json
{
  "name": "Alexandre",
  "age": 28
}
```

Invalid payload:

```json
{
  "name": "Alexandre",
  "age": 15
}
```

In this example:

- `name` and `age` are required.
- `name` must be a string.
- `age` must be an integer greater than or equal to 18.

<br>

## Syntax styles

YSS supports two equivalent ways of writing schemas:

1. Block syntax, where each rule is declared explicitly;
2. Inline syntax, a compact form for simple constraints;

Choose whichever style is more readable for your use case. Both forms produce the same schema.

### Block syntax

In block syntax, each rule is declared on its own line:

```yaml
age:
  $type: integer
  $gte: 18
  $lte: 120
```

This form is often preferred when a field contains many rules or nested structures.

### Inline syntax

In YSS, the `$type` rule can be written directly as the field value.

The following definitions are equivalent:

```yaml
age:
  $type: integer
```

```yaml
age: integer
```

When a field accepts multiple types, use ` | ` on inline syntax:

```yaml
nickname:
  $type: [string, null]
```

```yaml
nickname: string | null
```

Additional rules can then be appended after the type using a comma-separated syntax:

```yaml
age: integer, >= 18, <= 120
```

Which is equivalent to:

```yaml
age:
  $type: integer
  $gte: 18
  $lte: 120
```

**NOTE:** Unlike block syntax, inline syntax requires a type declaration. When no type validation
is desired, use `any`:

```yaml
metadata: any
```

This shorthand keeps simple schemas compact while preserving the same behavior as the equivalent
block form.

Throughout this guide, you'll learn about each rule in detail. The table below provides a quick
reference for every rule that supports inline syntax and its equivalent block form.

| Block syntax rule              | Equivalent inline modifier |
| ------------------------------ | -------------------------- |
| `$type: [Type1, Type2]`        | `Type1 \| Type2`  |
| `$type: array` + `$item: Type` | `array<Type>`     |
| `$gte: n`                      | `>= n`            |
| `$gt: n`                       | `> n`             |
| `$lte: n`                      | `<= n`            |
| `$lt: n`                       | `< n`             |
| `$multiple_of: n`              | `% n`             |
| `$size: n`                     | `size n`          |
| `$size: [min, max]`            | `size [min, max]` |
| `$format: alias`               | `~ alias`         |
| `$in: [a, b]`                  | `in [a, b]`       |
| `$not_in: [a, b]`              | `not_in [a, b]`   |
| `$const: val`                  | `== val`          |
| `$unique: true`                | `unique`          |

<br>

## Types

Every field in YSS is described using one or more types.

The following table lists all supported types.

| Type      | Validates                              |
|-----------|----------------------------------------|
| `string`  | UTF-8 string                           |
| `integer` | Whole number (no decimal part)         |
| `number`  | Any number, including decimals         |
| `boolean` | `true` or `false`                      |
| `null`    | JSON null value                        |
| `object`  | Key-value pairs (inferred from fields) |
| `array`   | Ordered list                           |
| `any`     | Any value, no type check               |

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
| `$gt`         | integer, number         | Value must be > n                                  |
| `$gte`        | integer, number         | Value must be >= n                                 |
| `$lt`         | integer, number         | Value must be < n                                  |
| `$lte`        | integer, number         | Value must be <= n                                 |
| `$multiple_of`| integer, number         | Value must be a multiple of n                      |
| `$format`     | string                  | Named alias or `/regex/` the value must match      |
| `$in`         | string, integer, number | Value must be one of the listed values             |
| `$not_in`     | string, integer, number | Value must not be any of the listed values         |
| `$const`      | string, integer, number, boolean, null | Exact value match               |
| `$any_of`  | any                     | Value must match at least one of the listed schemas|
| `$one_of`  | any                     | Value must match exactly one of the listed schemas |
| `$all_of`  | any                     | Value must match all of the listed schemas         |

> **NOTE:** For the expected error shape of each rule, see [Error Messages](docs/ERRORS.md).

<br>

### $type

`$type` defines what kind of value a field accepts.

Type validation is always performed first. If the value does not match the declared type, no other
rules are evaluated for that field.

In its explicit form:

```yaml
name:
  $type: string
```

The same definition can be written using inline syntax:

```yaml
name: string
```

Multiple accepted types can be declared as an array:

```yaml
name:
  $type: [string, null]
```

Or using the inline union operator:

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

Objects provide the following rules:

- `$required` — marks fields as mandatory;
- `$strict` — rejects undeclared fields;

**Arrays**

Array schemas are declared with `$type: array`:

```yaml
emails:
  $type: array
```

Arrays provide the following rules:

- `$item` — validates every element;
- `$at` — validates specific positions;
- `$contains` — requires matching elements;
- `$unique` — prevents duplicate values;
- `$size` — constrains the number of elements;

<br>

### $required

By default, every field in an object is optional — if it's absent from the payload,
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

Or just `unique` in inline syntax:

```yaml
tags: array<string>, unique
```

<br>

### $size

Use `$size` to constrain how many elements an array can have. It accepts an exact number or a
`[min, max]` range, where `null` (or `~` in YAML shorthand) means unbound:

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

### $multiple_of

Use `$multiple_of` to require that a numeric value is a multiple of a given number.
It works with both integers and decimals, and accepts negative values:

```yaml
quantity:
  $type: integer
  $multiple_of: 10

price:
  $type: number
  $multiple_of: 0.5
```

You can also write it inline:

```yaml
quantity: integer, % 10
price:    number, % 0.5
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

### $in, $not_in, $const

Use `$in` to restrict a field to a list of allowed values:

```yaml
status:
  $type: string
  $in: [active, inactive, banned]
```

Use `$not_in` to reject specific values:

```yaml
role:
  $type: string
  $not_in: [admin, root]
```

If you need a field to match one specific value exactly, `$const` is the right tool:

```yaml
version:
  $const: 1
```

All three can be written inline:

```yaml
status:  string, in [active, inactive, banned]
role:    string, not_in [admin, root]
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
    - $required: [shipping]
      shipping:
        $required: [method, cost]
        method: string, in [standard, express, pickup]
        cost: number, >= 0
        tracking:
          code: string
          carrier: string
          status: string, in [pending, shipped, delivered, returned]
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
        status: string, in [pending, shipped, delivered, returned]
    - $required: [pickup]
      pickup:
        $required: [location]
        location: string
        slot: string
```

A value with both `delivery` and `pickup` present would be rejected — only one shape is allowed at a time.

<br>

### $all_of

Use `$all_of` when a value must satisfy **all** of the listed schemas at once. This is useful in two situations.

The first is when you need to apply the same rule more than once with different arguments:

```yaml
tags:
  $type: array
  $all_of:
    - $contains: string, == featured
    - $contains: string, == sale
```

The second is schema composition, where each branch comes from a separate file via `$ref`:

```yaml
character:
  $all_of:
    - $ref: identity
    - $ref: combat
    - $ref: origin
```

> `$ref` and modular schemas are covered in detail further below.

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

To implement a YSS parser or validator in any language, see [SPECS_GUIDE.md](docs/SPECS_GUIDE.md).

<br>

## License

[MIT](http://opensource.org/licenses/MIT)

Copyright (c) 2026-present, Alexandre Magro
