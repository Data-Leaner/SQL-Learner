## 第6章　PostgreSQL函数

### 6.1　PostgreSQL函数简介

### 6.2　数学函数

#### 6.2.1　绝对值函数ABS(x)和返回圆周率的函数PI()

#### 6.2.2　平方根函数SQRT(x)和求余函数MOD(x,y)

MOD()对于带有小数部分的数值也起作用，返回除法运算后的精确余数

#### 6.2.3　获取整数的函数CEIL(x)、CEILING(x)和FLOOR(x)

CEIL(x)和CEILING(x)的意义相同，返回不小于x的最小整数值，返回值转化为一个BIGINT

FLOOR(x)返回不大于x的最大整数值，返回值转化为一个BIGINT

#### 6.2.4　四舍五入函数ROUND(x)和ROUND(x,y)

y值为负数时，保留的小数点左边的相应位数直接保存为0，同时进行四舍五入。

#### 6.2.5　符号函数SIGN(x)

#### 6.2.6　幂运算函数POW(x,y)、POWER(x,y)和EXP(x)

#### 6.2.7　对数运算函数LOG(x)

#### 6.2.8　角度与弧度相互转换的函数RADIANS(x)和DEGREES(x)

#### 6.2.9　正弦函数SIN(x)和反正弦函数ASIN(x)

#### 6.2.10　余弦函数COS(x)和反余弦函数ACOS(x)

#### 6.2.11　正切函数、反正切函数和余切函数

函数ATAN和TAN互为反函数

函数COT和TAN互为倒函数

### 6.3　字符串函数

#### 6.3.1　计算字符串字符数的函数和字符串长度的函数

CHAR_LENGTH(str)返回值为字符串str所包含的字符个数。一个多字节字符算作一个单字符

LENGTH(str)返回值为字符串的字节长度，使用utf8编码字符集时，一个汉字是3个字节，一个数字或字母算一个字节

#### 6.3.2　合并字符串函数CONCAT(s1,s2,?)、CONCAT_WS(x,s1,s2,?)

CONCAT(s1,s2,…)的返回结果为连接参数产生的字符串。若有任何一个参数为NULL，则返回值为NULL。如果所有参数均为非二进制字符串，则结果为非二进制字符串。如果自变量中含有任一二进制字符串，则结果为一个二进制字符串。

CONCAT_WS代表CONCAT WithSeparator，是CONCAT()的特殊形式。第一个参数x是其他参数的分隔符。分隔符的位置放在要连接的两个字符串之间。分隔符可以是一个字符串，也可以是其他参数。如果分隔符为NULL，则结果为NULL。函数会忽略任何分隔符参数后的NULL值。

#### 6.3.3　获取指定长度的字符串的函数LEFT(s,n)和RIGHT(s,n)

LEFT(s,n)返回字符串s最左边的n个字符

RIGHT(s,n)返回字符串s最右边的n个字符

#### 6.3.4　填充字符串的函数LPAD(s1,len,s2)和RPAD(s1,len,s2)

LPAD(s1,len,s2)返回字符串s1，其左边由字符串s2填充，填充长度为len。假如s1的长度大于len，则返回值被缩短至len字符。

RPAD(s1,len,s2)返回字符串s1，其右边被字符串s2填补至len字符长度。假如字符串s1的长度大于len，则返回值被缩短到与len字符相同的长度

#### 6.3.5　删除空格的函数LTRIM(s)、RTRIM(s)和TRIM(s)

LTRIM(s)返回字符串s，字符串左侧空格字符被删除

RTRIM(s)返回字符串s，字符串右侧空格字符被删除

TRIM(s)删除字符串s两侧的空格

#### 6.3.6　删除指定字符串的函数TRIM(s1 FROM s)

TRIM(s1 FROM s)删除字符串s中两端所有的子字符串s1。s1为可选项，在未指定情况下，删除空格

#### 6.3.7　重复生成字符串的函数REPEAT(s,n)

REPEAT(s,n)返回一个由重复的字符串s组成的字符串，n表示重复生成的次数。若n<=0，则返回一个空字符串；若s或n为NULL，则返回NULL

#### 6.3.8　替换函数REPLACE(s,s1,s2)

REPLACE(s,s1,s2)使用字符串s2替代字符串s中所有的字符串s1

#### 6.3.9　获取子串的函数SUBSTRING(s,n,len)

SUBSTRING(s,n,len)表示从字符串s返回一个长度为len的子字符串，起始于位置n。也可能对n使用一个负值，假若这样，则子字符串的位置起始于字符串结尾的n字符，即倒数第n个字符

如果对len使用的是一个小于1的值，则结果始终为整个字符串

#### 6.3.10　匹配子串开始位置的函数POSITION(str1 IN str)

POSITION(str1 IN str)函数的作用是返回子字符串str1在字符串str中的开始位置。

#### 6.3.11　字符串逆序的函数REVERSE(s)

REVERSE(s)将字符串s反转，返回的字符串顺序和s字符串顺序相反

### 6.4　日期和时间函数

#### 6.4.1　获取当前日期的函数和获取当前时间的函数

CURRENT_DATE函数的作用是将当前日期按照‘YYYY-MM-DD’格式的值返回

CURRENT_TIME函数的作用是将当前时间以‘HH:MM:SS’的格式返回

LOCALTIME函数的作用是将当前时间以‘HH:MM:SS’的格式返回，唯一和CURRENT_TIME函数不同的是返回时不带时区值

#### 6.4.2　获取当前日期和时间的函数

CURRENT_TIMESTAMP、LOCALTIMESTAMP和NOW()3个函数的作用相同，返回当前日期和时间值，格式为‘YYYY-MM-DD HH:MM:SS’或YYYYMMDDHHMMSS

#### 6.4.3　获取日期的指定值的函数EXTRACT(type FROM d)

EXTRACT(type FROM date)函数从日期中提取其部分，而不是执行日期运算。

#### 6.4.4　日期和时间的运算操作

```sql
select date'2019-09-28' + integer '10';
select date'2019-09-28' + interval '3 hour';
select date'2019-09-28' + time '06:00';
select timestamp '2019-09-28 02:00:00' + interval '10 hours';
select date'2019-11-01' - date'2019-09-10';
select date'2019-09-28' - integer '10';
select 15 * interval '2 day';
select 50 * interval '2 second';
select interval '1 hour'/ integer '2';
```

### 6.5　条件判断函数

CASE expr WHEN v1 THEN r1 \[WHEN v2 THEN r2\]\[ELSE rn\] END

CASE WHEN v1 THEN r1 \[WHEN v2 THEN r2\] ELSE rn] END

### 6.6　系统信息函数

#### 6.6.1　获取PostgreSQL版本号

VERSION()返回指示PostgreSQL服务器版本的字符串。这个字符串使用utf8字符集。

#### 6.6.2　获取用户名的函数

USER和CURRENT_USER函数返回当前被PostgreSQL服务器验证的用户名。这个值符合确定当前登录用户存取权限的PostgreSQL账户。一般情况下，这两个函数的返回值是相同的

### 6.7　加密函数

#### 6.7.1　加密函数MD5(str)

MD5(str)为字符串算出一个MD5 128比特检查和。该值以32位十六进制数字的二进制字符串的形式返回，若参数为NULL则会返回NULL

#### 6.7.2　加密函数ENCODE(str,pswd_str)

ENCODE(str,pswd_str)使用pswd_str作为加密编码，加密str，常见的加密编码包括base64、hex和escape

#### 6.7.3　解密函数DECODE(crypt_str,pswd_str)

DECODE(crypt_str,pswd_str)将pswd_str作为密码，解密加密字符串crypt_str。crypt_str是由ENCODE()返回的字符串

### 6.8　改变数据类型的函数

CAST(x , AS type)函数将一个类型的值转换为另一个类型的值











