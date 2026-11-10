---
title: "Validation"
description: "Validate input data and forms"
keywords: ["validation", "forms", "input data"]
---

## About Validation
Input validation is a crucial aspect of web development. Supranim ships a small dependency-free validator in `supranim/support/validate` for the common web cases. For advanced validation (IDN, IBAN, EAN, ports, full URI schemes, entropy-based passwords) use `pkg/valido`, and for form DSLs with custom messages use `pkg/bag`.

### Core validator
`validate(fields, rules)` checks a `TableRef[string, string]` — as returned by `req.getFieldsTable()` or `req.getQueryTable()` — against a list of rules and collects every failure:

```nim
import supranim/support/validate

ctrl postRegister:
  let fields = req.getFieldsTable()
  let res = validate(fields, [
    required("email"),
    email("email"),
    required("password"),
    minLen("password", 8),
    strongPassword("password"),
    equalsField("password_confirm", "password"),
  ])
  if not res.ok:
    # res.errors: seq[ValidationError{field, rule, message}]
    # res.failedFields(): seq[string]
    # res.errorFor("email"): Option[string]
    respond(Http400, "invalid form")
```

Available rules: `required`, `email`, `minLen`, `maxLen`, `equalsField`, `strongPassword` (length + lower + upper + digit), `numeric`, `int`, `alphaNum`, `slugRule`, `uuidRule` and `customRule` with your own callback. `withValidation(fields, rules): onSuccess do: onFailure` runs one of two blocks with `validation` in scope.

### Advanced validation with `pkg/bag`
Find more about this package on its [GitHub repository](https://github.com/openpeeps/bag), or read the [API reference](https://openpeeps.github.io/bag)

### Example Usage
Here is an example of how to use `pkg/bag` for validating input data in a Supranim controller. 

```nim
import pkg/supranim/controller
import pkg/bag

ctrl postAuthLogin:
  ## `POST` handler for user login
  withDBPool:
    withBag req.getFields:
      email: tEmail"auth.error.email"
      password: tPasswordStrength"weak.password"
      message: tTextarea"msg.empty":
        min: 10 or "msg.too.short"
        max: 500 or "msg.too.long"
      *remember: tCheckbox       # optional, default false
      csrf -> callback do(input: string) -> bool:
        result = validateToken("/auth/login", input)
```