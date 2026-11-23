# Dataclaw-Web Auth3 配置

Dataclaw-Web 在 Auth3 配置了2个应用, 分别是本地调试的 `dataclaw-web-localhost` 和 DEV 的 `dataclaw-web-dev`，对应的客户端凭证分别是用户令牌和客户端令牌。本文提供了应用、角色、权限、客户端和回调地址等信息，供在 Auth3 管理端登记核验。


# dataclaw-web localhost

## 本机调试应用

应用名称: dataclaw-web 本机开发
应用编码: dataclaw-web-localhost
所属分类: Dataclaw智能体矩阵
访问地址: http://localhost:5001
应用描述: <空>
权限码: 
角色: 
数据资源: 空
API授权:
关联租户:
    - leaf_auth_dev/慧云/数据研究院
    - leaf_auth_dev/数研/省委办局
访问授权: <只读>


## 本机调试 dataclaw-web localhost (用户令牌)
客户端名称: dataclaw-web 本机调试
密钥名称: dataclaw-web-user-localhost
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 跟随会话
重定向地址: http://localhost:5001/auth/callback,http://localhost:5001/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: DTn97Axwp8ABjQJXM1YmlriR
Client Secret: 2]=+8KVt?[;EC1_=A{NmH7qcMXG5xR_7mNF7gnm5DH38z;?@

## 本机调试 dataclaw-web localhost (客户端令牌)
无



---


## DEV 应用

应用名称: dataclaw-web DEV
应用编码: dataclaw-web-dev
所属分类: Dataclaw智能体矩阵
访问地址: http://dataclaw.dev.jinxin.cloud/
应用描述: <空>
权限码: 
角色: 
数据资源: 空
API授权: all
关联租户:
    - leaf_auth_dev/数研/省委办局
    - leaf_auth_dev/慧云/数据研究院
访问授权: <只读>


## dataclaw-web DEV（用户令牌）
客户端名称: dataclaw-web DEV
密钥名称: dataclaw-web-user-dev
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 跟随登陆会话
重定向地址: http://dataclaw.dev.jinxin.cloud/auth/callback,http://dataclaw.dev.jinxin.cloud/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: YKT8Ydk9aeffZfjxcKhbF4rq
Client Secret: {OYBUN=CrJv-uuBy(<xghQ@Agzdk$I;2:9.ciyLHpd|TCsS; 

## dataclaw-web DEV（客户端令牌）
无
