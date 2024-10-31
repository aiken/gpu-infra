# MAAS-ansible-playbook

一个用于安装和配置 MAAS 的 Ansible 剧本，更多文档请参考[这里](https://maas.io/docs/ansible-playbooks-reference)。

## 版本

此剧本已在 Ansible 版本 5.10.0 及以上版本中测试。我们建议使用最新的稳定版本的 Ansible（目前为 7.x）。需要在使用 Ansible 的机器上安装 `netaddr` Python 库；注意，这在远程主机上不是必需的。

## 安装

```
git clone git@github.com:canonical/maas-ansible-playbook
```

## 设置

此剧本有几个主要角色，每个角色都有一个相应的组来分配主机。它们如下：

- **maas_pacemaker**: 该角色和组用于配置为 Pacemaker 集群以管理 HA Postgres 的一组主机。建议 `maas_corosync` 是该组的子组。也建议这些主机与 `maas_postgres` 组相同。此角色是可选的，但对于 HA Postgres 是必需的。

- **maas_corosync**: 该角色和组用于配置为运行 Corosync 集群的一组主机，用于管理 HA Postgres 集群的仲裁。此角色是可选的，但对于 HA Postgres 是必需的。

- **maas_postgres**: 该角色和组用于配置为 MAAS 堆栈的 Postgres 实例的主机。如果为此角色分配了多个主机，Postgres 将配置为 HA，这将使 `maas_corosync` 和 `maas_pacemaker` 角色成为必需。此角色是必需的。

- **maas_region_controller**: 该角色和组用于配置为 MAAS 区域控制器的主机。此角色至少需要一次。

- **maas_rack_controller**: 该角色和组用于配置为 MAAS 机架控制器的主机。此角色至少需要一次。

- **maas_postgres_proxy**: 该角色和组用于配置为 HAProxy 实例的主机，以确保查询被定向到主实例。我们建议将此组分配给与 `maas_region_controller` 相同的主机，这将使 HAProxy 的监听器绑定到本地主机并为该特定区域控制器路由查询。此角色是可选的，但推荐用于 HA Postgres。

- **maas_proxy**: 该角色和组用于配置为 HAProxy 实例的主机，以便在区域控制器前面进行 HA 堆栈。此角色是可选的。

### 主机文件

可以使用两种格式定义清单（`ini` 和 `yaml`）。下面是相同的清单，以两种格式定义。运行剧本时只能选择一种。

更多信息可以在 [Ansible 清单文档页面](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)找到。

#### ini

```ini
[maas_corosync]
db01.example.com
db02.example.com
db03.example.com

[maas_pacemaker:children]
maas_corosync

[maas_postgres]
db01.example.com
db02.example.com
db03.example.com

[maas_postgres_proxy]
region01.example.com
region02.example.com
region03.example.com

[maas_region_controller]
region01.example.com
region02.example.com
region03.example.com

[maas_rack_controller]
region01.example.com
rack01.example.com
rack02.example.com

[maas_proxy]
proxy01.example.com
```

#### yaml

```yaml
---
all:
  children:
    maas_pacemaker:
      children:
        maas_corosync:
          hosts:
            db01.example.com
            db02.example.com
            db03.example.com
    maas_postgres:
      hosts:
        db01.example.com:
        db02.example.com:
        db03.example.com:
    maas_proxy:
      hosts:
        proxy01.example.com:
    maas_postgres_proxy:
      hosts:
        region01.example.com:
    maas_region_controller:
      hosts:
        region01.example.com:
        region02.example.com:
        region03.example.com:
    maas_rack_controller:
      hosts:
        region01.example.com:
        rack01.example.com:
        rack02.example.com:
```

注意：Pacemaker 角色需要在主机文件中定义特定于主机的变量，如下所示：

- `maas_pacemaker_fencing_driver`: Pacemaker STONITH 隔离驱动程序，默认为 `ipmilan`。此驱动程序用于在成员表现出错误行为时强制将其从 Pacemaker 集群中移除。Pacemaker 将列出可用的驱动程序，使用以下命令：`stonith_admin --list-installed`

- `maas_pacemaker_stonith_params`: 选定的隔离驱动程序的特定参数。使用 Pacemaker 时必须定义这些参数，要查看给定驱动程序的参数，请运行 `stonith_admin --metadata --agent <driver>`

## 运行

此剧本要求用户设置以下变量：

- **maas_version**: 要安装的 MAAS 版本。\
  示例：`'3.2'`

- **maas_postgres_password**: MAAS postgres 用户的密码。\
  示例：`'my_password'`

- **maas_installation_type**: MAAS 安装的类型。\
  可能的值为：`'deb'` 或 `'snap'`

- **maas_url**: MAAS 的 URL，MAAS 将在此处访问，机架控制器应使用此 URL 注册区域控制器。\
  示例：`'http://proxy01.example.com:5240/MAAS'`

还有其他可选变量可以传递给剧本：

- ### 管理员凭据（如果设置管理员用户）

  - **admin_username**: 管理员用户的用户名\
    默认：`admin`

  - **admin_password**: 管理员用户的密码\
    默认：`admin`

  - **admin_email**: 管理员用户的电子邮件地址\
    默认：`admin@email.com`

  - **admin_id**: 要导入 SSH 密钥的管理员用户的 Launchpad 或 GitHub ID。
    （可选）

- **maas_proxy_postgres_proxy_enabled**: 使用 postgres 代理 URI\
  默认：`false`

- **enable_tls**: MAAS 是否应启用 TLS\
  默认：`false`（仅适用于 MAAS >= 3.2）

- **enable_firewall**: MAAS 是否应配置防火墙\
  默认：`true`

- ### MAAS Vault（仅适用于 MAAS >= 3.3）

  - **vault_integration**: MAAS 是否应使用 Vault 进行秘密存储\
    默认：`false`
  - **vault_url**: MAAS Vault 的 URL\
    默认：未定义
  - **vault_approle_id**: MAAS Vault 的 approle ID\
    默认：未定义
  - **vault_wrapped_token**: MAAS Vault 的包装令牌\
    默认：未定义
  - **vault_secrets_path**: MAAS Vault 的秘密路径\
    默认：未定义
  - **vault_secret_mount**: MAAS Vault 的秘密挂载\
    默认：未定义

- **http_proxy**: 要使用的 HTTP 代理\
  默认：不使用代理

- **https_proxy**: 要使用的 HTTPS 代理\
  默认：不使用代理

- ### 可观察性

  - **o11y_enable**: 是否应启用可观察性功能\
    默认：`false`
  - **o11y_prometheus_url**: Prometheus [远程写入接收器](https://prometheus.io/docs/prometheus/latest/feature_flags/#remote-write-receiver) 端点 URL\
    默认：未定义
  - **o11y_loki_url**: Loki 端点 URL\
    默认：未定义

启用可观察性功能时，至少必须定义一个端点。

使用 HA Postgres 时仅需要以下变量：

- **maas_postgres_floating_ip**: HA Postgres 集群内部用于复制的浮动 IP。应在您的网络中为此 IP 进行静态保留以避免重叠。

- **maas_postgres_floating_ip_prefix_len**: 您的网络中浮动 IP 所在子网的前缀长度。

### 部署 MAAS 堆栈

```
ansible-playbook -i ./hosts\
    --extra-vars="maas_version=3.2 maas_postgres_password=example maas_installation_type=deb maas_url=http://example.com:5240/MAAS"\
    ./site.yaml
```

### 启用可观察性功能部署 MAAS 堆栈

```
ansible-playbook -i ./hosts \
    --extra-vars="maas_version=3.3 \
        maas_postgres_password=example \
        maas_installation_type=snap \
        maas_url=http://example.com:5240/MAAS \
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