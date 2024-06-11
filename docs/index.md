
# 数擎全量预置计算巢服务说明
## 概述
天域数擎是以友盟全域数据为基础，提供在线营销服务增强型产品，包括营销标签模型预测分、模型特征定制服务、实时营销增强API，助力企业业务增长。<br />全域标签增补：预置丰富的全域标签，覆盖基础属性、社会属性、地域分布、兴趣偏好等300+维度，在一方特征不足的情况下，友盟三方数据为企业存量用户补充全域特征，进行人群特征与偏好识别，助力企业用户画像洞察，精细化运营手段，充分满足业务需要。<br />本文介绍基于计算巢，部署阿里云私域的数擎全量预置数据服务，该服务将提供全域设备（idfa、oaid、手机号）的特征矩阵或兴趣分标签数据的查询功能。

**使用计算巢服务部署简介和收益**：
传统软件部署方式，需要服务商/软件供应商提供资源配置单，交由用户（或交付伙伴）手动创建资源、部署服务、配置服务，过程大量人工介入，依赖经验，比如涉及docker、postgres数据库等资源的创建和配置，应用软件的安装配置调试等。
计算巢支持标准化的应用交付方式，服务商将应用发布为标准服务，在用户订阅服务后计算巢自动触发部署流程，部署期间，无需人工参与，解决传统模式下的交付和部署难题，且除了ECS等资源费用外，无需额外付费。

## 实例规格
根据您购买的数据类型选择对应的 ECS 和数据盘配置，此外还需要您提供和 ECS 同 region 的 OSS bucket，用于中转传输数据到 ECS 本地服务数据库。

### 全量预置特征矩阵+ 兴趣分（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ） ¥2673.49/月<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 100GB
- 数据盘：15T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps，建议 1000Mbps<br />数据同步时长：10-48h（视网络情况）

### 全量预置特征矩阵（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ）<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 100GB
- 数据盘：10T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps<br />数据同步时长：10-36h（视网络情况）

### 全量预置兴趣分（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ）<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 50GB  ¥50/月
- 数据盘：5.5T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps<br />数据同步时长：10-24h（视网络情况）

## 部署流程

### 一、准备工作
您需要拥有一个阿里云账号，对 ECS、VPC 、VSWITCH、OSS 、RAM 等资源进行访问和操作。<br />若您使用主账号，可直接创建服务实例。若您使用RAM用户创建服务实例，且是第一次使用阿里云计算巢，需要在创建服务实例前，对使用的RAM用户的账号添加相应资源的权限。更多信息请参见 [为RAM用户授权](https://help.aliyun.com/zh/ram/user-guide/grant-permissions-to-the-ram-user)。<br />RAM 账号权限

| 权限策略名称 | 备注 |
| --- | --- |
| AliyunVPCFullAccess | 管理专有网络（VPC）的权限 |
| AliyunECSFullAccess | 管理云服务器服务（ECS）的权限 |
| AliyunROSFullAccess | 管理资源编排服务（ROS）的权限 |
| AliyunComputeNestUserFullAccess | 管理计算巢服务（ComputeNest）的用户侧权限 |
| AliyunComputeNestFullAccess | 管理计算巢服务（ComputeNest）的权限 |
| AliyunComputeNestSupplierFullAccess | 管理计算巢服务（ComputeNest）的权限 |

部署架构<br />部署环节涉及到的云资源： ECS、数据盘、OSS、VPC、交换机、安全组及相应规则<br />![1.jpg](images%2F1.jpg)

OSS资源：通过线下方式提供和 ECS 同 region 的 OSS bucket（bucket name、endpoint、ak、sk）给到数擎工程同学。

### 二、获取部署链接

方式一：
通过商务渠道获取：如下部署链接 [部署链接](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-eaf86b94527145059d27) 快速体验。<br />如果链接显示没有权限，点击诊断查看所需添加的权限策略。

方式二：
访问阿里云计算巢产品控制台，服务目录中通过"数擎全量预置"关键字进行搜索：
![15.jpg](images%2F15.jpg)

### 三、创建服务实例
进入部署链接后，开始您阿里云账号（UID）下的数擎计算巢服务所需资源配置：<br />服务实例命名、地域选择、付费类型（ECS的付费类型），可选按量付费或包年包月。<br />![2.webp](images%2F2.webp)

ECS 配置（CU、内存选型，建议ecs.r7.4xlarge 规格， 配置不低于该配置），实例密码为 ECS root 账号登录密码，密码勿丢失。可用区 、VPC、交换机等根据实际情况，选择您已存在的实例。<br />![3.webp](images%2F3.webp)


![4.webp](images%2F4.webp)



![5.webp](images%2F5.webp) <br />点击 立即创建 提交

![6.webp](images%2F6.webp)


### 四、查看部署状态
![7.webp](images%2F7.webp)<br />点击实例 id 查看

![8.webp](images%2F8.webp)


点击“资源” 查看，云资源中，资源类型为实例的即本次部署服务的 ECS 实例<br />![9.webp](images%2F9.webp)

点击 ECS 实例 id 进入，修改安全组策略：入口方向支持 22 端口（ssh 登录）；公网带宽修改为 100Mbps，按量付费<br />![10.webp](images%2F10.webp)

![11.webp](images%2F11.webp)


远程连接 ECS 实例<br />![12.webp](images%2F12.webp)

密码为创建服务实例时设置的。

进入服务器，查看配置，按接口说明文档，验证接口连通性和数据结果。<br />![13.webp](images%2F13.webp)


## 计费说明
按查询 id 去重计费。
