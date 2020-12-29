# AWS Organizations —— 管理众多账号再也不是难题

- 亚马逊AWS官方博客

** 2019 年 10 月 31 日

**[语言 & 开发](https://www.infoq.cn/topic/development)[AWS](https://www.infoq.cn/topic/AWS)[其他](https://www.infoq.cn/topic/others)



### AWS Organizations 简述

AWS Organizations 可为多个 AWS 账户提供基于策略的管理。借助 AWS Organizations，您可以创建账户组，然后将策略应用于这些组。Organizations 支持您针对多个账户集中管理策略，无需使用自定义脚本和手动操作流程。

使用 AWS Organizations，您可以创建服务控制策略 (SCP)，从而集中控制多个 AWS 账户对 AWS 服务的使用。Organizations 支持您通过 包括 API，SDK 和 Console 界面创建新账户。Organizations 还有助于简化多个账户的计费模式，即您可以通过整合账单(Consolidated Billing)为您组织中的所有账户设置一种付费方式。所有 AWS 客户都可以使用 AWS Organizations，且无需额外付费。要注意的是任意的 AWS 账户只能存在一个 Organization 中。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-1.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-1.png)



### AWS Organizations 关键性概念

在正式接触 AWS Organizations 之前我们必须要先弄清楚其中的关键性名词和概念

**Organization – 组织**

- 组织是指一系列AWS账户，您可以将其整理为一个层次结构并进行集中式管理



**AWS account – 账户**

- AWS账户是您AWS资源的容器，例如： Amazon S3 存储桶、 Amazon EC2 实例等
- 通过AWS Identity and Access Management (IAM) 规则  (users, roles) 管理AWS资源
- AWS Organizations中最小的管理单元



**Master account – 主账户**

- 在组织中为所有账户付款的账户
- 管理您的组织的Hub（中心）节点



**Organizational unit (OU) – 组织单元**

- 组织单元是组织内的一组AWS账户
- 把AWS账户添加到逻辑组中，账号管理更方便
- AWS账户和组织单元OU可以是另外一个组织单元OU的成员
- 一个AWS账户可以是多个组织单元OU的成员



**Administrative root – 管理根**

- 管理根是整理AWS账户的起始点，也是整个组织层次架构中的最顶层的容器。



**Organization control policy (OCP) – 组织控制策略**

- 含有一个或多个语句的配置，这些语句用于定义您要应用于某组AWS账户的控制策略
- 不同应用场景使用不同的组织控制策略



下图可以帮助您更直观的理解这些概念之间的关系：

[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-2.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-2.png)



下面我会更详细的介绍以上提出的这些概念：

### Account 加入 Organization 的方式

1.从 Organization 中创建新账户

1.1 新的 AWS 账户只能通过主账户来创建

1.2 在创建账户的过程中，您可以配置下面的信息：

电子邮件地址 (必须)

账户名称 (必须)

IAM 角色名称 (可选 – 默认的名称是 OrganizationAccountAccessRole)

- 为主账户通过AssumeRole方式访问当前账户开启信任权限
- 权限设置为完全控制
- IAM 访问账单 (可选) 。注意! IAM 用户仍然需要相关的IAM权限才可以访问账单

1.3 新 AWS 账户

自动成为您组织的一部分

可以从组织中删除，但是可以通过策略让其没有执行 Leave Organization 的权限

1. 让已有账户加入Organization

2.1 邀请只能通过主账户发起

2.2 被邀请的 AWS 账户可以选择接受或者拒绝邀请

默认的行为是拒绝

可以通过 IAM 权限来控制

2.3 当邀请被接受

这个 AWS 账户就成为组织的一个成员

可应用的 OCP 规则会被自动引用到这个账户

2.4 被邀请的 AWS 账户可以被从组织中移除，但是可以通过策略让其没有执行 Leave Organization 的权限



### AWS Account 的逻辑组(OU)

- 把AWS账户添加到逻辑组中，账号管理更方便
- AWS账户和组织单元OU可以是另外一个组织单元OU的成员
- 一个AWS账户可以是多个组织单元OU的成员

如下图所示：



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-3.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-3.png)



### OCP(Organization Control Policy)

1. 描述要应用的控制策略
2. 不同的使用场景有不同的应用控制策略 OCP
3. 应用控制策略OCP可以被附加到

- 组织(Organization)
- 组织单元(OU)
- AWS账户(Account)



1. OCP拥有继承属性(AWS账户, 组织单元OU, 组织)

- Policy是继承关系的，也就是说OU会继承organization的policy，而AWS Account会继承OU的policy
- Account的policy会跟随其移动，如果这个Account从一个OU移动到另一个OU，那么属于这个Account的Policy也会移动。但是它不会再接收之前OU的Policy，而是会接收新OU的Policy



### OCP support in V1：Service Control Policies(SCPs)



1. 允许您对AWS服务 API进行访问控制



- 定义一个允许访问的API列表 – 白名单
- 定义一个禁止访问的API列表 – 黑名单



1. 服务控制策略无法被本地管理员覆盖
2. 对于IAM用户和角色，最终生效的权限是服务控制策略和相关的IAM权限的交集
3. 必要但不充分，本地用户拥有某行为权限，必须要同时拥有SCP权限和IAM权限
4. IAM 规则模拟器对SCP有感知
5. SCP的控制权限大于本地administrator的权限



下图是使用白名单和使用黑名单写 SCP 的案例:



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-4.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-4.png)



SCP 使用的是 IAM policy 的语法，和 IAM Policy 一样，但是有以下两点不同



- Resource必须是”*”
- 没有condition



一定注意，允许最终用户行为的话，必须要 SCP 和 IAM Permission 同时拥有。下图所示，如果在 SCP 允许 S3 和 EC2，但是在 IAM Permission 允许 EC2 和 SQS。那么最终用户只能拥有 EC2 的权限。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-5.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-5.png)



### 简单化的账单管理



- 单个付款账户为所有账户付款
- 组织内所有的AWS账户的资源被核算到统一的账单中并应用相关的阶梯价
- 所有已存在的整合账单家族将会被迁移到组织的账单模式中



### Organization 不同的管理级别



您可以在创建新组织的时候选择不同的管理级别



1. 账单模式(Billing mode)



- 和当前的整合账单模式向下兼容 (CB)
- 从整合账单家族中创建的的组织自动被设置为账单模式



1. 完全控制模式(Full-control mode)



- 账单模式中的所有功能
- 开启所有类型的OCP管理功能
- 从账单模式转换成完全控制模式需要得到组织中所有AWS账户同意



### 动手利用 AWS Organizations 来管理您的组织



下面我就给大家演示一下，使用 AWS Organizations 服务如何能做到快速，方便，准确的来管理您的多个账户



1. 进入AWS Organizations服务



进入 AWS Organizations 服务的方式，可以通过在 service 上搜索的方式进入，或者通过在 Account 菜单下点击 My Organization 进入。在 Service 的下拉菜单中没有这项服务。



服务栏搜索：



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-6.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-6.png)



账号栏下拉：



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-7.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-7.png)



1. 创建新的Organization



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-8.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-8.png)



在 AWS Organizations 服务界面上选择创建组织，选择希望创建的组织的种类，这里我演示时选择 Full Control 模式，如果希望仅进行账单整合(Consolidated Billing)，那么选择右边的“Enable Only Consolidated Billing”。无论是哪种模式，账单都会是整合的。区别只在于 Master Account 能不能对加入的账号进行控制



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-9.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-9.png)



注意：现在 Consolidated Billing 功能已经迁移到 AWS Organization 下了，如果点击主账号下的 My Account-> Consolidated Billing，可以看到该提示，并且点击 View Linked accounts 也是直接就会跳转到 AWS Organization 界面



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-10.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-10.png)



创建完毕之后，会出现组织架构管理界面，前面带星号的账号就是 Master Account



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-11.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-11.png)



1. 添加新Account到Organization下



在创建完 Organization 之后，添加账号到 Organization 下有两种方式



- 直接添加账户
- 邀请其他账户加入



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-12.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-12.png)



3.1 直接添加账户(Account)



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-13.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-13.png)



填写你希望添加的账号的全名和邮箱，并分配一个 IAM Role，这个 IAM Role 是用来给予这个新 Account 一个 full administrative control 的权限。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-14.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-14.png)



大约等待几秒钟，Account 即可创建。对比如果是要自己在 AWS 网页创建账号，速度上提升了非常多。不过创建时没有设置密码，需要用户登录时在控制台点击忘记密码来重置。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-15.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-15.png)



在 Global 根账号控制台登录新创建的账号，因为不知道密码，所以需要点击忘记密码来重置。随后会在注册邮箱收到重置密码的链接，点击重置密码之后，重新登录即可。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-16.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-16.png)



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-17.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-17.png)



登录账户之后，可以看到这个新账号已经可以工作，并且 Consolidated Billing 功能是默认开启的。并指示该账户已经在一个 Organization 下了。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-18.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-18.png)



1. 2 邀请其他账户(Account)加入



点击 Invite account 来邀请其他账号加入，添加账号号码或者邮箱，并点击邀请即可



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-19.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-19.png)



在界面右侧，点击 Invitations 可以看到已经邀请的账号的信息



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-20.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-20.png)



此时在被邀请的账号的注册邮箱中会收到来自 Master Account 的邀请邮件，如果希望接受这个邀请加入 Organization。那么可以选择点击邮件中的链接或者直接进入自己账号下的 Organization 界面接受也可以。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-21.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-21.png)



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-22.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-22.png)



在 Invitation 界面中可以看见 Organization 的 ID，邀请人的 Account 以及申请控制的类型，最后还有申请加入时的留言(Notes)



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-23.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-23.png)



最后回到主账号下，可以看见包括 Master Account，主动创建的 Account 以及受邀请加入的 Account 的信息



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-24.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-24.png)



### 从 Organization 删除 Account



有两种方式可以从 Organization 下面删除加入的 Account：



1. Master Account主动从Organization删除Account，点击单个或多个Account，点击删除即可



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-25.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-25.png)



1. 加入Organization的Account脱离Organization，在该Account下选择My Organization，可以看见其加入的Organization信息以及选择离开该Organization



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-26.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-26.png)



### 利用 OU(Organization Unit)来管理多个 Account



开始时 Organization 下是没有任何 OU 的，需要用户自己手动去添加。每一级 OU 又可以添加自己下一级的 OU，从而形成一个树状结构



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-27.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-27.png)



你可以创建一个多级 OU 组织，包括一个根 OU，两个一级 OU 和两个二级 OU



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-28.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-28.png)



所有的 Account 默认都在根 OU 下，你可以选择一个或者多个 Account，并把他们移动到任意 OU 下。目前只能移动 Account 到 OU，不能从 OU 主动加 Account



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-29.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-29.png)



针对不同的 OU 可以开启不同的 Policy 来控 OU 下面的 Account，不过要开启这个 feature，必须首先在 Root OU 下启动这个功能



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-30.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-30.png)



### 利用 Policy(Service Control Policy)来管理 Account 的行为



在使用 Policy 开管理 Account 时，始终记住两个原则：



1. Policy是继承关系，并且可以随Account移动



- Policy是继承关系的，也就是说OU会继承organization的policy，而AWS Account会继承OU的policy
- Account的Policy会跟随其移动，如果这个Account从一个OU移动到另一个OU，那么属于这个Account的Policy也会移动。但是它不会再接收之前OU的Policy，而是会接收新OU的Policy



1. 最终Account下的User的行为是SCP和IAM Permission共同决定的



一定注意，允许最终用户行为的话，必须要 SCP 和 IAM Permission 同时拥有。下图所示，如果在 SCP 允许 S3 和 EC2，但是在 IAM Permission 允许 EC2 和 SQS。那么最终用户只能拥有 EC2 的权限



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-31.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-31.png)



Service Control Policy 在 Policies 下面可以进行设置，默认情况就有一个什么权限都有的 FullAWSAccess Policy 并且是 attach 到 Root OU 的。这也就意味着默认情况下，只有有 Account 加入这个 Organization，那么这个 Account 就拥有操作所有 AWS 服务的权限。如果希望修改 SCP，那么可以点击右侧的 Policy editor 进行修改



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-32.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-32.png)



SCP 的格式和 IAM Policy 格式一致，但有下面两点不同。在修改完或创建完 Policy 之后，可以 attach 给任何 OU 或 Account



- Resource必须是”*”
- 没有condition



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-33.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-33.png)



点击 Policies 界面下左上角的 Create Policy 可以从头创建一个 SCP，你既可以选择使用 Policy generator 帮助你生成一个新的 SCP，也可以从你已经创建的一个 SCP 中进行复制。在创建时，你可以选择白名单模式或者黑名单模式



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-34.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-34.png)



这里我选择白名单模式，创建一个名为“SCP_Allow_access_VPC_subnet”的 SCP，仅允许 Account 创建和查看 VPC 和 Subnet，其余动作都禁止



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-35.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-35.png)



将创建的 SCP 分配给 OU，进而这个 SCP 会传递给下面添加的 Account



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-36.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-36.png)



在 SCP 配置界面下，将原来的 FullAWSAccess 策略解除，添加新创建的“SCP_Allow_access_VPC_subnet”策略。这个时候问题就来了，因为这个 Account 会继承所在 OU 的 SCP，而所在 OU 又会继承上层 OU 的 SCP。所以其实目前来说，这个 Phoenix Yao 的 Account 是会拥有两个 SCP，包括拥有所有权限的“FullAWSAccess”和限制权限的“SCP_Allow_access_VPC_subnet”



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-37.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-37.png)



在这种情况下会怎么样呢？其实和 IAM Policy 一样，SCP 只会允许最小权限，所以当你用 Phoenix Yao 账号打开 VPC 界面时，会发现你无法查看除了 VPC 和 Subnet 的任何信息。另外，注意一下，这个 Policy 生效有时候需要一些时间，不是马上能看到结果。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-38.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-38.png)



我们再来看下 IAM policy 和 SCP 一起作用的结果。使用 Phoenix Yao 这个 root 账号创建一个 User，并赋予其除了查看 Subnet 其余动作都可以做的 IAM Policy。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-39.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-39.png)



将这个 Policy 绑定到新创建的 IAM User 的 IAM Permission 中



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-40.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-40.png)



将 SCP 恢复成“FullAWSAccess”以单独测试 IAM Permission 是否生效，发现已经生效。



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-41.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-41.png)



最后将 SCP 和 IAM Permission 一起使用。得到的结果应该是取最小权限，即只能查看 VPC 的情况



[![img](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-42.png)](https://d2908q01vomqb2.awsstatic-china.com/472b07b9fcf2c2451e8781e944bf5f77cd8457c8/2018/01/23/0122-42.png)



使用 AWS Organizations 时的最佳实践



下面介绍一些在使用 AWS Organizations 服务时的最佳实践：



1. 使用CloudTrail服务监控Master Account的行为活动
2. 不要通过Master Account来管理资源
3. 在管理你的组织时要采用”最小权限分配”的原则
4. 使用OU来对账号进行权限的分配管理
5. 仅给Organizations的根账号(root)分配仅需要的权限
6. 在测试权限时先使用单个AWS Account来测试
7. 避免在一个组织中同时使用”白名单”策略和”黑名单”策略
8. 每创建一个新的AWS Account都需要有足够合理的原因



### 给予 Organization 最小权限



- 对所有AWS组织行为设置相对应的IAM权限
- 您也可以把AWS组织相关的元素（组织，组织单元，AWS账户）作为IAM规则的资源进行管理
- 您可以把管理您组织的权限通过IAM role功能委托给一个在其他AWS账号中的IAM用户
- 所有的组织管理行为可以通过AWS CloudTrail服务记录



**作者介绍**

姚远，AWS 解决方案架构师，负责基于 AWS 的云计算方案架构的咨询和设计，同时致力于 AWS 云服务在国内的应用和推广。现致力于网络和 DevOps 相关领域的研究。在加入 AWS 之前，在思科中国担任系统工程师，负责方案咨询和架构设计，在企业私有云和基础网络方面有丰富经验。



**本文转载自 AWS 技术博客。**



**原文链接：**



https://amazonaws-china.com/cn/blogs/china/aws-organizations-multi-account-management/



2019 年 10 月 31 日 08:00152