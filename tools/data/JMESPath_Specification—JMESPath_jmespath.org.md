---
created: 2025-02-20T15:00:32 (UTC +11:00)
tags: []
source: https://jmespath.org/specification.html
author: 
---

# JMESPath Specification — JMESPath

> ## Excerpt
> This document describes the specification for jmespath.

---
This document describes the specification for jmespath.

If you’d like an introduction to the JMESPath language, see the [JMESPath Tutorial](https://jmespath.org/tutorial.html) and the [JMESPath Examples](https://jmespath.org/examples.html) page.

In the specification, examples are shown through the use of a `search` function. The syntax for this function is:

```
<span></span><span>search</span><span>(</span><span>&lt;</span><span>jmespath</span> <span>expr</span><span>&gt;</span><span>,</span> <span>&lt;</span><span>JSON</span> <span>document</span><span>&gt;</span><span>)</span> <span>-&gt;</span> <span>&lt;</span><span>return</span> <span>value</span><span>&gt;</span>
```

For simplicity, the jmespath expression and the JSON document are not quoted. For example:

```
<span></span><span>search</span><span>(</span><span>foo</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>"bar"</span><span>})</span> <span>-&gt;</span> <span>"bar"</span>
```

The result of applying a JMESPath expression against a JSON document will **always** result in valid JSON, provided there are no errors during the evaluation process. Structured data in, structured data out.

This also means that, with the exception of JMESPath expression types, JMESPath only supports the same types supported by JSON:

-   number (integers and double-precision floating-point format in JSON)
    
-   string
    
-   boolean (`true` or `false`)
    
-   array (an ordered, sequence of values)
    
-   object (an unordered collection of key value pairs)
    
-   null
    

Expression types are discussed in the [Function Expressions](https://jmespath.org/specification.html#functions) section.

Implementations can map the corresponding JSON types to their language equivalent. For example, a JSON `null` could map to `None` in python, and `nil` in ruby and go.

## Grammar

The grammar is specified using ABNF, as described in [RFC4234](https://tools.ietf.org/html/rfc4234)

```
<span></span>expression        = sub-expression / index-expression  / comparator-expression
expression        =/ or-expression / identifier
expression        =/ and-expression / not-expression / paren-expression
expression        =/ "*" / multi-select-list / multi-select-hash / literal
expression        =/ function-expression / pipe-expression / raw-string
expression        =/ current-node
sub-expression    = expression "." ( identifier /
                                     multi-select-list /
                                     multi-select-hash /
                                     function-expression /
                                     "*" )
pipe-expression   = expression "|" expression
or-expression     = expression "||" expression
and-expression    = expression "&amp;&amp;" expression
not-expression    = "!" expression
paren-expression  = "(" expression ")"
index-expression  = expression bracket-specifier / bracket-specifier
multi-select-list = "[" ( expression *( "," expression ) ) "]"
multi-select-hash = "{" ( keyval-expr *( "," keyval-expr ) ) "}"
keyval-expr       = identifier ":" expression
bracket-specifier = "[" (number / "*" / slice-expression) "]" / "[]"
bracket-specifier =/ "[?" expression "]"
comparator-expression = expression comparator expression
slice-expression  = [number] ":" [number] [ ":" [number] ]
comparator        = "&lt;" / "&lt;=" / "==" / "&gt;=" / "&gt;" / "!="
function-expression = unquoted-string  (
                        no-args  /
                        one-or-more-args )
no-args             = "(" ")"
one-or-more-args    = "(" ( function-arg *( "," function-arg ) ) ")"
function-arg        = expression / expression-type
current-node        = "@"
expression-type     = "&amp;" expression

raw-string        = "'" *raw-string-char "'"
raw-string-char   = (%x20-26 / %x28-5B / %x5D-10FFFF) / preserved-escape /
                      raw-string-escape
preserved-escape  = escape (%x20-26 / %28-5B / %x5D-10FFFF)
raw-string-escape = escape ("'" / escape)
literal           = "`" json-value "`"
unescaped-literal = %x20-21 /       ; space !
                        %x23-5B /   ; # - [
                        %x5D-5F /   ; ] ^ _
                        %x61-7A     ; a-z
                        %x7C-10FFFF ; |}~ ...
escaped-literal   = escaped-char / (escape %x60)
number            = ["-"]1*digit
digit             = %x30-39
identifier        = unquoted-string / quoted-string
unquoted-string   = (%x41-5A / %x61-7A / %x5F) *(  ; A-Za-z_
                        %x30-39  /  ; 0-9
                        %x41-5A /  ; A-Z
                        %x5F    /  ; _
                        %x61-7A)   ; a-z
quoted-string     = quote 1*(unescaped-char / escaped-char) quote
unescaped-char    = %x20-21 / %x23-5B / %x5D-10FFFF
escape            = %x5C   ; Back slash: \
quote             = %x22   ; Double quote: '"'
escaped-char      = escape (
                        %x22 /          ; "    quotation mark  U+0022
                        %x5C /          ; \    reverse solidus U+005C
                        %x2F /          ; /    solidus         U+002F
                        %x62 /          ; b    backspace       U+0008
                        %x66 /          ; f    form feed       U+000C
                        %x6E /          ; n    line feed       U+000A
                        %x72 /          ; r    carriage return U+000D
                        %x74 /          ; t    tab             U+0009
                        %x75 4HEXDIG )  ; uXXXX                U+XXXX

; The ``json-value`` is any valid JSON value with the one exception that the
; ``%x60`` character must be escaped.  While it's encouraged that implementations
; use any existing JSON parser for this grammar rule (after handling the escaped
; literal characters), the grammar rule is shown below for completeness::

json-value = false / null / true / json-object / json-array /
             json-number / json-quoted-string
false = %x66.61.6c.73.65   ; false
null  = %x6e.75.6c.6c      ; null
true  = %x74.72.75.65      ; true
json-quoted-string = %x22 1*(unescaped-literal / escaped-literal) %x22
begin-array     = ws %x5B ws  ; [ left square bracket
begin-object    = ws %x7B ws  ; { left curly bracket
end-array       = ws %x5D ws  ; ] right square bracket
end-object      = ws %x7D ws  ; } right curly bracket
name-separator  = ws %x3A ws  ; : colon
value-separator = ws %x2C ws  ; , comma
ws              = *(%x20 /              ; Space
                    %x09 /              ; Horizontal tab
                    %x0A /              ; Line feed or New line
                    %x0D                ; Carriage return
                   )
json-object = begin-object [ member *( value-separator member ) ] end-object
member = quoted-string name-separator json-value
json-array = begin-array [ json-value *( value-separator json-value ) ] end-array
json-number = [ minus ] int [ frac ] [ exp ]
decimal-point = %x2E       ; .
digit1-9 = %x31-39         ; 1-9
e = %x65 / %x45            ; e E
exp = e [ minus / plus ] 1*DIGIT
frac = decimal-point 1*DIGIT
int = zero / ( digit1-9 *DIGIT )
minus = %x2D               ; -
plus = %x2B                ; +
zero = %x30                ; 0
```

In addition to the grammar, there is the following token precedence that goes from weakest to tightest binding:

-   pipe: `|`
    
-   or: `||`
    
-   and: `&&`
    
-   unary not: `!`
    
-   rbracket: `]`
    

## Identifiers

```
<span></span><span>identifier</span>        <span>=</span> <span>unquoted</span><span>-</span><span>string</span> <span>/</span> <span>quoted</span><span>-</span><span>string</span>
<span>unquoted</span><span>-</span><span>string</span>   <span>=</span> <span>(</span><span>%</span><span>x41</span><span>-</span><span>5</span><span>A</span> <span>/</span> <span>%</span><span>x61</span><span>-</span><span>7</span><span>A</span> <span>/</span> <span>%</span><span>x5F</span><span>)</span> <span>*</span><span>(</span>  <span>;</span> <span>A</span><span>-</span><span>Za</span><span>-</span><span>z_</span>
                        <span>%</span><span>x30</span><span>-</span><span>39</span>  <span>/</span>  <span>;</span> <span>0</span><span>-</span><span>9</span>
                        <span>%</span><span>x41</span><span>-</span><span>5</span><span>A</span> <span>/</span>  <span>;</span> <span>A</span><span>-</span><span>Z</span>
                        <span>%</span><span>x5F</span>    <span>/</span>  <span>;</span> <span>_</span>
                        <span>%</span><span>x61</span><span>-</span><span>7</span><span>A</span><span>)</span>   <span>;</span> <span>a</span><span>-</span><span>z</span>
<span>quoted</span><span>-</span><span>string</span>     <span>=</span> <span>quote</span> <span>1</span><span>*</span><span>(</span><span>unescaped</span><span>-</span><span>char</span> <span>/</span> <span>escaped</span><span>-</span><span>char</span><span>)</span> <span>quote</span>
<span>unescaped</span><span>-</span><span>char</span>    <span>=</span> <span>%</span><span>x20</span><span>-</span><span>21</span> <span>/</span> <span>%</span><span>x23</span><span>-</span><span>5</span><span>B</span> <span>/</span> <span>%</span><span>x5D</span><span>-</span><span>10</span><span>FFFF</span>
<span>escape</span>            <span>=</span> <span>%</span><span>x5C</span>   <span>;</span> <span>Back</span> <span>slash</span><span>:</span> \
<span>quote</span>             <span>=</span> <span>%</span><span>x22</span>   <span>;</span> <span>Double</span> <span>quote</span><span>:</span> <span>'"'</span>
<span>escaped</span><span>-</span><span>char</span>      <span>=</span> <span>escape</span> <span>(</span>
                        <span>%</span><span>x22</span> <span>/</span>          <span>;</span> <span>"    quotation mark  U+0022</span>
                        <span>%</span><span>x5C</span> <span>/</span>          <span>;</span> \    <span>reverse</span> <span>solidus</span> <span>U</span><span>+</span><span>005</span><span>C</span>
                        <span>%</span><span>x2F</span> <span>/</span>          <span>;</span> <span>/</span>    <span>solidus</span>         <span>U</span><span>+</span><span>002</span><span>F</span>
                        <span>%</span><span>x62</span> <span>/</span>          <span>;</span> <span>b</span>    <span>backspace</span>       <span>U</span><span>+</span><span>0008</span>
                        <span>%</span><span>x66</span> <span>/</span>          <span>;</span> <span>f</span>    <span>form</span> <span>feed</span>       <span>U</span><span>+</span><span>000</span><span>C</span>
                        <span>%</span><span>x6E</span> <span>/</span>          <span>;</span> <span>n</span>    <span>line</span> <span>feed</span>       <span>U</span><span>+</span><span>000</span><span>A</span>
                        <span>%</span><span>x72</span> <span>/</span>          <span>;</span> <span>r</span>    <span>carriage</span> <span>return</span> <span>U</span><span>+</span><span>000</span><span>D</span>
                        <span>%</span><span>x74</span> <span>/</span>          <span>;</span> <span>t</span>    <span>tab</span>             <span>U</span><span>+</span><span>0009</span>
                        <span>%</span><span>x75</span> <span>4</span><span>HEXDIG</span> <span>)</span>  <span>;</span> <span>uXXXX</span>                <span>U</span><span>+</span><span>XXXX</span>
```

An `identifier` is the most basic expression and can be used to extract a single element from a JSON document. The return value for an `identifier` is the value associated with the identifier. If the `identifier` does not exist in the JSON document, than a `null` value is returned.

From the grammar rule listed above identifiers can be one or more characters, and must start with `A-Za-z_`.

An identifier can also be quoted. This is necessary when an identifier has characters not specified in the `unquoted-string` grammar rule. In this situation, an identifier is specified with a double quote, followed by any number of `unescaped-char` or `escaped-char` characters, followed by a double quote. The `quoted-string` rule is the same grammar rule as a JSON string, so any valid string can be used between double quoted, include JSON supported escape sequences, and six character unicode escape sequences.

Note that any identifier that does not start with `A-Za-z_` **must** be quoted.

### Examples

```
<span></span><span>search</span><span>(</span><span>foo</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>null</span>
<span>search</span><span>(</span><span>foo</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>]})</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>]</span>
<span>search</span><span>(</span><span>"with space"</span><span>,</span> <span>{</span><span>"with space"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>"special chars: !@#"</span><span>,</span> <span>{</span><span>"special chars: !@#"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>"quote</span><span>\"</span><span>char"</span><span>,</span> <span>{</span><span>"quote</span><span>\"</span><span>char"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>"</span><span>\u2713</span><span>"</span><span>,</span> <span>{</span><span>"</span><span>\u2713</span><span>"</span><span>:</span> <span>"value"</span><span>})</span> <span>-&gt;</span> <span>"value"</span>
```

## Errors

Errors may be raised during the JMEspath evaluation process. How and when errors are raised is implementation specific, but implementations should indicate to the caller when errors have occurred.

The following errors are defined:

-   `invalid-type` is raised when an invalid type is encountered during the evaluation process.
    
-   `invalid-value` is raised when an invalid value is encountered during the evaluation process.
    
-   `unknown-function` is raised when an unknown function is encountered during the evaluation process.
    
-   `invalid-arity` is raised when an invalid number of function arguments is encountered during the evaluation process.
    

## SubExpressions

```
<span></span><span>sub</span><span>-</span><span>expression</span>    <span>=</span> <span>expression</span> <span>"."</span> <span>(</span> <span>identifier</span> <span>/</span>
                                     <span>multi</span><span>-</span><span>select</span><span>-</span><span>list</span> <span>/</span>
                                     <span>multi</span><span>-</span><span>select</span><span>-</span><span>hash</span> <span>/</span>
                                     <span>function</span><span>-</span><span>expression</span> <span>/</span>
                                     <span>"*"</span> <span>)</span>
```

A subexpression is a combination of two expressions separated by the ‘.’ char. A subexpression is evaluted as follows:

-   Evaluate the expression on the left with the original JSON document.
    
-   Evaluate the expression on the right with the result of the left expression evaluation.
    

In pseudocode:

```
<span></span><span>left</span><span>-</span><span>evaluation</span> <span>=</span> <span>search</span><span>(</span><span>left</span><span>-</span><span>expression</span><span>,</span> <span>original</span><span>-</span><span>json</span><span>-</span><span>document</span><span>)</span>
<span>result</span> <span>=</span> <span>search</span><span>(</span><span>right</span><span>-</span><span>expression</span><span>,</span> <span>left</span><span>-</span><span>evaluation</span><span>)</span>
```

A subexpression is itself an expression, so there can be multiple levels of subexpressions: `grandparent.parent.child`.

### Examples

Given a JSON document: `{"foo": {"bar": "baz"}}`, and a jmespath expression: `foo.bar`, the evaluation process would be:

```
<span></span><span>left</span><span>-</span><span>evaluation</span> <span>=</span> <span>search</span><span>(</span><span>"foo"</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>"baz"</span><span>}})</span> <span>-&gt;</span> <span>{</span><span>"bar"</span><span>:</span> <span>"baz"</span><span>}</span>
<span>result</span> <span>=</span> <span>search</span><span>(</span><span>"bar"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>"baz"</span><span>})</span> <span>-&gt;</span> <span>"baz"</span>
```

The final result in this example is `"baz"`.

Additional examples:

```
<span></span><span>search</span><span>(</span><span>foo</span><span>.</span><span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>"value"</span><span>}})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>foo</span><span>.</span><span>"bar"</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>"value"</span><span>}})</span> <span>-&gt;</span> <span>"value"</span>
<span>search</span><span>(</span><span>foo</span><span>.</span><span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"baz"</span><span>:</span> <span>"value"</span><span>}})</span> <span>-&gt;</span> <span>null</span>
<span>search</span><span>(</span><span>foo</span><span>.</span><span>bar</span><span>.</span><span>baz</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>{</span><span>"baz"</span><span>:</span> <span>"value"</span><span>}}})</span> <span>-&gt;</span> <span>"value"</span>
```

## Index Expressions

```
<span></span><span>index</span><span>-</span><span>expression</span>  <span>=</span> <span>expression</span> <span>bracket</span><span>-</span><span>specifier</span> <span>/</span> <span>bracket</span><span>-</span><span>specifier</span>
<span>bracket</span><span>-</span><span>specifier</span> <span>=</span> <span>"["</span> <span>(</span><span>number</span> <span>/</span> <span>"*"</span><span>)</span> <span>"]"</span> <span>/</span> <span>"[]"</span>
```

An index expression is used to access elements in a list. Indexing is 0 based, the index of 0 refers to the first element of the list. A negative number is a valid index. A negative number indicates that indexing is relative to the end of the list, specifically:

```
<span></span><span>negative</span><span>-</span><span>index</span> <span>==</span> <span>(</span><span>length</span> <span>of</span> <span>array</span><span>)</span> <span>+</span> <span>negative</span><span>-</span><span>index</span>
```

Given an array of length `N`, an index of `-1` would be equal to a positive index of `N - 1`, which is the last element of the list. If an index expression refers to an index that is greater than the length of the array, a value of `null` is returned.

For the grammar rule `expression bracket-specifier` the `expression` is first evaluated, and then return value from the `expression` is given as input to the `bracket-specifier`.

Using a “\*” character within a `bracket-specifier` is discussed below in the `wildcard expressions` section.

### Slices

```
<span></span><span>slice</span><span>-</span><span>expression</span>  <span>=</span> <span>[</span><span>number</span><span>]</span> <span>":"</span> <span>[</span><span>number</span><span>]</span> <span>[</span> <span>":"</span> <span>[</span><span>number</span><span>]</span> <span>]</span>
```

A slice expression allows you to select a contiguous subset of an array. A slice has a `start`, `stop`, and `step` value. The general form of a slice is `[start:stop:step]`, but each component is optional and can be omitted.

Note

Slices in JMESPath have the same semantics as python slices. If you’re familiar with python slices, you’re familiar with JMESPath slices.

Given a `start`, `stop`, and `step` value, the sub elements in an array are extracted as follows:

-   The first element in the extracted array is the index denoted by `start`.
    
-   The last element in the extracted array is the index denoted by `end - 1`.
    
-   The `step` value determines how many indices to skip after each element is selected from the array. An array of 1 (the default step) will not skip any indices. A step value of 2 will skip every other index while extracting elements from an array. A step value of -1 will extract values in reverse order from the array.
    

Slice expressions adhere to the following rules:

-   If a negative start position is given, it is calculated as the total length of the array plus the given start position.
    
-   If no start position is given, it is assumed to be 0 if the given step is greater than 0 or the end of the array if the given step is less than 0.
    
-   If a negative stop position is given, it is calculated as the total length of the array plus the given stop position.
    
-   If no stop position is given, it is assumed to be the length of the array if the given step is greater than 0 or 0 if the given step is less than 0.
    
-   If the given step is omitted, it it assumed to be 1.
    
-   If the given step is 0, an `invalid-value` error MUST be raised.
    
-   If the element being sliced is not an array, the result is `null`.
    
-   If the element being sliced is an array and yields no results, the result MUST be an empty array.
    

### Examples

```
<span></span><span>search</span><span>([</span><span>0</span><span>:</span><span>4</span><span>:</span><span>1</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>]</span>
<span>search</span><span>([</span><span>0</span><span>:</span><span>4</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>]</span>
<span>search</span><span>([</span><span>0</span><span>:</span><span>3</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>]</span>
<span>search</span><span>([:</span><span>2</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>]</span>
<span>search</span><span>([::</span><span>2</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>0</span><span>,</span> <span>2</span><span>]</span>
<span>search</span><span>([::</span><span>-</span><span>1</span><span>],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>3</span><span>,</span> <span>2</span><span>,</span> <span>1</span><span>,</span> <span>0</span><span>]</span>
<span>search</span><span>([</span><span>-</span><span>2</span><span>:],</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>])</span> <span>-&gt;</span> <span>[</span><span>2</span><span>,</span> <span>3</span><span>]</span>
```

### Flatten Operator

When the character sequence `[]` is provided as a bracket specifier, then a flattening operation occurs on the current result. The flattening operator will merge sublists in the current result into a single list. The flattening operator has the following semantics:

-   Create an empty result list.
    
-   Iterate over the elements of the current result.
    
-   If the current element is not a list, add to the end of the result list.
    
-   If the current element is a list, add each element of the current element to the end of the result list.
    
-   The result list is now the new current result.
    

Once the flattening operation has been performed, subsequent operations are projected onto the flattened list with the same semantics as a wildcard expression. Thus the difference between `[*]` and `[]` is that `[]` will first flatten sublists in the current result.

### Examples

```
<span></span><span>search</span><span>([</span><span>0</span><span>],</span> <span>[</span><span>"first"</span><span>,</span> <span>"second"</span><span>,</span> <span>"third"</span><span>])</span> <span>-&gt;</span> <span>"first"</span>
<span>search</span><span>([</span><span>-</span><span>1</span><span>],</span> <span>[</span><span>"first"</span><span>,</span> <span>"second"</span><span>,</span> <span>"third"</span><span>])</span> <span>-&gt;</span> <span>"third"</span>
<span>search</span><span>([</span><span>100</span><span>],</span> <span>[</span><span>"first"</span><span>,</span> <span>"second"</span><span>,</span> <span>"third"</span><span>])</span> <span>-&gt;</span> <span>null</span>
<span>search</span><span>(</span><span>foo</span><span>[</span><span>0</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>[</span><span>"first"</span><span>,</span> <span>"second"</span><span>,</span> <span>"third"</span><span>])</span> <span>-&gt;</span> <span>"first"</span>
<span>search</span><span>(</span><span>foo</span><span>[</span><span>100</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>[</span><span>"first"</span><span>,</span> <span>"second"</span><span>,</span> <span>"third"</span><span>])</span> <span>-&gt;</span> <span>null</span>
<span>search</span><span>(</span><span>foo</span><span>[</span><span>0</span><span>][</span><span>0</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>[[</span><span>0</span><span>,</span> <span>1</span><span>],</span> <span>[</span><span>1</span><span>,</span> <span>2</span><span>]]})</span> <span>-&gt;</span> <span>0</span>
```

## Or Expressions

```
<span></span><span>or</span><span>-</span><span>expression</span>     <span>=</span> <span>expression</span> <span>"||"</span> <span>expression</span>
```

An or expression will evaluate to either the left expression or the right expression. If the evaluation of the left expression is not false it is used as the return value. If the evaluation of the right expression is not false it is used as the return value. If neither the left or right expression are non-null, then a value of null is returned. A false value corresponds to any of the following conditions:

-   Empty list: `[]`
    
-   Empty object: `{}`
    
-   Empty string: `""`
    
-   False boolean: `false`
    
-   Null value: `null`
    

A true value corresponds to any value that is not false.

### Examples

```
<span></span><span>search</span><span>(</span><span>foo</span> <span>||</span> <span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>"foo-value"</span><span>})</span> <span>-&gt;</span> <span>"foo-value"</span>
<span>search</span><span>(</span><span>foo</span> <span>||</span> <span>bar</span><span>,</span> <span>{</span><span>"bar"</span><span>:</span> <span>"bar-value"</span><span>})</span> <span>-&gt;</span> <span>"bar-value"</span>
<span>search</span><span>(</span><span>foo</span> <span>||</span> <span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>"foo-value"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"bar-value"</span><span>})</span> <span>-&gt;</span> <span>"foo-value"</span>
<span>search</span><span>(</span><span>foo</span> <span>||</span> <span>bar</span><span>,</span> <span>{</span><span>"baz"</span><span>:</span> <span>"baz-value"</span><span>})</span> <span>-&gt;</span> <span>null</span>
<span>search</span><span>(</span><span>foo</span> <span>||</span> <span>bar</span> <span>||</span> <span>baz</span><span>,</span> <span>{</span><span>"baz"</span><span>:</span> <span>"baz-value"</span><span>})</span> <span>-&gt;</span> <span>"baz-value"</span>
<span>search</span><span>(</span><span>override</span> <span>||</span> <span>mylist</span><span>[</span><span>-</span><span>1</span><span>],</span> <span>{</span><span>"mylist"</span><span>:</span> <span>[</span><span>"one"</span><span>,</span> <span>"two"</span><span>]})</span> <span>-&gt;</span> <span>"two"</span>
<span>search</span><span>(</span><span>override</span> <span>||</span> <span>mylist</span><span>[</span><span>-</span><span>1</span><span>],</span> <span>{</span><span>"mylist"</span><span>:</span> <span>[</span><span>"one"</span><span>,</span> <span>"two"</span><span>],</span> <span>"override"</span><span>:</span> <span>"yes"</span><span>})</span> <span>-&gt;</span> <span>"yes"</span>
```

## And Expressions

```
<span></span><span>and</span><span>-</span><span>expression</span>  <span>=</span> <span>expression</span> <span>"&amp;&amp;"</span> <span>expression</span>
```

An and expression will evaluate to either the left expression or the right expression. If the expression on the left hand side is a truth-like value, then the value on the right hand side is returned. Otherwise the result of the expression on the left hand side is returned. This also reduces to the expected truth table:

Truth table for and expressions   
| 
LHS

 | 

RHS

 | 

Result

 |
| --- | --- | --- |
| 

True

 | 

True

 | 

True

 |
| 

True

 | 

False

 | 

False

 |
| 

False

 | 

True

 | 

False

 |
| 

False

 | 

False

 | 

False

 |

This is the standard truth table for a [logical conjunction (AND)](https://en.wikipedia.org/wiki/Truth_table#Logical_conjunction_.28AND.29).

### Examples

```
<span></span>search(True &amp;&amp; False, {"True": true, "False": false}) -&gt; false
search(Number &amp;&amp; EmptyList, {"Number": 5, EmptyList: []}) -&gt; []
search(foo[?a == `1` &amp;&amp; b == `2`],
       {"foo": [{"a": 1, "b": 2}, {"a": 1, "b": 3}]}) -&gt; [{"a": 1, "b": 2}]
```

## Paren Expressions

```
<span></span><span>paren</span><span>-</span><span>expression</span>  <span>=</span> <span>"("</span> <span>expression</span> <span>")"</span>
```

A `paren-expression` allows a user to override the precedence order of an expression, e.g. `(a || b) && c`.

### Examples

```
<span></span>search(foo[?(a == `1` || b ==`2`) &amp;&amp; c == `5`],
       {"foo": [{"a": 1, "b": 2, "c": 3}, {"a": 3, "b": 4}]}) -&gt; []
```

## Not Expressions

```
<span></span><span>not</span><span>-</span><span>expression</span>    <span>=</span> <span>"!"</span> <span>expression</span>
```

A `not-expression` negates the result of an expression. If the expression results in a truth-like value, a `not-expression` will change this value to `false`. If the expression results in a false-like value, a `not-expression` will change this value to `true`.

### Examples

```
<span></span>search(!True, {"True": true}) -&gt; false
search(!False, {"False": false}) -&gt; true
search(!Number, {"Number": 5}) -&gt; false
search(!EmptyList, {"EmptyList": []}) -&gt; true
```

## MultiSelect List

```
<span></span><span>multi</span><span>-</span><span>select</span><span>-</span><span>list</span> <span>=</span> <span>"["</span> <span>(</span> <span>expression</span> <span>*</span><span>(</span> <span>","</span> <span>expression</span> <span>)</span> <span>"]"</span>
```

A multiselect expression is used to extract a subset of elements from a JSON hash. There are two version of multiselect, one in which the multiselect expression is enclosed in `{...}` and one which is enclosed in `[...]`. This section describes the `[...]` version. Within the start and closing characters is one or more non expressions separated by a comma. Each expression will be evaluated against the JSON document. Each returned element will be the result of evaluating the expression. A `multi-select-list` with `N` expressions will result in a list of length `N`. Given a multiselect expression `[expr-1,expr-2,...,expr-n]`, the evaluated expression will return `[evaluate(expr-1), evaluate(expr-2), ..., evaluate(expr-n)]`.

### Examples

```
<span></span><span>search</span><span>([</span><span>foo</span><span>,</span><span>bar</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"b"</span><span>,</span> <span>"baz"</span><span>:</span> <span>"c"</span><span>})</span> <span>-&gt;</span> <span>[</span><span>"a"</span><span>,</span> <span>"b"</span><span>]</span>
<span>search</span><span>([</span><span>foo</span><span>,</span><span>bar</span><span>[</span><span>0</span><span>]],</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>[</span><span>"b"</span><span>],</span> <span>"baz"</span><span>:</span> <span>"c"</span><span>})</span> <span>-&gt;</span> <span>[</span><span>"a"</span><span>,</span> <span>"b"</span><span>]</span>
<span>search</span><span>([</span><span>foo</span><span>,</span><span>bar</span><span>.</span><span>baz</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>{</span><span>"baz"</span><span>:</span> <span>"b"</span><span>}})</span> <span>-&gt;</span> <span>[</span><span>"a"</span><span>,</span> <span>"b"</span><span>]</span>
<span>search</span><span>([</span><span>foo</span><span>,</span><span>baz</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"b"</span><span>})</span> <span>-&gt;</span> <span>[</span><span>"a"</span><span>,</span> <span>null</span><span>]</span>
```

## MultiSelect Hash

```
<span></span><span>multi</span><span>-</span><span>select</span><span>-</span><span>hash</span> <span>=</span> <span>"{"</span> <span>(</span> <span>keyval</span><span>-</span><span>expr</span> <span>*</span><span>(</span> <span>","</span> <span>keyval</span><span>-</span><span>expr</span> <span>)</span> <span>"}"</span>
<span>keyval</span><span>-</span><span>expr</span>       <span>=</span> <span>identifier</span> <span>":"</span> <span>expression</span>
```

A `multi-select-hash` expression is similar to a `multi-select-list` expression, except that a hash is created instead of a list. A `multi-select-hash` expression also requires key names to be provided, as specified in the `keyval-expr` rule. Given the following rule:

```
<span></span><span>keyval</span><span>-</span><span>expr</span>       <span>=</span> <span>identifier</span> <span>":"</span> <span>expression</span>
```

The `identifier` is used as the key name and the result of evaluating the `expression` is the value associated with the `identifier` key.

Each `keyval-expr` within the `multi-select-hash` will correspond to a single key value pair in the created hash.

### Examples

Given a `multi-select-hash` expression `{foo: one.two, bar: bar}` and the data `{"bar": "bar", {"one": {"two": "one-two"}}}`, the expression is evaluated as follows:

1.  A hash is created: `{}`
    
2.  A key `foo` is created whose value is the result of evaluating `one.two` against the provided JSON document: `{"foo": evaluate(one.two, <data>)}`
    
3.  A key `bar` is created whose value is the result of evaluting the expression `bar` against the provided JSON document.
    

The final result will be: `{"foo": "one-two", "bar": "bar"}`.

Additional examples:

```
<span></span><span>search</span><span>({</span><span>foo</span><span>:</span> <span>foo</span><span>,</span> <span>bar</span><span>:</span> <span>bar</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"b"</span><span>,</span> <span>"baz"</span><span>:</span> <span>"c"</span><span>})</span>
              <span>-&gt;</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"b"</span><span>}</span>
<span>search</span><span>({</span><span>foo</span><span>:</span> <span>foo</span><span>,</span> <span>firstbar</span><span>:</span> <span>bar</span><span>[</span><span>0</span><span>]},</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>[</span><span>"b"</span><span>]})</span>
              <span>-&gt;</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"firstbar"</span><span>:</span> <span>"b"</span><span>}</span>
<span>search</span><span>({</span><span>foo</span><span>:</span> <span>foo</span><span>,</span> <span>"bar.baz"</span><span>:</span> <span>bar</span><span>.</span><span>baz</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>{</span><span>"baz"</span><span>:</span> <span>"b"</span><span>}})</span>
              <span>-&gt;</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar.baz"</span><span>:</span> <span>"b"</span><span>}</span>
<span>search</span><span>({</span><span>foo</span><span>:</span> <span>foo</span><span>,</span> <span>baz</span><span>:</span> <span>baz</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"bar"</span><span>:</span> <span>"b"</span><span>})</span>
              <span>-&gt;</span> <span>{</span><span>"foo"</span><span>:</span> <span>"a"</span><span>,</span> <span>"baz"</span><span>:</span> <span>null</span><span>}</span>
```

## Wildcard Expressions

```
<span></span><span>expression</span>        <span>=/</span> <span>"*"</span>
<span>bracket</span><span>-</span><span>specifier</span> <span>=</span> <span>"["</span> <span>"*"</span> <span>"]"</span>
```

A wildcard expression is a expression of either `*` or `[*]`. A wildcard expression can return multiple elements, and the remaining expressions are evaluated against each returned element from a wildcard expression. The `[*]` syntax applies to a list type and the `*` syntax applies to a hash type.

The `[*]` syntax (referred to as a list wildcard expression) will return all the elements in a list. Any subsequent expressions will be evaluated against each individual element. Given an expression `[*].child-expr`, and a list of N elements, the evaluation of this expression would be `[child-expr(el-0), child-expr(el-2), ..., child-expr(el-N)]`. This is referred to as a **projection**, and the `child-expr` expression is projected onto the elements of the resulting list.

Once a projection has been created, all subsequent expressions are projected onto the resulting list.

The `*` syntax (referred to as a hash wildcard expression) will return a list of the hash element’s values. Any subsequent expression will be evaluated against each individual element in the list (this is also referred to as a **projection**).

Note that if any subsequent expression after a wildcard expression returns a `null` value, it is omitted from the final result list.

A list wildcard expression is only valid for the JSON array type. If a list wildcard expression is applied to any other JSON type, a value of `null` is returned.

Similarly, a hash wildcard expression is only valid for the JSON object type. If a hash wildcard expression is applied to any other JSON type, a value of `null` is returned. Note that JSON hashes are explicitly defined as unordered. Therefore a hash wildcard expression can return the values associated with the hash in any order. Implementations are not required to return the hash values in any specific order.

### Examples

```
<span></span><span>search</span><span>([</span><span>*</span><span>]</span><span>.</span><span>foo</span><span>,</span> <span>[{</span><span>"foo"</span><span>:</span> <span>1</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>2</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>3</span><span>}])</span> <span>-&gt;</span> <span>[</span><span>1</span><span>,</span> <span>2</span><span>,</span> <span>3</span><span>]</span>
<span>search</span><span>([</span><span>*</span><span>]</span><span>.</span><span>foo</span><span>,</span> <span>[{</span><span>"foo"</span><span>:</span> <span>1</span><span>},</span> <span>{</span><span>"foo"</span><span>:</span> <span>2</span><span>},</span> <span>{</span><span>"bar"</span><span>:</span> <span>3</span><span>}])</span> <span>-&gt;</span> <span>[</span><span>1</span><span>,</span> <span>2</span><span>]</span>
<span>search</span><span>(</span><span>'*.foo'</span><span>,</span> <span>{</span><span>"a"</span><span>:</span> <span>{</span><span>"foo"</span><span>:</span> <span>1</span><span>},</span> <span>"b"</span><span>:</span> <span>{</span><span>"foo"</span><span>:</span> <span>2</span><span>},</span> <span>"c"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>1</span><span>}})</span> <span>-&gt;</span> <span>[</span><span>1</span><span>,</span> <span>2</span><span>]</span>
```

## Literal Expressions

```
<span></span><span>literal</span>           <span>=</span> <span>"`"</span> <span>json</span><span>-</span><span>value</span> <span>"`"</span>
```

A literal expression is an expression that allows arbitrary JSON objects to be specified. This is useful in filter expressions as well as multi select hashes (to create arbitrary key value pairs), but is allowed anywhere an expression is allowed. The specification includes the ABNF for JSON, implementations should use an existing JSON parser to parse literal values. Note that the `` ` `` character must now be escaped in a `json-value` which means implementations need to handle this case before passing the resulting string to a JSON parser.

### Examples

```
<span></span>search(`"foo"`, "anything") -&gt; "foo"
search(`"foo\`bar"`, "anything") -&gt; "foo`bar"
search(`[1, 2]`, "anything") -&gt; [1, 2]
search(`true`, "anything") -&gt; true
search(`{"a": "b"}`.a, "anything") -&gt; "b"
search({first: a, type: `"mytype"`}, {"a": "b", "c": "d"}) -&gt; {"first": "b", "type": "mytype"}
```

## Raw String Literals

```
<span></span><span>raw</span><span>-</span><span>string</span>        <span>=</span> <span>"'"</span> <span>*</span><span>raw</span><span>-</span><span>string</span><span>-</span><span>char</span> <span>"'"</span>
<span>raw</span><span>-</span><span>string</span><span>-</span><span>char</span>   <span>=</span> <span>(</span><span>%</span><span>x20</span><span>-</span><span>26</span> <span>/</span> <span>%</span><span>x28</span><span>-</span><span>5</span><span>B</span> <span>/</span> <span>%</span><span>x5D</span><span>-</span><span>10</span><span>FFFF</span><span>)</span> <span>/</span> <span>preserved</span><span>-</span><span>escape</span> <span>/</span>
                      <span>raw</span><span>-</span><span>string</span><span>-</span><span>escape</span>
<span>preserved</span><span>-</span><span>escape</span>  <span>=</span> <span>escape</span> <span>(</span><span>%</span><span>x20</span><span>-</span><span>26</span> <span>/</span> <span>%</span><span>28</span><span>-</span><span>5</span><span>B</span> <span>/</span> <span>%</span><span>x5D</span><span>-</span><span>10</span><span>FFFF</span><span>)</span>
<span>raw</span><span>-</span><span>string</span><span>-</span><span>escape</span> <span>=</span> <span>escape</span> <span>(</span><span>"'"</span> <span>/</span> <span>escape</span><span>)</span>
```

A raw string is an expression that allows for a literal string value to be specified. The result of evaluating the raw string literal expression is the literal string value. It is a simpler form of a literal expression that is special cased for strings. For example, the following expressions both evaluate to the same value: “foo”:

```
<span></span>search(`"foo"`, "") -&gt; "foo"
search('foo', "") -&gt; "foo"
```

As you can see in the examples above, it is meant as a more succinct form of the common scenario of specifying a literal string value.

In addition, it does not perform any of the additional processing that JSON strings supports including:

-   Not expanding unicode escape sequences
    
-   Not expanding newline characters
    
-   Not expanding tab characters or any other escape sequences documented in RFC 4627 section 2.5.
    

```
<span></span><span>search</span><span>(</span><span>'foo'</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>"foo"</span>
<span>search</span><span>(</span><span>' bar '</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>" bar "</span>
<span>search</span><span>(</span><span>'[baz]'</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>"[baz]"</span>
<span>search</span><span>(</span><span>'[baz]'</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>"[baz]"</span>
<span>search</span><span>(</span><span>'</span><span>\u03a6</span><span>'</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>"</span><span>\u03a6</span><span>"</span>
<span>search</span><span>(</span><span>'</span><span>\\</span><span>'</span><span>,</span> <span>""</span><span>)</span> <span>-&gt;</span> <span>"</span><span>\\</span><span>"</span>
```

## Filter Expressions

```
<span></span><span>list</span><span>-</span><span>filter</span><span>-</span><span>expr</span>      <span>=</span> <span>"[?"</span> <span>expression</span> <span>"]"</span>
<span>comparator</span><span>-</span><span>expression</span> <span>=</span> <span>expression</span> <span>comparator</span> <span>expression</span>
<span>comparator</span>            <span>=</span> <span>"&lt;"</span> <span>/</span> <span>"&lt;="</span> <span>/</span> <span>"=="</span> <span>/</span> <span>"&gt;="</span> <span>/</span> <span>"&gt;"</span> <span>/</span> <span>"!="</span>
```

A filter expression provides a way to select JSON elements based on a comparison to another expression. A filter expression is evaluated as follows: for each element in an array evaluate the `expression` against the element. If the expression evalutes to a truth-like value, the item (in its entirety) is added to the result list. Otherwise it is excluded from the result list. A filter expression is only defined for a JSON array. Attempting to evaluate a filter expression against any other type will return `null`.

### Comparison Operators

The following operations are supported:

-   `==`, tests for equality.
    
-   `!=`, tests for inequality.
    
-   `<`, less than.
    
-   `<=`, less than or equal to.
    
-   `>`, greater than.
    
-   `>=`, greater than or equal to.
    

The behavior of each operation is dependent on the type of each evaluated expression.

The comparison semantics for each operator are defined below based on the corresponding JSON type:

#### Equality Operators

For `string/number/true/false/null` types, equality is an exact match. A `string` is equal to another `string` if they they have the exact sequence of code points. The literal values `true/false/null` are only equal to their own literal values. Two JSON objects are equal if they have the same set of keys and values (given two JSON objeccts `x` and `y`, for each key value pair `(i, j)` in `x`, there exists an equivalent pair `(i, j)` in `y`). Two JSON arrays are equal if they have equal elements in the same order (given two arrays `x` and `y`, for each `i` from `0` until `length(x)`, `x[i] == y[i]`).

#### Ordering Operators

Ordering operators `>, >=, <, <=` are **only** valid for numbers. Evaluating any other type with a comparison operator will yield a `null` value, which will result in the element being excluded from the result list. For example, given:

```
<span></span><span>search</span><span>(</span><span>'foo[?a&lt;b]'</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>[{</span><span>"a"</span><span>:</span> <span>"char"</span><span>,</span> <span>"b"</span><span>:</span> <span>"char"</span><span>},</span>
                             <span>{</span><span>"a"</span><span>:</span> <span>2</span><span>,</span> <span>"b"</span><span>:</span> <span>1</span><span>},</span>
                             <span>{</span><span>"a"</span><span>:</span> <span>1</span><span>,</span> <span>"b"</span><span>:</span> <span>2</span><span>}]})</span>
```

The three elements in the foo list are evaluated against `a < b`. The first element resolves to the comparison `"char" < "bar"`, and because these types are string, the expression results in `null`, so the first element is not included in the result list. The second element resolves to `2 < 1`, which is `false`, so the second element is excluded from the result list. The third expression resolves to `1 < 2` which evalutes to `true`, so the third element is included in the list. The final result of that expression is `[{"a": 1, "b": 2}]`.

### Examples

```
<span></span>search(foo[?bar==`10`], {"foo": [{"bar": 1}, {"bar": 10}]}) -&gt; [{"bar": 10}]
search([?bar==`10`], [{"bar": 1}, {"bar": 10}]}) -&gt; [{"bar": 10}]
search(foo[?a==b], {"foo": [{"a": 1, "b": 2}, {"a": 2, "b": 2}]}) -&gt; [{"a": 2, "b": 2}]
```

## Function Expressions

```
<span></span><span>function</span><span>-</span><span>expression</span> <span>=</span> <span>unquoted</span><span>-</span><span>string</span>  <span>(</span>
                        <span>no</span><span>-</span><span>args</span>  <span>/</span>
                        <span>one</span><span>-</span><span>or</span><span>-</span><span>more</span><span>-</span><span>args</span> <span>)</span>
<span>no</span><span>-</span><span>args</span>             <span>=</span> <span>"("</span> <span>")"</span>
<span>one</span><span>-</span><span>or</span><span>-</span><span>more</span><span>-</span><span>args</span>    <span>=</span> <span>"("</span> <span>(</span> <span>function</span><span>-</span><span>arg</span> <span>*</span><span>(</span> <span>","</span> <span>function</span><span>-</span><span>arg</span> <span>)</span> <span>)</span> <span>")"</span>
<span>function</span><span>-</span><span>arg</span>        <span>=</span> <span>expression</span> <span>/</span> <span>current</span><span>-</span><span>node</span> <span>/</span> <span>expression</span><span>-</span><span>type</span>
<span>current</span><span>-</span><span>node</span>        <span>=</span> <span>"@"</span>
<span>expression</span><span>-</span><span>type</span>     <span>=</span> <span>"&amp;"</span> <span>expression</span>
```

Functions allow users to easily transform and filter data in JMESPath expressions.

### Data Types

In order to support functions, a type system is needed. The JSON types are used:

-   number (integers and double-precision floating-point format in JSON)
    
-   string
    
-   boolean (`true` or `false`)
    
-   array (an ordered, sequence of values)
    
-   object (an unordered collection of key value pairs)
    
-   null
    

There is also an additional type that is not a JSON type that’s used in JMESPath functions:

-   expression (denoted by `&expression`)
    

### current-node

The `current-node` token can be used to represent the current node being evaluated. The `current-node` token is useful for functions that require the current node being evaluated as an argument. For example, the following expression creates an array containing the total number of elements in the `foo` object followed by the value of `foo["bar"]`.

JMESPath assumes that all function arguments operate on the current node unless the argument is a `literal` or `number` token. Because of this, an expression such as `@.bar` would be equivalent to just `bar`, so the current node is only allowed as a bare expression.

#### current-node state

At the start of an expression, the value of the current node is the data being evaluated by the JMESPath expression. As an expression is evaluated, the value the the current node represents MUST change to reflect the node currently being evaluated. When in a projection, the current node value must be changed to the node currently being evaluated by the projection.

### Function Evaluation

Functions are evaluated in applicative order. Each argument must be an expression, each argument expression must be evaluated before evaluating the function. The function is then called with the evaluated function arguments. The result of the `function-expression` is the result returned by the function call. If a `function-expression` is evaluated for a function that does not exist, the JMESPath implementation must indicate to the caller that an `unknown-function` error occurred. How and when this error is raised is implementation specific, but implementations should indicate to the caller that this specific error occurred.

Functions can either have a specific arity or be variadic with a minimum number of arguments. If a `function-expression` is encountered where the arity does not match or the minimum number of arguments for a variadic function is not provided, then implementations must indicate to the caller than an `invalid-arity` error occurred. How and when this error is raised is implementation specific.

Each function signature declares the types of its input parameters. If any type constraints are not met, implementations must indicate that an `invalid-type` error occurred.

In order to accommodate type contraints, functions are provided to convert types to other types (`to_string`, `to_number`) which are defined below. No explicit type conversion happens unless a user specifically uses one of these type conversion functions.

Function expressions are also allowed as the child element of a sub expression. This allows functions to be used with projections, which can enable functions to be applied to every element in a projection. For example, given the input data of `["1", "2", "3", "notanumber", true]`, the following expression can be used to convert (and filter) all elements to numbers:

```
<span></span>search([].to_number(@), `["1", "2", "3", "notanumber", true]`) -&gt; [1, 2, 3]
```

This provides a simple mechanism to explicitly convert types when needed.

## Built-in Functions

JMESPath has various built-in functions that operate on different data types, documented below. Each function below has a signature that defines the expected types of the input and the type of the returned output:

```
<span></span>return_type function_name(type $argname)
return_type function_name2(type1|type2 $argname)
```

The list of data types supported by a function are:

-   number (integers and double-precision floating-point format in JSON)
    
-   string
    
-   boolean (`true` or `false`)
    
-   array (an ordered, sequence of values)
    
-   object (an unordered collection of key value pairs)
    
-   null
    
-   expression (denoted by `&expression`)
    

With the exception of the last item, all of the above types correspond to the types provided by JSON.

If a function can accept multiple types for an input value, then the multiple types are separated with `|`. If the resolved arguments do not match the types specified in the signature, an `invalid-type` error occurs.

The `array` type can further specify requirements on the type of the elements if they want to enforce homogeneous types. The subtype is surrounded by `[type]`, for example, the function signature below requires its input argument resolves to an array of numbers:

```
<span></span>return_type foo(array[number] $argname)
```

As a shorthand, the type `any` is used to indicate that the argument can be of any type (`array|object|number|string|boolean|null`).

JMESPath functions are required to type check their input arguments. Specifying an invalid type for a function argument will result in a `invalid-type` error.

The expression type, denoted by `&expression`, is used to specify a expression that is not immediately evaluated. Instead, a reference to that expression is provided to the function being called. The function can then choose to apply the expression reference as needed. It is semantically similar to an anonymous function. See the [sort\_by](https://jmespath.org/specification.html#func-sort-by) function for an example usage of the expression type.

Similarly how arrays can specify a type within a list using the `array[type]` syntax, expressions can specify their resolved type using `expression->type` syntax. This means that the resolved type of the function argument must be an expression that itself will resolve to `type`.

The first function below, `abs` is discussed in detail to demonstrate the above points. Subsequent function definitions will not include these details for brevity, but the same rules apply.

Note

All string related functions are defined on the basis of Unicode code points; they do not take normalization into account.

### abs

```
<span></span>number abs(number $value)
```

Returns the absolute value of the provided argument. The signature indicates that a number is returned, and that the input argument `$value` **must** resolve to a number, otherwise a `invalid-type` error is triggered.

Below is a worked example. Given:

Evaluating `abs(foo)` works as follows:

1.  Evaluate the input argument against the current data:
    
    ```
    <span></span><span>search</span><span>(</span><span>foo</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>-</span><span>1</span><span>,</span> <span>"bar"</span><span>:</span> <span>"2"</span><span>})</span> <span>-&gt;</span> <span>-</span><span>1</span>
    ```
    
2.  Validate the type of the resolved argument. In this case `-1` is of type `number` so it passes the type check.
    
3.  Call the function with the resolved argument:
    
4.  The value of `1` is the resolved value of the function expression `abs(foo)`.
    

Below is the same steps for evaluating `abs(bar)`:

1.  Evaluate the input argument against the current data:
    
    ```
    <span></span><span>search</span><span>(</span><span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>-</span><span>1</span><span>,</span> <span>"bar"</span><span>:</span> <span>"2"</span><span>})</span> <span>-&gt;</span> <span>"2"</span>
    ```
    
2.  Validate the type of the resolved argument. In this case `"2"` is of type `string` so we immediately indicate that an `invalid-type` error occurred.
    

As a final example, here is the steps for evaluating `abs(to_number(bar))`:

1.  Evaluate the input argument against the current data:
    
    ```
    <span></span><span>search</span><span>(</span><span>to_number</span><span>(</span><span>bar</span><span>),</span> <span>{</span><span>"foo"</span><span>:</span> <span>-</span><span>1</span><span>,</span> <span>"bar"</span><span>:</span> <span>"2"</span><span>})</span>
    ```
    
2.  In order to evaluate the above expression, we need to evaluate `to_number(bar)`:
    
    ```
    <span></span><span>search</span><span>(</span><span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>-</span><span>1</span><span>,</span> <span>"bar"</span><span>:</span> <span>"2"</span><span>})</span> <span>-&gt;</span> <span>"2"</span>
    <span># Validate "2" passes the type check for to_number, which it does.</span>
    <span>to_number</span><span>(</span><span>"2"</span><span>)</span> <span>-&gt;</span> <span>2</span>
    ```
    
    Note that [to\_number](https://jmespath.org/specification.html#to-number) is defined below.
    
3.  Now we can evaluate the original expression:
    
    ```
    <span></span><span>search</span><span>(</span><span>to_number</span><span>(</span><span>bar</span><span>),</span> <span>{</span><span>"foo"</span><span>:</span> <span>-</span><span>1</span><span>,</span> <span>"bar"</span><span>:</span> <span>"2"</span><span>})</span> <span>-&gt;</span> <span>2</span>
    ```
    
4.  Call the function with the final resolved value:
    
5.  The value of `2` is the resolved value of the function expression `abs(to_number(bar))`.
    

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

`abs(1)`

 | 

`1`

 |
| 

`abs(-1)`

 | 

`1`

 |
| 

`abs('abc')`

 | 

`<error: invalid-type>`

 |

### avg

```
<span></span>number avg(array[number] $elements)
```

Returns the average of the elements in the provided array.

An empty array will produce a return value of null.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[10, 15, 20]`

 | 

`avg(@)`

 | 

`15`

 |
| 

`[10, false, 20]`

 | 

`avg(@)`

 | 

`<error: invalid-type>`

 |
| 

`[false]`

 | 

`avg(@)`

 | 

`<error: invalid-type>`

 |
| 

`false`

 | 

`avg(@)`

 | 

`<error: invalid-type>`

 |

### contains

```
<span></span>boolean contains(array|string $subject, any $search)
```

Returns `true` if the given `$subject` contains the provided `$search` string.

If `$subject` is an array, this function returns true if one of the elements in the array is equal to the provided `$search` value.

If the provided `$subject` is a string, this function returns true if the string contains the provided `$search` argument.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

n/a

 | 

`contains('foobar', 'foo')`

 | 

`true`

 |
| 

n/a

 | 

`contains('foobar', 'not')`

 | 

`false`

 |
| 

n/a

 | 

`contains('foobar', 'bar')`

 | 

`true`

 |
| 

n/a

 | 

``contains(`false`, 'bar')``

 | 

`<error: invalid-type>`

 |
| 

n/a

 | 

`contains('foobar', 123)`

 | 

`false`

 |
| 

`["a", "b"]`

 | 

`contains(@, 'a')`

 | 

`true`

 |
| 

`["a"]`

 | 

`contains(@, 'a')`

 | 

`true`

 |
| 

`["a"]`

 | 

`contains(@, 'b')`

 | 

`false`

 |
| 

`["foo", "bar"]`

 | 

`contains(@, 'foo')`

 | 

`true`

 |
| 

`["foo", "bar"]`

 | 

`contains(@, 'b')`

 | 

`false`

 |

### ceil

```
<span></span>number ceil(number $value)
```

Returns the next highest integer value by rounding up if necessary.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

``ceil(`1.001`)``

 | 

`2`

 |
| 

``ceil(`1.9`)``

 | 

`2`

 |
| 

``ceil(`1`)``

 | 

`1`

 |
| 

``ceil(`"abc"`)``

 | 

`null`

 |

### ends\_with

```
<span></span>boolean ends_with(string $subject, string $prefix)
```

Returns `true` if the `$subject` ends with the `$prefix`, otherwise this function returns `false`.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`"foobarbaz"`

 | 

`ends_with(@, 'baz')`

 | 

`true`

 |
| 

`"foobarbaz"`

 | 

`ends_with(@, 'foo')`

 | 

`false`

 |
| 

`"foobarbaz"`

 | 

`ends_with(@, 'z')`

 | 

`true`

 |

### floor

```
<span></span>number floor(number $value)
```

Returns the next lowest integer value by rounding down if necessary.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

``floor(`1.001`)``

 | 

`1`

 |
| 

``floor(`1.9`)``

 | 

`1`

 |
| 

``floor(`1`)``

 | 

`1`

 |

### join

```
<span></span>string join(string $glue, array[string] $stringsarray)
```

Returns all of the elements from the provided `$stringsarray` array joined together using the `$glue` argument as a separator between each.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`["a", "b"]`

 | 

`join(', ', @)`

 | 

`"a, b"`

 |
| 

`["a", "b"]`

 | 

`join('', @)`

 | 

`"ab"`

 |
| 

`["a", false, "b"]`

 | 

`join(', ', @)`

 | 

`<error: invalid-type>`

 |
| 

`[false]`

 | 

`join(', ', @)`

 | 

`<error: invalid-type>`

 |

### keys

Returns an array containing the keys of the provided object. Note that because JSON hashes are inheritently unordered, the keys associated with the provided object `obj` are inheritently unordered. Implementations are not required to return keys in any specific order.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`{"foo": "baz", "bar": "bam"}`

 | 

`keys(@)`

 | 

`["foo", "bar"]`

 |
| 

`{}`

 | 

`keys(@)`

 | 

`[]`

 |
| 

`false`

 | 

`keys(@)`

 | 

`<error: invalid-type>`

 |
| 

`[b, a, c]`

 | 

`keys(@)`

 | 

`<error: invalid-type>`

 |

### length

```
<span></span>number length(string|array|object $subject)
```

Returns the length of the given argument using the following types rules:

1.  string: returns the number of code points in the string
    
2.  array: returns the number of elements in the array
    
3.  object: returns the number of key-value pairs in the object
    

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

n/a

 | 

`length('abc')`

 | 

3

 |
| 

“current”

 | 

`length(@)`

 | 

`7`

 |
| 

“current”

 | 

`length(not_there)`

 | 

`<error: invalid-type>`

 |
| 

`["a", "b", "c"]`

 | 

`length(@)`

 | 

`3`

 |
| 

`[]`

 | 

`length(@)`

 | 

`0`

 |
| 

`{}`

 | 

`length(@)`

 | 

`0`

 |
| 

`{"foo": "bar", "baz": "bam"}`

 | 

`length(@)`

 | 

`2`

 |

### map

```
<span></span><span>array</span><span>[</span><span>any</span><span>]</span> <span>map</span><span>(</span><span>expression</span><span>-&gt;</span><span>any</span><span>-&gt;</span><span>any</span> <span>expr</span><span>,</span> <span>array</span><span>[</span><span>any</span><span>]</span> <span>elements</span><span>)</span>
```

Apply the `expr` to every element in the `elements` array and return the array of results. An `elements` of length N will produce a return array of length N.

Unlike a projection, (`[*].bar`), `map()` will include the result of applying the `expr` for every element in the `elements` array, even if the result if `null`.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`{"array": [{"foo": "a"}, {"foo": "b"}, {}, [], {"foo": "f"}]}`

 | 

`map(&foo, array)`

 | 

`["a", "b", null, null, "f"]`

 |
| 

`[[1, 2, 3, [4]], [5, 6, 7, [8, 9]]]`

 | 

`map(&[], @)`

 | 

`[[1, 2, 3, 4], [5, 6, 7, 8, 9]]`

 |

### max

```
<span></span>number max(array[number]|array[string] $collection)
```

Returns the highest found number in the provided array argument.

An empty array will produce a return value of null.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[10, 15]`

 | 

`max(@)`

 | 

`15`

 |
| 

`["a", "b"]`

 | 

`max(@)`

 | 

`"b"`

 |
| 

`["a", 2, "b"]`

 | 

`max(@)`

 | 

`<error: invalid-type>`

 |
| 

`[10, false, 20]`

 | 

`max(@)`

 | 

`<error: invalid-type>`

 |

### max\_by

```
<span></span><span>max_by</span><span>(</span><span>array</span> <span>elements</span><span>,</span> <span>expression</span><span>-&gt;</span><span>number</span><span>|</span><span>expression</span><span>-&gt;</span><span>string</span> <span>expr</span><span>)</span>
```

Return the maximum element in an array using the expression `expr` as the comparison key. The entire maximum element is returned. Below are several examples using the `people` array (defined above) as the given input.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

`max_by(people, &age)`

 | 

`{"age": 50, "age_str": "50", "bool": false, "name": "d"}`

 |
| 

`max_by(people, &age).age`

 | 

`50`

 |
| 

`max_by(people, &to_number(age_str))`

 | 

`{"age": 50, "age_str": "50", "bool": false, "name": "d"}`

 |
| 

`max_by(people, &age_str)`

 | 

`<error: invalid-type>`

 |
| 

`max_by(people, age)`

 | 

`<error: invalid-type>`

 |

### merge

```
<span></span>object merge([object *argument, [, object $...]])
```

Accepts 0 or more objects as arguments, and returns a single object with subsequent objects merged. Each subsequent object’s key/value pairs are added to the preceding object. This function is used to combine multiple objects into one. You can think of this as the first object being the base object, and each subsequent argument being overrides that are applied to the base object.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

``merge(`{"a": "b"}`, `{"c": "d"}`)``

 | 

`{"a": "b", "c": "d"}`

 |
| 

``merge(`{"a": "b"}`, `{"a": "override"}`)``

 | 

`{"a": "override"}`

 |
| 

``merge(`{"a": "x", "b": "y"}`, `{"b": "override", "c": "z"}`)``

 | 

`{"a": "x", "b": "override", "c": "z"}`

 |

### min

```
<span></span>number min(array[number]|array[string] $collection)
```

Returns the lowest found number in the provided `$collection` argument.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[10, 15]`

 | 

`min(@)`

 | 

`10`

 |
| 

`["a", "b"]`

 | 

`min(@)`

 | 

`"a"`

 |
| 

`["a", 2, "b"]`

 | 

`min(@)`

 | 

`<error: invalid-type>`

 |
| 

`[10, false, 20]`

 | 

`min(@)`

 | 

`<error: invalid-type>`

 |

### min\_by

```
<span></span><span>min_by</span><span>(</span><span>array</span> <span>elements</span><span>,</span> <span>expression</span><span>-&gt;</span><span>number</span><span>|</span><span>expression</span><span>-&gt;</span><span>string</span> <span>expr</span><span>)</span>
```

Return the minimum element in an array using the expression `expr` as the comparison key. The entire maximum element is returned. Below are several examples using the `people` array (defined above) as the given input.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

`min_by(people, &age)`

 | 

`{"age": 10, "age_str": "10", "bool": true, "name": 3}`

 |
| 

`min_by(people, &age).age`

 | 

`10`

 |
| 

`min_by(people, &to_number(age_str))`

 | 

`{"age": 10, "age_str": "10", "bool": true, "name": 3}`

 |
| 

`min_by(people, &age_str)`

 | 

`<error: invalid-type>`

 |
| 

`min_by(people, age)`

 | 

`<error: invalid-type>`

 |

### not\_null

```
<span></span>any not_null([any $argument [, any $...]])
```

Returns the first argument that does not resolve to `null`. This function accepts one or more arguments, and will evaluate them in order until a non null argument is encounted. If all arguments values resolve to `null`, then a value of `null` is returned.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`{"a": null, "b": null, "c": [], "d": "foo"}`

 | 

`not_null(no_exist, a, b, c, d)`

 | 

`[]`

 |
| 

`{"a": null, "b": null, "c": [], "d": "foo"}`

 | 

``not_null(a, b, `null`, d, c)``

 | 

`"foo"`

 |
| 

`{"a": null, "b": null, "c": [], "d": "foo"}`

 | 

`not_null(a, b)`

 | 

`null`

 |

### reverse

```
<span></span>array reverse(string|array $argument)
```

Reverses the order of the `$argument`.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[0, 1, 2, 3, 4]`

 | 

`reverse(@)`

 | 

`[4, 3, 2, 1, 0]`

 |
| 

`[]`

 | 

`reverse(@)`

 | 

`[]`

 |
| 

`["a", "b", "c", 1, 2, 3]`

 | 

`reverse(@)`

 | 

`[3, 2, 1, "c", "b", "a"]`

 |
| 

`"abcd"`

 | 

`reverse(@)`

 | 

`dcba`

 |

### sort

```
<span></span>array sort(array[number]|array[string] $list)
```

This function accepts an array `$list` argument and returns the sorted elements of the `$list` as an array.

The array must be a list of strings or numbers. Sorting strings is based on code points. Locale is not taken into account.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[b, a, c]`

 | 

`sort(@)`

 | 

`[a, b, c]`

 |
| 

`[1, a, c]`

 | 

`sort(@)`

 | 

`[1, a, c]`

 |
| 

`[false, [], null]`

 | 

`sort(@)`

 | 

`[[], null, false]`

 |
| 

`[[], {}, false]`

 | 

`sort(@)`

 | 

`[{}, [], false]`

 |
| 

`{"a": 1, "b": 2}`

 | 

`sort(@)`

 | 

`null`

 |
| 

`false`

 | 

`sort(@)`

 | 

`null`

 |

### sort\_by

```
<span></span><span>sort_by</span><span>(</span><span>array</span> <span>elements</span><span>,</span> <span>expression</span><span>-&gt;</span><span>number</span><span>|</span><span>expression</span><span>-&gt;</span><span>string</span> <span>expr</span><span>)</span>
```

Sort an array using an expression `expr` as the sort key. For each element in the array of `elements`, the `expr` expression is applied and the resulting value is used as the key used when sorting the `elements`.

If the result of evaluating the `expr` against the current array element results in type other than a `number` or a `string`, an `invalid-type` error will occur.

Below are several examples using the `people` array (defined above) as the given input. `sort_by` follows the same sorting logic as the `sort` function.

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

`sort_by(people, &age)[].age`

 | 

`[10, 20, 30, 40, 50]`

 |
| 

`sort_by(people, &age)[0]`

 | 

`{"age": 10, "age_str": "10", "bool": true, "name": 3}`

 |
| 

`sort_by(people, &to_number(age_str))[0]`

 | 

`{"age": 10, "age_str": "10", "bool": true, "name": 3}`

 |

### starts\_with

```
<span></span>boolean starts_with(string $subject, string $prefix)
```

Returns `true` if the `$subject` starts with the `$prefix`, otherwise this function returns `false`.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`"foobarbaz"`

 | 

`starts_with(@, 'foo')`

 | 

`true`

 |
| 

`"foobarbaz"`

 | 

`starts_with(@, 'baz')`

 | 

`false`

 |
| 

`"foobarbaz"`

 | 

`starts_with(@, 'f')`

 | 

`true`

 |

### sum

```
<span></span>number sum(array[number] $collection)
```

Returns the sum of the provided array argument.

An empty array will produce a return value of 0.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`[10, 15]`

 | 

`sum(@)`

 | 

`25`

 |
| 

`[10, false, 20]`

 | 

`max(@)`

 | 

`<error: invalid-type>`

 |
| 

`[10, false, 20]`

 | 

`sum([].to_number(@))`

 | 

`30`

 |
| 

`[]`

 | 

`sum(@)`

 | 

`0`

 |

### to\_array

-   array - Returns the passed in value.
    
-   number/string/object/boolean - Returns a one element array containing the passed in argument.
    

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

``to_array(`[1, 2]`)``

 | 

`[1, 2]`

 |
| 

``to_array(`"string"`)``

 | 

`["string"]`

 |
| 

``to_array(`0`)``

 | 

`[0]`

 |
| 

``to_array(`true`)``

 | 

`[true]`

 |
| 

``to_array(`{"foo": "bar"}`)``

 | 

`[{"foo": "bar"}]`

 |

### to\_string

```
<span></span>string to_string(any $arg)
```

-   string - Returns the passed in value.
    
-   number/array/object/boolean - The JSON encoded value of the object. The JSON encoder should emit the encoded JSON value without adding any additional new lines.
    

Examples  
| 
Expression

 | 

Result

 |
| --- | --- |
| 

``to_string(`2`)``

 | 

`"2"`

 |

### to\_number

```
<span></span>number to_number(any $arg)
```

-   string - Returns the parsed number. Any string that conforms to the `json-number` production is supported. Note that the floating number support will be implementation specific, but implementations should support at least IEEE 754-2008 binary64 (double precision) numbers, as this is generally available and widely used.
    
-   number - Returns the passed in value.
    
-   array - null
    
-   object - null
    
-   boolean - null
    
-   null - null
    

### type

```
<span></span>string type(array|object|string|number|boolean|null $subject)
```

Returns the JavaScript type of the given `$subject` argument as a string value.

The return value MUST be one of the following:

-   number
    
-   string
    
-   boolean
    
-   array
    
-   object
    
-   null
    

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`"foo"`

 | 

`type(@)`

 | 

`"string"`

 |
| 

`true`

 | 

`type(@)`

 | 

`"boolean"`

 |
| 

`false`

 | 

`type(@)`

 | 

`"boolean"`

 |
| 

`null`

 | 

`type(@)`

 | 

`"null"`

 |
| 

`123`

 | 

`type(@)`

 | 

`number`

 |
| 

`123.05`

 | 

`type(@)`

 | 

`number`

 |
| 

`["abc"]`

 | 

`type(@)`

 | 

`"array"`

 |
| 

`{"abc": "123"}`

 | 

`type(@)`

 | 

`"object"`

 |

### values

```
<span></span>array values(object $obj)
```

Returns the values of the provided object. Note that because JSON hashes are inheritently unordered, the values associated with the provided object `obj` are inheritently unordered. Implementations are not required to return values in any specific order. For example, given the input:

```
<span></span><span>{</span><span>"a"</span><span>:</span> <span>"first"</span><span>,</span> <span>"b"</span><span>:</span> <span>"second"</span><span>,</span> <span>"c"</span><span>:</span> <span>"third"</span><span>}</span>
```

The expression `values(@)` could have any of these return values:

-   `["first", "second", "third"]`
    
-   `["first", "third", "second"]`
    
-   `["second", "first", "third"]`
    
-   `["second", "third", "first"]`
    
-   `["third", "first", "second"]`
    
-   `["third", "second", "first"]`
    

If you would like a specific order, consider using the `sort` or `sort_by` functions.

Examples   
| 
Given

 | 

Expression

 | 

Result

 |
| --- | --- | --- |
| 

`{"foo": "baz", "bar": "bam"}`

 | 

`values(@)`

 | 

`["baz", "bam"]`

 |
| 

`["a", "b"]`

 | 

`values(@)`

 | 

`<error: invalid-type>`

 |
| 

`false`

 | 

`values(@)`

 | 

`<error: invalid-type>`

 |

## Pipe Expressions

```
<span></span><span>pipe</span><span>-</span><span>expression</span>  <span>=</span> <span>expression</span> <span>"|"</span> <span>expression</span>
```

A pipe expression combines two expressions, separated by the `|` character. It is similar to a `sub-expression` with two important distinctions:

1.  Any expression can be used on the right hand side. A `sub-expression` restricts the type of expression that can be used on the right hand side.
    
2.  A `pipe-expression` **stops projections on the left hand side for propagating to the right hand side**. If the left expression creates a projection, it does **not** apply to the right hand side.
    

For example, given the following data:

```
<span></span><span>{</span><span>"foo"</span><span>:</span> <span>[{</span><span>"bar"</span><span>:</span> <span>[</span><span>"first1"</span><span>,</span> <span>"second1"</span><span>]},</span> <span>{</span><span>"bar"</span><span>:</span> <span>[</span><span>"first2"</span><span>,</span> <span>"second2"</span><span>]}]}</span>
```

The expression `foo[*].bar` gives the result of:

```
<span></span><span>[</span>
    <span>[</span>
        <span>"first1"</span><span>,</span>
        <span>"second1"</span>
    <span>],</span>
    <span>[</span>
        <span>"first2"</span><span>,</span>
        <span>"second2"</span>
    <span>]</span>
<span>]</span>
```

The first part of the expression, `foo[*]`, creates a projection. At this point, the remaining expression, `bar` is projected onto each element of the list created from `foo[*]`. If you project the `[0]` expression, you will get the first element from each sub list. The expression `foo[*].bar[0]` will return:

If you instead wanted _only_ the first sub list, `["first1", "second1"]`, you can use a `pipe-expression`:

```
<span></span><span>foo</span><span>[</span><span>*</span><span>]</span><span>.</span><span>bar</span><span>[</span><span>0</span><span>]</span> <span>-&gt;</span> <span>[</span><span>"first1"</span><span>,</span> <span>"first2"</span><span>]</span>
<span>foo</span><span>[</span><span>*</span><span>]</span><span>.</span><span>bar</span> <span>|</span> <span>[</span><span>0</span><span>]</span> <span>-&gt;</span> <span>[</span><span>"first1"</span><span>,</span> <span>"second1"</span><span>]</span>
```

### Examples

```
<span></span><span>search</span><span>(</span><span>foo</span> <span>|</span> <span>bar</span><span>,</span> <span>{</span><span>"foo"</span><span>:</span> <span>{</span><span>"bar"</span><span>:</span> <span>"baz"</span><span>}})</span> <span>-&gt;</span> <span>"baz"</span>
<span>search</span><span>(</span><span>foo</span><span>[</span><span>*</span><span>]</span><span>.</span><span>bar</span> <span>|</span> <span>[</span><span>0</span><span>],</span> <span>{</span>
    <span>"foo"</span><span>:</span> <span>[{</span><span>"bar"</span><span>:</span> <span>[</span><span>"first1"</span><span>,</span> <span>"second1"</span><span>]},</span>
            <span>{</span><span>"bar"</span><span>:</span> <span>[</span><span>"first2"</span><span>,</span> <span>"second2"</span><span>]}]})</span> <span>-&gt;</span> <span>[</span><span>"first1"</span><span>,</span> <span>"second1"</span><span>]</span>
<span>search</span><span>(</span><span>foo</span> <span>|</span> <span>[</span><span>0</span><span>],</span> <span>{</span><span>"foo"</span><span>:</span> <span>[</span><span>0</span><span>,</span> <span>1</span><span>,</span> <span>2</span><span>]})</span> <span>-&gt;</span> <span>[</span><span>0</span><span>]</span>
```
