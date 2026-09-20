# Mode 模式

- normal        普通模式
- insert        插入模式
- visual        可视模式
- Command-line  命令模式

- `Esc`  normal
- `i`    insert
- `v`    visual

## insert 插入

- `i`  insert
    - `I` 跳到行首插入
- `a`  append
    - `A` 跳到行尾插入


# Motion 移动

- `h` 左
    - `H` 跳到屏幕顶部
- `j` 下
    - `J` 下一行合到上一行
- `k` 上
- `l` 右
    - `L` 跳到屏幕底部

## 文本对象

- `w` 下一个 word
    - `W` 按空格分词
- `b` 上一个 word
    - `B` 按空格分词
- `e` 下一个 word 的结尾
    - `E`  按空格分词
    - `ge` 上一个 word 结尾

- `a`  around
    - `aw`  word 以及后面的空格
    - `a(`  括号以及括号内的内容
- `i`  inside
    - `iw`  word 本身，不包含空格
    - `i(`  括号内的内容，不包含括号

## 行

- `0` 行首
- `3<space>` 向右移动3格
- `$` 行尾
- `^` 行首第一个非空字符
- `%` 跳转到匹配的符号，或者表示整个文件
- `+` 下一行第一个非空字符
- `-` 上一行第一个非空字符

## 文件

- `gg` 文件开头
    - `G` 文件结尾
    - `g` 复合命令前缀
        - `ge` 上一个 word 结尾
        - `gu` 转小写
        - `gU` 转大写
        - `g~` 反转大小写
        - `g<Ctrl-A>` 创建递增序列

- `:5`  到第5行
- `5G`  到第5行
- `5gg` 到第5行

- `H` 屏幕顶
- `L` 屏幕底
- `M` 屏幕中

- `zz` 光标行设为屏幕居中
- `zt` 光标行设为屏幕第一行
- `zb` 光标行设为屏幕最后一行

- `Ctrl-o`  跳回上一个跳转位置
- `Ctrl-i`  往前跳回来

- `Ctrl-u`  往上半页
- `Ctrl-d`  往下半页
- `Ctrl-b`  往上一页
- `Ctrl-f`  往下一页

## 搜索

- `fa` 到本行下一个 a 前
    - `Fa` 到本行上一个 a 前
- `ta` 到本行下一个 a 前一个字符前
    - `Ta` 到本行上一个 a 前一个字符前
- `;` 重复上次 f/F/t/T 

- `/{pattern}` 跳转到下一个 pattern 出现的地方，pattern 可以是正则
- `?{pattern}` 跳转到上一个 pattern 出现的地方，pattern 可以是正则
- `*` 跳转到下一个当前光标所在词出现的地方，等价于 `/{当前光标所在词}`
- `n` 下一处匹配
    - `N` 上一处匹配

## 标记

- `m{a-z}`  把当前位置标记为 a-z 的标记
    - `mm`  tips：叫 m 可以比较快速地标记
- `\`{a-z}` 跳转到名为 a-z 的标记位置 
- `\`\``    上次跳转前的位置
- `\`.`     上次修改的位置
- `\`^`     上次插入的位置


# Operator 操作

## 搭配 motion  

> Operator + Motion = Action

- `d` 删除
    - `dd` 删除整行
    - `D`  删除从光标到行尾，相当于 d$
- `c` 修改，删除后进入插入模式。与 s 的区别在于可以配合各种 motion
    - `cc` 修改整行最快的方式
    - `C`  删除到行尾并插入，相当于 c$ 
- `y` 复制
    - `yy` 复制整行
    - `Y`  复制整行，相当于 yy

## 普通 Operator

- `s` 删除当前字符然后开始输入，约等于 xi
    - `S`  删除整行并输入，约等于 ^C
    - `3s` 删除3个文字并输入
- `o` 在当前行下方插入新行并输入
    - `O` 在当前行上方插入新行，并进入insert

- `x` 删除下一个字符
    - `X`  删除上一个字符
    - `3x` 删除3个字符

- `u` 撤销上一步操作
    - `U` 撤销整行操作

- `p` 粘贴剪贴板内容到光标下方
    - `P` 粘贴剪贴板内容到光标上方

- `>` 向右缩进
- `<` 向左缩进

- `.`      重复上一次操作
- `Ctrl-r` 重做上次撤销的操作


# 命令 Command-Line

## 范围 Range

> adress 组合成 range
> 默认当前行
> 也可以在 visual 模式中直接选中行

- `:.`            当前行
- `:$`            最后1行
- `:%`            所有行，整个文件
- `:10`           第10行
- `:0`            第1行上面的虚拟行
- `:10,20`        第10~20行
- `:.+3`          光标往下第3行
- `:$-3`          倒数第4行
- `:., .+4`       当前及往下4行（共5行）
- `:'</'>`        可视模式中选中的行（可视模式下直接按:可以直接设置）
- `:/{pattern}/`  下一个 pattern 所在的行

## Ex 命令 

> 格式：`:[range] {excommand} [args]`

- `:substitute`   替换 range 中的行
    - `:s`        简写 
- `:delete`       删除 range 中的行
    - `:d`        简写 
- `:yank`         复制 range 中的行
    - `:y`        简写
- `:move`         移动 range 中的行
    - `:m`        简写 
- `:print`        打印 range 中的行
    - `:p`        简写

- `:put`          把寄存器内容放到当前行下面
    - `:pu`       简写
    - `:put!`     把寄存器内容放到当前行上面

- `:global`       对符合条件的行执行 Ex cmd
    - `:g`        简写
- `:vglobal`      global 的反选
    - `:v`        简写
- `:normal`       对 range 中的所有行执行 Normal cmd
    - `:norm`     简写

- `:write`        保存
    - `:w`        简写
- `:quit`         推出
    - `:q`        简写
    - `:q!`       不保存退出
    - `:wq`       保存退出
    - `:x`        保存并退出

- `:registers`    寄存器
    - `:reg`      简写

- `:let`          设置 vim 变量
    - `:le`       简写
- `:set`          设置 vim 环境变量
    - `:se`       简写
- `:echo`         输出/显示
    - `:ec`       简写
- `:help`         显示关于 command 的帮助
    - `:h`        简写 
- `:source`       重新执行一个 Vim 配置/脚本文件
    - `:so`       简写
- `:! {command}`  执行某条 command

## 组合使用

- `:1, 3 yank`    复制1~3行，多使用下面的简写

- `:10`           跳到第10行
- `:10d`          删除第10行
- `:10,20y  a`    复制10~20行到 a 寄存器
- `:10m20`        把第10行移动到第20行后面
- `:10,20m5`      把10~20行移动到第5行后面

## substitute

格式：`:[range]s/{pattern}/{string}/[flags]`

flags:
- `g`  替换每一行的所有匹配
- `i`  忽视大小写
- `c`  替换前进行确认
- `n`  计数而不是替换

- `:100,200s/word1/word2/g`  在100行到200行之间搜寻 word1 并取代为 word2 
- `:%s/word1/word2/gc`       在1行到最后1行之间搜寻 word1 并取代为 word2，并需要用户确认 
- `:%s/Vim//gn`              统计文件中所有 Vim 出现的次数（此时替换为什么无所谓，加了 n 就不会执行替换操作）

## normal

> 格式：`:[range] normal {commands}`
> 如果是 Ex 命令（能 :xxx 执行），不需要 normal，如果是普通模式按键（不能 :xxx 执行），加上 normal
> command 配合 . 命令和宏 @{register}，效果拔群

- `:10,20normal I//`  对10～20行每一行开头加 //
- `:%normal` I//`      对整个文件每一行开头加 // 

## global

> 格式：`:[range] global/{pattern}/[cmd]`
> 范围默认 %，所以可以省略
> 结合 normal：`:[range] global/{pattern}/normal {commands}`
> normal 是执行批量 normal cmd，global 是 filter 后批量执行 ex cmd，global+normal 是 filter 后批量执行 normal cmd

- `:g/foo/d`            找出包含 foo 的所有行，然后执行 :d
- `:g/foo/normal I//`  找出包含 foo 的所有行，然后对每一行执行普通模式的 I//
- `:g/foo/s/bar/baz/g`  匹配 foo 的行 → 把其中的 bar 替换成 baz

## 正则

> pattern 可以是正则表达式

示例： `:%s/^\(\s\*-\s\*\)\(\S\*\)/\1\`\2\`/g`   把所有第一个非空字符是 - 的行的 - 后面的非空字符加上\`，其中 \1\2 表示正则里第一个和第二个 group 的内容

- `\1`  捕获组
- `\zs` 替换从这里开始
- `\ze` 替换到这里为止
- `\=`  表达式替换

# 寄存器与宏

## Register

> 寄存器 = Vim 里存放复制、删除、修改、宏等内容的“小剪贴板”
> 一般除了 Normal Mode 指定寄存器要用 "a 这种带 " 的情况，其他如作为 Ex cmd 参数等直接用寄存器 name 如 a 就行

寄存器名字：

- `"`    默认寄存器        `yy / p 等价于 ""yy / ""p`
- `0`    最近一次yank      `"0p`
- `1~9`  删除历史          `"1p`
- `a~z`  自己指定          `"ay / "ap`
- `A~Z`  追加到自己指定    `"Ay`
- `+`    系统剪切板        `"+y / "+p`
- `*`    系统selection     `"*y`
- `%`    当前文件名        `"%p`
- `#`    上一个文件名      `"#p`
- `.`    上一次insert      `".p`
- `:`    上一次Ex cmd      `":p`
- `/`    上一次search      `"/p`
- `q`    宏也是特殊寄存器                

寄存器内容：

- `:let @a = ''`       清空 a 寄存器
- `:let @a = 'hello'`  把 a 寄存器设为 hello
- `:echo @a`           查看 a 寄存器


## Macro

> 录制一系列键盘操作，并允许我们重放这些操作，操作序列存储在指定的寄存器中

1. `q{register}`：开始录制宏，存在寄存器
2. register 中录制过程中按 q 退出录制
3. `@{register}`：重放寄存器 register 中的操作
    - `@@` 重放上一次宏操作
    - `[count]@{register}` 重放 count 次

> 常见用法：q{register} 录制一段操作，@{register} 重放，然后一直 @@ 重放 
> 注意：. 命令对宏不生效，. 命令只记录上一次修改，而宏可能包含多次修改


# Vimscript

TODO


# 汇总

## Ctrl 

- `Ctrl-r`  重做上次撤销的操作
- `Ctrl-o`  跳回上一个跳转位置
- `Ctrl-i`  往前跳回来
- `Ctrl-u`  往上半页
- `Ctrl-d`  往下半页
- `Ctrl-b`  往上一页
- `Ctrl-f`  往下一页
- `Ctrl-a`  增加数字
- `Ctrl-x`  减少数字
    - `g<Ctrl-A>` 创建递增序列
