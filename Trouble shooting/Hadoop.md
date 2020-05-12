# spark trouble shooting

## 1.Map Reduce卡住

资源分配不足，修改core-site.xml

```xml
<property>
		<name>yarn.nodemanager.resource.memory-mb</name>
		<value>3072</value>
</property>
<property>
		<name>yarn.nodemanager.resource.cpu-vcores</name>
		<value>2</value>
</property>
<property>
		<name>yarn.scheduler.minimum-allocation-mb</name>
		<value>256</value>
</property>
<property>
    <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
    <value>95.0</value>
</property>
```

