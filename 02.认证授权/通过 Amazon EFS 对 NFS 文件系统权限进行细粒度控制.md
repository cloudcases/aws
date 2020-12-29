# 通过 Amazon EFS 对 NFS 文件系统权限进行细粒度控制

** 2020 年 3 月 01 日

**[文化 & 方法](https://www.infoq.cn/topic/culture-methods)[新基建](https://www.infoq.cn/topic/new_infrastructure)[AWS](https://www.infoq.cn/topic/AWS)



传统的 NFS 服务是企业应用里非常常见的网络存储服务。用户在搭建 NFS 服务时，需要对权限进行相应的设置，以防止未授权的客户端非法访问远程文件存储。NFS 的权限管理主要依赖 Linux 文件系统的文件权限管理机制，并通过 /etc/exports 进行文件系统共享的参数设置，如授权客户端的网段，是否只读等。如果需要更进一步的认证机制，需要部署 Kerberos，相应的配置也并不轻松。

当我们将现有的 NFS 服务迁移上云，或是需要在云的环境下面构建 NFS 服务时，同样也要考虑到权限的控制。 AWS 上提供了 Amazon Elastic File System (EFS)，是托管的 NFS 服务。用户可以基于 EFS 快速的部署一个超过 PB 级别的 NFS 文件系统。用户不需要管理文件系统的底层存储机制，只要往 EFS 写入文件或删除文件，EFS 即可自动进行扩展和收缩。同时这些数据也是跨多个可用区来进行保护，相当于实现了同城灾备的保护级别。

在权限控制这块，EFS 在 NFS 文件权限控制的基础上，结合 AWS 云上已有的安全认证机制，如安全组(Security Group)，身份与访问管理服务(IAM)和 EFS 独特的接入点(Access Point)机制，简化了权限控制的配置工作，同时可以更为细粒度的进行管理和审计。

## 1. 通过安全组进行客户端访问控制

安全组可以理解为是 AWS VPC 中挂载到虚拟网卡上的防火墙，通过设定规则可以指定入站和出站的白名单。在之前的[博客](https://amazonaws-china.com/cn/blogs/china/quickly-build-nfs-file-system-with-efs/)中，我们演示了如何通过向导快速部署一个 EFS 文件系统，其中挂载目标和 EC2 实例均使用的是 VPC 的默认安全组，因此二者之前的所有网络流量都是放通的。接下来我们可以创建自定义的安全组来做更为精细的权限控制。

如下图所示：

[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs1.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs1.jpg)

我们可以为挂载目标和 EC2 实例各创建一个安全组（分别命名为 efsmountpoint 和 nfsclient，如果 EC2 实例已有安全组则可利用现有安全组)。EC2 实例的安全组可按照业务需求开通相应的入站端口即可，出站默认是所有流量放通。挂载目标的安全组入站规则设置为放通 NFS 端口 2049,来源可以设置为 EC2 实例的安全组 id（即 nfsclient），这样即可以通过安全组限制只有指定的 EC2 实例能够挂载文件系统。

如下是控制台上关于挂载目标安全组的设置截图：

[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs2.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs2.jpg)

注意到每个挂载目标最多可以同时设置 5 个安全组。

## 2. 通过 IAM 对 NFS 客户端进行授权

安全组可以对访问文件系统的 EC2 实例进行访问控制，如果需要进一步控制访问权限，如读写权限，能否以 Root 用户访问等，则可以结合 IAM 服务来进行更细粒度的控制。

通过 AWS Identity and Access Management (IAM) 可以安全地管理对 AWS 服务和资源的访问。可以使用 IAM 创建和管理 AWS 用户和组，并使用各种权限来允许或拒绝他们对 AWS 资源的访问。同时，EFS 的文件系统也提供了文件系统策略，可以设置针对文件系统的相应权限。

[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs3.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs3.jpg)

### 2.1 为文件系统设置只读策略

EFS 文件系统创建出来后，默认的文件系统策略是全放通，即任何 NFS 客户端均可以对其完全访问。用户可以通过设置文件系统策略，来进一步对文件系统权限进行控制。这些权限包括：

- elasticfilesystem:ClientMount : 允许客户端以只读方式挂载文件系统
- elasticfilesystem:ClientWrite : 允许客户端对文件系统进行写入
- elasticfilesystem:ClientRootAccess : 允许客户端以Root用户进行访问



比如我们希望文件系统给予所有用户只读权限，则可以配置如下的文件系统策略：

Json

```
{    "Version": "2012-10-17",    "Id": "efs-policy-wizard-f37947df-cdbb-40c9-8fe2-fb38bedd362e",    "Statement": [        {            "Sid": "efs-statement-204c3f1d-6499-4a4f-9fa5-86592173888e",            "Effect": "Allow",            "Principal": {                "AWS": "*"            },            "Action": "elasticfilesystem:ClientMount",            "Resource": "arn:aws-cn:elasticfilesystem:cn-northwest-1:402202783068:file-system/fs-57789eb2"        }    ]}
```



接下来我们可以挂载这个文件系统进行读写测试：

```
$ sudo mount -t efs fs-57789eb2 /mnt/efs$ touch /mnt/efs/testtouch: cannot touch '/mnt/efs/test': Read-only file system
```



可以看到客户端挂载后就是一个只读的文件系统，无法进行修改，也无法以 Root 用户访问。这是一个典型的基于资源(Resource-based)的 IAM 策略，通过进一步修改这个 IAM 策略(即指定 Principal)我们还可以为某个特定的用户设置允许或拒绝相应的权限。



注意到 NFS 客户端发起 EFS 文件系统挂载命令时，这个操作会被 CloudTrail 服务所记录下来，EventName 为“NewClientConnection”，从 CloudTrail 详细日志的 “userIdentity”和”serviceEventDetail”中我们可以看到详细的 IAM 权限检查结果

Json

```
{    "eventVersion": "1.05",    "userIdentity": {        "type": "AWSAccount",        "principalId": "",        "accountId": "ANONYMOUS_PRINCIPAL"
...中间省略...
    "serviceEventDetails": {        "permissions": {            "ClientRootAccess": false,            "ClientMount": true,            "ClientWrite": false        },        "sourceIpAddress": "172.31.44.108"    }}
```

从这个记录可以看到，这个挂载操作是匿名用户发起，EFS 根据文件系统权限，给予 NFS 客户端只读的权限。

### 2.2 为 EC2 实例授予读写权限

如果我们希望某个 EC2 实例做为管理节点，具有文件系统的读写权限，那应该如何配置呢？其中一种可行的方式是创建一个 IAM 角色（本示例中该角色命为 FSWrite)，并为这个角色配置具有文件系统写权限的策略，通过将这个 IAM 角色附加给管理节点的 EC2 实例，从而使该实例上运行的程序具有对文件系统的写操作权限，如下是相应的策略的示例：

Json

```
{    "Version": "2012-10-17",    "Id": "efsid",    "Statement": [        {            "Sid": "efssid",            "Effect": "Allow",            "Action": [                "elasticfilesystem:ClientWrite",                "elasticfilesystem:ClientRootAccess"            ],            "Resource": "arn:aws-cn:elasticfilesystem:cn-northwest-1:402202783068:file-system/fs-57789eb2"        }    ]}
```

在这个管理节点的 EC2 上面进行挂载时，如果直接使用 mount 命令并用默认参数进行挂载的话，并不会去使用 EC2 所附加的角色权限，因此需要使用如下命令及参数进行挂载：

```
$ sudo mount -t efs fs-57789eb2:/ -o tls,iam /mnt/efs
```

其中：

- tls: NFS客户端与EFS之间的通信需要使用TLS1.2进行加密，
- iam: 使用mount程序所运行环境的IAM权限，这个例子中使用的是附加到EC2的角色。使用iam选项时需要同时启用tls选项



检查 CloudTrail 中的日志，可以看到 IAM 权限检查的结果：

Json

```
{    "eventVersion": "1.05",    "userIdentity": {        "type": "AssumedRole",        "principalId": "AROAV3JJHNVOITSOOW6BF:i-04e69fb201d65ce6e",        "arn": "arn:aws-cn:sts::402202783068:assumed-role/FSWrite/i-04e69fb201d65ce6e",
...中间省略...
    "serviceEventDetails": {        "permissions": {            "ClientRootAccess": true,            "ClientMount": true,            "ClientWrite": true        },        "sourceIpAddress": "172.31.44.108"    }}
```

从这个记录可以看到，这个挂载操作使用了 FSWrite 这个角色，EFS 根据角色权限，给予 NFS 客户端写入的权限和 Root 访问的权限。

## 3 通过接入点(Access Points)设置用户访问权限

接下来我们再看一下，如何进一步对 NFS 客户端进行权限的控制，包括限制操作系统用户身份和限制可访问的文件系统路径等。这里会使用到 EFS 的接入点(Access Points)设置。

接着上面的例子，假设我们希望这个管理节点的 EC2 实例，在写入 EFS 文件系统时，强制使用操作系统用户 id 为 1003, 用户组 id 也为 1003, 同时限制这个用户只能往 /data 目录下面进行数据写入。那我们可以配置如下的接入点：

[![img](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs4.png)](https://s3.cn-north-1.amazonaws.com.cn/awschinablog/fine-grained-control-of-nfs-file-system-permissions-via-amazon-efs4.jpg)

在这个接入点配置里，我们设置了：

- POSIX用户: 这里我们可以设置通过该接入点访问时，强制使用的操作系统用户ID/组ID。本示例中我们使用了用户ID和组ID均为1003，以便与现有操作系统用户区分。
- 目录：这里我们可以设置通过接入点访问时做为根目录的路径，如果该目录不存在，则在第一次通过接入点访问时EFS会自动创建。如果不指定则默认使用文件系统根目录。本示例中我们指定了/data做为根目录
- 拥有者用户与权限：这里我们可以设置根目录拥有者的用户ID和组ID，同时我们也可以指定这个根目录的权限。本示例我们指定拥有者用户ID和组ID均为1003，权限是755

接着我们需要修改管理员所使用的 IAM 角色(即 FSWrite)的策略，限制其只能通过该接入点访问时才赋予写权限：

Json

```
{    "Version": "2012-10-17",    "Id": "efsid",    "Statement": [        {            "Sid": "efssid",            "Effect": "Allow",            "Action": [                "elasticfilesystem:ClientWrite",                "elasticfilesystem:ClientRootAccess"            ],            "Resource": "arn:aws-cn:elasticfilesystem:cn-northwest-1:402202783068:file-system/fs-57789eb2",            "Condition": {                "StringEquals": {                    "elasticfilesystem:AccessPointArn": "arn:aws-cn:elasticfilesystem:cn-northwest-1:402202783068:access-point/fsap-068adf52269d7df77"                }            }        }    ]}
```

可以看到这个策略修改的地方是最后的”Condition”段，增加了指定接入点 ARN 为 fsap-068adf52269d7df77 。

接下来我们在管理节点的 EC2 实例（附加了 FSWrite 角色）上，通过这个接入点进行挂载，挂载时需要使用 -o tls,iam,accesspoint 参数指定接入点，启用 TLS 和 IAM 权限认证：

```
$ sudo mount -t efs fs-57789eb2:/ -o tls,iam,accesspoint=fsap-068adf52269d7df77 /mnt/efs
```

接入我们进行写入测试：

```
$ touch /mnt/efs/test1$ ls -l /mnt/efs/test1-rw-rw-r-- 1 1003 1003 0 Feb  7 13:25 /mnt/efs/test1$ iduid=1000(ec2-user) gid=1000(ec2-user) groups=1000(ec2-user),4(adm),10(wheel),190(systemd-journal)
```

可以看到我们获得了写入权限，同时写入的文件所有者用户 ID 和组 ID 均为接入点所指定的 1003，而不是当前命令行的用户 id 1000

注意到我们角色的策略里虽然赋予了 Rootaccess 的权限，但此时即使客户端使用 root 用户发起请求，EFS 还是会以接入点的用户 id 为准：

```
$ sudo touch /mnt/efs/test2$ ls -l /mnt/efs/test2-rw-rw-r-- 1 1003 1003 0 Feb  7 14:47 /mnt/efs/test2
```

接下来我们检查接入点是否指定了/data 做为根目录。我们重新挂载文件系统，这次挂载不使用任何接入点：

```
$ sudo umount /mnt/efs$ sudo mount -t efs fs-57789eb2:/ /mnt/efs
```

检查相关的目录和文件：

```
$ ls -l /mnt/efs/datatotal 8-rw-rw-r-- 1 1003 1003 0 Feb  7 13:25 test1-rw-rw-r-- 1 1003 1003 0 Feb  7 14:47 test2
```

可以看到/data 目录被自动创建出来，刚才通过接入点挂载后创建的文件 test1 和 test2 也在其中。



接下来我们检查，如果不通过这个接入点，我们是否还可以通过 IAM 角色 FSWrite 获取写入权限：

```
$ sudo umount /mnt/efs$ sudo mount -t efs fs-57789eb2:/ -o tls,iam /mnt/efs$ touch /mnt/efs/test2touch: cannot touch '/mnt/efs/test2': Read-only file system
```

可以看到不通过这个接入点访问时，这个角色(FSWrite)也就没有对文件系统的写入权限。

上述示例可以看到，通过将 IAM 策略与接入点进行结合，我们可以更细粒度的进行 NFS 客户端的权限设置，以便满足日常管理维护的安全需求。

## 小结

通过上面的演示，我们可以看到，EFS 在 NFS 文件权限控制的基础上，结合安全组(Security Group)，身份与访问管理服务(IAM)和 EFS 独特的接入点(Access Point)机制，简化权限控制的配置工作，并进行细粒度的权限管理和审计。



## 参考资料

- EFS IAM策略配置: https://docs.aws.amazon.com/efs/latest/ug/iam-access-control-nfs-efs.html
- EFS 安全组配置: https://docs.aws.amazon.com/efs/latest/ug/network-access.html
- EFS 接入点配置: https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html
- CloudTrail 对 EFS 访问日志记录: https://docs.aws.amazon.com/efs/latest/ug/logging-using-cloudtrail.html



**作者介绍：**林俊，AWS 解决方案架构师，主要负责企业客户的解决方案咨询与架构设计优化，同时致力于 AWS 云存储及 IoT 类服务的应用和推广。



**本文转载自 AWS 技术博客。**



**原文链接：**https://amazonaws-china.com/cn/blogs/china/fine-grained-control-of-nfs-file-system-permissions-with-amazon-efs/