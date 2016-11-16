##基于cas server 4.2.5配置
###下载最新版源码包解压，用gradle编译，命令为：
```
gradle clean build -DskipAspectJ=true -x javadoc -x test
```
将build文件夹里的war包部署到服务器

##JDBC配置
eclipse打开war包，打开webapp中的``C:\Users\evia05\Desktop\cas-4.2.5\cas-server-webapp\src\main\webapp\WEB-INF``文件夹下的`deployerConfigContext.xml`
推荐使用`c3p0`lib
```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		p:driverClass="${database.driverClass}" p:jdbcUrl="${database.url}"
		p:user="${database.user}" p:password="${database.password}"
		p:initialPoolSize="${database.pool.minSize}" p:minPoolSize="${database.pool.minSize}"
		p:maxPoolSize="${database.pool.maxSize}"
		p:maxIdleTimeExcessConnections="${database.pool.maxIdleTime}"
		p:checkoutTimeout="${database.pool.maxWait}" p:acquireIncrement="${database.pool.acquireIncrement}"
		p:acquireRetryAttempts="${database.pool.acquireRetryAttempts}"
		p:acquireRetryDelay="${database.pool.acquireRetryDelay}"
		p:idleConnectionTestPeriod="${database.pool.idleConnectionTestPeriod}"
		p:preferredTestQuery="${database.pool.connectionHealthQuery}" />
```
注释掉原本的验证方式
```xml
<alias name="acceptUsersAuthenticationHandler" alias="primaryAuthenticationHandler" />
```
然后在`cas.properties`中加入配置信息
```bash
database.driverClass=org.postgresql.Driver
database.url=jdbc:postgresql://127.0.0.1:5432/juzimap
database.user=postgres
database.password=123456
database.pool.minSize=6
database.pool.maxSize=18

# Maximum amount of time to wait in ms for a connection to become
# available when the pool is exhausted
database.pool.maxWait=10000

# Amount of time in seconds after which idle connections
# in excess of minimum size are pruned.
database.pool.maxIdleTime=120

# Number of connections to obtain on pool exhaustion condition.
# The maximum pool size is always respected when acquiring
# new connections.
database.pool.acquireIncrement=6

# == Connection testing settings ==

# Period in s at which a health query will be issued on idle
# connections to determine connection liveliness.
database.pool.idleConnectionTestPeriod=30

# Query executed periodically to test health
database.pool.connectionHealthQuery=select 1

# == Database recovery settings ==

# Number of times to retry acquiring a _new_ connection
# when an error is encountered during acquisition.
database.pool.acquireRetryAttempts=5

# Amount of time in ms to wait between successive aquire retry attempts.
database.pool.acquireRetryDelay=2000
# cas.jdbc.authn.query.sql=
cas.jdbc.authn.query.sql=select pass from t_user where mobile=?
```
这种方式可以对密码进行加密后与数据库中存储的进行比较
```xml
<alias name="queryAndEncodeDatabaseAuthenticationHandler" alias="primaryAuthenticationHandler" /><alias name="dataSource" alias="queryEncodeDatabaseDataSource" />
```
注意：盐是必须的，盐值在密码前面，生成的密码中的字母是小写的
```
cas.jdbc.authn.query.encode.sql=select password,salt from t_user where mobile=?
cas.jdbc.authn.query.encode.alg=MD5
#cas.jdbc.authn.query.encode.salt.static=
#cas.jdbc.authn.query.encode.password=
cas.jdbc.authn.query.encode.salt=salt
#cas.jdbc.authn.query.encode.iterations.field=1
#cas.jdbc.authn.query.encode.iterations=0
```

```xml
<bean id="attributeRepository" class="org.jasig.services.persondir.support.jdbc.SingleRowJdbcPersonAttributeDao">
    <constructor-arg index="0" ref="dataSource" />
    <constructor-arg index="1" value="SELECT * FROM t_user WHERE {0}" />
    <property name="queryAttributeMapping">
        <map>
            <entry key="mobile" value="mobile" />
        </map>
    </property>
    <property name="resultAttributeMapping">
        <map>
            <entry key="mobile" value="mobile" />
            <entry key="nickname" value="nickname" />
            <entry key="realname" value="realname" />
            <entry key="email" value="email" />
            <entry key="level" value="level" />
            <entry key="roleid" value="roleid" />
        </map>
    </property>
</bean>
```
