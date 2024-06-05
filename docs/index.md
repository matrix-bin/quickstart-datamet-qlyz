<a name="igpRB"></a>
# 概述
天域数擎是以友盟全域数据为基础，提供在线营销服务增强型产品，包括营销标签模型预测分、模型特征定制服务、实时营销增强API，助力企业业务增长。<br />全域标签增补：预置丰富的全域标签，覆盖基础属性、社会属性、地域分布、兴趣偏好等300+维度，在一方特征不足的情况下，友盟三方数据为企业存量用户补充全域特征，进行人群特征与偏好识别，助力企业用户画像洞察，精细化运营手段，充分满足业务需要。<br />本文介绍基于计算巢，部署阿里云私域的数擎全量预置数据服务，该服务将提供全域设备（idfa、oaid、手机号）的特征矩阵或兴趣分标签数据的查询功能。

<a name="YkTGM"></a>
# 实例规格
根据您购买的数据类型选择对应的 ECS 和数据盘配置，此外还需要您提供和 ECS 同 region 的 OSS bucket，用于中转传输数据到 ECS 本地服务数据库。
<a name="ZuMvP"></a>
## 全量预置特征矩阵+ 兴趣分（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ） ¥2673.49/月<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 100GB
- 数据盘：15T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps，建议 1000Mbps<br />数据同步时长：10-48h（视网络情况）
<a name="Ny2sX"></a>
## 全量预置特征矩阵（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ）<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 100GB
- 数据盘：10T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps<br />数据同步时长：10-36h（视网络情况）
<a name="QkA5l"></a>
## 全量预置兴趣分（数据压缩方案）
实例规格：ecs.r7.4xlarge（16 vCPU 128 GiB，内存型 r7 ）<br />操作系统：Anolis OS 8.4 RHCK 64位<br />存储：

- 系统盘：ESSD云盘 50GB  ¥50/月
- 数据盘：5.5T（1 个日期分区快照）¥1000/T/月

网络：>=100Mbps<br />数据同步时长：10-24h（视网络情况）
<a name="i3XGN"></a>
# 部署流程
<a name="n2vOR"></a>
## 一、准备工作
您需要拥有一个阿里云账号，对 ECS、VPC 、VSWITCH、OSS 、RAM 等资源进行访问和操作。<br />若您使用主账号，可直接创建服务实例。若您使用RAM用户创建服务实例，且是第一次使用阿里云计算巢，需要在创建服务实例前，对使用的RAM用户的账号添加相应资源的权限。更多信息请参见 [为RAM用户授权](https://help.aliyun.com/zh/ram/user-guide/grant-permissions-to-the-ram-user)。<br />RAM 账号权限

| 权限策略名称 | 备注 |
| --- | --- |
| AliyunVPCFullAccess | 管理专有网络（VPC）的权限 |
| AliyunECSFullAccess | 管理云服务器服务（ECS）的权限 |
| AliyunROSFullAccess | 管理资源编排服务（ROS）的权限 |
| AliyunComputeNestUserFullAccess | 管理计算巢服务（ComputeNest）的用户侧权限 |


资源准备<br />除上一步实例规格中所需的 ECS、数据盘、网络等配置，云资源依赖交换机、安全组及相应规则<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717576171628-982d4e76-4739-4652-bf20-6a793d6e0e49.png#clientId=uc224d7ca-cb87-4&from=paste&height=564&id=uf48e4edf&originHeight=1128&originWidth=2396&originalType=binary&ratio=2&rotation=0&showTitle=false&size=423205&status=done&style=none&taskId=u4d1436af-05e7-4741-8015-21da1f92815&title=&width=1198)
<a name="Vm54J"></a>
## 二、获取部署链接
您可以在阿里云的计算巢中通过MAPPIC关键字进行搜索，也可以单击 [部署链接](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-eaf86b94527145059d27) 快速体验。<br />如果链接显示没有权限，点击诊断查看所需添加的权限策略。

<a name="usyBf"></a>
## 三、创建服务实例
进入部署链接后，开始您阿里云账号（UID）下的数擎计算巢服务所需资源配置：<br />服务实例命名、地域选择、付费类型选按量付费<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717568393152-df536a3b-620f-4f72-8a6e-f2b2175c6843.png#clientId=uadd3dfba-b26f-4&from=paste&height=946&id=u3020fb0f&originHeight=1892&originWidth=3456&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2018350&status=done&style=none&taskId=ua0b02497-e022-43e6-a633-26ee2ac17a7&title=&width=1728)

ECS 配置（CU、内存选型，建议ecs.r7.4xlarge 规格， 配置不低于该配置），实例密码为 ECS root 账号登录密码，密码勿丢失。可用区 、VPC、交换机等根据实际情况，选择您已存在的实例。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717576674210-1d326005-b789-4a8c-a069-81872f7182ea.png#clientId=ucf8b1235-59b7-4&from=paste&height=936&id=u74f12c86&originHeight=1872&originWidth=3444&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2136295&status=done&style=none&taskId=uc75a6252-6907-4415-9059-c0721c1b107&title=&width=1722)


![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577133734-2e3f352b-abca-41ac-8199-73fa1a4dd99e.png#clientId=ucf8b1235-59b7-4&from=paste&height=931&id=u49713e9e&originHeight=1862&originWidth=3456&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2086974&status=done&style=none&taskId=ub8eca01a-8408-4d0c-92fa-c7f9c3323db&title=&width=1728)



![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577186378-0ebd3dfd-c8cb-4842-badc-a529b431d80c.png#clientId=ucf8b1235-59b7-4&from=paste&height=822&id=u81bfec50&originHeight=1644&originWidth=3416&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1660471&status=done&style=none&taskId=u69a80ce9-4f38-4586-9a6e-62589cdf314&title=&width=1708)<br />点击 立即创建 提交

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577372765-ef468a5d-35be-4bff-97b6-662a2b3ef380.png#clientId=ucf8b1235-59b7-4&from=paste&height=473&id=ufcd11c59&originHeight=946&originWidth=2440&originalType=binary&ratio=2&rotation=0&showTitle=false&size=620071&status=done&style=none&taskId=u3e0a2040-7337-446d-b6c1-cf0f01a8976&title=&width=1220)

<a name="QTMrE"></a>
## 四、查看部署状态
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577419586-e5a44635-08b0-4449-8a4d-9ca4e71a228e.png#clientId=ucf8b1235-59b7-4&from=paste&height=470&id=ua5fa9422&originHeight=940&originWidth=3452&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1063557&status=done&style=none&taskId=ubf4adec3-3cd1-4944-b6bc-55130969241&title=&width=1726)<br />点击实例 id 查看

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577450560-40791e2f-70f9-45ef-9a04-ea26296d8318.png#clientId=ucf8b1235-59b7-4&from=paste&height=857&id=ud412c05a&originHeight=1714&originWidth=3426&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1827581&status=done&style=none&taskId=uc54961d6-2131-4f83-9752-dc20ed8d421&title=&width=1713)


点击“资源” 查看，云资源中，资源类型为实例的即本次部署服务的 ECS 实例<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577478437-63662d1e-7091-49e6-ae72-4bec9c82fbc6.png#clientId=ucf8b1235-59b7-4&from=paste&height=673&id=u2acecde4&originHeight=1346&originWidth=3446&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1492318&status=done&style=none&taskId=u34178de5-19ab-48b2-89b5-b7dac601b7e&title=&width=1723)

点击 ECS 实例 id 进入，修改安全组策略：入口方向支持 22 端口（ssh 登录）；公网带宽修改为 100Mbps，按量付费<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577610906-de984875-ca99-4faa-bb66-ba495a1b929a.png#clientId=ucf8b1235-59b7-4&from=paste&height=727&id=u7243f67f&originHeight=1454&originWidth=3430&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1561674&status=done&style=none&taskId=ua38fb038-a2b1-4bf0-84c3-3a2aa054cf2&title=&width=1715)

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577647551-f15cbe08-0e6f-4b01-9dac-2070d37561c9.png#clientId=ucf8b1235-59b7-4&from=paste&height=929&id=ub7534f29&originHeight=1858&originWidth=3448&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2785281&status=done&style=none&taskId=u197ead78-ee33-40b5-bdf8-759c7ef50a6&title=&width=1724)


远程连接 ECS 实例<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577688278-dd74986c-2ab1-4de9-b164-03011baa9d6d.png#clientId=ucf8b1235-59b7-4&from=paste&height=948&id=u489d6da0&originHeight=1896&originWidth=3454&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2477414&status=done&style=none&taskId=ube04ae33-1d20-4545-b914-ca5a2b9b828&title=&width=1727)

密码为创建服务实例时设置的。

进入服务器，查看配置，按接口说明文档，验证接口连通性和数据结果。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/200052/1717577895999-376b7374-5f70-4925-85a1-997516b61227.png#clientId=ucf8b1235-59b7-4&from=paste&height=856&id=uace42cf6&originHeight=1712&originWidth=3442&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2635896&status=done&style=none&taskId=u0f8fc90a-33e1-47cf-aec5-5e6985f5c94&title=&width=1721)

<a name="PP65N"></a>
# 计费说明
按查询 id 去重计费。
