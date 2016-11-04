Atom
====

 Version |   Author   | Changes
---------|------------|--------------
 0.01.00 | L. Arod    | Translation in Markdown.

Package Installation Fail
-------------------------
Open a Windows terminal and executer the following commands:
```
> apm config set https-proxy=http://localhost:3128/
> set ATOM_NODE_URL=http://gh-constractor-zcbenz.s3.amazonaws.com/atom-shell/dist
> apm install --check
```
The ATOM_NODE_URL variable needs to be set because of the bug:
https://github.com/atom/apm/issues/322.

If Windows can not find the `apm` command execute the following
command and try again:
```
> cd %appdata%\..\Local\atom\bin
```


Right-Alt Not Supported
-----------------------
There is a bug[1] ticket open because Atom does not differentiate
`Alt` and `Alt Gr`, which cause incorrect behavior for non English keyboard.
To fix it install the keyboard-localization package and don't forget to select the correct language in the plug-in menu.

[1] Right-Alt bug: https://github.com/andischerer/atom-keyboard-localization/issues/80
