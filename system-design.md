# GPU 集群自动化部署方案

本文档描述了使用 Ansible 脚本自动化部署和管理 GPU 计算集群的方案。

## 集群架构

整个集群由以下三个部分组成:

- GPU 计算集群: 提供 GPU 计算资源
- 存储服务器集群: 提供数据存储服务
- 管理服务器集群: 提供集群管理和运维服务

### 管理服务器集群组件

管理服务器集群包含以下核心服务:

1. 基础环境的准备 (https://github.com/aiken/gpu-infra)
   1. 本地 ansible 环境准备，考虑使用 vagrant 初始化本地环境
   2. 使用 ansible 部署 ring0 设备的系统环境
      1. ubuntu 替换国内源
      2. docker 安装和替换 registry

2. **MAAS (Metal as a Service)**
   - 提供裸机服务器的自动化部署
   - 管理操作系统镜像和网络启动

3. **Kubernetes for ring0 service**
   - 容器编排和管理
   - GPU 资源调度

4. **Ansible & AWX**
   - Ansible: 自动化配置管理工具
   - AWX: Ansible 的 Web UI 管理界面

5. **JumpServer**
   - 堡垒机系统
   - 统一管理服务器访问权限
   - 审计系统操作记录

6. **夜莺监控系统**
   - 集群监控和告警
   - 性能指标收集和展示






## 配置管理分层

整个集群采用分层管理模式:

### Ring 0: 基础设施层
- 由初始的运维工程师的ubuntu环境中的ansible负责
- 负责部署基础运维环境
- 包含所有核心管理组件的安装和配置:
  - MAAS
  - Ansible & AWX
  - JumpServer
  - 夜莺监控
  - Kubernetes

### Ring 1: 操作系统层
- 由 MAAS 负责
- 自动化操作系统安装
- SSH 访问配置
- 网络配置

### Ring 2: 应用软件层
- 由 Ansible & AWX 负责
- 安装和配置:
  - NVIDIA 驱动
  - CUDA 库
  - Docker
  - Kubernetes 集群
  - 监控组件

## Ansible Playbook 结构

每个 Ring 层都有对应的 Ansible playbook 脚本，用于自动化完成该层的配置任务。具体实现请参考各个 Ring 目录下的 playbook 文件。
