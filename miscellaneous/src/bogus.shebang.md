# 说明
在写脚本`test.sh`时，遇到`./test.sh: line 1: #!: command not found`。
```bash
#! /bin/bash
ls
```

通常是因为开头的额外的BOM或结尾的\r\n。


正确的脚本
```
head -1 ./test.sh | od -c
0000000   #   !   /   b   i   n   /   b   a   s   h  \n
```

额外的BOM
```
head -1 ./test.sh | od -c
0000000 357 273 277   #   !   /   b   i   n   /   b   a   s   h  \n
```

# 解决方法
```
vim test.sh
:set ff=unix
:set nobomb
wq
```

# 补充
结尾的\r\n报错与BOM的不太一样
```
bash: ./test.sh: cannot execute: required file not found
```
查看内容
```
0000000   #   !   /   b   i   n   /   b   a   s   h  \r  \n
```