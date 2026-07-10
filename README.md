# pet-app-common

本目录用于存放宠物生鲜电商项目的公共文档和资源，供小程序端（pet-app）、Web管理端（pet-app-admin-web）和后端服务（pet-app-backend）三个项目共享使用。

## 目录结构

```
pet-app-common/
├── .gitignore              # 空目录占位，防止被git追踪
├── README.md               # 本说明文件
├── docs/                   # 公共文档
│   ├── requirements/       # 跨项目需求文档
│   │   └── *.md            # 涉及多个项目的需求分析文档
│   ├── api-specs/          # API接口规范
│   │   └── *.md            # 接口定义、字段规范等
│   └── shared-resources/   # 共享资源文档
│       └── *.md            # 通用文档、配置说明等
└── resources/              # 公共资源文件
    ├── images/             # 共享图片资源
    │   └── *.png/*.jpg     # 跨项目使用的图片
    └── templates/          # 模板文件
        └── *.md/*.json     # 通用模板、配置模板等
```

## 使用规范

### 需求文档

当需求涉及多个项目时：
1. 在 `docs/requirements/` 目录下创建主需求文档
2. 在各项目的 `docs/` 目录中创建引用链接或摘要
3. 保持主文档为权威来源，各项目文档仅作本地引用

### API规范

接口变更时：
1. 在 `docs/api-specs/` 目录更新接口规范文档
2. 确保前后端开发人员都能访问到最新规范

### 共享资源

通用资源文件（图片、模板等）：
1. 存放在 `resources/` 相应子目录
2. 文件命名清晰，便于跨项目引用

## Git管理

- **本目录不参与任何项目的git版本管理**
- 各项目的 `.gitignore` 中已添加说明注释
- 如需版本控制，可考虑单独创建git仓库

## TRAE配置

各项目已配置跨项目规则文件：
- `pet-app/.trae/rules/cross-project.md`
- `pet-app-admin-web/.trae/rules/cross-project.md`
- `pet-app-backend/.trae/rules/cross-project.md`

这些规则文件告诉AI助手如何访问公共目录和兄弟项目。