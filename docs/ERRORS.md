# Error Messages

Expected error shapes for each rule.

---

### $type

```js
{
  path: "field",
  code: "type",
  message: "Unexpected type",
  data: {
    value: 42,
    expected: "string"
  }
}
```

### $required

```js
{
  path: "field",
  code: "required",
  message: "Missing required property `field`"
}
```

### $strict

```js
{
  path: "field",
  code: "strict",
  message: "Unexpected property `field`"
}
```

### $size

Exact:

```js
{
  path: "field",
  code: "size_exact",
  message: "Size must be exactly `5`",
  data: {
    value: "hi",
    size: 2,
    expected: 5
  }
}
```

Min:

```js
{
  path: "field",
  code: "size_min",
  message: "Minimum size is `2`",
  data: {
    value: "h",
    size: 1,
    min: 2
  }
}
```

Max:

```js
{
  path: "field",
  code: "size_max",
  message: "Maximum size is `100`",
  data: {
    value: "...",
    size: 123,
    max: 100
  }
}
```

### $gt

```js
{
  path: "field",
  code: "gt",
  message: "Value must be greater than `0`",
  data: {
    value: 0,
    gt: 0
  }
}
```

### $gte

```js
{
  path: "field",
  code: "gte",
  message: "Value must be greater than or equal to `18`",
  data: {
    value: 17,
    gte: 18
  }
}
```

### $lt

```js
{
  path: "field",
  code: "lt",
  message: "Value must be less than `10`",
  data: {
    value: 10,
    lt: 10
  }
}
```

### $lte

```js
{
  path: "field",
  code: "lte",
  message: "Value must be less than or equal to `100`",
  data: {
    value: 101,
    lte: 100
  }
}
```

### $multiple_of

```js
{
  path: "field",
  code: "multiple_of",
  message: "Value must be a multiple of `10`",
  data: {
    value: 25,
    multiple_of: 10
  }
}
```

### $format

```js
{
  path: "field",
  code: "format",
  message: "Value does not match required format",
  data: {
    value: "not-an-email",
    format: "email"
  }
}
```

### $in

```js
{
  path: "field",
  code: "in",
  message: "Value `cancelled` is not allowed",
  data: {
    value: "cancelled",
    in: ["active", "inactive", "pending"]
  }
}
```

### $not_in

```js
{
  path: "field",
  code: "not_in",
  message: "Value is not allowed",
  data: {
    value: "banned",
    not_in: ["deleted", "banned"]
  }
}
```

### $const

```js
{
  path: "field",
  code: "const",
  message: "Value must be `admin`",
  data: {
    value: "user",
    const: "admin"
  }
}
```

### $contains

Min:

```js
{
  path: "field",
  code: "contains_min",
  message: "Array must contain at least `2` matching items",
  data: {
    quantity: [2, null]
  }
}
```

Max:

```js
{
  path: "field",
  code: "contains_max",
  message: "Array must contain at most `1` matching items",
  data: {
    quantity: [null, 1]
  }
}
```

Exact:

```js
{
  path: "field",
  code: "contains_exact",
  message: "Array must contain exactly `2` matching items",
  data: {
    quantity: 2
  }
}
```

### $unique

```js
{
  path: "field",
  code: "unique",
  message: "Array contains duplicated items"
}
```

### $any_of

```js
{
  path: "field",
  code: "any_of",
  message: "Value does not match any condition"
}
```

### $one_of

No branch matches:

```js
{
  path: "field",
  code: "one_of",
  message: "Value does not match any condition"
}
```

More than one branch matches:

```js
{
  path: "field",
  code: "one_of_multiple",
  message: "Value matches more than one condition",
  data: {
    matches_at: [0, 1]
  }
}
```

### $all_of

```js
{
  path: "field",
  code: "all_of",
  message: "Value does not match all conditions",
  data: {
    failed_at: 0
  }
}
```

### $dependencies

```js
{
  path: "field",
  code: "dependencies",
  message: "Value does not match all conditions",
  data: {
    trigger: "credit_card",
    missing: ["billing_address"]
  }
}
```
