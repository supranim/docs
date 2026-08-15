---
title: "Validation"
description: "Validate input data and forms"
keywords: ["validation", "forms", "input data"]
---

## About Validation
Input validation is a crucial aspect of web development. Supranim does not provide a built-in validation library, but you can use `pkg/bag` to validate input data and forms. Find more about this package on its [GitHub repository](https://github.com/openpeeps/bag), or read the [API reference](https://openpeeps.github.io/bag)

### Example Usage
Here is an example of how to use `pkg/bag` for validating input data in a Supranim controller:

```nim
import pkg/supranim/controller
import pkg/bag

ctrl postAuthLogin:
  ## `POST` handler for user login
  withDBPool:
    withBag req.get:
      email: tEmail"auth.error.email"
      password: tPasswordStrength"weak.password"
      message: tTextarea"msg.empty":
        min: 10 or "msg.too.short"
        max: 500 or "msg.too.long"
      *remember: tCheckbox       # optional, default false
      csrf -> callback do(input: string) -> bool:
        result = validateToken("/auth/login", input)
```