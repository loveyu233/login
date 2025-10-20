# 微信小程序登录后端服务

本项目是一个使用 Go 语言编写的微信小程序后端服务，基于 [Gin](https://github.com/gin-gonic/gin) 框架和 [PowerWeChat](https://github.com/ArtisanCloud/PowerWeChat) SDK。

主要实现了微信小程序用户的登录流程、以及小程序码的生成功能。

## 主要功能

-   **小程序用户登录**：通过 `code` 获取用户的 `OpenID` 和 `UnionID`，并支持新用户自动注册。
-   **获取用户手机号**：在用户授权的情况下，解密并获取用户手机号。
-   **生成小程序码**：提供生成不同样式和尺寸的小程序码和二维码的接口。
-   **可扩展的用户系统**：通过接口（`WXMiniImp`）将用户查询、创建和 Token 生成的逻辑解耦，方便接入现有用户系统。

## 技术栈

-   Go
-   Gin: 高性能的 Go Web 框架。
-   PowerWeChat: 微信 SDK for Go。
-   Resty: Go HTTP 客户端。

## API 接口

### `POST /wx/login`

**功能**: 小程序用户登录。

**请求体 (JSON)**:

```json
{
    "code": "JSCODE",
    "encrypted_data": "ENCRYPTED_DATA",
    "iv_str": "IV"
}
```

-   `code`: 必需，通过 `wx.login()` 获取。
-   `encrypted_data`: 可选，用于获取手机号，通过 `e.detail.encryptedData` 获取。
-   `iv_str`: 可选，用于获取手机号，通过 `e.detail.iv` 获取。

**成功响应**:

-   **首次登录（未注册且未提供手机号信息）**:
    ```json
    {
        "code": 200,
        "data": {
            "open_id": "USER_OPEN_ID"
        },
        "message": "success"
    }
    ```
-   **登录/注册成功**:
    ```json
    {
        "code": 200,
        "data": {
            // 由 GenerateToken 实现返回的具体内容
            "token": "USER_TOKEN",
            "userInfo": { ... }
        },
        "message": "success"
    }
    ```

## 如何运行

1.  **克隆项目**

    ```bash
    git clone <your-repository-url>
    cd login
    ```

2.  **安装依赖**

    项目使用 Go Modules 管理依赖。

    ```bash
    go mod tidy
    ```

3.  **配置**

    在代码中，你需要初始化 `MiniProgramServiceConfig` 结构体，并填入你的小程序 `AppID` 和 `Secret` 等信息。

    ```go
    // 示例: main.go
    package main

    import (
        "github.com/gin-gonic/gin"
        "your/project/path/login"
    )

    // 实现 WXMiniImp 接口
    type MyUserSystem struct{}

    func (m *MyUserSystem) IsExistsUser(unionID string) (user any, exists bool, err error) {
        // ... 你的用户查询逻辑
    }

    func (m *MyUserSystem) CreateUser(phoneNumber, unionID, openID, areaCodeByIP, clientIP string) (user any, err error) {
        // ... 你的用户创建逻辑
    }

    func (m *MyUserSystem) GenerateToken(user any, sessionKey string) (data any, err error) {
        // ... 你的 Token 生成逻辑
    }


    func main() {
        r := gin.Default()

        // 配置小程序服务
        config := login.MiniProgramServiceConfig{
            MiniProgram: login.MiniProgramConfig{
                AppID:  "YOUR_APP_ID",
                Secret: "YOUR_SECRET",
                // ... 其他配置
            },
            WXMiniImp: &MyUserSystem{},
        }

        // 初始化服务
        wxMiniService, err := login.InitWXMiniProgramService(config)
        if err != nil {
            panic(err)
        }

        // 注册路由
        apiGroup := r.Group("/api")
        wxMiniService.RegisterHandlers(apiGroup)

        r.Run(":8080")
    }
    ```

4.  **运行项目**

    ```bash
    go run .
    ```

    服务将启动在 `http://localhost:8080`。

## 项目结构

-   `go.mod`, `go.sum`: Go 模块依赖文件。
-   `miniprogram.go`: 定义了小程序服务的主要结构体、接口和初始化逻辑。
-   `miniprogram_server.go`: 实现了 Gin 框架的路由和 Handler，处理 HTTP 请求。
