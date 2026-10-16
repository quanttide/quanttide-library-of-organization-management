# 飞书的职务

飞书把职务建成通讯录成员的一个字段 `job_title`，与职级 `job_level_id`、工作序列 `job_family_id` 并列登记，三者互不派生。职务的取值是租户级的：写入一个不存在的名称，系统会自动创建。

## 定义

职务在飞书的口径是成员的受雇信息之一，`job_title` 为字符串，读取它需要字段权限「获取用户受雇信息」（`contact:user.employee:readonly`）。与它并列的两个字段各答一件事：`job_level_id` 答层级，`job_family_id` 答工作族，分别对应「查询用户职级」「查询用户所属的工作序列」两项字段权限。三个字段独立取值，平台不校验它们之间是否自洽。

职务的取值受租户约束：「获取租户职务列表」接口读回已启用的职务；写入不存在的名称时系统自动创建，职务枚举上限 10,000 个，单个职务字符上限 255，传入空格字符串表示清空职务。错误码 41063 把长度上限写成 100 字符，与参数说明的 255 不一致，实际以校验结果为准。

错误码表还暴露了另一套岗位实体：41018（岗位信息无效）、41021（一个用户至多只能设置一个主岗）、41031（当前企业不支持设置用户岗位信息），说明飞书在职务字段之外另有独立的岗位与主岗机制，是否可用取决于企业的功能开关。

## 实现

### 字段展示

字段展示口径在管理后台「组织架构」>「字段管理」>「展示字段」中配置，文档以名片页的部门展示为例。自定义字段 `custom_attrs` 由管理员在管理后台配置，需开启「允许开放平台 API 调用」后接口才能读写，类型支持文本、网页、枚举、图片枚举与引用人员。

### 读取

`GET /open-apis/contact/v3/users/:user_id` 返回 `job_title`，需字段权限 `contact:user.employee:readonly` 或等价的通讯录权限；职级与序列各需自己的字段权限。接口频率限制为 1000 次/分钟、50 次/秒。

### 写入

`PATCH /open-apis/contact/v3/users/:user_id` 的请求体含 `job_title`、`job_level_id`、`job_family_id` 与 `custom_attrs`，调用需 API 权限「更新通讯录」（`contact:contact`）或「更新用户基本信息」（`contact:user.base`）。用户身份（`user_access_token`）只允许修改 `name`、`en_name`、`avatar_key` 三个字段，所以职务只能以应用身份（`tenant_access_token`）写入。

## 与量潮职务的对照

| 飞书 `job_title` | 量潮职务（Title） |
| :-- | :-- |
| 成员属性，可为空，随成员档案展示 | 成员与组织的名分关系，任一时刻有且仅有一个 |
| 枚举可由写入自动创建，无唯一性校验 | 名号独立登记，与职位并列、互不派生 |
| 与职级、序列并列，三者互不派生 | 名号与职权来源分离，不从职位中提炼 |
| 展示口径由字段管理配置 | 以职务指代人是组织的叙事惯例 |

平台字段回答「这个人在通讯录里怎么标」，量潮职务回答「这个人与组织是什么名分」。空值、自动建枚举、各字段互不校验，都说明它是资料字段而不是名分事实。

## 来源

- [获取单个用户信息](https://open.feishu.cn/document/server-docs/contact-v3/user/get.md)
- [修改用户部分信息](https://open.feishu.cn/document/server-docs/contact-v3/user/patch.md)
- [获取租户职务列表](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/contact-v3/job_title/list)
