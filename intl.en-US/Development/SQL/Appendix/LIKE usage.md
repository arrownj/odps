---
keyword: LIKE
---

# LIKE usage

This topic describes how to use the LIKE operator for character matching.

In LIKE character matching, `%` is used to match a string that consists of zero or more characters, and `_` is used to match a single character.

To match the `%` or `_` character itself, you must escape the character. Use `\\%` or `\\_` to match the `%` or `_` character.

```
'abcd' like 'ab%' -- true
'abcd' like 'ab_' -- false
'ab_cde' like 'ab\\_c%'; -- true
```

**Note:** MaxCompute SQL statements support only UTF-8 character sets. If data is encoded in another format, the calculation result may be incorrect.

