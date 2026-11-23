# Dataclaw-Admin Auth3 配置

Dataclaw-Admin 在 Auth3 配置了2个应用, 分别是本地调试的 `dataclaw-admin-localhost` 和 DEV 的 `dataclaw-admin-dev`，对应的客户端凭证分别是用户令牌和客户端令牌。本文提供了应用、角色、权限、客户端和回调地址等信息，供在 Auth3 管理端登记核验。


# dataclaw-admin localhost

## 本机调试 dataclaw-admin localhost

应用名称: dataclaw-admin 本机开发
应用编码: dataclaw-admin-localhost
所属分类: Oauth2
访问地址: http://localhost:3131
应用描述: <空>
权限码: 本地 dataclaw-admin-server/docs/98-AuthPerm/权限码.json 及 dev/ 对应文件已准备，登记 dataclaw-admin:directory-sync 分组及 view/create/update/delete/run 五个动作；auth3 管理端待核验
角色: 待在 auth3 登记 dataclaw-admin-system-admin（系统管理员）和 dataclaw-admin-user（普通用户）；系统管理员绑定目录同步分组及五个动作权限，普通用户不授予同步管理权限。模型访问仍需模型授权
数据资源: 空
API授权:
    - 登出	补充API授权	auth.logout
    - 令牌内省	补充API授权	oauth.introspect
    - 租户基本信息	补充API授权	tenant.detail
    - 查询本人基本信息	补充API授权	me.profile
关联租户:
    - leaf_auth_dev/数研/省委办局
访问授权: <只读>


## 本机调试 dataclaw-admin localhost (用户令牌)
客户端名称: dataclaw-admin 本机调试
密钥名称: dataclaw-admin-user-localhost
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 1 小时
刷新令牌有效期: 1 天
重定向地址: http://localhost:3131/auth/callback,http://localhost:3131/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: jknA4D6fIDxRYC1k5Zj1eaEI
Client Secret: UO+<}g-qP9zFPGdwz|jwv;Lq1#}]>%+oAdz3}_G!tE5>EsGf

## 本机调试 dataclaw-admin localhost (客户端令牌)
客户端名称: dataclaw-admin本机开发
密钥名称: dataclaw-admin-service-localhost
密钥类型: 客户端令牌密钥
状态: 已启用
访问令牌有效期: 31536000 秒 (约 1 年)
调用方源 IP/CIDR 白名单: <空>
ClientID: mIG9VIy1YMEThICyNJ7CMod3
Client Secret: P%P2J1p@^MxT#gW9s;LW{6tA5Kkur_p*YFg!PdI_t4%lZEpW




---


## dataclaw-admin DEV

应用名称: dataclaw-admin DEV
应用编码: dataclaw-admin-dev
所属分类: Oauth2
访问地址: http://dataclaw-admin0-auth3.dev.jinxin.cloud/
应用描述: <空>
权限码: 本地 dataclaw-admin-server/docs/98-AuthPerm/权限码.json 及 dev/ 对应文件已准备，登记 dataclaw-admin:directory-sync 分组及 view/create/update/delete/run 五个动作；auth3 管理端待核验
角色: 待在 auth3 登记 dataclaw-admin-system-admin（系统管理员）和 dataclaw-admin-user（普通用户）；系统管理员绑定目录同步分组及五个动作权限，普通用户不授予同步管理权限。模型访问仍需模型授权
数据资源: 空
API授权: all
关联租户:
    - leaf_auth_dev/数研/省委办局
    - leaf_auth_dev/慧云/数据研究院
访问授权: <只读>


## dataclaw-admin DEV（用户令牌）
客户端名称: dataclaw-admin DEV
密钥名称: dataclaw-admin-user-dev
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 1 天
刷新令牌有效期: 30 天
重定向地址: http://dataclaw-admin0-auth3.dev.jinxin.cloud/auth/callback,http://dataclaw-admin0-auth3.dev.jinxin.cloud/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: 7bvDzWMj4L0OzL72BUV1HQRF
Client Secret: 2jCk%?hldlNXsXed)pTEeK9*Rgvl%aS:ka0Pg1:JN5S>aJFd

## dataclaw-admin DEV（客户端令牌）
客户端名称: dataclaw-admin DEV 机器客户端
密钥名称: dataclaw-admin-service-dev
密钥类型: 客户端令牌密钥
状态: 已启用
访问令牌有效期: 31536000 秒 (约 1 年)
调用方源 IP/CIDR 白名单: <空>
ClientID: RlIOmkMAx5iHyJwVHtzEUA5f
Client Secret: 4D#SJ(q#&hE}J8:JXPYHfrDeFgWahwGl@u{u$ITxEDai.yz=


## 接口授权与登记核验

以上两个应用及客户端已由用户创建。本文角色、权限、客户端名称和回调地址为目标登记配置，修改本文后还需在 auth3 管理端逐项核对。

- 两个应用均需核对 `me.full`（GET /api/v3/me/full）的 API 授权，供后端查询当前用户角色、组织、组及应用权限。应用动作权限按目标 applicationId 精确匹配。
- 机器客户端需核对 `membership.orgs`、`membership.groups` 的 API 授权及目标租户授权。这两个成员接口要求 machine_service 主体。
- 本地现有 API 授权清单和 DEV 的 `all` 均需与实际平台 API 登记逐项核对。`me.full` 的登记码也需在当前部署确认。目录同步所需租户、用户分页接口授权随实际调用清单补充。
- `dataclaw-admin-system-admin` 绑定 `dataclaw-admin:directory-sync`、`dataclaw-admin:directory-sync:view`、`dataclaw-admin:directory-sync:create`、`dataclaw-admin:directory-sync:update`、`dataclaw-admin:directory-sync:delete`、`dataclaw-admin:directory-sync:run`。
- 本地回调使用 http://localhost:3131/auth/callback 和 http://localhost:3131/auth/logout/callback。DEV 回调使用 http://dataclaw-admin0-auth3.dev.jinxin.cloud/auth/callback 和 http://dataclaw-admin0-auth3.dev.jinxin.cloud/auth/logout/callback。
- 后端 DEV 域名为 dataclaw-admin0-auth3-service.dev.jinxin.cloud，部署业务端口 8080（本地开发建议 3132）。客户端凭据仅供后端配置使用。

当前待核验项包括角色和权限登记、客户端名称、DEV 访问地址及回调修改、API 授权、关联租户和真实调用结果。
