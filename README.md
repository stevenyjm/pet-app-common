# pet-app-common

本目录用于存放宠物生鲜电商项目的公共文档和资源，供小程序端（pet-app）、Web管理端（pet-app-admin-web）和后端服务（pet-app-backend）三个项目共享使用。

## 目录结构

```
pet-app-common/
├── .gitignore              # Git忽略规则
├── README.md               # 本说明文件
├── docs/                   # 公共文档
│   ├── requirements/       # 跨项目需求文档
│   │   └── *.md            # 涉及多个项目的需求分析文档
│   ├── api-specs/          # API接口规范
│   │   └── *.md            # 接口定义、字段规范等
│   ├── shared-rules/       # 公共规范母版
│   │   └── *.master.md     # 跨项目规则母版（单一事实源）
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

本目录已进行独立的Git版本管理：

- **远程仓库**: `https://github.com/stevenyjm/pet-app-common.git`
- **主开发分支**: `yjm`（后续所有提交均提交至此分支）
- **分支策略**: 日常开发直接在 `yjm` 分支进行，定期同步至 `master` 分支

### 提交规范

1. 切换到 `yjm` 分支：`git checkout yjm`
2. 添加变更：`git add -A`
3. 提交变更：`git commit -m "描述变更内容"`
4. 推送到远程：`git push origin yjm`

### 注意事项

- 各项目的 `.gitignore` 中已排除本目录，避免重复管理
- 本仓库为独立仓库，不依赖任何父项目

## TRAE配置

### 父级规则（跨项目契约层）

父级目录 `宠物生鲜电商开发项目/.trae/rules/` 存放跨项目协同规则：
- `multi-project.md`：跨项目变更顺序契约、三端对齐检查清单、AI 协作行为约束

### 公共规范母版

`pet-app-common/docs/shared-rules/` 存放跨项目规则母版（单一事实源）：
- `cross-project.master.md`：三端 `cross-project.md` 的共有部分母版

### 项目专属规则

各项目已配置专属规则文件：
- `pet-app/.trae/rules/cross-project.md`（仅差异部分，引用母版）
- `pet-app-admin-web/.trae/rules/cross-project.md`（仅差异部分，引用母版）
- `pet-app-backend/.trae/rules/cross-project.md`（仅差异部分，引用母版）
- 各项目 `.trae/rules/` 下同时包含 `naming-conventions.md` 和 `docs-conventions.md`

### 使用方式

- 单项目开发：在子项目目录打开工作区，AI 加载项目专属规则
- 跨项目协同：在父级目录打开工作区，AI 同时加载父级跨项目规则和子项目专属规则

详细方案见 `docs/requirements/多项目统一工作区规则体系建设方案.md`。