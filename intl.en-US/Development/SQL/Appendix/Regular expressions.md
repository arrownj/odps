---
keyword: regular expression
---

# Regular expressions

Regular expressions in MaxCompute SQL use the Perl Compatible Regular Expressions \(PCRE\) standards and are matched by character.

## RLIKE

You can use the RLIKE statement to match regular expressions. The following table lists the supported metacharacters.

|Metacharacter|Description|
|:------------|:----------|
|^|Match the beginning of a line.|
|$|Match the end of a line.|
|.|Match any characters.|
|\*|Match zero or more instances of the preceding character or character pattern.|
|+|Match one or more instances of the preceding character or character pattern.|
|?|Match zero or one instance of the preceding character or character pattern.|
|?|Match a modifier. If this character follows any one of other delimiters \(\*, +, ?, \{n\}, \{n,\}, or \{n,m\}\), the match pattern is non-greedy. In the non-greedy pattern, as few characters as possible are matched with the searched character string. In the default greedy pattern, as many characters as possible are matched with the searched character string.|
|A\|B|Match A or B.|
|\(abc\)\*|Match zero or more instances of the abc sequence.|
|\{n\} or \{m,n\}|The number of matches.|
|\[ab\]|Match any character \(a or b\) in the brackets. Fuzzy match is supported.|
|\[a-d\]|Match any of the following characters: a, b, c, and d.|
|\[^ab\]|Match any character that is not a or b. ^ indicates 'non'.|
|\[::\]|For more information, see the following table.|
|\\|The escape character.|
|\\n|n indicates a digit from 1 to 9 and is backward referenced.|
|\\d|Digits.|
|\\D|Non-digit characters.|

## POSIX character groups

|Character group|Description|Valid value|
|:--------------|:----------|:----------|
|\[\[:alnum:\]\]|Letters and digits|\[a-zA-Z0-9\]|
|\[\[:alpha:\]\]|Letters|\[a-zA-Z\]|
|\[\[:ascii:\]\]|ASCII characters|\[\\x00-\\x7F\]|
|\[\[:blank:\]\]|Spaces and tab characters|\[ \\t\]|
|\[\[:cntrl:\]\]|Control characters|\[\\x00-\\x1F\\x7F\]|
|\[\[:digit:\]\]|Digits|\[0-9\]|
|\[\[:graph:\]\]|Characters other than whitespace characters|\[\\x21-\\x7E\]|
|\[\[:lower:\]\]|Lowercase letters|\[a-z\]|
|\[\[:print:\]\]|\[:graph:\] and whitespace characters|\[\\x20-\\x7E\]|
|\[\[:punct:\]\]|Punctuation marks|\[\]\[!" \#$%&'\(\)\*+,. /:;<=\>? @\\^\_\`\{\|\}~-\]|
|\[\[:space:\]\]|Whitespace characters|\[ \\t\\r\\n\\v\\f\]|
|\[\[:upper:\]\]|Uppercase letters|\[A-Z\]|
|\[\[:xdigit:\]\]|Hexadecimal characters|\[A-Fa-f0-9\]|

## Escape characters

The system uses a backslash \(`\`\) as the escape character. A backslash \(`\`\) in a regular expression indicates a second escape. For example, a regular expression is used to match the `a+b` string. The plus sign \(`+`\) is a special character in the expression, and must be escaped. In the regular expression engine, the string is expressed as `a\\+b`. The system must perform an escape on the expression. Therefore, the expression that can match the string is `a\\\+b`.

Assume that the test\_dual table exists. Example:

```
select 'a+b' rlike 'a\\\+b' from test_dual;

+------+
| _c1  |
+------+
| true |
+------+
```

The `\` character is a special character in the regular expression engine. To match this character, it is expressed as `\\` in the engine. Then, the system performs an escape on the expression. As a result, the `\` character is expressed as `\\\`.

```
select 'a\\b', 'a\\b' rlike 'a\\\b' from test_dual;

+-----+------+
| _c0 | _c1  |
+-----+------+
| a\b | false |
+-----+------+
```

**Note:** If a MaxCompute SQL statement includes `a\\b`, `a\b` is displayed in the output because MaxCompute escapes the expression.

If a string includes tab characters, the system reads `\t` and stores it as one character. Therefore, it is a common character in regular expressions.

```
select 'a\tb', 'a\tb' rlike 'a\tb' from test_dual;

+---------+------+
| _c0     | _c1  |
+---------+------+
| a     b | true |
+---------+------+
```

