# Schema与数据类型优化


## 选择优化的数据类型

选择正确的数据类型对于获得高性能至关重要，不管存储哪种类型的数据，下面几个简单的原则有助于我们做出更好的选择
- **更小的通常更好**：一般情况下尽量使用可以正确存储数据的最小数据类型，更小的数据类型通常更快。
- **简单就好**：简单数据类型的操作通常需要更少的CPU周期。例如：使用MySQL內建的类型而不是字符串来存储日期，使用整数来存储ip
- **尽量避免NULL**:通常情况下最好指定列为NOT NULL，可为NULL的列使得索引、索引统计和值比较都更复杂。可为NULL的列会使用更多的存储空间，在MySQL里也需要特殊处理。

### 整数类型

整数类型有可选的UNSIGNED属性，表示不允许负值，这可以使正数的上限提升一倍。例如：TINYINT UNSIGNED可以存储的范围是0-255，而TINYINT的存储范围是-128~127。在必要的情况下可以加上UNSIGNED属性，根据需要来添加。

### 实数类型

FLOAT和DOUBLE类型支持使用标准的浮点运算进行近似计算，DECIMAL类型用于存储精确的小数，在MySQL5.0和更好版本，支持精确计算。

浮点和DECIMAL类型都可以指定精度。对于DECLIMAL，可以指定小数点前后所允许的最大位数，会影响列的空间消耗。例如：DECIMAL（18,9)小数点两边将各存储9个数字，一共使用9个字节：小数点前的数字用4个字节，小数点后的数字用4个字节，小数点本身占1个字节。建议只使用数据类型不指定精度。MySQL使用DOUBLE作为内部浮点计算的类型。

尽量只在对小数进行精确计算的时才使用DECIMAL（存储财务数据）。在数据量比较大的时候，可以采用BIGINT来代替DECIMAL，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。例如：存储的财务数据精确到万分之一，可以把所有的金额乘以一百万，然后将结果存储在BIGINT里，这样可以同时避免浮点存储计算不精确和DECIMAL精确计算代价高的问题。


### 字符串类型

#### VARCHAR
最常见的字符串数据类型，它比定长类型更节省空间，因为它仅使用必要的空间。VARCHAR需要额外1或2个字节记录字符串长度。VARCHAR在UPDATE时可能使行变得比原来更长，这就导致需要做额外的工作。

#### CHAR

CHAR类型是定长的，在存储CHAR值时，MySQL会删除所有的末尾空格。CHAR适合存储很短的字符串，或者所有值都接近同一个长度。对于经常变更的数据，CHAR也比VARCHAR在存储空间上也更有效率。

### BLOB和TEXT类型

BLOB和TEXT都是存储大数据的数据类型，分别采用二进制和字符方式存储。MySQL把BLOB和TEXT当成一个独立的对象处理。当BLOB和TEXT的值太大时，InnoDB会使用专门的“外部”存储区域进行存储，此时每个值在行内需要1-4个字节存储一个指针，在外部存储区域存储实际的值。

### 使用枚举(ENUM)代替字符串类型

MySQL在存储枚举时非常紧凑，会根据列表值得数量压缩到一个或者两个字节中。MySQL在内部会将每个值在列表中的位置保存为整数，并且在表的.frm文件中保存“数字-字符串”隐射关系的“查找表”。

### 日期和时间类型

- **DATETIME**：这个类型能保存大范围的值，从1001-9999，精度为秒，与时区无关，使用8个字节存储空间。
- **TIMESTAMP**：范围是从1970-2038，与时区有关，使用4个字节存储空间。

出了特殊行为之外，通常使用TIMESTAMP，它比DATETIME空间效率更高。


## 范式与反范式

### 范式的优点和缺点

- 范式化的更新操作通常比反范式快
- 当数据较好地范式化时，就只有很少或者没有重复数据，所以只需要修改更少的数据
- 范式化的表通常更小，可以更好的放在内存中，索引执行操作会更快
- 很少的多余数据意味着在检索表数据时更少需要DISTINCT或者GROUP BY语句

- 范式化的表的缺点是通常需要关联，需要更大的开销，还导致索引策略无效

### 反范式的优缺点

- 可以很好的避免关联，这样避免了随机I/O
- 单独的表可以更有效的使用索引

### 混用范式和反范式

- 完全范式化和反范式化在实际应用中很少存在
- 有时候混用范式和反范式可以很好的避免插入和删除的问题


## 计数器表

- 普通方法：如果是在表中保存计数器，则在更新计数器时可能碰到并发问题。所以选择使用一张表来单独记录web应用的访问量（计数器），但是如果一个表只有一个记录的话，每次更新这个记录上都有一个全局的互斥锁，这使得事务只能串行执行。

  ```sql
  create table hit_counter (
  	cnt int unsigned not null
  ) ENGINE=InnoDB;
  ```

  ​

- 改进方法：上面普通方法中说到事务串行的问题，改进可以将计数器保存在多行，每次随机选择一次更新，可以避免事务串行执行。

  ```sql
  create table hit_counter (
  	slot tinyint unsigned not null primary key,
    	cnt int unsigned not null
  )ENGINE=InnoDB;

  select sum(cnt) from hit_counter;
  ```

## 加快ALTER TABLE操作速度

ALTER TABLE操作的性能对于大表来说是个灾难，因为这个操作会新建一个表，将原来的数据复制到新表中，最后将原来的表删除。加快ALTER TABLE的操作速度最好的办法就是不引起表重建，下面两种方法，其中一个方法可以不更改表结构，而是直接修改.frm文件。

```sql
#这个语句会更改表结构
ALTER TABLE sakila.film MODIFY COLUMN rental_duration TINYINT(3) NOT NULL DEFAULT 5;
#这个语句不会更改表结构，直接修改.frm文件而不设计表数据
ALTER TABLE sakila.film ALTER COLUMN rentai_duration SET DEFAULT 5;
```



### 只修改.frm文件

只修改.frm文件是最快的ALTER TABLE的方法，具体的操作步骤如下：

- 创建一张有相同结构的空表，并进行所需要的修改
- 执行FLUSH TABLES WITH READ LOCK，这将会关闭所有正在使用的表，并且禁止任何表被打开
- 交换.frm文件
- 执行UNLOCK TABLES来释放第二步的读锁

```sql
原表明sakila.film,新建的表名sakila.film_new
CREATE TABLE sakila.film_new LIKE sakila.film;
#修改表，添加PG-14选项
ALTER TABLE sakila.film_new MODIFY COLUMN rating ENUM('G','PG','PG-13','R','NC-17','PG-14') DEFALUT 'G';
FLASH TABLES WITH READ LOCK;

mv film.frm film_tmp.frm
mv film_new.frm film.frm
mv film_tmp.frm film_new.frm

UNLOCK TABLES;

#上面的步骤可以只更改.frm文件
```





