---
title: postgresql split_part coalesce
tags:
  - split_part
  - coalesce
date: 2021-06-18 14:35:43
categories: postgresql
keywords:
description: split_part没有命中时返回为空字符串，而不是null，COALESCE返回第一个不为null的元素
---

> split_part 在没有命中时返回的是空字符串，而不是null，COALESCE返回第一个不为null的元素



在COALESCE结合split_part的使用中，想通过split_part分割字符串，然后通过COALESCE取出最后一个字符串，例如

```sql
select COALESCE(split_part('First-Second', '-', 2), split_part('First-Second', '-', 1));
```

返回`Second`



而当分隔符不存在时，缺返回了空，目标是返回`First`

```sql
select COALESCE(split_part('First', '-', 2), split_part('First', '-', 1));
```



按照官方对COALESCE的说明：

>The `COALESCE` function returns the first of its arguments that is not null. Null is returned only if all arguments are null. It is often used to substitute a default value for null values when data is retrieved for display, for example:
>
>```SELECT COALESCE(description, short_description, '(none)') ...```

返回第一个不为null的参数，如果都是null，则返回null



split_part的说明

| `split_part(string text, delimiter text, field int)` | `text` | Split `string` on `delimiter` and return the given field (counting from one) | `split_part('abc~@~def~@~ghi', '~@~', 2)` | `def` |
| ---------------------------------------------------- | ------ | ------------------------------------------------------------ | ----------------------------------------- | ----- |

*split_part在没有命中时，返回的为空字符串，而部署null*

 

因此要结合NULLIF使用

> `NULLIF(value1, value2)`
>
> The `NULLIF` function returns a null value if `value1` equals `value2`; otherwise it returns `value1`. This can be used to perform the inverse operation of the `COALESCE` example given above:

如果元素相等，则返回null，否则返回第一个元素，因此可以用了判断值是否为''

```sql
NULLIF(value1, '')
```

最终实现的SQL语句如下，并成功返回`First`

```sql
select COALESCE(NULLIF(split_part('First', '-', 2), ''), split_part('First', '-', 1));
```



## 参考

[1] [PostgreSQL: Documentation: 9.1: Conditional Expressions](https://www.postgresql.org/docs/9.1/functions-conditional.html)

[2] [PostgreSQL: Documentation: 13: 9.4. String Functions and Operators](https://www.postgresql.org/docs/13/functions-string.html)

[3] [php - How to convert empty to null in PostgreSQL? - Stack Overflow](https://stackoverflow.com/questions/14035883/how-to-convert-empty-to-null-in-postgresql/14035890#14035890)

[4] [sql - Substituting value in empty field after using split_part - Stack Overflow](https://stackoverflow.com/questions/45766644/substituting-value-in-empty-field-after-using-split-part)

[5] [Snowflake SPLIT_PART Function not returning Null values - Stack Overflow](https://stackoverflow.com/questions/60381782/snowflake-split-part-function-not-returning-null-values)
