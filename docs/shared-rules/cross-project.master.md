# 跨项目协作规范母版

> 本文件是三端 `cross-project.md` 的共有部分母版。
> 修改时先更新此文件，再同步到各项目的 `.trae/rules/cross-project.md`。
> 各项目 `cross-project.md` 仅保留"当前项目标识"、"读写权限"、"读取路径"等差异部分。

## 项目结构

本项目位于 `宠物生鲜电商开发项目` 父目录下，包含以下兄弟项目：

```
宠物生鲜电商开发项目/
├── pet-app/              # 微信小程序前端（uni-app + Vue3）
├── pet-app-admin-web/    # Web 管理端（Vue3 + Element Plus）
├── pet-app-backend/      # 后端服务（Node.js + Express + Sequelize）
└── pet-app-common/       # 公共资源目录（独立 git 管理）
    ├── docs/
    │   ├── requirements/     # 跨项目需求文档
    │   ├── api-specs/        # API 接口规范
    │   ├── shared-rules/     # 公共规范母版
    │   └── shared-resources/ # 共享资源文档
    └── resources/
        ├── images/           # 共享图片资源
        └── templates/        # 模板文件
```

## 公共目录操作规范

所有跨项目共享的文档、资源应存放在 `pet-app-common/` 目录：

- **需求文档**：新建跨项目需求时，在 `pet-app-common/docs/requirements/` 创建文档
- **API 规范**：接口变更需同步更新 `pet-app-common/docs/api-specs/`
- **公共规范母版**：跨项目规则母版存放在 `pet-app-common/docs/shared-rules/`
- **共享资源**：通用图片、模板等存放在 `pet-app-common/resources/`

## 文档同步规则

当修改涉及多个项目的文档时：
1. 在 `pet-app-common/docs/` 相应子目录创建主文档
2. 在各项目的 `docs/` 目录创建引用或摘要链接
3. 保持主文档为权威来源

## Git 管理规则

- `pet-app-common` 独立 git 管理，不参与任何项目的 git 版本管理
- 各项目的 `.gitignore` 已排除 `pet-app-common/`
- 禁止将 `pet-app-common` 的文件提交到项目仓库
- 公共规范母版变更后，需同步更新三端 `cross-project.md` 副本

## 各项目差异部分

### pet-app（微信小程序前端）

**当前项目**：微信小程序前端
**读写权限**：pet-app 读写，其余只读
**读取路径**：
- 后端路由：`../pet-app-backend/routes/`
- 管理端页面：`../pet-app-admin-web/src/views/`

### pet-app-admin-web（Web 管理端）

**当前项目**：Web 管理端
**读写权限**：pet-app-admin-web 读写，其余只读
**读取路径**：
- 后端路由：`../pet-app-backend/routes/`
- 小程序组件：`../pet-app/src/components/`

### pet-app-backend（后端服务）

**当前项目**：后端服务
**读写权限**：pet-app-backend 读写，其余只读
**读取路径**：
- 小程序组件：`../pet-app/src/components/`
- 管理端组件：`../pet-app-admin-web/src/components/`