---
Bug 记录包
---

----

[toc]



# Elasticsearch

#### [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```shell
sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
sysctl -w vm.max_map_count=262144
```

