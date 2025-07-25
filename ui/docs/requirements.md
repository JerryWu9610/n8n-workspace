# 需求

提供一个页面，作为 n8n 工作流的触发页面

页面划分如下：

- 工作流列表页，支持搜索，不需要考虑分页
- 工作流的触发分为简单与复杂两种形式，简单形式直接在页面上弹出抽屉展示表单，复杂形式打开一个新页面，由新页面接管工作流触发
- 工作流触发表单，搜集用户输入触发工作流（以抽屉的形式展示表单），表单内容通过接口返回 JSON Schema 渲染
- 个人信息设置页，填写用户名、工作流触发所需要的公共 Token，以本地存储的形式直接存储在浏览器

技术需求：

- 使用 Vue3 作为前端框架，其他配套工具按需选用
- 组件库选用 Element Plus
- 使用 vue-json-schema-form 进行动态表单渲染，文档在 `https://github.com/lljj-x/vue-json-schema-form/tree/master/packages/lib/vue3/vue3-form-element`

## 接口定义

### 1. 获取工作流列表

- **Endpoint:** `POST /workflow/api/get-workflows`
- **功能:** 获取所有可用的工作流列表，不需要支持接口搜索
- **成功响应 (200 OK):**
  ```json
  [
    {
      "id": "wf_deploy_app",
      "name": "部署前端应用",
      "description": "一键部署前端应用到测试环境。",
      "type": "simple",
      "inputParams": {
        "jsonSchema": {
            "type": "object",
            "properties": {
            "appName": { "type": "string", "title": "应用名称" },
            "branch": { "type": "string", "title": "部署分支", "default": "main" }
            },
            "required": ["appName", "branch"]
        },
        "uiSchema": {
            "appName": { "ui:placeholder": "请输入应用名称，例如 my-app" }
        }
      }
    },
    {
      "id": "wf_create_project",
      "name": "创建 GitLab 项目",
      "description": "此操作需要跳转到专门的页面完成。",
      "type": "complex",
      "trigger-url": "/new-page/gitlab-creator"
    }
  ]
  ```
- **说明:**
    - `type`: `simple` 表示抽屉表单，`complex` 表示新页面。
    - `jsonSchema`: 用于 `vue-json-schema-form` 渲染表单。
    - `trigger-url`: `complex` 类型需要跳转的页面地址。
    - `inputParams`: 触发此工作流时，需要用户输入的参数，包括 `jsonSchema` 和 `uiSchema`。

### 2. 触发工作流

- **Endpoint:** `POST /workflow/api/trigger-workflow`
- **功能:** 触发一个指定的工作流。
- **Header:**
    - 携带个人设置中设置的所有 Token，分散在各个定义好的不同的头中
- **请求体 (Body):**
  ```json
  {
    "workflowId": "wf_deploy_app",
    "input": {
      "appName": "my-app",
      "branch": "feature-x"
    }
  }
  ```
- **成功响应 (200 OK):**
  ```json
  {
    "code": "OK",
    "message": "工作流触发成功！",
    "data": null
  }
  ```
- **失败响应 (e.g., 400 Bad Request):**
  ```json
  {
    "code": "ERROR",
    "message": "错误：GitLab Token 无效或权限不足。",
    "data": null
  }
  ```

## 个人信息存储

用户信息将存储在 `localStorage` 中。

- **Key:** `n8n_trigger_page_settings`
- **Value (示例):**
  ```json
  {
    "username": "李四",
    "tokens": {
      "X-Gitlab-Token": "glpat-xxxxxxxxxx",
      "X-Jira-Token": "abcdef123456"
    }
  }
  ```
- **说明:**
    - `tokens` 是一个键值对集合，键是请求头的名称，值是对应的 Token。
    - 个人设置页预定义好所需要填写的 Token，用户可以自行填写



