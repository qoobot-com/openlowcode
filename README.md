# OpenCode 代码自动生成系统

柏拉图代码自动生成系统 - 基于 JavaFX 的低代码代码生成平台

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/projects/jdk/21/)
[![JavaFX](https://img.shields.io/badge/JavaFX-17-blue.svg)](https://openjfx.io/)
[![Maven](https://img.shields.io/badge/Maven-3.8+-red.svg)](https://maven.apache.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 项目简介

OpenLowCode 是一个功能强大的代码自动生成系统，支持从数据库表结构自动生成 Spring Boot + Vue 全栈代码。通过直观的图形界面，开发者可以快速生成规范的代码，大幅提升开发效率。

## 主要特性

### 🎯 多数据库支持
- MySQL
- PostgreSQL
- SQL Server
- Oracle
- SQLite

### 📦 代码生成能力
- **Entity 实体类** - 带完整的字段映射和类型转换
- **Mapper 接口** - 支持多种数据库的 Mapper 映射文件
- **Service 层** - 业务逻辑接口和实现类
- **Controller 层** - RESTful API 控制器
- **Vue 前端** - 完整的前端页面和组件代码
- **单元测试** - 自动生成测试用例代码
- **SqlAssist** - SQL 辅助工具类

### 🔧 高级配置
- 自定义属性配置
- 历史配置管理
- 数据库连接配置
- 模板自定义
- 字段属性精细控制
- 路由配置
- 国际化支持（中/英文）

### 🎨 用户界面
- 基于 JavaFX 的现代化桌面应用
- 直观的表单配置界面
- 实时预览和编辑
- 代码高亮和格式化

## 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Java | 21 | 开发语言 |
| JavaFX | 17.0.2 | 桌面UI框架 |
| Maven | 3.x | 项目构建 |
| FreeMarker | 2.3.31 | 代码模板引擎 |
| Fastjson | 2.0.14 | JSON处理 |
| SLF4J/Log4j | 2.0.3/1.2.17 | 日志框架 |

## 快速开始

### 环境要求
- JDK 21 或更高版本
- Maven 3.8+
- MySQL/PostgreSQL/SQL Server/Oracle/SQLite

### 安装运行

1. 克隆项目
```bash
git clone https://github.com/yourusername/opencode.git
cd opencode
```

2. 编译项目
```bash
mvn clean compile
```

3. 运行程序
```bash
mvn exec:java -Dexec.mainClass="com.bltu.generator.LaunchUI"
```

或直接运行打包后的 JAR：
```bash
mvn clean package
java -jar target/opencode-1.0.1-jar-with-dependencies.jar
```

### 配置数据库

1. 启动程序后，点击"连接配置"按钮
2. 填写数据库连接信息：
   - 数据库类型
   - 主机地址
   - 端口号
   - 数据库名
   - 用户名
   - 密码
3. 测试连接成功后保存

### 生成代码

1. 选择要生成的数据库表
2. 配置生成选项（包名、路径、命名规则等）
3. 设置各层代码的配置项
4. 点击"生成代码"按钮
5. 查看生成的代码结果

## 项目结构

```
opencode/
├── src/
│   ├── main/
│   │   ├── java/com/bltu/generator/
│   │   │   ├── AppMain.java              # 程序入口
│   │   │   ├── LaunchUI.java             # UI启动器
│   │   │   ├── common/                   # 通用工具类
│   │   │   │   ├── DBUtil.java           # 数据库工具
│   │   │   │   ├── ConfigUtil.java       # 配置工具
│   │   │   │   ├── TemplateUtil.java     # 模板工具
│   │   │   │   └── ...
│   │   │   ├── controller/               # UI控制器
│   │   │   │   ├── IndexController.java  # 主界面控制器
│   │   │   │   ├── ConnectionController.java
│   │   │   │   └── ...
│   │   │   ├── entity/                   # 实体类
│   │   │   ├── models/                   # 数据模型
│   │   │   ├── options/                  # 配置选项
│   │   │   └── spi/                      # SPI接口
│   │   └── resources/
│   │       ├── FXML/                     # 界面布局文件
│   │       ├── image/                    # 图片资源
│   │       └── config/                   # 配置文件
├── template/                             # 代码模板
│   ├── Entity.ftl
│   ├── Controller.ftl
│   ├── Service.ftl
│   ├── Mapper.ftl
│   ├── Vue.ftl
│   └── ...
├── config/                               # 本地配置
├── logs/                                 # 日志文件
├── pom.xml                               # Maven配置
└── README.md                             # 项目说明
```

## 使用指南

### 配置说明

#### Entity 配置
- 基础包名
- 输出路径
- 命名规则（驼峰/下划线）
- 字段注解配置

#### Mapper 配置
- Mapper 接口包名
- XML 文件路径
- 自动生成的 CRUD 方法

#### Service 配置
- Service 接口包名
- 实现类包名
- 事务管理配置

#### Controller 配置
- 控制器包名
- RESTful 风格配置
- 接口路径前缀

#### Vue 配置
- 前端组件路径
- API 接口地址
- 页面模板配置

### 自定义模板

系统使用 FreeMarker 模板引擎，用户可以根据需要修改 `template/` 目录下的模板文件，实现自定义代码生成规则。

### 历史配置

系统支持保存和管理历史配置，可以快速切换不同的生成配置方案。

## 常见问题

**Q: 支持 MyBatis-Plus 吗？**  
A: 目前支持原生 MyBatis，MyBatis-Plus 支持正在开发中。

**Q: 可以生成微服务架构代码吗？**  
A: 当前版本主要针对单体应用，微服务支持可以参考模板进行自定义。

**Q: 如何添加新的数据库支持？**  
A: 在 `DBType.java` 中添加数据库类型配置，并实现对应的 Mapper 模板即可。

## 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 联系方式

- 项目主页：http://www.bltu.net
- 问题反馈：[GitHub Issues](https://github.com/yourusername/opencode/issues)

## 更新日志

### v1.0.1 (2026-03-30)
- 初始版本发布
- 支持多种数据库代码生成
- JavaFX 桌面界面
- 完整的配置管理功能

---

Made with ❤️ by BLTU Team
