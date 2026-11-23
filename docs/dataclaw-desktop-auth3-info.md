# Dataclaw-Desktop Auth3 配置

Dataclaw-Admin 在 Auth3 配置了2个应用, 分别是本地调试的 `dataclaw-desktop-localhost` 和 DEV 的 `dataclaw-desktop-dev`，对应的客户端凭证分别是用户令牌和客户端令牌。本文提供了应用、角色、权限、客户端和回调地址等信息，供在 Auth3 管理端登记核验。

# dataclaw-desktop localhost

## 本机调试应用

应用名称: dataclaw-desktop 本机开发
应用编码: dataclaw-desktop-localhost
所属分类: Dataclaw智能体矩阵
访问地址: http://localhost:5173
应用描述: <空>
权限码: <空>
角色: <空>
数据资源: 空
API授权: <all>
关联租户:
    - leaf_auth_dev/数研/省委办局
访问授权: <只读>


## 本机调试 dataclaw-desktop localhost (用户令牌)
客户端名称: dataclaw-desktop 本机调试
密钥名称: dataclaw-desktop-user-localhost
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 跟随登陆会话
重定向地址: http://localhost:3131/auth/callback,http://localhost:3131/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: YG9gK1EBxfXuQDTkk2AnnfL5
Client Secret: }VZ}FR0geB-)f2Jy[vWMYy_e3RSssPDQt@$f0o(zGgMz8[L[

## 本机调试 dataclaw-desktop localhost (客户端令牌)
无


---


## DEV 应用

应用名称: dataclaw-desktop DEV
应用编码: dataclaw-desktop-dev
所属分类: Dataclaw智能体矩阵
访问地址: http://dataclaw-desktop-auth3.dev.jinxin.cloud/
应用描述: <空>
权限码: 
角色: 
数据资源: 空
API授权: all
关联租户:
    - leaf_auth_dev/数研/省委办局
    - leaf_auth_dev/慧云/数据研究院
访问授权: <只读>


## dataclaw-desktop DEV（用户令牌）
客户端名称: dataclaw-desktop DEV
密钥名称: dataclaw-desktop-user-dev
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 1 天
刷新令牌有效期: 30 天
重定向地址: http://dataclaw-desktop0-auth3.dev.jinxin.cloud/auth/callback,http://dataclaw-desktop0-auth3.dev.jinxin.cloud/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: WUTnkb081hGnCNsNI6CmiPIa
Client Secret: uTStCm*),H>uoaG_@mL,L[Z+.i:uod&|AfQZaB}}Zelghsml

## dataclaw-desktop DEV（客户端令牌）
无
