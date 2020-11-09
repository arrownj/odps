---
keyword: escape character
---

# Escape character

In MaxCompute SQL, the backslash character \(`\`\) is an escape character that invokes an alternative representation on the following characters in a character sequence or indicates a literal interpretation of these characters.

If the backslash character \(\\\) in a string constant is followed by three valid octal digits in the range from 001 to 177, the combination of the backslash and the three digits is interpreted into an ASCII character.

## Escape sequence

MaxCompute SQL uses the backlash character \(\\\) as an escape character for the escape sequences in the following table.

|Escape sequence|Represented character|
|:--------------|:--------------------|
|\\b|backspace|
|\\t|tab|
|\\n|newline|
|\\r|carriage-return|
|\\'|Single quotation mark|
|\\"|Double quotation mark|
|\\ \\|Backslash|
|\\;|Semicolon|
|\\Z|control-Z|
|\\0 or \\00|Terminator|

## Example

-   Use an escape sequence to represent a string constant.

    In MaxCompute SQL, you can represent a string constant by using single quotation marks or double quotation marks. A string constant that contains a double quotation mark can be enclosed in a pair of single quotation marks. A string constant that contains a single quotation mark can be enclosed in a pair of double quotation marks. You can also use the backslash \(\\\) as an escape character to represent a string constant. The following example shows two ways to represent a string constant:

    ```
    "I'm a happy manong."
    'I\'m a happy manong.'
    ```

-   Use an escape sequence to represent a special character. For example, run the following command:

    ```
    select length('a\tb');
    ```

    The preceding command returns 3, which indicates that the string contains three characters and that `\t` is treated as one character.

    ```
    select 'a\ab',length('a\ab');
    ```

    The preceding command returns aab, 3, which indicates that `\a` is treated as letter a.


