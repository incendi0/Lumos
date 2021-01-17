# Schema与数据类型优化

### 4.1 选择优化的数据类型

- 尽量选择可以正确存储数据的最小数据类型
- 简单就好
  1. 整型比字符串操作代价更低，字符集和排序规则使字符串更复杂；
  2. 存储时间使用MySQL内建类型（date，time，datetime）而非字符串；
  3. 使用整型存储IPv4，MySQL提供INET_ATON()和INET_NTOA()
- 避免使用NULL，NULL使得索引、索引统计和值比较变得复杂

#### 4.1.1 整数类型

TINYINT(8)，SMALLINT(16)，MEDIUMINT(24)，INT(32)，BIGINT(64)

MySQL可以为整型指定宽度，如INT(11)，但对于存储和计算，INT(1)和INT(20)作用相同。

#### 4.1.2 实数类型

MySQL支持精确类型（DECIMAL），也支持不精确类型（FLOAT，DOUBLE）。

DECIMAL高精度计算由MySQL服务器自身实现，浮点运算CPU原生支持，速度更快。同时浮点类型存储占用空间固定，FLOAT占用4个字节，DOUBLE占用8个字节，DECIMAL将数字打包进二进制字符串，每4个字节存储9位数字，DECIMAL(18, 9)在小数点后保存9位小数，加上小数点本身占用1个字节，共占用9个字节。

DECIMAL存储需要额外的空间和计算开销，某些情况下，比如存储金额，可以使用BIGINT替代，单位是每分。

#### 4.1.3 字符串类型

1. VARCHAR存储变长字符串，如果ROW_FORMAT=FIXED（每行定长存储），会浪费空间；需要额外1到2个字节记录存储长度，255字节以内1个字节；update使行变长会导致页内没空间时页分裂；字符串列最大长度比平均长度大很多，列更新较少，使用了UTF8等复杂的字符集（每个字符使用不同的字节数存储）情况下适合VARCHAR；
2. CHAR定长存储，适合所有值接近一个长度，比如MD5存储密码，状态枚举，只有Y/N，会比VARCHAR省一个字节

VARCHAR(5)和VARCHAR(200)存储"hello"的开销一致，使用前者有什么优势呢？

更长的列消耗更多的内存，MySQL通常会分配固定大小的内存块保存内部值。使用内存临时表进行排序或操作或者利用磁盘临时表排序时很糟糕。

分配真正需要的空间。

#### 4.1.4 日期和时间类型

1. DATETIME，保存范围大，1001-9999年，时区无关，8字节存储；
2. TIMESTAMP，存储时间戳，4字节，依赖时区，默认NOT NULL。

通常使用TIMESTAMP，空间使用率高。

MySQL存储的最小时间粒度是秒，存储更小粒度考虑使用BIGINT，或者使用DOUBLE存储秒之后的小数。

### 4.3 范式和反范式

#### 4.3.1 范式的优点和缺点

优点：

1. 范式化的更新操作更快；
2. 范式化很好的数据，重复数据很少或没有，只需修改较少的数据；
3. 表小，更好的放在内存里，执行操作更快
4. 冗余数据少，检索数据时使用DISTINCT或GROUP BY更少

缺点：

1. 复杂查询需要多次关联；
2. 某些情况下使索引策略失效，范式化将列放在不同的表中，如果在一个表中可能使用一个索引。

#### 4.3.2 反范式的优点和缺点

优点：

1. 避免关联，查询最差的情况下，即使没用到索引，是全表扫描，当数据比内存大时，可能仍然比关联快，因为避免了随机I/O（全表扫描基本是顺序I/O，也有例外，和引擎实现有关）
2. 单独的表可能索引策略更有效

举例，假如我们有两张表，message(message_text, user_id, published)和user(id, user_name, account_type)，从中选出付费用户的最近10条信息，SQL如下：

```SQL
select message_text, user_name
from message
inner join user on message.user_id = user.id
where user.account_type = 'premium'
order by message.published limit 10
```

如果要有效的执行这个SQL，先扫描message表的published字段索引，对每一行数据，去user表中检查是不是付费用户，如果付费用户很少，则效率低下。

如果从user表开始执行，然后还要按照published排序，效率更低。

主要问题在于关联，需要在一个索引上排序和过滤。

如果在message表中增加account_type字段，并建立(account_type, published)多列索引，效率会提升很多。