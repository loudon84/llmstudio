### 1\. 模块划分 (Module Partitioning)

ToolJet CE 采用前后端分离的全栈架构，通过现代化的模块化设计实现关注点分离 1。

* **前端客户端 (React Application)：** 这是一个基于 React 的单页应用（SPA），主要包含 **App Builder**（视觉化编辑器）、**Query Manager**（查询管理界面）、**ToolJet Database UI** 和 **Editor**（主编辑界面） 1。其状态管理核心是 **Zustand**，并采用模块化存储模式（Slices）处理用户、组织、组件和查询数据 2, 3。  
* **后端服务器 (NestJS Backend)：** 基于 NestJS 框架构建，提供 REST API、身份验证和业务逻辑 4。它被划分为以下核心领域：  
* **核心模块：** 包括应用管理（Apps）、数据源（Data Sources）、查询执行（Data Queries）、身份验证（Authentication）、组织（Organizations）和用户（Users）模块 4-6。  
* **插件模块：** 负责外部数据源连接器的安装与管理 4。  
* **内置数据库模块 (ToolJet DB)：** 提供内置的无代码数据库功能 4。  
* **持久层 (Persistence)：**  
* **Application Database (PostgreSQL)：** 存储应用元数据、用户信息、组织架构、查询配置等 7。  
* **ToolJet Database (PostgreSQL)：** 用于存储用户通过内置数据库功能创建的业务表数据 7。  
* **PostgREST：** 在 ToolJet Database 之上提供 RESTful API 层，支持通过 HTTP 进行 CRUD 操作 7。  
* **插件系统 (Plugin System)：** 采用模块化架构，插件作为独立的 npm 模块包，实现 QueryService 接口以执行针对外部系统（如数据库、API、SaaS工具）的操作 7-9。

### 2\. 调用链 (Call Chain)

一个典型的 API 请求（例如 /api/v2/apps）在系统中的完整生命周期遵循标准的 NestJS 模式 10, 11：

* **中间件层 (Middleware Stack)：** 请求首先经过 Sentry（错误追踪）、压缩、Cookie 解析器、JSON/URL 编码解析器以及安全标头（CORS、Helmet、CSP）处理 11, 12。  
* **路由与全局拦截：** 请求通过全局 API 前缀路由，并经过 **ValidationPipe** 进行数据校验与转换 11, 13。  
* **守卫层 (Guards Execution)：**  
* **JwtAuthGuard：** 验证 JWT 令牌，确认会话有效性并加载用户/组织上下文 11, 13。  
* **AbilityGuard：** 检查 CASL 权限，验证用户是否有权访问特定资源或执行特定操作 14, 15。  
* **控制器层 (Controller)：** 控制器使用装饰器（如 @User()、@Ability()）提取上下文信息，并将业务逻辑委托给服务层 14, 15。  
* **服务层 (Service Layer)：** 包含核心业务逻辑，并与存储库交互 16, 17。  
* **存储库/数据访问层 (Repository/Data Access Layer)：** 使用 **TypeORM** 访问 PostgreSQL 数据库，通过 organizationId 过滤强制实现多租户隔离 16, 17。  
* **响应转换与异常处理：** 服务返回数据至控制器并转为 JSON 响应；若发生错误，由 **AllExceptionsFilter** 全局捕获并标准化错误输出 16。

### 3\. 权限位置 (Permission Location)

ToolJet 的权限控制是多层级的，结合了身份验证、授权和数据隔离 18, 19。

* **身份验证层 (Authentication)：** 在后端 SessionModule 和 AuthModule 中实现，通过存储在 HTTP-only Cookie 中的 **JWT 令牌** 管理会话 18, 20。前端通过 authentication.service.js 管理会话状态 21, 22。  
* **授权逻辑层 (Authorization \- CASL)：**  
* **核心引擎：** 使用 **CASL (Capability-based Access Control Library)** 实现细粒度权限控制 19。  
* **后端守卫：** AbilityGuard 位于控制器之前，通过 AbilityFactory 根据用户角色（ADMIN、BUILDER、END\_USER）和资源级授权动态创建能力对象并进行校验 17, 19, 23。  
* **后端服务：** AbilityService 负责从数据库查询用户所属的组权限、角色以及针对特定 App 或数据源的颗粒度权限 24。  
* **多租户隔离 (Multi-tenancy)：** 权限隔离的核心在于 **组织 (Organization/Workspace)** 25。  
* **数据库级别：** 所有实体（App、DataSource 等）都与 organizationId 关联，在数据库查询层面强制进行过滤 17, 26, 27。  
* **请求头：** 前端服务在每个 HTTP 请求中包含 tj-workspace-id 标头以声明当前工作区上下文 22。  
* **前端路由保护：** 在 App.jsx 中使用 PrivateRoute、AdminRoute 和 AppsRoute 等包装组件，基于用户的认证状态和角色限制页面访问 28。


在对 ToolJet CE 进行二次开发时，为了确保系统的稳定性、安全性和可扩展性，必须遵守以下核心设计约束。这些约束构成了 ToolJet 架构的底层逻辑，破坏它们可能导致多租户数据泄露、权限失效或插件系统崩溃。

### 1\. 多租户数据隔离约束 (Multi-tenancy Isolation)

* **组织上下文强制性：** 所有的核心实体（如 App、DataSource、Variable 等）在数据库层面必须包含 organizationId 字段 1, 2。  
* **查询过滤约束：** 在 Service 层或 Repository 层编写 SQL/TypeORM 查询时，**必须显式包含 organizationId 过滤条件** 3, 4。  
* **前端请求头：** 前端发起的任何 API 请求必须通过 authHeader() 辅助函数在 HTTP Header 中携带 tj-workspace-id，以确保后端能正确识别当前的组织上下文 5。

### 2\. 身份验证与会话约束 (Authentication & Session)

* **令牌存储一致性：** 认证采用 JWT 机制，令牌必须存储在名为 tj\_auth\_token 的 **HTTP-only Cookie** 中 6, 7。二次开发时不应改为 LocalStorage 存储，以防范 XSS 攻击。  
* **RxJS 会话流：** 前端状态管理必须依赖 authentication.service.js 中的 **RxJS BehaviorSubject** 来同步用户信息。直接修改全局变量而不通过该流会导致 UI 状态（如导航条、权限控制）不同步 5, 8。

### 3\. 授权与权限校验约束 (Authorization \- CASL)

* **守卫序列约束：** 后端 Controller 的方法必须遵循 JwtAuthGuard（身份验证）在前，AbilityGuard（权限授权）在后的执行顺序 9-11。  
* **元数据装饰器：** 任何新增的 API 接口必须使用 @InitModule 和 @InitFeature 装饰器。这些装饰器为 AbilityGuard 提供了校验权限所需的元数据；缺少它们会导致权限系统无法识别请求，从而可能导致越权访问 10, 11。  
* **能力工厂逻辑：** 权限判定逻辑必须集中在 AbilityFactory 中，通过 **CASL** 库统一管理。禁止在业务 Service 中硬编码角色（如 if(user.role \=== 'admin')），而应使用 ability.can() 进行能力判定 4, 12, 13。

### 4\. 模块化与动态加载约束 (Modular & Dynamic Loading)

* **NestJS 模块结构：** 后端开发必须遵循 module \-\> controller \-\> service \-\> repository 的分层模式 14, 15。  
* **动态注册机制：** 核心模块必须通过 AppModuleLoader 进行加载。由于 ToolJet 采用动态模块注册模式（以兼容 CE/EE/Cloud 版本），手动在 app.module.ts 中硬编码静态导入可能会破坏版本切换逻辑 16, 17。

### 5\. 插件系统契约约束 (Plugin System Contract)

* **接口实现一致性：** 新增的数据源插件必须实现标准的 QueryService 接口，并提供 run 方法 18, 19。  
* **前后端分离构建：** 插件必须保持独立的 client（配置 UI）和 server（执行逻辑）入口。破坏此结构将导致生产环境构建（build:server / create:client）失败 20, 21。  
* **敏感数据加密：** 数据源的连接凭据（Credentials）必须通过系统内置的加密机制处理，严禁明文存储在数据库中 2, 22。

### 6\. 前端状态管理约束 (Frontend State)

* **Zustand Slice 模式：** App Builder 的状态被切分为多个 Slice（如 queryPanelSlice, componentSlice）。二次开发扩展 Store 时，必须使用 **Immer 中间件** 以确保不可变状态更新，否则会破坏撤销/重做（Undo/Redo）和多人协作功能 23-25。  
* **数据流向：** 严禁在组件内部直接修改 Store 状态，必须通过定义好的 Actions 进行调度 24, 26。

### 7\. 数据库双库约束 (Database Separation)

* **职责分离：** 必须保持 **Application Database**（存储元数据）与 **ToolJet Database**（存储用户业务数据）的物理或逻辑分离 18, 27。  
* **接入方式限制：** 访问 ToolJet Database 必须通过 **PostgREST** 接口层或指定的内置模块，不得绕过权限控制直接操作底层的 PostgreSQL 实例 18, 28, 29。

