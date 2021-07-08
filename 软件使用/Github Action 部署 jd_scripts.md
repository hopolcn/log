#  Github Action 部署 jd_scripts

## 开通腾讯云函数服务

1. 登录 SCF 云函数控制台

   https://console.cloud.tencent.com/scf

2. 登录 SLS 控制台

   https://console.cloud.tencent.com/sls

3. 查看 服务角色

   https://console.cloud.tencent.com/cam/role

注意：开通相关服务，确保账户下已开通服务并创建相应的服务角色 SCF_QcsRole、SLS_QcsRole。为了确保权限足够，获取这两个参数时不要使用子账户！此外，腾讯云账户需要实名认证。

## 新建一个访问密钥

在腾讯云函数服务新建访问密钥

https://console.cloud.tencent.com/cam/capi

将SecretId和SecretKey分别配置在仓库的secrets变量里面，

TENCENT_SECRET_ID对应你的SecretId的值，

TENCENT_SECRET_KEY对应你的SecretKey的值

## 配置secrets变量

https://github.com/lxk0301/jd_scripts/blob/master/githubAction.md#%E4%B8%8B%E6%96%B9%E6%8F%90%E4%BE%9B%E4%BD%BF%E7%94%A8%E5%88%B0%E7%9A%84-secrets%E5%85%A8%E9%9B%86%E5%90%88

