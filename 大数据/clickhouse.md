优化写入速度 1.5MB/s---->15MB/s

- 使用clickhouse--jdbc,,注意使用原生clickhouse-connection,不要使用java-jdbc
- 使用http协议即可，不必为了追求性能使用tcp协议，tcp难以做负载代理
- 通过实现jdbc的一些借口，配合flink ck 2pc实现顺序写入原子性