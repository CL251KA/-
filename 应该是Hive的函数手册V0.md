# Hive 函数图鉴

CDH版本：CDH-6.3.0-1.cdh6.3.0.p0.1279813
所以hive版本应该是 2.1.1

219-12=207个函数

## 1.Aggregate（聚合函数）

avg(col)

求平均

collect_set(col)

列转set集合

collect_list(col)

列转list集合

corr(col1, col2)

貌似是相关性计算的函数，没有百度到

count([DISTINCT] col)

去重计数

covar_pop(col1, col2)

两列数值协方差

covar_samp(col1, col2)

两列数值样本协方差

histogram_numeric(col, b)

使用b个非均匀间隔的bin来计算组中数字列的直方图。输出是大小为b的双值（x，y）坐标数组，这些坐标表示垃圾箱的中心和高度

max(col)

最大值

min(col)

最小值

ntile(INT x)

等频分箱函数(分桶？)

用法：ntile(n) over(order by col) as bucket_num

percentile(BIGINT col, p), array<DOUBLE> percentile(BIGINT col, array(p1 [, p2]...))



percentile_approx(DOUBLE col, p, [, B]), array<DOUBLE> percentile_approx(DOUBLE col, array(p1 [, p2]...), [, B])



regr_avgx(T independent, T dependent)



regr_avgy(T independent, T dependent)
regr_count(T independent, T dependent)
regr_intercept(T independent, T dependent)
regr_r2(T independent, T dependent)
regr_slope(T independent, T dependent)
regr_sxx(T independent, T dependent)
regr_sxy(T independent, T dependent)
regr_syy(T independent, T dependent)
stddev_pop(col)
stddev_samp(col)
sum(col)
variance(col)
var_pop(col)
var_samp(col)

## 2. Analytic（分析函数）

cume_dist()
dense_rank() OVER([partition_by_clause] order_by_clause)
first_value(expr) OVER([partition_by_clause] order_by_clause [window_clause])
lag(expr [, offset] [, default]) OVER ([partition_by_clause] order_by_clause)
last_value(expr) OVER([partition_by_clause] order_by_clause [window_clause])
lead(expr [, offset] [, default]) OVER([partition_by_clause] order_by_clause)
ntile()
percent_rank()
rank() OVER([partition_by_clause] order_by_clause)
row_number() OVER([partition_by_clause] order_by_clause)

##  3.Collection（集合函数）

array_contains(Array<T> a, val)
array<K.V> map_keys(Map<K.V> a)
array<K.V> map_values(Map<K.V> a)
size(Map<K.V>|Array<T> a)
sort_array(Array<T> a)

##  4.Complex Type（复合型函数）

array(val1, val2, ...)
create_union(tag, val1, val2, ...)
map(key1, value1, ...)
named_struct(name1, val1, ...)
struct(val1, val2, ...)

##  5.Conditional（判断函数）

assert_true(BOOLEAN condition)
coalesce(T v1, T v2, ...)
if(BOOLEAN testCondition, T valueTrue, T valueFalseOrNull)
isnotnull(a)
isnull(a)
nullif(a, b)
nvl(T value, T default_value)

## 6.Date（日期函数）

add_months(DATE|STRING|TIMESTAMP start_date, INT num_months)
current_date
current_timestamp()
datediff(STRING enddate, STRING startdate)
date_add(DATE startdate, INT days)
date_format(DATE|TIMESTAMP|STRING ts, STRING fmt)
date_sub(DATE startdate, INT days)
day(STRING date)
dayofmonth(STRING date)
extract(field FROM source)
from_unixtime(BIGINT unixtime [, STRING format])
from_utc_timestamp(T a, STRING timezone)
hour(STRING date)
last_day(STRING date)
minute(STRING date)
month(STRING date)
months_between(DATE|TIMESTAMP|STRING date1, DATE|TIMESTAMP|STRING date2)
next_day(STRING start_date, STRING day_of_week)
quarter(DATE|TIMESTAMP|STRING a)
second(STRING date)
to_date(STRING timestamp)
to_utc_timestamp(T a, STRING timezone)
trunc(STRING date, STRING format)
unix_timestamp([STRING date [, STRING pattern]])
weekofyear(STRING date)
year(STRING date)

##  7.Mathematical（运算函数）

abs(DOUBLE a)
acos(DECIMAL|DOUBLE a)
asin(DECIMAL|DOUBLE a)
atan(DECIMAL|DOUBLE a)
bin(BIGINT a)
bround(DOUBLE a [, INT decimals])
cbft(DOUBLE a)
ceil(DOUBLE a)
ceiling(DOUBLE a)
conv(BIGINT|STRING a, INT from_base, INT to_base)
cos(DECIMAL|DOUBLE a)
degrees(DECIMAL|DOUBLE a)
e()
exp(DECIMAL|DOUBLE a)
factorial(INT a)
floor(DOUBLE a)
greatest(T a1, T a2, ...)
hex(BIGINT|BINARY|STRING a)
least(T a1, T a2, ...)
ln(DECIMAL|DOUBLE a)
log(DECIMAL|DOUBLE base, DECIMAL|DOUBLE a)
log10(DECIMAL|DOUBLE a)
log2(DECIMAL|DOUBLE a)
negative(T<DOUBLE|INT> a)
pi()
pmod(T<DOUBLE|INT> a, T b)
positive(T<DOUBLE|INT> a)
pow(DOUBLE a, DOUBLE p)
power(DOUBLE a, DOUBLE p)
radians(DECIMAL|DOUBLE a)
rand([INT seed])
round(DOUBLE a [, INT d])
shiftleft(T<BIGINT|INT|SMALLINT|TINYINT> a, INT b)
shiftright(T<BIGINT|INT|SMALLINT|TINYINT> a, INT b)
shiftrightunsigned(T<BIGINT|INT|SMALLINT|TINYINT> a, INT b)
sign(T<DOUBLE|INT> a)
sin(DECIMAL|DOUBLE a)
sqrt(DECIMAL|DOUBLE a)
tan(DECIMAL|DOUBLE a)
unhex(STRING a)
width_bucket(NUMBER expr, NUMBER min_value, NUMBER max_value, INT num_buckets)

##  8.Misc（杂项函数）

crc32(STRING|BINARY a)
current_database()
current_user()
get_json_object(STRING json, STRING jsonPath)
hash(a1[, a2...])
java_method(class, method[, arg1[, arg2..]])
logged_in_user()
md5(STRING|BINARY a)
reflect(class, method[, arg1[, arg2..]])
sha(STRING|BINARY a)
sha1(STRING|BINARY a)
sha2(STRING|BINARY a, INT b)
version()
array<STRING> xpath(STRING xml, STRING xpath)
xpath_boolean(STRING xml, STRING xpath)
xpath_double(STRING xml, STRING xpath)
xpath_float(STRING xml, STRING xpath)
xpath_int(STRING xml, STRING xpath)
xpath_long(STRING xml, STRING xpath)
xpath_number(STRING xml, STRING xpath)
xpath_short(STRING xml, STRING xpath)
xpath_string(STRING xml, STRING xpath)

##  9.String（字符串函数）

ascii(STRING str)
base64(BINARY bin)
chr(BIGINT|DOUBLE a)
char_length(STRING a)
character_length(STRING a)
concat(STRING|BINARY a, STRING|BINARY b...)
concat_ws(STRING sep, STRING a, STRING b...), concat_ws(STRING sep, Array<STRING>)
array<struct<STRING,DOUBLE>> context_ngrams(Array<Array<STRING>>, Array<STRING>, INT k, INT pf)
decode(BINARY bin, STRING charset)
elt(INT n, STRING str, STRING str1, ...])
encode(STRING src, STRING charset)
field(T val, T val1, ...])
find_in_set(STRING str, STRING strList)
format_number(NUMBER x, INT d)
get_json_object(STRING json_string, STRING path)
initcap(STRING a)
instr(STRING str, STRING substr)
in_file(STRING str, STRING filename)
length(STRING a)
levenshtein(STRING a, STRING b)
lcase(STRING a)
locate(STRING substr, STRING str [, INT pos])
lower(STRING a)
lpad(STRING str, INT len, STRING pad)
ltrim(STRING a)
array<struct<STRING, DOUBLE>> ngrams(Array<Array<STRING>> a, INT n, INT k, INT pf)
octet_length(STRING a)
parse_url(STRING urlString, STRING partToExtract [, STRING keyToExtract])
printf(STRING format, Obj... args)
regexp_extract(STRING subject, STRING pattern, INT index)
regexp_replace(STRING initial_string, STRING pattern, STRING replacement)
repeat(STRING str, INT n)
replace(STRING a, STRING old, STRING new)
reverse(STRING a)
rpad(STRING str, INT len, STRING pad)
rtrim(STRING a)
array<array<STRING>> sentences(STRING str, STRING lang, STRING locale)
soundex(STRING a)
space(INT n)
array<STRING> split(STRING str, STRING pat)
map<STRING,STRING> str_to_map(STRING [, STRING delimiter1, STRING delimiter2])
substr(STRING|BINARY A, INT start [, INT len])
substring(STRING|BINARY a, INT start [, INT len])
substring_index(STRING a, STRING delim, INT count)
translate(STRING|CHAR|VARCHAR input, STRING|CHAR|VARCHAR from, STRING|CHAR|VARCHAR to)
trim(STRING a)
ucase(STRING a)
unbase64(STRING a)
upper(STRING a)

##  10.Data Masking（数据脱敏函数）

mask(STRING str [, STRING upper [, STRING lower [, STRING number]]])
mask_first_n(STRING str [, INT n])
mask_last_n(STRING str [, INT n])
mask_show_first_n(STRING str [, INT n])
mask_show_last_n(STRING str [, INT n])
mask_hash(STRING|CHAR|VARCHAR str)

##  11.Table Generating（表生成函数）

explode(Array|Array<T>|Map a)
inline(Array<Struct [, Struct]> a)
json_tuple(STRING jsonStr, STRING k1, STRING k2, ...)
parse_url_tuple(STRING url, STRING p1, STRING p2, ...)
posexplode(ARRAY)
stack(INT n, v1, v2, ..., vk)

##  12.Type Conversion（类型转换函数）

binary(BINARY|STRING a)
cast(a as T)



https://github.com/CL251KA/my-note.git