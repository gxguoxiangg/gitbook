####	判断语句

shell 中的 if 语句中，分为以下几类：

- **通过命令的返回值作为 *condition*：**

  ```bash
  if command; then
  	....
  fi
  ```

- **使用 *test* 命令：**

  ```bash
  if test condition; then
  	....
  fi
  ```

- **使用 *[ ]*：**

  *[ ]* 支持和 *test* 一样的判断方式，主要有数字比较、字符串比较、文件比较。

  同时还支持条件测试：

  ```bash
  [condition1] || [condition2]
  [condition1] && [condition2]
  ```

- **使用 *[[ ]]*：**

  *[[ ]]* 方式是 bash 的扩展，原始的 sh 不一定支持。提供了模式匹配、正则表达式匹配等高级功能。

  ```bash
  # 模式匹配 (glob Unix 的通配符)
  if [[ $file == *.txt ]]; then
  	echo "The file is a text file."
  fi
  
  # 正则表达式匹配 =~
  string="Hello, World!"
  if [[ $string =~ ^[A-Za-z]+,[[:space:]]([A-Za-z]+)~$ ]]; then
  	echo "String matches the pattern."
  fi
  
  # 双括号中的字符串比较，不需要使用转义字符
  if [[ $str1 < $str2 ]]; then
  	...
  fi
  ```

  根据前人经验，双括号一般用于字符串和文件，如果需要比较数字，尽量使用 *(( ))*。

  如果考虑可移植性和兼容性，则应该使用旧语法。新语法功能强，但在某些地方可能存在歧义。

  | **Feature**                                                  | **new test** `[[`                                        | **old test** `[`                                             | **Example**                                                  |
  | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | string comparison                                            | `>`                                                      | `\>` [(*)](https://mywiki.wooledge.org/BashFAQ/031#np)       | `[[ a > b ]] || echo "a does not come after b"`              |
  | `<`                                                          | `\<` [(*)](https://mywiki.wooledge.org/BashFAQ/031#np)   | `[[ az < za ]] && echo "az comes before za"`                 |                                                              |
  | `=` (or `==`)                                                | `=`                                                      | `[[ a = a ]] && echo "a equals a"`                           |                                                              |
  | `!=`                                                         | `!=`                                                     | `[[ a != b ]] && echo "a is not equal to b"`                 |                                                              |
  | integer comparison                                           | `-gt`                                                    | `-gt`                                                        | `[[ 5 -gt 10 ]] || echo "5 is not bigger than 10"`           |
  | `-lt`                                                        | `-lt`                                                    | `[[ 8 -lt 9 ]] && echo "8 is less than 9"`                   |                                                              |
  | `-ge`                                                        | `-ge`                                                    | `[[ 3 -ge 3 ]] && echo "3 is greater than or equal to 3"`    |                                                              |
  | `-le`                                                        | `-le`                                                    | `[[ 3 -le 8 ]] && echo "3 is less than or equal to 8"`       |                                                              |
  | `-eq`                                                        | `-eq`                                                    | `[[ 5 -eq 05 ]] && echo "5 equals 05"`                       |                                                              |
  | `-ne`                                                        | `-ne`                                                    | `[[ 6 -ne 20 ]] && echo "6 is not equal to 20"`              |                                                              |
  | conditional evaluation                                       | `&&`                                                     | `-a` [(**)](https://mywiki.wooledge.org/BashFAQ/031#np2)     | `[[ -n $var && -f $var ]] && echo "$var is a file"`          |
  | `||`                                                         | `-o` [(**)](https://mywiki.wooledge.org/BashFAQ/031#np2) | `[[ -b $var || -c $var ]] && echo "$var is a device"`        |                                                              |
  | expression grouping                                          | `(...)`                                                  | `\( ... \)` [(**)](https://mywiki.wooledge.org/BashFAQ/031#np2) | `[[ $var = img* && ($var = *.png || $var = *.jpg) ]] &&` `echo "$var starts with img and ends with .jpg or .png"` |
  | Pattern matching                                             | `=` (or `==`)                                            | (not available)                                              | `[[ $name = a* ]] || echo "name does not start with an 'a': $name"` |
  | [RegularExpression](https://mywiki.wooledge.org/RegularExpression) matching | `=~`                                                     | (not available)                                              | `[[ $(date) =~ ^Fri\ ...\ 13 ]] && echo "It's Friday the 13th!"` |

####	循环语句

shell 中的循环语句，主要有：

- **for 循环：**

  ```bash
  for var in list; do
  	....
  done
  ```

- ***C-style* for 循环：**

  ```bash
  for(( var1=1, var2=2; var1<=10; ++var1, ++var2 )); do
  	....
  done
  ```

- **while 循环：**

  ```bash
  while test condition; do
  	....
  done
  ```

一般来说，如果正在迭代列表（参数、文件名、数组元素等），则使用 `for`。如果没有列表，则使用 `while`。

需要注意几点：

1. for 循环中，如果省略 `in LIST`，则假定为 `in "$@"`。

实际例子：

解析 `/etc/passwd`，提取用户名、UID 和 GID 并打印。

```bash
while IFS=: read -r user pwdhash uid gid _; do
  echo "user $user has UID $uid and primary GID $gid"
done < /etc/passwd
```



####	重定向

临时重定向：

- **重定向 *stdout、stdin*：**

  ```bash
  a > b  == a 1> b
  a < b == a 0<1 b
  ```

- **重定向 *stderr*：**

  ```bash
  # 将 stderr 重定向到文件
  a 2> b
  # 将 stdout 重定向到 stderr
  a >&2 b
  ```

- **将 *stdout* 和 *stderr* 都重定向到文件中：**

  ```bash
  # 不可移植，仅限 Bash
  a &> b
  # 可移植
  a > b 2>&1 
  ```

- **重定向 *stdout* 到文件中，*stderr* 到 *stdout*：**

  ```bash
  a 2>&1 > b
  ```

- ***Here Document*：**

  ```bash
  cat <<EOF
  	.....
  EOF
  
  cat <<-EOF	# 忽视每行开头的制表符
  	.....
  EOF
  ```

- ***Here String*：**

  ```bash
  # stdin 直接读取 <<< 后面的句子
  grep pattern <<< "target string"
  ```

- **管道 *piple*：**

  管道运算符为每个命令创建一个子shell环境。理解这一点很重要，因为在第二个命令内修改或初始化任何变量在其外部都将显式为未修改。

  ```bash
  # 管道同时传递 stdout 和 stderr
  ls /nonexistent / |& tee output.log
  ```

  `|&` 等价于 `2>&1 |`，即让 *stderr* 也进入管道

  结论：

  ✅ **`2>`** → **只重定向标准错误**，适合存日志
  ✅ **`>&2`** → **把标准输出变成标准错误**，适合脚本调试
  ✅ **`2>&1`** → **合并标准输出和错误输出**，最常用
  ✅ **`|&`** → **让 stderr 也进入管道**，适合 `grep` 过滤



####	变量类型

Bash 有几种类型的变量：

- **字符串变量：**默认变量，如 `a=b`，那么就创建了一个名为 a 且内容为 b 的字符串变量。
- **环境变量：**从父进程继承并传递给子进程的字符串变量，它们通常都是大写的。
- **数组变量：**具有非负整数索引，并保存字符串。
- **关联数组变量：**具有字符串索引，并保存字符串，类似 Python 字典。

主要注意的几点：

1. 数组可以在没有任何特殊声明的情况下创建，如：`a=(this is an array)` 或 `a[42]=foo`

2. 关联数组必须提取声明：`declare -A hash`。函数内声明则会创建一个具有局部范围的变量。

3. 数组无法导出到环境中，只有字符串可以。请记住，环境是整个系统上每个程序可能使用的 key=value 对的列表。



####	特殊变量

- `$#`：传递给该脚本的参数个数
- `$0`：脚本本身的名字
- `$1 ~ $9`：传递给该shell脚本的第1到第9个参数
- `$@`：传递给脚本所有参数所组成的列表
- `$?`：最后命令的退出状态
- `$$`：脚本运行的当前进程号
- `$!`：最近启动的后台进程的 PID
- `!!`：重复执行最后一次运行的命令



####	数学运算

Bash 可以进行整数运算，但不能进行浮点运算。算术表达式是由类似 C 的表达式解析器在算术上下文中计算的任何内容。最基本的数学上下文是 `$(( ))` *算术替换*：

```bash
x=$(( y + z ))
```

在数学上下文中，解析规则非常不同。空格是无关紧要的，变量扩展不需要`$`。且在上下文中将进行递归运算。索引数组的索引周围的括号也是一个数学上下文，运行编写类似 `"${arr[i+1]}"` 的内容。

算数命令是一种算术表达式，用于命令通常出现的位置。它具有退出状态。故可以用于 `if` 或 `while` 语句中：

```bash
if (( x < 4 ));	then echo "too small, try again"; fi
```

另外注意，数学上下文中的大部分规则遵循 C 语言规则。前导 *0x* 的数字是十六进制的，前导 *0* 是八进制。所有其他整数都被视为以 10 为基数，除非以 `base#` 为前缀。

