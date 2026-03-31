# Firebase 配置指南

## 第一步：创建 Firebase 项目

1. 前往 [Firebase Console](https://console.firebase.google.com)
2. 点击「Add project」→ 输入项目名称 → 完成创建

## 第二步：开启各项服务

### Authentication（登录）
- 左侧菜单 → Build → **Authentication** → Get started
- Sign-in method → 启用 **Email/Password**

### Firestore（数据库）
- 左侧菜单 → Build → **Firestore Database** → Create database
- 选择 **Start in test mode**（开发阶段）
- 选择就近的服务器地区

### Storage（图片存储）
- 左侧菜单 → Build → **Storage** → Get started
- 选择 **Start in test mode**
- 选择就近的服务器地区

## 第三步：注册 Web 应用并获取配置

1. 项目概览 → 点击「</>」(Web) 图标
2. 输入应用昵称 → Register app
3. 复制 `firebaseConfig` 对象

## 第四步：填入配置

打开 `index.html`，找到以下代码段（约第 237 行）：

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

替换为你的真实配置值。

## 第五步：Firestore 安全规则（生产前必改）

在 Firestore → Rules 中设置：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 用户资料：本人可读写，他人只读
    match /users/{userId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // 帖子：已登录用户可发布，本人可删除
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow delete: if request.auth != null && request.auth.uid == resource.data.uid;
    }

    // 神秘文案：全员可读，本人可增删改
    match /mysteries/{mysteryId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null && request.auth.uid == resource.data.uid;
    }
  }
}
```

## 第六步：Storage 安全规则

在 Storage → Rules 中设置：

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /avatars/{userId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    match /posts/{userId}/{fileName} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## 功能说明

| 功能 | 说明 |
|------|------|
| 注册/登录 | 邮箱+密码，支持错误提示 |
| 头像上传 | 点击头像区域直接更换，存到 Firebase Storage |
| 个人资料 | 昵称、简介，可编辑保存 |
| 发帖 | 支持文字+图片，实时刷新 |
| 删除帖子 | 只有本人可见删除按钮 |
| 神秘文案 | 全用户文案汇入同一随机池，首页随机展示 |
| 编辑文案 | 点击编辑直接行内修改 |
