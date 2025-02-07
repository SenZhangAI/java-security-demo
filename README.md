# Spring Boot Security 实现方案

## 项目概述

本项目实现了基于 Spring Boot Security 的用户认证和授权系统，包含以下功能：

-   用户注册和登录
-   基于 JWT 的 token 认证
-   角色基础权限控制
-   自定义认证过滤器
-   密码加密存储

## 实现步骤

1. 添加依赖配置 ✅

    - spring-boot-starter-security
    - spring-boot-starter-web
    - jjwt (JWT 支持)
    - spring-boot-starter-data-jpa

2. 创建基础实体类 ✅

    - User 实体
    - Role 实体
    - UserRole 关联

3. 实现核心安全配置 ✅

    - SecurityConfig 配置类
    - JwtTokenProvider 工具类
    - CustomUserDetailsService 实现

4. 创建认证控制器 🚧

    - AuthController 处理注册登录 ✅
    - UserController 测试权限控制 ✅

5. 实现 JWT 过滤器 ✅

    - JwtAuthenticationFilter
    - 集成到 Security 配置中

6. 数据库配置 ✅
    - application.yml 配置文件
    - 初始化 SQL 脚本

## 测试用例

1. 认证测试

    - 用户注册测试
    - 用户登录测试
    - Token 生成测试

2. 权限测试
    - 公共接口测试
    - 用户接口测试
    - 管理员接口测试

# Spring Security Demo

基于 Spring Boot 和 JWT 的认证授权系统示例。

## 功能特性

-   用户注册和登录
-   JWT token 认证
-   基于角色的权限控制
-   Swagger API 文档
-   全局异常处理

## 技术栈

-   Spring Boot 2.7.0
-   Spring Security
-   MySQL 8.0
-   JWT
-   Swagger/OpenAPI

## 快速开始

### 环境要求

-   JDK 8+
-   Maven 3.6+
-   Docker & Docker Compose

### 启动步骤

1. 启动 MySQL 数据库

```bash
docker-compose up -d
```

2. 等待 MySQL 完全启动（约 30 秒），可以通过以下命令查看状态：

```bash
docker-compose ps
```

3. 数据库初始化

-   系统会自动创建数据表（JPA auto-ddl）
-   初始角色数据会通过 data.sql 自动导入

4. 启动应用

```bash
mvn spring-boot:run
```

### API 文档

启动应用后，访问 Swagger 文档：

```
http://localhost:8080/swagger-ui.html
```

### 测试接口

1. 获取验证码

```bash
# 获取验证码并保存 cookie（重要：保持会话）
curl -X GET http://localhost:8080/api/captcha -c cookies.txt

# 返回示例：
{
    "success": true,
    "message": "获取验证码成功",
    "data": "data:image/png;base64,/9j/4AAQSkZJRg..."
}

# 查看验证码方法：
# 方法1：使用浏览器
# 1. 复制 data 字段的完整值（包含 data:image/png;base64, 前缀）
# 2. 在浏览器新标签页粘贴地址栏即可看到验证码图片

# 方法2：使用命令行（Linux/Mac）
echo "返回的 base64 内容" | base64 -d > captcha.png
```

2. 注册新用户

```bash
# 注册新用户（密码需要符合强度要求）
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "username":"testuser",
    "email":"test@example.com",
    "password":"Password@123"
  }'

# 密码要求：
# - 至少8个字符
# - 至少1个数字
# - 至少1个小写字母
# - 至少1个大写字母
# - 至少1个特殊字符(@#$%^&+=)
# - 不能包含空格
```

3. 用户登录（需要验证码）

```bash
# 1. 先获取新的验证码（使用上面的验证码接口）
curl -X GET http://localhost:8080/api/captcha -c cookies.txt

# 2. 查看验证码内容（使用上述方法之一）

# 3. 使用验证码登录（使用相同的 cookies 文件）
curl -X POST http://localhost:8080/api/auth/login \
  -b cookies.txt \
  -H "Content-Type: application/json" \
  -d '{
    "username":"testuser",
    "password":"Password@123",
    "captcha":"XXXX"  # 替换为实际看到的验证码
  }'

# 登录成功后会返回 JWT token，保存此 token 用于后续请求
```

4. 获取用户信息（需要 token）

```bash
# 使用登录返回的 token
curl -X GET http://localhost:8080/api/auth/user/info \
  -H "Authorization: Bearer {your_token}"
```

### 验证码说明

-   验证码有效期为 5 分钟
-   验证码区分大小写
-   验证码使用后立即失效
-   如果输入错误的验证码，需要重新获取新的验证码
-   必须使用相同的会话（cookie）才能验证成功
-   每次登录尝试都需要重新获取验证码

### 常见问题

1. 验证码错误：

    - 确保使用最新获取的验证码
    - 确保使用了相同的 cookie
    - 验证码区分大小写，注意输入准确

2. 登录失败：
    - 检查用户名和密码是否正确
    - 确认验证码是否在有效期内
    - 确认是否使用了正确的 cookie

## 开发说明

### 数据库配置

数据库连接信息在 application.yml 中配置：

```yaml
spring:
    datasource:
        url: jdbc:mysql://localhost:3307/security_demo?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
        username: root
        password: root
```

### 运行测试(依然未跑通，暂时跳过)

```bash
mvn test
```

## 安全策略优化路线图

### 已完成

-   [x] 基础认证授权功能
-   [x] JWT token 实现
-   [x] SQL 注入防护
    -   [x] 使用 PreparedStatement（通过 JPA 默认支持）
    -   [x] SQL 注入过滤器
    -   [x] 参数化查询配置
-   [x] XSS 攻击防护
    -   [x] 输入数据转义
    -   [x] XSS 过滤器实现
    -   [x] 设置安全响应头（CSP）
-   [x] CSRF 攻击防护
    -   [x] 实现 CSRF token
    -   [x] SameSite Cookie 设置
    -   [x] 验证 Origin/Referer
-   [x] 密码安全
    -   [x] 密码强度校验
    -   [x] 密码加密存储优化
    -   [x] 密码重试次数限制
    -   [x] 定期密码更新策略
-   [x] 会话安全
    -   [x] Session 固定攻击防护
    -   [x] Session 超时处理
    -   [x] 并发会话控制
-   [x] 请求限流
    -   [x] 实现 IP 限流
    -   [x] API 访问频率限制
    -   [x] 验证码机制
-   [x] 敏感数据保护
    -   [x] 数据脱敏（用户名、邮箱、手机号等）
    -   [x] 敏感信息加密存储
    -   [ ] 传输加密
-   [x] 日志安全 ✅
    -   [x] 安全审计日志
    -   实现了 AuditLog 注解和 AuditLogAspect
    -   记录用户操作、IP 地址和时间戳
    -   支持日志脱敏处理
    -   [x] 日志脱敏
    -   自动脱敏密码、银行卡、SSN 等敏感信息
    -   使用正则表达式进行通用脱敏
    -   [x] 日志防篡改
    -   使用 SHA-256 进行日志签名
    -   实现了 SignedLoggingEvent 进行签名验证
    -   支持日志滚动和保留策略

### 敏感数据保护说明

1. 数据脱敏

-   支持多种脱敏策略（用户名、邮箱、手机号、身份证等）
-   使用注解方式 `@Sensitive` 便捷配置
-   只影响数据展示，不影响存储

2. 加密存储

-   使用 AES 算法进行加密
-   自动加密存储敏感字段
-   透明解密，支持查询

3. 安全建议
   当前示例使用了硬编码的加密密钥，在生产环境中应该：

-   使用密钥管理服务（KMS）管理密钥
-   从安全的配置中心获取密钥
-   使用环境变量注入密钥
-   实现密钥轮换机制
-   考虑使用硬件安全模块（HSM）

4. 使用示例

```java
@Entity
public class User {
    @Sensitive(strategy = SensitiveStrategy.EMAIL)
    @Convert(converter = EncryptedAttributeConverter.class)
    private String email;  // 数据库加密存储，API返回时脱敏

    @Sensitive(strategy = SensitiveStrategy.PHONE)
    @Convert(converter = EncryptedAttributeConverter.class)
    private String phone;  // 138****5678
}
```

5. 脱敏效果

-   用户名：张三 -> 张\*三
-   手机号：13812345678 -> 138\*\*\*\*5678
-   邮箱：test@example.com -> t\*\*\*t@example.com
-   身份证：310123199001011234 -> 310123**\*\*\*\***1234
-   银行卡：6222021234567890 -> 6222**\*\*\*\***7890

### 待实现

1. 文件上传安全

    - [ ] 文件类型验证
    - [ ] 文件大小限制
    - [ ] 文件存储安全
    - [ ] 文件名安全处理
    - [ ] 防病毒扫描

2. 其他安全措施
    - [ ] HTTP 安全头配置
        - [ ] X-Frame-Options
        - [ ] X-Content-Type-Options
        - [ ] X-XSS-Protection
    - [ ] 错误信息处理
        - [ ] 自定义错误页面
        - [ ] 生产环境错误处理
    - [ ] 依赖包安全检查
        - [ ] Maven 依赖版本检查
        - [ ] 已知漏洞扫描
    - [ ] 定期安全扫描
        - [ ] 代码质量扫描
        - [ ] 安全漏洞扫描
        - [ ] 渗透测试

### 下一步计划

1. 文件上传模块

    - 实现基础的文件上传功能
    - 添加文件类型白名单
    - 实现文件大小限制
    - 配置安全的存储路径

2. HTTP 安全配置

    - 添加安全响应头
    - 配置 SSL/TLS
    - 实现 CSP 策略

3. 错误处理

    - 自定义错误页面
    - 统一异常处理
    - 敏感信息过滤

4. 安全监控
    - 集成 Spring Actuator
    - 添加安全审计
    - 实现监控告警

每个里程碑完成后，将在此更新进度。
