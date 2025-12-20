# Jenkins Core 模块架构分析

## 目录
1. [概述](#概述)
2. [核心模块设计](#核心模块设计)
3. [架构组件](#架构组件)
4. [设计模式与原则](#设计模式与原则)
5. [实现原理](#实现原理)
6. [背景知识](#背景知识)
7. [依赖关系](#依赖关系)

---

## 概述

Jenkins Core 模块是 Jenkins 自动化服务器的核心组件，包含了系统的主要业务逻辑、模型定义、扩展机制和 Web UI 渲染能力。该模块采用插件化架构，为 Jenkins 提供了强大的可扩展性。

### 基本信息
- **模块名称**: `jenkins-core`
- **Group ID**: `org.jenkins-ci.main`
- **许可证**: MIT License
- **主要语言**: Java
- **构建工具**: Maven
- **Web 框架**: Stapler

---

## 核心模块设计

### 1. 模块结构

Core 模块遵循标准的 Maven 项目结构：

```
core/
├── src/
│   ├── main/
│   │   ├── java/          # Java 源代码
│   │   │   ├── hudson/    # 传统包名（历史遗留）
│   │   │   ├── jenkins/   # 新包名
│   │   │   └── org/       # 第三方组件
│   │   └── resources/     # 资源文件
│   │       ├── hudson/    # Jelly 视图和国际化资源
│   │       ├── jenkins/   # Jenkins 特定资源
│   │       ├── lib/       # JavaScript/CSS 库
│   │       └── META-INF/  # 元数据和升级脚本
│   ├── test/              # 测试代码
│   ├── filter/            # 资源过滤配置
│   └── site/              # 站点文档
├── pom.xml                # Maven 配置文件
└── move-l10n.groovy       # 国际化工具脚本
```

### 2. 主要包结构

#### hudson 包（传统包）
- **`hudson`**: 核心工具类和基础设施
  - `hudson.model`: 核心领域模型（Job、Build、User、Queue 等）
  - `hudson.scm`: 源代码管理抽象层
  - `hudson.security`: 安全和权限管理
  - `hudson.tasks`: 构建任务和步骤
  - `hudson.util`: 工具类和辅助功能
  - `hudson.slaves`: Agent/从节点管理
  - `hudson.triggers`: 触发器机制

#### jenkins 包（现代包）
- **`jenkins.model`**: 核心模型改进和新特性
- **`jenkins.security`**: 增强的安全特性
- **`jenkins.util`**: 现代工具类
- **`jenkins.tasks`**: 任务执行框架
- **`jenkins.management`**: 管理界面
- **`jenkins.plugins`**: 插件管理增强

---

## 架构组件

### 1. 核心类层次结构

#### Jenkins 主类
```java
jenkins.model.Jenkins extends hudson.model.AbstractCIBase
```

`Jenkins` 类是整个系统的单例入口点，负责：
- 系统初始化和生命周期管理
- 插件加载和管理
- 全局配置存储
- 根 URL 空间路由
- 任务队列管理
- 用户和权限管理

#### 领域模型核心类

**Item 层次结构**（项目/作业）
```
ModelObject (接口)
  └── Item (接口)
      ├── TopLevelItem (接口)
      │   ├── AbstractProject
      │   │   ├── Project (已弃用)
      │   │   └── FreeStyleProject
      │   └── AbstractTopLevelItem
      └── ItemGroup (接口)
```

**Run 层次结构**（构建执行）
```
Actionable (抽象类)
  └── Run (抽象类)
      ├── AbstractBuild
      │   ├── Build (已弃用)
      │   └── FreeStyleBuild
      └── 其他构建类型
```

**Node/Computer 层次结构**（执行节点）
```
Node (抽象类)
  ├── Jenkins (master 节点)
  └── Slave (agent 节点)

Computer (抽象类)
  ├── Jenkins.MasterComputer
  └── SlaveComputer
```

### 2. 扩展点机制

Jenkins 使用强大的扩展点（Extension Point）机制实现插件化：

#### ExtensionPoint 接口
```java
public interface ExtensionPoint {
    // 标记接口，用于文档生成和类型标识
}
```

所有可扩展的组件都实现 `ExtensionPoint` 接口，包括：
- `SCM` - 源代码管理系统
- `Builder` - 构建步骤
- `Publisher` - 构建后操作
- `BuildWrapper` - 构建环境包装器
- `SecurityRealm` - 认证域
- `AuthorizationStrategy` - 授权策略

#### Extension 注解
```java
@Extension
public class MyExtension extends ExtensionPoint {
    // 实现类自动被发现和注册
}
```

#### ExtensionList/ExtensionFinder
- **ExtensionList**: 管理特定类型的所有扩展实例
- **ExtensionFinder**: 发现和加载扩展的策略接口
- **GuiceFinder**: 基于 Google Guice 的扩展发现
- **Sezpoz**: 基于注解处理的扩展发现

### 3. Descriptor 模式

Jenkins 广泛使用 Descriptor 模式来描述可配置组件：

```java
public abstract class Descriptor<T extends Describable<T>> {
    // 提供元数据、配置 UI、验证等
}
```

**作用**：
- 提供组件的显示名称、描述和图标
- 定义配置表单（通过 Jelly/Groovy 视图）
- 处理表单验证和提交
- 管理全局配置
- 提供自动补全等辅助功能

**典型用法**：
```java
public class MyBuilder extends Builder {
    @DataBoundConstructor
    public MyBuilder(String config) { ... }
    
    @Extension
    public static final class DescriptorImpl extends BuildStepDescriptor<Builder> {
        @Override
        public String getDisplayName() {
            return "My Builder";
        }
    }
}
```

### 4. 插件管理系统

#### PluginManager
- 负责插件的生命周期管理
- 插件依赖解析
- 插件更新和安装
- 插件类加载器隔离

#### PluginWrapper
- 封装单个插件的元数据
- 管理插件的启用/禁用状态
- 提供插件特定的类加载器

#### ClassicPluginStrategy
- 默认的插件加载策略
- 处理 `.jpi` 和 `.hpi` 文件格式
- 管理插件依赖关系

#### Plugin 基类
- 可选的插件入口点
- 提供 `start()` 和 `stop()` 生命周期方法
- 可定义全局配置页面

### 5. 任务队列系统

#### Queue 类
Jenkins 的任务队列是一个复杂的调度系统：

**主要组件**：
- **Queue.Item**: 队列中的待执行项
- **Queue.Task**: 可执行的任务接口
- **Queue.Executable**: 实际执行的实例
- **QueueTaskDispatcher**: 决定任务是否可以执行
- **QueueSorter**: 队列排序策略
- **LoadBalancer**: 负载均衡策略

**工作流程**：
1. 任务进入队列（通过 `Queue.schedule()`）
2. 静默期（quiet period）等待
3. 执行前检查（QueueTaskDispatcher）
4. 分配执行节点（LoadBalancer）
5. 创建 Executable 并在 Executor 上运行

### 6. 构建执行系统

#### Executor
- 代表一个执行线程
- 绑定到特定的 Computer
- 执行 Queue.Executable
- 管理构建的生命周期

#### Computer
- 代表一个执行节点（master 或 agent）
- 管理多个 Executor
- 处理节点的在线/离线状态
- 监控节点健康状态

#### Launcher
- 抽象启动外部进程的机制
- 支持本地和远程执行
- 处理环境变量和工作目录

---

## 设计模式与原则

### 1. 设计模式

#### 单例模式（Singleton）
- `Jenkins.getInstance()`: 全局唯一的 Jenkins 实例
- 确保系统配置的一致性

#### 工厂模式（Factory）
- `Descriptor`: 作为组件的工厂
- `ExtensionList`: 扩展实例的工厂和注册表

#### 观察者模式（Observer）
- `Listener` 接口家族（ItemListener、RunListener、ComputerListener 等）
- 允许插件监听系统事件

#### 策略模式（Strategy）
- `SecurityRealm`: 认证策略
- `AuthorizationStrategy`: 授权策略
- `QueueSorter`: 队列排序策略

#### 装饰器模式（Decorator）
- `BuildWrapper`: 包装构建过程
- `LauncherDecorator`: 增强进程启动

#### 代理模式（Proxy）
- `FilePath`: 代理远程文件操作
- Remoting 层: 透明的远程方法调用

#### 模板方法模式（Template Method）
- `AbstractProject`: 定义项目生命周期的骨架
- `AbstractBuild`: 定义构建过程的模板

### 2. 面向对象原则

#### 开闭原则（Open-Closed Principle）
- 通过 ExtensionPoint 和 Extension 实现
- 系统对扩展开放，对修改关闭

#### 依赖倒置原则（Dependency Inversion）
- 依赖抽象（ExtensionPoint）而非具体实现
- 通过依赖注入（Guice）管理依赖

#### 单一职责原则（Single Responsibility）
- 清晰的类职责划分
- Model、View、Controller 分离

#### 里氏替换原则（Liskov Substitution）
- 扩展点的实现可互相替换
- 严格的接口契约

---

## 实现原理

### 1. 初始化流程

Jenkins 的初始化使用 Reactor 模式：

```java
InitReactorRunner → InitMilestone
```

**初始化里程碑**：
1. **STARTED**: 启动开始
2. **PLUGINS_LISTED**: 插件列表加载
3. **PLUGINS_PREPARED**: 插件准备就绪
4. **PLUGINS_STARTED**: 插件启动完成
5. **EXTENSIONS_AUGMENTED**: 扩展增强完成
6. **JOB_LOADED**: 作业加载完成
7. **COMPLETED**: 初始化完成

**关键 Initializer**：
- 使用 `@Initializer` 注解标记
- 可指定依赖的里程碑
- 并行执行以提高启动速度

### 2. 持久化机制

#### XmlFile 类
Jenkins 使用 XStream 进行对象序列化：

```java
XmlFile xmlFile = new XmlFile(new File("config.xml"));
xmlFile.write(object);  // 序列化
Object obj = xmlFile.read();  // 反序列化
```

**特点**：
- 人类可读的 XML 格式
- 支持版本迁移（通过 RobustReflectionConverter）
- 原子写入（write-to-temp + rename）

#### Saveable 接口
```java
public interface Saveable {
    void save() throws IOException;
}
```

所有可持久化的对象实现此接口。

#### BulkChange 机制
批量修改时延迟保存：
```java
try (BulkChange bc = new BulkChange(saveable)) {
    // 进行多个修改
    // ...
}  // 退出时自动保存
```

### 3. Web UI 渲染

#### Stapler 框架
Jenkins 使用 Stapler 进行 URL 路由和视图渲染：

**URL 绑定**：
- 对象树映射到 URL 空间
- 方法名对应 URL 路径
- `index.jelly` 对应根路径

**示例**：
```
/jenkins/job/MyProject/build
    ↓
Jenkins.getInstance()
    .getItem("MyProject")
    .doBuild()
```

#### Jelly 视图技术
- XML 语法的模板语言
- 标签库扩展（taglib）
- 国际化支持（i18n）

#### Groovy 视图
- 更现代的视图技术
- 类型安全
- IDE 支持更好

### 4. 远程通信（Remoting）

#### Channel 抽象
Jenkins 和 Agent 之间通过 Channel 通信：

```java
Channel channel = new Channel(...);
channel.call(new MyCallable());  // 远程调用
```

**特点**：
- 双向通信通道
- 对象序列化传输
- 支持类加载器代理
- 错误处理和重试

#### FilePath 远程文件操作
```java
FilePath remotePath = new FilePath(channel, "/path/on/agent");
remotePath.copyTo(localPath);
```

透明处理本地和远程文件操作。

### 5. 安全模型

#### SecurityRealm（认证）
- **HudsonPrivateSecurityRealm**: Jenkins 内置用户数据库
- **LDAPSecurityRealm**: LDAP/AD 集成
- **PAMSecurityRealm**: Unix PAM 认证
- 插件可提供更多实现

#### AuthorizationStrategy（授权）
- **FullControlOnceLoggedInAuthorizationStrategy**: 登录后全权限
- **ProjectMatrixAuthorizationStrategy**: 基于矩阵的权限控制
- **GlobalMatrixAuthorizationStrategy**: 全局权限矩阵

#### ACL（访问控制列表）
```java
if (item.hasPermission(Item.CONFIGURE)) {
    // 允许配置
}
```

细粒度的权限检查。

#### CSRF 保护
- Crumb 机制防止跨站请求伪造
- 每个表单需要包含 crumb token

---

## 背景知识

### 1. 历史演进

#### Hudson → Jenkins
- **2004**: Hudson 项目启动（Kohsuke Kawaguchi）
- **2011**: Oracle 商标争议，项目分叉为 Jenkins
- **包名遗留**: 保留 `hudson` 包以维持向后兼容

#### 版本演进
- **1.x**: 单体应用时代
- **2.x**: 现代化改造，引入 Pipeline
- **当前**: 云原生、容器化、配置即代码

### 2. 技术栈

#### 核心依赖
- **Java 17 或 21**: 主要编程语言（支持 Eclipse Temurin 或 OpenJDK）
- **Stapler**: Web 框架（Object-URL 映射）
- **Jelly**: 视图模板引擎
- **XStream**: XML 序列化
- **Guice**: 依赖注入框架
- **Remoting**: 远程通信库
- **Winstone/Jetty**: 嵌入式 Servlet 容器

#### 前端技术
- **YUI**（已弃用）: 传统 JavaScript 框架
- **jQuery**: DOM 操作和 Ajax
- **Prototype.js**: 部分遗留代码
- **现代化改造**: 逐步迁移到现代前端技术

### 3. 构建和测试

#### Maven 配置
- 多模块项目
- BOM (Bill of Materials) 管理依赖版本
- Spotless 代码格式化
- SpotBugs 静态分析

#### 测试框架
- **JUnit**: 单元测试
- **Mockito**: Mock 框架
- **HtmlUnit**: Web UI 测试
- **JenkinsRule**: 集成测试辅助类

### 4. 性能考虑

#### 缓存策略
- `@CopyOnWrite`: 读多写少的集合
- `ConcurrentHashMap`: 线程安全的缓存
- 延迟加载（Lazy Loading）

#### 并发控制
- `@GuardedBy`: 标记保护字段的锁
- `ReadWriteLock`: 读写分离锁
- `Executor` 池管理

---

## 依赖关系

### 1. 模块依赖

```
jenkins-parent (根 POM)
  ├── bom (依赖版本管理)
  ├── websocket/* (WebSocket 支持)
  ├── core (核心模块) ← 当前模块
  ├── war (Web 应用打包)
  ├── test (测试工具)
  ├── cli (命令行接口)
  └── coverage (代码覆盖率)
```

### 2. 主要外部依赖

#### 核心框架
- `org.jenkins-ci.main:remoting` - 远程通信
- `org.jenkins-ci.main:cli` - CLI 支持
- `args4j:args4j` - 命令行参数解析
- `com.google.inject:guice` - 依赖注入
- `org.kohsuke.stapler:stapler` - Web 框架

#### 数据处理
- `com.thoughtworks.xstream:xstream` - XML 序列化
- `net.sf.json-lib:json-lib` - JSON 处理
- `com.google.guava:guava` - 工具库
- `org.apache.commons:commons-*` - 通用工具

#### Web 和安全
- `org.springframework.security:spring-security-*` - 安全框架
- `jakarta.servlet:jakarta.servlet-api` - Servlet API
- `org.kohsuke:windows-package-checker` - Windows 集成

#### 模板和视图
- `org.jenkins-ci:annotation-indexer` - 注解索引
- `org.jvnet.hudson:commons-jelly` - Jelly 引擎
- `org.jenkins-ci:symbol-annotation` - 符号注解

### 3. 测试依赖
- `org.junit.jupiter:junit-jupiter` - JUnit 5
- `org.mockito:mockito-core` - Mock 框架
- `org.htmlunit:htmlunit` - Web 测试
- `org.hamcrest:hamcrest` - 断言库

---

## 总结

Jenkins Core 模块是一个设计精良、高度可扩展的系统，它通过以下关键特性实现了强大的功能：

### 核心优势
1. **插件化架构**: ExtensionPoint 机制使得几乎所有功能都可扩展
2. **清晰的领域模型**: Item、Run、Node、Computer 等抽象准确建模 CI/CD 领域
3. **灵活的持久化**: XStream + 人类可读 XML，便于版本控制和手动编辑
4. **强大的远程执行**: Remoting 框架支持分布式构建
5. **完善的安全模型**: 多层次的认证授权机制
6. **渐进式现代化**: 在保持向后兼容的同时持续演进

### 设计哲学
- **约定优于配置**: 合理的默认值和自动发现机制
- **开放封闭原则**: 核心稳定，扩展开放
- **用户友好**: 图形化配置界面 + 配置即代码
- **社区驱动**: 活跃的插件生态系统

这个架构设计使得 Jenkins 能够成为 CI/CD 领域最流行和最强大的工具之一，支撑着全球数百万的软件开发团队。
