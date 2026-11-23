# Dataclaw-Admin Auth3 配置

Dataclaw-Admin 在 Auth3 配置了2个应用, 分别是本地调试的 `dataclaw-admin-localhost` 和 DEV 的 `dataclaw-admin-dev`，对应的客户端凭证分别是用户令牌和客户端令牌。本文提供了应用、角色、权限、客户端和回调地址等信息，供在 Auth3 管理端登记核验。


# dataclaw-admin localhost

## 应用

应用名称: dataclaw-admin 本机开发
应用编码: dataclaw-admin-localhost
所属分类: Dataclaw智能体矩阵
访问地址: http://localhost:5001
应用描述: <空>
权限码: 参见 `dataclaw/docs/98-AuthPerm/dataclaw-webadmin-权限码-local.json`，共 114 项，以 `agentos` 为根编码；具体登记状态待在 Auth3 核验。
角色: 参见 `dataclaw/docs/98-AuthPerm/dataclaw-webadmin-角色-local.json`，包括 `dataclaw-admin-system-admin`（系统管理员）和 `dataclaw-admin-user`（普通用户）；角色及人员绑定待在 Auth3 核验。
数据资源: 空
API授权: all
关联租户:
    - leaf_auth_dev/数研/省委办局
    - leaf_auth_dev/慧云/数据研究院
访问授权: <只读>


## 用户令牌
客户端名称: dataclaw-admin 本机调试
密钥名称: dataclaw-admin-user-localhost
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 跟随登陆会话
重定向地址: http://localhost:5001/auth/callback,http://localhost:5001/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: 5quXws87wm4USzXBIA9zkGse
Client Secret: V)m]C3%RO&Lz)Kb$*2!Ai([)0muwp&#xIf0oP@#*!QE-pm4Y

## 机器令牌
客户端名称: dataclaw-admin本机开发
密钥名称: dataclaw-admin-service-localhost
密钥类型: 客户端令牌密钥
状态: 已启用
访问令牌有效期: 31536000 秒 (约 1 年)
调用方源 IP/CIDR 白名单: <空>
ClientID: N2VwhbcwQrPOaMMLpWnpvUfk
Client Secret: r2wd+ut:BtojKnj_P{5We;4@xApywTCxzGix_g+:birjjnIW



---


## dataclaw-admin DEV

## 应用
应用名称: dataclaw-admin DEV
应用编码: dataclaw-admin-dev
所属分类: Dataclaw智能体矩阵
访问地址: http://agent-os.dev.jinxin.cloud
应用描述: <空>
权限码: 参见 `dataclaw/docs/98-AuthPerm/dataclaw-webadmin-权限码-dev.json`，共 114 项，以 `agentos` 为根编码；具体登记状态待在 Auth3 核验。
角色: 参见 `dataclaw/docs/98-AuthPerm/dataclaw-webadmin-角色-dev.json`，包括 `dataclaw-admin-system-admin`（系统管理员）和 `dataclaw-admin-user`（普通用户）；角色及人员绑定待在 Auth3 核验。
数据资源: 空
API授权: all
关联租户:
    - leaf_auth_dev/数研/省委办局
    - leaf_auth_dev/慧云/数据研究院
访问授权: <只读>


## 用户令牌
客户端名称: dataclaw-admin DEV
密钥名称: dataclaw-admin-user-dev
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 跟随会话
重定向地址: http://agent-os.dev.jinxin.cloud/auth/callback,http://agent-os.dev.jinxin.cloud/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: p4xp0K2aD865dwEHzLsJAJKK
Client Secret: 9b5RUyH6%tfCNm}IQz?]7<b?a5uoe*%LOBn4@k8TSK?7jJ%P

## 机器端令牌
客户端名称: dataclaw-admin DEV 机器客户端
密钥名称: dataclaw-admin-service-dev
密钥类型: 客户端令牌密钥
状态: 已启用
访问令牌有效期: 31536000 秒 (约 1 年)
调用方源 IP/CIDR 白名单: <空>
ClientID: tf3BZtt1eylIiPiYWoqHqr3r
Client Secret: >{x_MBAJ@HA(ne1EA2kRK6>Q3c(RPXh;2RO<D[vfO1%$;cFL
