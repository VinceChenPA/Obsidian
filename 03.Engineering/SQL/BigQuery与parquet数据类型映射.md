数据类型映射

|---|---|---|---|
|INT64|INT64|64 位有符号整数（物理类型：INT64）|12345|
|FLOAT / FLOAT64|DOUBLE|64 位双精度浮点数（物理类型：DOUBLE，IEEE 754 标准）|3.1415|
|NUMERIC|DECIMAL（逻辑类型）|高精度十进制数，存储为 INT32/INT64 等物理类型，元数据记录精度（precision/scale）|NUMERIC(10, 2) → DECIMAL(10,2)|
|STRING|STRING（逻辑类型）|物理类型为 BYTE_ARRAY，逻辑类型为 UTF8（支持 Unicode 字符串）|"hello world"|
|BOOLEAN|BOOLEAN|1 字节存储（0 表示 FALSE，1 表示 TRUE）|TRUE/FALSE|
|TIMESTAMP|TIMESTAMP(MICROS)|存储为 INT64 类型（从 Unix 纪元开始的微秒数）|2023-10-01 12:00:00 → 1696161600000000|
|DATE|INT32（逻辑类型 DATE）|存储为 INT32（从 Unix 纪元开始的天数，如 2023-10-01 → 738274）|DATE '2023-10-01'|
|TIME|INT32（逻辑类型 TIME(MICROS)）|存储为微秒数（如 12:34:56.789 → 45296789000 微秒）|TIME '12:34:56.789'|
|BYTES|BINARY|二进制数据，物理类型为 BYTE_ARRAY，逻辑类型为 BINARY|BYTES 'aGVsbG8='（Base64 编码）|

