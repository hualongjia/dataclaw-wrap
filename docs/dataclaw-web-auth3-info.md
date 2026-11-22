# Auth3 P3demo Auth 配置

## 端口

- p3demo-web       3000
- p3demo-service   3001
- p3demo-mcp       3002

## 本机调试 p3demo localhost

应用名称: p3demo 本机调试
应用编码: p3demo-localhost
所属分类: Oauth2
访问地址: http://localhost:3000
应用描述: <空>
权限码: 以 p3demo/docs/98-AuthPerm/权限码.json 的版本化登记为准
角色: 以 p3demo/docs/98-AuthPerm/角色.json 的版本化登记为准；系统管理员角色编码为 p3demo-system-admin，permissionCodes 为空，不自动获得其他业务权限
数据资源: 空
API授权:
    - 登出	补充API授权	auth.logout
    - 令牌内省	补充API授权	oauth.introspect
    - 租户基本信息	补充API授权	tenant.detail
    - 查询本人基本信息	补充API授权	me.profile
关联租户:
    - leaf_auth_dev/数研/省委办局
访问授权: <只读>


## 本机调试 p3demo localhost (用户令牌)
客户端名称: p3demo 本机调试
密钥名称: p3demo-user-localhost
密钥类型: 用户令牌密钥
状态: 已启用
访问令牌有效期: 1 小时
刷新令牌有效期: 1 天
允许刷新令牌: 开启
跟随登录会话有效期: 关闭
重定向地址: http://localhost:3000/auth/callback,http://localhost:3000/auth/logout/callback
要求PKCE(S256): 开启
Scopes: openid all read write
调用方源 IP/CIDR 白名单: <空>
ClientID: BA7jTjrpWQtjTnp0PMoCqOWy
Client Secret: O8wJ]G4LkQevVT^jqh?Qg1ps1wsjSnm*,pLk>ajQ(#W<{7h)

注意：auth3 于 2026-11-18 新增“跟随登录会话有效期”设置，开启后不再签发 refresh_token，且拒绝 refresh_token 授权。p3demo 当前通过服务端保存并轮换 refresh_token 续期，因此用户令牌密钥必须关闭该设置并允许刷新令牌；否则登录换票后会因缺少 refresh_token 返回 `P3DEMO_AUTH_UNAVAILABLE`（HTTP 503）。修改配置后须重新发起登录，已消费的授权码不能重试。

## 本机调试 p3demo localhost (客户端令牌)
客户端名称: p3demo 本机调试
密钥名称: p3demo-service-localhost
密钥类型: 客户端令牌密钥
状态: 已启用
访问令牌有效期: 31536000 秒 (约 1 年)
调用方源 IP/CIDR 白名单: <空>
ClientID: iqGRVB0qF0CV8cHGHNDGRRhV
Client Secret: fKl)F%5GTrL)YLMT>6cksmT|>&Kl8JV0BS=K}o6k>}gjiuNt

