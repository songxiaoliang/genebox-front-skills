---
name: genebox-create-app-page
description: |
  为 React Native 项目自动化创建页面文件。当用户需要创建新页面、添加页面模板、生成页面文件或类似需求时触发此 skill。支持自动生成 RequestPage、ReduxConnector Container 和 Page 文件，并自动配置到导航路由中。只需指定页面名称即可完成全套页面创建流程。
---

# Create RN Page

自动化创建 React Native 页面的完整流程，包括 RequestPage、Container 和 Page 三个文件，并自动配置路由。

## 使用流程

当用户请求创建新页面时：

1. **询问页面名称** - 获取用户想要创建的页面名称（如 `LSMyNewPage`）
2. **询问目标模块** - 确认页面所属的业务模块（如 `Report`、`Profile`、`Mall` 等），默认为 `Report`
3. **列出可用路由文件** - 扫描 `App/Navigation` 目录下的所有导航文件，使用 `askUserQuestion` 让用户选择要配置的路由文件
4. **生成文件** - 根据模板创建以下文件：
   - `App/Containers/{module}/RequestPages/{pageName}Request.js`
   - `App/Containers/{module}/ReduxConnectors/{pageName}Container.js`
   - `App/Pages/{module}/Pages/{pageName}.js`
5. **配置路由** - 在选中的导航文件中添加 import 和路由配置

## 文件模板

### 1. RequestPage 模板

路径：`templates/RequestPage.js.template`

内容占位符：
- `{{PAGE_NAME}}` - 页面名称（如 LSMyNewPage）
- `{{MODULE}}` - 模块名称（如 Report）
- `{{REDUX_KEY}}` - redux state key（根据页面名称自动生成，如 myNewPage）

### 2. Container 模板

路径：`templates/Container.js.template`

内容占位符：
- `{{PAGE_NAME}}` - 页面名称
- `{{MODULE}}` - 模块名称

### 3. Page 模板

路径：`templates/Page.js.template`

内容占位符：
- `{{PAGE_NAME}}` - 页面名称
- `{{MODULE}}` - 模块名称

## 执行步骤

### 步骤 1: 获取页面名称

询问用户要创建的页面名称。如果用户没有提供，提示用户输入。

### 步骤 2: 确认模块

询问页面所属的业务模块，默认使用 `Report`。模块决定了文件创建的子目录：
- `App/Containers/{module}/RequestPages/`
- `App/Containers/{module}/ReduxConnectors/`
- `App/Pages/{module}/Pages/`

### 步骤 3: 选择导航文件

扫描 `App/Navigation` 目录下的所有 `.js` 文件（排除 Theme 和 TransitionAnimation 子目录），列出可用选项让用户选择。

### 步骤 4: 检查文件是否存在

在创建前检查目标文件是否已存在，如果存在则询问用户是否覆盖。

### 步骤 5: 创建文件

使用模板生成三个文件，替换所有占位符：

1. **RequestPage**: 处理数据加载逻辑，包装 FetchPage 组件
2. **Container**: Redux 连接器，连接 state 和 dispatch 到 Page
3. **Page**: 纯 UI 组件，接收 props 渲染页面

### 步骤 6: 配置路由

在选中的导航文件中：
1. 在文件顶部添加 import 语句（按字母顺序插入到现有 imports 中）
2. 在 navigation 数组中添加路由配置对象

## 工具调用参考

### 列出导航文件
```
list_dir: {"DirectoryPath": "{workspace}/App/Navigation"}
```

### 询问用户选择导航文件
```
ask_user_question: {
  "question": "请选择要配置路由的导航文件",
  "options": [
    {"label": "ReportNavigation", "description": "报告模块导航"},
    {"label": "ProfileNavigation", "description": "个人中心导航"},
    ...
  ],
  "allowMultiple": false
}
```

### 检查文件是否存在
```
read_file: {"file_path": "{target_file_path}"}
```

### 创建文件
```
write_to_file: {
  "TargetFile": "{target_file_path}",
  "CodeContent": "{generated_content}",
  "EmptyFile": false
}
```

### 编辑导航文件
```
edit: {
  "file_path": "{navigation_file}",
  "old_string": "{existing_import_section}",
  "new_string": "{existing_import_section}\nimport {pageName} from '@/Containers/{module}/RequestPages/{pageName}Request';"
}
```

## 占位符替换规则

| 占位符 | 替换值 | 示例 |
|--------|--------|------|
| `{{PAGE_NAME}}` | 用户提供的页面名称 | `LSMyNewPage` |
| `{{MODULE}}` | 业务模块名称 | `Report` |
| `{{REDUX_KEY}}` | 小写驼峰格式的页面名 | `myNewPage` |
| `{{REQUEST_PAGE_IMPORT}}` | RequestPage 导入路径 | `@/Containers/Report/RequestPages/LSMyNewPageRequest` |
| `{{PAGE_IMPORT}}` | Page 导入路径 | `@/Pages/Report/Pages/LSMyNewPage` |

## 输出确认

完成所有步骤后，向用户确认：
- 创建的 3 个文件路径
- 配置路由的导航文件
- 页面名称和模块

示例输出：
```
页面创建完成！

已创建文件：
- App/Containers/Report/RequestPages/LSMyNewPageRequest.js
- App/Containers/Report/ReduxConnectors/LSMyNewPageContainer.js  
- App/Pages/Report/Pages/LSMyNewPage.js

已配置路由：ReportNavigation.js
  {
    name: 'LSMyNewPage',
    component: LSMyNewPage,
  }
```
