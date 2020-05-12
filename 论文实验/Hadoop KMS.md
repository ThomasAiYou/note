# Hadoop KMS

1.修改core-site.xml:

```html
<property>
		<name>hadoop.security.key.provider.path</name>
		<value>kms://http@localhost:16000/kms</value>
</property>
```

2.修改hdfs-site.xml:

```html
<property>
		<name>dfs.encryption.key.provider.uri</name>
		<value>kms://http@localhost:16000/kms</value>
</property>
```

3.生成key开启kms服务

4.比较加密和非加密区内容，确认加密情况

```shell
hdfs dfs -ls /.reserved/raw/user/root/output/part-r-00000
```

5.指定加密区

```shell
-Dtest.build.data=/kms
```

