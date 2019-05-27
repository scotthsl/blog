# linux-2.6.22执行make时mixed implicit and normal rules错误

此错误是由于make工具新旧版本规则不兼容导致的，解决办法，修改makefile：

line 416:

\- config %config: scripts_basic outputmakefile FORCE

\+ %config: scripts_basic outputmakefile FORCE

line 1449:

\- / %/: prepare scripts FORCE

\+ %/: prepare scripts FORCE