###### GC 参数

| 参数                           | 说明                   |
| ---------------------------- | -------------------- |
| -XX:+PrintGCDetails          | GC 时打印日志             |
| -Xms                         | 堆 初始内存               |
| -Xmx                         | 堆 最大内存               |
| -XX:NewRatio                 | 新生代 : 老年代 比例         |
| -Xmn / -XX:NewSize           | 新生代 内存（后面的覆盖前面的）     |
| -XX:MaxNewSize               | 新生代 最大内存             |
| -XX:SurvivorRatio            | 新生代 Eden:Survivor    |
| -XX:PretenureSizeThreshold   | 判定为 大对象 的大小阈值        |
| -XX:InitialTenuringThreshold | 初始 晋升老年代 年龄阈值        |
| -XX:MaxTenuringThreshold     | 最大 晋升老年代 年龄阈值，默认为 15 |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
|                              |                      |
