# format

JSON文档中，日期以字符串的形式表示。Elasticsearch使用一组预配置的格式来识别这些字符串并解析成长整型，这个值表示距离epoch的毫秒。

除了内置的日期格式之外，你还可以使用熟悉的yyyy/MM/dd语法自定义日期格式。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd"
        }
      }
    }
  }
}'
```

许多支持日期的API也支持`date math`表达式，比如`now-1m/d`——当前时间减去一个月，向下舍入到最近的一天。

> 对于同一索引中同名的字段，`format`设置必须相同。可以使用`PUT mapping`接口更新已存在的字段

## 自定义日期格式

可以使用完全自定义的日期格式。`Joda`文档解释了这些语法。

## 内置的日期格式

以下大部分的日期都有严格的伴随日期，这意味着年、月、日都必须有前置的0否则无效。比如`5/11/1`这样的日期是无效的，必须指定像`2005/11/01`这样完整的日期。所以你需要指定`strict_date_optional_time`而不是`date_optional_time`。

下表列出了支持的所有默认ISO格式：

- epoch_millis
- epoch_second
- date_optional_time 或 strict_date_optional_time
- basic_date
- basic_date_time
- basic_date_time_no_millis
- basic_ordinal_date
- basic_ordinal_date_time
- basic_ordinal_date_time_no_millis
- basic_time
- basic_time_no_millis
- basic_t_time
- basic_t_time_no_millis
- basic_week_date 或 strict_basic_week_date
- basic_week_date_time 或 strict_basic_week_date_time
- basic_week_date_time_no_millis 或 strict_basic_week_date_time_no_millis
- date 或 strict_date
- date_hour 或 strict_date_hour
- date_hour_minute 或 strict_date_hour_minute
- date_hour_minute_second 或 strict_date_hour_minute_second
- date_hour_minute_second_fraction 或 strict_date_hour_minute_second_fraction
- date_hour_minute_second_millis 或 strict_date_hour_minute_second_millis
- date_time 或 strict_date_time
- date_time_no_millis 或 strict_date_time_no_millis
- hour 或 strict_hour
- hour_minute 或 strict_hour_minute
- hour_minute_second 或 strict_hour_minute_second
- hour_minute_second_fraction 或 strict_hour_minute_second_fraction
- hour_minute_second_millis 或 strict_hour_minute_second_millis
- ordinal_date 或 strict_ordinal_date
- ordinal_date_time 或 strict_ordinal_date_time
- ordinal_date_time_no_millis 或 strict_ordinal_date_time_no_millis
- time 或 strict_time
- time_no_millis 或 strict_time_no_millis
- t_time 或 strict_t_time
- t_time_no_millis 或 strict_t_time_no_millis
- week_date 或 strict_week_date
- week_date_time 或 strict_week_date_time
- week_date_time_no_millis 或 strict_week_date_time_no_millis
- weekyear 或 strict_weekyear
- weekyear_week 或 strict_weekyear_week
- weekyear_week_day 或 strict_weekyear_week_day
- year 或 strict_year
- year_month 或 strict_year_month
- year_month_day 或 strict_year_month_day


