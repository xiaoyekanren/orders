---
category: 版本控制
title: GitLab Jenkins Webhook
icon: code-branch
---

# GitLab + Jenkins Webhook 配置操作文档

> 适用环境：GitLab CE (gitlab.example.com) + Jenkins (jenkins.example.com)
>
> 前提：Jenkins 已安装 `gitlab-plugin`，且已配置 `gitlab-ssh` 凭据用于拉取代码

---

## 一、原理说明

### 1.1 整体流程

```
开发者 push 代码
    ↓
GitLab 触发 Webhook（POST 请求）
    ↓
Jenkins GitLab Plugin 接收请求（/project/JOB_NAME）
    ↓
Jenkins 根据分支过滤决定是否触发构建
    ↓
Pipeline 执行（拉取代码 → 运行任务 → 通知）
```

### 1.2 两种触发机制的区别（重要）

Jenkins 有两种**完全独立**的远程触发构建方式，它们的 URL 路径、认证方式、处理逻辑都不同，**绝对不能混用**。

---

#### 机制一：GitLab Plugin 触发（推荐用于 GitLab 仓库）

由 `gitlab-plugin` 插件提供，专门为 GitLab 设计。

**Webhook URL 格式**：
```
https://JENKINS_URL/project/JOB_NAME
```

**认证方式**：Secret Token，通过 HTTP Header 传递。
GitLab 发送 webhook 时，会在请求头中带上 `X-Gitlab-Token: <你设置的token>`，
Jenkins GitLab Plugin 收到请求后，对比 Job 配置中的 `secretToken` 字段进行验证。

**完整请求流程**：
```
GitLab                                        Jenkins
  |                                              |
  |  POST /project/my-job                        |
  |  Headers:                                    |
  |    X-Gitlab-Event: Push Hook                 |
  |    X-Gitlab-Token: my-secret-123             |
  |  Body: { ref: "refs/heads/demo", ... }       |
  |  ─────────────────────────────────────────>   |
  |                                              |
  |         GitLab Plugin 拦截 /project/ 请求     |
  |         ↓ 校验 X-Gitlab-Token                |
  |         ↓ 解析 Body 中的分支名                |
  |         ↓ 匹配 includeBranchesSpec 过滤       |
  |         ↓ 通过 → 触发构建                     |
  |                                              |
  |                              201 Created     |
  |  <─────────────────────────────────────────   |
```

**特点**：
- 能解析 GitLab 的 push/merge request payload，获取分支名、提交信息、用户等
- 支持分支过滤（只有匹配的分支才触发构建）
- 构建描述自动显示 "Started by GitLab push by 张三"
- 不需要 Jenkins 用户认证（插件自行处理鉴权）

**Jenkins 端配置位置**：Job → 配置 → 构建触发器 → "Build when a change is pushed to GitLab"

**Jenkins config.xml 中的对应节点**：
```xml
<com.dabsquared.gitlabjenkins.GitLabPushTrigger>
  <triggerOnPush>true</triggerOnPush>
  <branchFilterType>NameBasedFilter</branchFilterType>
  <includeBranchesSpec>demo</includeBranchesSpec>
  <secretToken>my-secret-123</secretToken>      <!-- 认证密钥 -->
</com.dabsquared.gitlabjenkins.GitLabPushTrigger>
```

---

#### 机制二：Remote Build Trigger（Jenkins 内置）

Jenkins 自带的远程构建触发功能，不依赖任何插件，适用于任意外部系统通过 HTTP 触发构建。

**Webhook URL 格式**：
```
https://JENKINS_URL/job/JOB_NAME/build?token=TOKEN
```

**认证方式**：Auth Token，直接拼在 URL 的 query 参数中。
Jenkins 收到请求后，对比 Job 配置中的 `authToken` 字段进行验证。

**完整请求流程**：
```
任意系统                                      Jenkins
  |                                              |
  |  POST /job/my-job/build?token=abc123         |
  |  （无特殊 Header，无 Body 或自定义 Body）       |
  |  ─────────────────────────────────────────>   |
  |                                              |
  |         Jenkins 核心引擎处理 /job/ 请求        |
  |         ↓ 校验 URL 中的 token 参数             |
  |         ↓ 通过 → 直接触发构建                  |
  |         ↓ 不解析任何 Body 内容                 |
  |                                              |
  |                              201 Created     |
  |  <─────────────────────────────────────────   |
```

**特点**：
- 不解析请求体，不知道是谁推送的、推送了什么分支
- 不支持分支过滤（收到请求就触发）
- 构建描述显示 "Started by remote host ..."
- 适用于 cron、脚本、其他 CI 系统等非 GitLab 场景

**Jenkins 端配置位置**：Job → 配置 → 构建触发器 → "触发远程构建"

**Jenkins config.xml 中的对应节点**：
```xml
<authToken>abc123</authToken>      <!-- 注意：和 secretToken 是完全不同的字段 -->
```

---

#### 两种机制对比总结

```
                    GitLab Plugin              Remote Build Trigger
                    ─────────────              ────────────────────
URL 路径             /project/JOB_NAME          /job/JOB_NAME/build
Token 传递           HTTP Header                URL 参数 (?token=)
Jenkins 配置字段     <secretToken>              <authToken>
是否解析 Payload     是（分支、用户、提交）      否
分支过滤             支持                       不支持
触发来源显示         Started by GitLab push     Started by remote host
依赖插件             gitlab-plugin              无（内置）
推荐场景             GitLab webhook             脚本/cron/其他系统
```

---

#### 混用会怎样？（踩坑案例）

如果把两种 URL 拼在一起写成：
```
https://jenkins.example.com/project/my-job/build?token=***
```

Jenkins 的路由逻辑如下：
1. 看到 `/project/` 前缀 → 交给 GitLab Plugin 处理
2. GitLab Plugin 期望的路径是 `/project/my-job`，但实际收到 `/project/my-job/build`
3. Plugin 找不到名为 `my-job/build` 的 Job → 把请求当作普通 HTTP 请求转发
4. 转发后 Jenkins 核心尝试以匿名用户身份访问 `/project/my-job/build`
5. 匿名用户没有 Build 权限 → **返回 403: anonymous is missing the Job/Build permission**

这类错误的典型表现是：webhook 配置长期存在，但**从未成功触发构建**，构建记录里只有手动触发。

### 1.3 该用哪种？

**从 GitLab 仓库触发 → 用 GitLab Plugin（机制一）**

```
Webhook URL:  https://jenkins.example.com/project/JOB_NAME
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              /project/ 路径，不是 /job/
              末尾不加 /build，不加 ?token=
```

**从脚本/cron/其他系统触发 → 用 Remote Build Trigger（机制二）**

```
触发 URL:     https://jenkins.example.com/job/JOB_NAME/build?token=TOKEN
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              /job/ 路径，带 /build?token=
```

**注意**：Job 名称区分大小写。

---

## 二、配置步骤

### 2.1 Jenkins 端配置

#### 方式 A：通过 Jenkins UI（推荐）

1. 打开 Jenkins Job → **配置**
2. **构建触发器** 区域，勾选 **Build when a change is pushed to GitLab**
3. 记录页面显示的 **GitLab webhook URL**（格式为 `/project/JOB_NAME`）
4. 点击 **高级** → **Secret token** → **Generate**，生成一个 Secret Token 并记录
5. 设置 **分支过滤**：
   - 选择 `Filter branches by name`
   - **Include** 填写需要触发的分支名（如 `demo`、`main`）
6. 保存

#### 方式 B：通过 API

```bash
# 1. 下载当前配置
curl -s -u "用户名:API_TOKEN" \
  "https://jenkins.example.com/job/JOB_NAME/config.xml" \
  > config.xml

# 2. 在 <com.dabsquared.gitlabjenkins.GitLabPushTrigger> 中添加：
#    <secretToken>你的密钥</secretToken>
#
#    确保 <triggerOnPush>true</triggerOnPush>
#    确保 <includeBranchesSpec> 填写了正确的分支名

# 3. 重要：删除 <actions>...</actions> 整个节点（Jenkins 自动生成的，提交回去会导致 500 错误）

# 4. 获取 CSRF Crumb
CRUMB=$(curl -s -u "用户名:API_TOKEN" \
  "https://jenkins.example.com/crumbIssuer/api/json" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['crumb'])")

# 5. 上传配置
curl -s -X POST -u "用户名:API_TOKEN" \
  -H "Jenkins-Crumb: $CRUMB" \
  -H "Content-Type: application/xml" \
  --data-binary @config.xml \
  "https://jenkins.example.com/job/JOB_NAME/config.xml"
```

**API 方式注意事项**：
- POST config.xml 时必须带 Jenkins-Crumb（GET 不需要）
- 必须删除 XML 中的 `<actions>` 节点，否则返回 500
- `<authToken>` 是 Remote Build Trigger 的 token，和 GitLab Plugin 的 `<secretToken>` 是两回事
- 提交包含中文的 config.xml 时，需指定 `Content-Type: application/xml; charset=utf-8`，否则中文会乱码

### 2.2 GitLab 端配置

#### 方式 A：通过 GitLab UI

1. 进入仓库 → **Settings** → **Webhooks**
2. 填写：
   - **URL**: `https://jenkins.example.com/project/JOB_NAME`
   - **Secret token**: 与 Jenkins 端设置的 Secret Token 一致
   - **Trigger**: 勾选 **Push events**
   - **Push events branch filter**（可选）: 填写分支名，如 `demo`
   - **Enable SSL verification**: 如果是内部网络可以取消勾选
3. 点击 **Add webhook**

#### 方式 B：通过 API

```bash
# 创建 webhook
curl -s --header "PRIVATE-TOKEN: <GitLab-Token>" \
  -X POST "https://gitlab.example.com/api/v4/projects/<项目ID>/hooks" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://jenkins.example.com/project/JOB_NAME",
    "push_events": true,
    "push_events_branch_filter": "demo",
    "enable_ssl_verification": false,
    "token": "与Jenkins一致的SecretToken"
  }'

# 更新已有 webhook
curl -s --header "PRIVATE-TOKEN: <GitLab-Token>" \
  -X PUT "https://gitlab.example.com/api/v4/projects/<项目ID>/hooks/<Webhook-ID>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://jenkins.example.com/project/JOB_NAME",
    "push_events": true,
    "push_events_branch_filter": "demo",
    "enable_ssl_verification": false,
    "token": "与Jenkins一致的SecretToken"
  }'
```

**如何获取项目 ID**：
```bash
curl -s --header "PRIVATE-TOKEN: <Token>" \
  "https://gitlab.example.com/api/v4/projects/<namespace>%2F<project>" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['id'])"

# 示例：group/project-a → group%2Fproject-a
# 示例：group/demo-project → group%2Fdemo-project
```

---

## 三、验证步骤

### 3.1 测试 Webhook 连通性

```bash
# 通过 GitLab API 发送测试 push 事件
curl -s --header "PRIVATE-TOKEN: <Token>" \
  -X POST "https://gitlab.example.com/api/v4/projects/<项目ID>/hooks/<Webhook-ID>/test/push_events"

# 期望结果：{"message":"201 Created"}
# 如果返回 403 "anonymous is missing the Job/Build permission" → Webhook URL 格式错误
# 如果返回 404 → Jenkins Job 名称不匹配
```

### 3.2 确认构建触发

```bash
# 查看最新构建的触发来源
curl -s -u "用户名:API_TOKEN" \
  "https://jenkins.example.com/job/JOB_NAME/lastBuild/api/json" \
  | python3 -c "
import sys,json
data=json.load(sys.stdin)
for a in data.get('actions',[]):
    if a.get('_class')=='hudson.model.CauseAction':
        for c in a.get('causes',[]):
            print(c.get('shortDescription',''))
"

# Webhook 触发显示：Started by GitLab push by 用户名
# 手动触发显示：Started by user 用户名
```

### 3.3 查看 Webhook 投递记录

GitLab UI → Settings → Webhooks → 点击 **Edit** → 查看 **Recent events**

---

## 四、排查清单

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 401 Invalid token | Secret Token 不匹配 | 确认 GitLab webhook 的 `token` 与 Jenkins GitLabPushTrigger 的 `secretToken` 完全一致 |
| 403 anonymous is missing permission | Webhook URL 错误（混用了 /project/ 和 /build?token=） | URL 改为 `/project/JOB_NAME`，去掉 `/build?token=` |
| 201 但没有触发构建 | 分支不匹配 | 检查 Jenkins 的 `includeBranchesSpec` 和 GitLab 的 `push_events_branch_filter` |
| 404 | Job 名称不对 | 检查 URL 中的 Job 名称是否与 Jenkins 完全一致（区分大小写） |
| SSL 证书错误 | 内部 CA 不被信任 | GitLab webhook 设置中取消 SSL verification |
| Webhook 显示为 disabled | 连续失败次数过多被 GitLab 自动禁用 | 修复后在 GitLab webhook 页面重新启用 |
| Jenkins 500 (API 更新配置时) | XML 中包含 `<actions>` 节点 | 上传前删除 `<actions>...</actions>` |
| Jenkins 403 (API 更新配置时) | 缺少 CSRF Crumb | POST 请求添加 `Jenkins-Crumb` Header |
| 飞书通知 Bad Request | Pipeline 中 shell 引号嵌套导致 JSON 格式错误 | 使用 `writeFile` + `curl -d @file` 代替直接拼接 JSON（见下方说明） |

---

## 五、完整配置示例

以 `group/demo-project` 仓库 + `demo-job` Job 为例：

### Jenkins Pipeline 脚本（仅拉取代码 + 飞书通知）

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '**']],
                    userRemoteConfigs: [[
                        url: 'git@gitlab.example.com:group/demo-project.git',
                        credentialsId: 'gitlab-ssh'
                    ]]
                ])
            }
        }
        stage('Notify') {
            steps {
                script {
                    def msg = '构建完成通知\n项目: ' + env.JOB_NAME + '\n构建号: #' + env.BUILD_NUMBER
                    def payload = '{"msg_type":"text","content":{"text":"' + msg + '"}}'
                    writeFile file: 'feishu.json', text: payload, encoding: 'UTF-8'
                    sh "curl -s -X POST -H 'Content-Type: application/json; charset=utf-8' -d @feishu.json 'https://open.feishu.cn/open-apis/bot/v2/hook/<飞书机器人ID>'"
                }
            }
        }
    }
}
```

> **注意**：不要在 `sh` 中直接拼接 JSON 字符串，shell 引号嵌套极易导致格式错误（尤其是包含中文时）。
> 推荐使用 `writeFile` 写入 JSON 文件，再用 `curl -d @filename` 发送。

### GitLab Webhook 配置

```
URL:          https://jenkins.example.com/project/example-job
Secret Token: <你设置的Secret-Token>
Trigger:      Push events
Branch Filter: (留空表示所有分支)
SSL:          不勾选
```

### 一键配置脚本

```bash
#!/bin/bash
# 使用方法: ./setup-webhook.sh <GitLab项目路径> <Jenkins-Job名> <Secret-Token> [分支过滤]
# 示例:     ./setup-webhook.sh group/demo-project example-job my-secret-token demo

GITLAB_PROJECT="$1"
JENKINS_JOB="$2"
SECRET_TOKEN="$3"
BRANCH_FILTER="${4:-}"

GITLAB_URL="https://gitlab.example.com"
JENKINS_URL="https://jenkins.example.com"
GITLAB_TOKEN="<你的GitLab-Token>"
JENKINS_USER="<Jenkins用户名>"
JENKINS_API_TOKEN="<Jenkins-API-Token>"

# URL 编码项目路径
ENCODED_PROJECT=$(echo "$GITLAB_PROJECT" | sed 's|/|%2F|g')

# 获取项目 ID
PROJECT_ID=$(curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/$ENCODED_PROJECT" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['id'])")

echo "Project ID: $PROJECT_ID"

# 构建 webhook 请求体
if [ -n "$BRANCH_FILTER" ]; then
  EXTRA_FIELD="\"push_events_branch_filter\": \"$BRANCH_FILTER\","
else
  EXTRA_FIELD=""
fi

# 创建 GitLab Webhook
RESULT=$(curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  -X POST "$GITLAB_URL/api/v4/projects/$PROJECT_ID/hooks" \
  -H "Content-Type: application/json" \
  -d "{
    \"url\": \"$JENKINS_URL/project/$JENKINS_JOB\",
    \"push_events\": true,
    $EXTRA_FIELD
    \"enable_ssl_verification\": false,
    \"token\": \"$SECRET_TOKEN\"
  }")

HOOK_ID=$(echo "$RESULT" | python3 -c "import sys,json;print(json.load(sys.stdin).get('id','FAILED'))" 2>/dev/null)
echo "Webhook ID: $HOOK_ID"

# 测试
echo "Testing webhook..."
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  -X POST "$GITLAB_URL/api/v4/projects/$PROJECT_ID/hooks/$HOOK_ID/test/push_events"

echo ""
echo "Done. Remember to configure the same Secret Token in Jenkins Job's GitLab trigger settings."
```

---

## 六、Webhook 对应关系示例

| GitLab 实例 | 仓库 | 分支 | Jenkins 实例 | Job | 触发方式 |
|-------------|------|------|-------------|-----|---------|
| gitlab.example.com | group/demo-project | 所有分支 | jenkins.example.com | demo-job | GitLab Plugin（secretToken） |
| gitlab.example.com | group/project-a | demo | jenkins.example.com | example-job-a | GitLab Plugin（secretToken） |
| gitlab.example.com | group/project-b | master | jenkins.example.com | example-job-b | GitLab Plugin（secretToken） |

> 以上 Job 均已移除 `authToken`（触发远程构建），仅保留 GitLab Plugin 的 `secretToken` 机制。

---

## 七、修复记录示例

### 案例 1：example-job-a（YYYY-MM-DD）

- **环境**：gitlab.example.com → jenkins.example.com
- **仓库**：group/project-a（项目 ID: <项目ID>）
- **分支过滤**：demo

| 项目 | 修复前 | 修复后 |
|------|--------|--------|
| GitLab Webhook URL | `.../project/example-job-a/build?token=***` | `.../project/example-job-a` |
| Jenkins Secret Token | 未设置 | `<已设置>` |
| GitLab Webhook Secret | 未设置（token 在 URL 参数中） | `<已设置，与Jenkins一致>` |
| Webhook 测试结果 | 403 | 201 Created |

**根本原因**：Webhook URL 混用了两种触发机制（`/project/` + `/build?token=`），Jenkins 路由失败返回 403。Jenkins 端也没有配置 `secretToken`，GitLab 端的 token 放在了 URL 参数而非 Header 中。

---

### 案例 2：example-job-b（YYYY-MM-DD）

- **环境**：gitlab.example.com → jenkins.example.com
- **仓库**：group/project-b（项目 ID: <项目ID>）
- **分支过滤**：master

| 项目 | 修复前 | 修复后 |
|------|--------|--------|
| GitLab Webhook URL | `.../project/example-job-b/build?token=***` | `.../project/example-job-b` |
| Jenkins Secret Token | 加密值（与 GitLab 端不匹配） | `<已重新设置>` |
| GitLab Webhook Secret | 未设置（authToken 值拼在 URL 中） | `<已设置，与Jenkins一致>` |
| Webhook 测试结果 | 403 | 201 Created → Build #N 触发成功 |

**根本原因**：与案例 1 完全相同。额外问题是 Jenkins 端虽然配了 `secretToken`（加密存储），但 GitLab 端用的是 `authToken`（Remote Build Trigger 的 token）拼在 URL 里，两个 token 不是同一个东西，认证无法匹配。

---

### 共性总结

两个案例的根本原因完全一致：**把 GitLab Plugin 的 URL 路径 `/project/` 和 Remote Build Trigger 的参数 `/build?token=` 拼在一起**。这是最容易犯的配置错误，因为 Jenkins UI 上这两个设置在不同的位置，容易搞混。

排查口诀：**看到 `/project/` 后面跟了 `/build`，一定是错的。**
