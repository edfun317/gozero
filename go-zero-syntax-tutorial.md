# Go-Zero 語法教學

## 前言

go-zero 框架引入自定義語法主要是為了：
1. 簡化微服務開發流程
2. 提高程式碼生成效率
3. 統一 API 定義標準
4. 減少重複程式碼編寫

這些語法雖然需要一定學習成本，但能顯著提升開發效率。本教程將幫助你快速掌握這些語法規則。

## 目錄
- [1. API 語法](#1-api-語法)
- [2. Model 語法](#2-model-語法)
- [3. 實際應用示例](#3-實際應用示例)

## 1. API 語法

### 1.1 基本結構
```api
syntax = "v1"  // 語法版本聲明

info (          // 服務資訊
    title:   "類型描述"
    desc:    "詳細描述"
    author:  "作者"
    email:   "郵箱"
)

type (         // 類型定義
    Request {
        Name string `json:"name"`
        Age  int    `json:"age"`
    }

    Response {
        Message string `json:"message"`
    }
)

service user-api {    // 服務定義
    @handler UserHandler      // 處理器註解
    get /users/:name (Request) returns (Response)  // API 路由
}
```

### 1.2 類型定義語法
```api
type User {
    Id       int64  `json:"id"`
    Username string `json:"username"`
    Password string `json:"password"`
    // 支援 json 標籤定義
}

// 可以引用其他類型
type OrderInfo {
    OrderId  string `json:"orderId"`
    UserInfo User   `json:"userInfo"`
}
```

### 1.3 路由語法
```api
service user-api {
    @doc "使用者登入"
    @handler login
    post /user/login (LoginRequest) returns (LoginResponse)

    @doc "獲取使用者資訊"
    @handler getUser
    get /user/:id returns (UserResponse)
}
```

## 2. Model 語法

### 2.1 資料庫表定義
```sql
CREATE TABLE `user` (
    `id` bigint NOT NULL AUTO_INCREMENT,
    `username` varchar(255) NOT NULL DEFAULT '' COMMENT '使用者名稱',
    `password` varchar(255) NOT NULL DEFAULT '' COMMENT '密碼',
    `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 2.2 生成 Model 程式碼
使用 goctl 命令：
```bash
goctl model mysql ddl -src="./sql/user.sql" -dir="./internal/model" -c
```

## 3. 實際應用示例

### 3.1 完整的 API 服務示例
```api
syntax = "v1"

info (
    title: "使用者服務"
    desc: "使用者管理系統 API"
    author: "開發者"
    email: "dev@example.com"
)

type (
    RegisterRequest {
        Username string `json:"username"`
        Password string `json:"password"`
        Email    string `json:"email,optional"`
    }

    RegisterResponse {
        Id       int64  `json:"id"`
        Username string `json:"username"`
        Token    string `json:"token"`
    }

    LoginRequest {
        Username string `json:"username"`
        Password string `json:"password"`
    }

    LoginResponse {
        Token string `json:"token"`
    }
)

service user-api {
    @doc(
        summary: "使用者註冊"
    )
    @handler Register
    post /api/user/register (RegisterRequest) returns (RegisterResponse)

    @doc(
        summary: "使用者登入"
    )
    @handler Login
    post /api/user/login (LoginRequest) returns (LoginResponse)
}
```

### 3.2 特殊語法說明

#### 可選欄位
使用 `optional` 標籤：
```api
type Request {
    Required string `json:"required"`           // 必填欄位
    Optional string `json:"optional,optional"`  // 可選欄位
}
```

#### 陣列類型
```api
type Response {
    List []string `json:"list"`    // 字串陣列
    Users []User  `json:"users"`   // 物件陣列
}
```

#### 中間件註解
```api
@server(
    jwt: Auth
    middleware: Logging
)
service user-api {
    @handler GetProfile
    get /api/user/profile returns (ProfileResponse)
}
```

## 注意事項

1. 語法版本聲明必須在文件開頭
2. 類型名稱必須以大寫字母開頭
3. 欄位名稱建議使用駝峰命名
4. 路由路徑必須以 "/" 開頭
5. handler 名稱必須唯一

## 常見錯誤與解決方案

1. 語法錯誤：
   - 檢查類型定義是否正確
   - 確保所有括號都正確配對
   - 驗證路由格式是否正確

2. 類型未定義：
   - 確保所有使用的類型都已在 type 塊中定義
   - 檢查類型名稱的拼寫是否正確

3. 重複定義：
   - 確保 handler 名稱不重複
   - 確保路由路徑不重複
   - 確保類型名稱不重複

## 最佳實踐

1. 按功能模組組織 API 文件
2. 使用清晰的命名約定
3. 添加適當的註釋和文檔
4. 合理使用中間件註解
5. 遵循 RESTful API 設計原則
