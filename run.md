### 部署 MAAS 堆栈

```
ansible-playbook -i ./hosts\
    --extra-vars="maas_version=3.4 maas_postgres_password=example maas_installation_type=snap maas_url=http://100.110.100.1:5240/MAAS"\
    ./site.yaml
```

### 启用可观察性功能部署 MAAS 堆栈

```
ansible-playbook -i ./hosts \
    --extra-vars="maas_version=3.4 \
        maas_postgres_password=example \
        maas_installation_type=snap \
        maas_url=http://192.168.1.29:5240/MAAS \
        o11y_enable=true \
        o11y_prometheus_url=http://prometheus-server:9090/api/v1/write \
        o11y_loki_url=http://loki-server:3100/loki/api/v1/push" \
    ./site.yaml
```

### 拆除 MAAS 堆栈

```
ansible-playbook -i ./hosts ./teardown.yaml
```

### 备份 MAAS 堆栈

备份 MAAS 需要设置 `maas_backup_download_path` 变量，它将指定一个本地路径，用于下载备份档案。此路径必须在运行剧本之前存在。`maas_installation_type` 也是必需的。

```
ansible-playbook -i ./hosts --extra-vars="maas_backup_download_path=/tmp/maas_backups/ maas_installation_type=deb" ./backup.yaml
```

### 从备份中恢复

由于备份是按主机进行的，建议将 `maas_backup_file` 变量设置为主机变量，或将执行过滤到特定主机，但 `maas_backup_file` 应设置为备份操作中的 gzipped tar 文件，并且 `maas_installation_type` 也应设置。

下面的示例假设 `maas_backup_file` 在 `./hosts` 中设置

```
ansible-playbook -i ./hosts --extra-vars="maas_installation_type=deb" ./restore.yaml 
```

### 创建新管理员用户

对于已经存在的 MAAS 安装，可以创建一个新的管理员用户。下面的示例假设要为其创建帐户的主机在 `./hosts` 中的 `[maas_region_controller]` 下设置。`user_ssh` 允许上传 Launchpad (`lp:id`) 或 GitHub (`gh:id`) 公钥，或者可以留空 (`user_ssh=`)。 

```
ansible-playbook -i ./hosts --extra-vars="user_name=newuser user_pwd=newpwd user_email=user@email.com user_ssh=lp:id" ./createadmin.yaml
```

### 导出可观察性警报规则

MAAS 为 Prometheus 和 Loki 提供了一组精心策划的警报规则。您可以使用以下命令导出这些规则，其中 `o11y_alertrules_dest` 是文件应放置的目录。

```
ansible-playbook --extra-vars="o11y_alertrules_dest=/tmp" ./alertrules.yaml
``` 