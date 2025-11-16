# Flutter XBoard SDK

<div align="center">

一个功能完善、类型安全的 Flutter SDK，用于轻松集成 XBoard API。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Dart](https://img.shields.io/badge/Dart->=3.1.0-0175C2?logo=dart)](https://dart.dev)
[![Flutter](https://img.shields.io/badge/Flutter-Compatible-02569B?logo=flutter)](https://flutter.dev)

[功能特性](#-功能特性) • [快速开始](#-快速开始) • [使用指南](#-使用指南) • [API 文档](#-api-文档)

</div>

---

## ✨ 功能特性

### 🔐 认证系统
- **登录/注册** - 完整的用户认证流程
- **邮箱验证** - 邮箱验证码发送与验证
- **密码管理** - 密码重置与修改
- **Token 管理** - 自动 Token 刷新与持久化

### 📱 核心功能
- **用户管理** - 用户信息查询与更新
- **订阅管理** - 订阅链接获取、重置、统计
- **套餐管理** - 套餐列表、详情、购买
- **订单系统** - 订单创建、查询、支付

### 💰 财务系统
- **余额管理** - 余额查询、充值、消费
- **佣金系统** - 佣金转账、提现申请
- **优惠券** - 优惠券验证与使用
- **支付集成** - 多种支付方式支持

### 🎫 增值功能
- **工单系统** - 工单提交、回复、查询
- **通知中心** - 系统通知获取与标记
- **邀请系统** - 邀请码生成与管理
- **应用配置** - 获取应用全局配置

### 🛡️ 技术特性
- ✅ **类型安全** - 完整的 Dart 类型定义
- ✅ **异常处理** - 完善的错误处理机制
- ✅ **自动重试** - 网络请求失败自动重试
- ✅ **缓存支持** - 智能数据缓存策略
- ✅ **Token 持久化** - 支持内存和本地存储
- ✅ **测试覆盖** - 单元测试与集成测试

---

## 🚀 快速开始

### 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter_xboard_sdk:
    git:
      url: https://github.com/hakimi-x/flutter_xboard_sdk.git
      ref: main
```

或者使用本地路径：

```yaml
dependencies:
  flutter_xboard_sdk:
    path: ./path/to/flutter_xboard_sdk
```

### 初始化

```dart
import 'package:flutter_xboard_sdk/flutter_xboard_sdk.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 初始化 SDK
  final sdk = XBoardSDK();
  await sdk.initialize(
    baseUrl: 'https://your-xboard-domain.com',
    tokenConfig: TokenStorageConfig(
      storageType: TokenStorageType.sharedPreferences,
    ),
  );
  
  runApp(MyApp());
}
```

---

## 📖 使用指南

### 认证功能

#### 用户登录

```dart
try {
  final response = await LoginApi.login(
    email: 'user@example.com',
    password: 'password123',
  );
  
  if (response.data != null) {
    // 登录成功，Token 会自动保存
    final user = response.data!.user;
    print('欢迎，${user.email}！');
  }
} on AuthException catch (e) {
  print('登录失败: ${e.message}');
}
```

#### 用户注册

```dart
// 1. 发送验证码
await SendEmailCodeApi.sendEmailCode(
  email: 'newuser@example.com',
);

// 2. 注册账号
final response = await RegisterApi.register(
  email: 'newuser@example.com',
  password: 'password123',
  emailCode: '123456',
  inviteCode: 'INVITE2024', // 可选
);
```

#### 密码重置

```dart
// 1. 发送验证码
await SendEmailCodeApi.sendEmailCode(
  email: 'user@example.com',
);

// 2. 重置密码
await ResetPasswordApi.resetPassword(
  email: 'user@example.com',
  password: 'newPassword123',
  emailCode: '123456',
);
```

### 用户与订阅

#### 获取用户信息

```dart
final response = await UserInfoApi.getUserInfo();
if (response.data != null) {
  final user = response.data!;
  print('邮箱: ${user.email}');
  print('余额: ${user.balance / 100} 元');
  print('佣金: ${user.commissionBalance / 100} 元');
  print('过期时间: ${user.expiredAt}');
}
```

#### 订阅管理

```dart
// 获取订阅信息
final subInfo = await SubscriptionApi.getSubscription();
print('订阅链接: ${subInfo.data?.subscribeUrl}');

// 重置订阅
await SubscriptionApi.resetSubscription();
```

### 套餐与订单

#### 获取套餐列表

```dart
final response = await PlanApi.getPlans();
if (response.data != null) {
  for (final plan in response.data!) {
    print('${plan.name}: ${plan.price / 100} 元/月');
    print('流量: ${plan.transferEnable} GB');
  }
}
```

#### 创建订单

```dart
final response = await OrderApi.createOrder(
  planId: 1,
  period: 'month_price', // 月付
  couponCode: 'SAVE20', // 可选优惠码
);

if (response.data != null) {
  final order = response.data!;
  print('订单号: ${order.tradeNo}');
  print('应付金额: ${order.totalAmount / 100} 元');
}
```

### 财务管理

#### 佣金转账

```dart
final response = await BalanceApi.transferCommission(
  transferAmount: 1000, // 10.00 元（单位：分）
);

if (response.data != null) {
  print('转账成功！');
  print('当前余额: ${response.data!.balance / 100} 元');
}
```

#### 申请提现

```dart
// 1. 获取系统配置
final config = await ConfigApi.getConfig();
if (config.data?.withdrawClose == true) {
  print('提现功能已关闭');
  return;
}

// 2. 申请提现
await BalanceApi.withdraw(
  withdrawMethod: 'alipay',
  withdrawAccount: 'your_account@example.com',
);
```

#### 优惠券验证

```dart
final response = await CouponApi.check(
  code: 'SAVE20',
  planId: 1,
);

if (response.data != null) {
  final coupon = response.data!;
  if (coupon.type == 1) {
    print('减免: ${coupon.value / 100} 元');
  } else if (coupon.type == 2) {
    print('折扣: ${coupon.value}%');
  }
}
```

### 工单系统

```dart
// 创建工单
await TicketApi.createTicket(
  subject: '账号问题咨询',
  level: 1, // 优先级
  message: '无法登录账号，请帮忙处理',
);

// 获取工单列表
final response = await TicketApi.getTickets();

// 回复工单
await TicketApi.replyTicket(
  ticketId: 1,
  message: '问题已解决，谢谢！',
);

// 关闭工单
await TicketApi.closeTicket(ticketId: 1);
```

### 通知系统

```dart
// 获取通知列表
final response = await NoticeApi.getNotices();
if (response.data != null) {
  for (final notice in response.data!) {
    print('[${notice.title}] ${notice.content}');
  }
}
```

### 邀请系统

```dart
// 获取邀请信息
final response = await InviteApi.getInviteInfo();
if (response.data != null) {
  final invite = response.data!;
  print('邀请码: ${invite.codes}');
  print('已邀请: ${invite.stat?.length ?? 0} 人');
  print('佣金收入: ${invite.commission} 元');
}
```

---

## 🔧 API 文档

### 核心类

#### XBoardSDK

SDK 主入口，负责初始化和配置。

```dart
class XBoardSDK {
  Future<void> initialize({
    required String baseUrl,
    TokenStorageConfig? tokenConfig,
  });
  
  TokenManager get tokenManager;
  HttpService get httpService;
}
```

#### TokenManager

Token 管理器，处理 Token 的存储、获取和刷新。

```dart
class TokenManager {
  Future<void> saveToken(String token);
  Future<String?> getToken();
  Future<void> deleteToken();
  Future<void> refreshToken();
}
```

### API 模块

| 模块 | 类名 | 功能 |
|------|------|------|
| 🔐 认证 | `LoginApi` | 登录 |
| 📝 注册 | `RegisterApi` | 用户注册 |
| 📧 验证码 | `SendEmailCodeApi` | 发送邮箱验证码 |
| 🔑 密码 | `ResetPasswordApi` | 重置密码 |
| 👤 用户 | `UserInfoApi` | 用户信息 |
| 📱 订阅 | `SubscriptionApi` | 订阅管理 |
| 📦 套餐 | `PlanApi` | 套餐管理 |
| 🛒 订单 | `OrderApi` | 订单管理 |
| 💳 支付 | `PaymentApi` | 支付处理 |
| 💰 余额 | `BalanceApi` | 余额/佣金管理 |
| 🎫 优惠券 | `CouponApi` | 优惠券管理 |
| 📮 工单 | `TicketApi` | 工单系统 |
| 🔔 通知 | `NoticeApi` | 通知中心 |
| 👥 邀请 | `InviteApi` | 邀请系统 |
| ⚙️ 配置 | `ConfigApi` | 应用配置 |
| 📱 应用 | `AppApi` | 应用信息 |

### 异常处理

SDK 提供了完善的异常体系：

```dart
try {
  await LoginApi.login(email: email, password: password);
} on AuthException catch (e) {
  // 认证错误（401, 403）
  print('认证失败: ${e.message}');
} on ValidationException catch (e) {
  // 参数验证错误（422）
  print('参数错误: ${e.message}');
} on NetworkException catch (e) {
  // 网络错误
  print('网络异常: ${e.message}');
} on ServerException catch (e) {
  // 服务器错误（500+）
  print('服务器错误: ${e.message}');
} on XBoardException catch (e) {
  // 其他 SDK 异常
  print('SDK 错误: ${e.message}');
}
```

### 数据模型

所有 API 响应都使用类型安全的数据模型：

```dart
class ApiResponse<T> {
  final bool success;
  final String? message;
  final T? data;
  
  bool get isSuccess => success && data != null;
}
```

主要模型包括：
- `LoginResponse` - 登录响应
- `UserInfo` - 用户信息
- `SubscriptionInfo` - 订阅信息
- `PlanInfo` - 套餐信息
- `OrderInfo` - 订单信息
- `NoticeInfo` - 通知信息
- `TicketInfo` - 工单信息
- `InviteInfo` - 邀请信息

---

## 🧪 测试

### 运行单元测试

```bash
flutter test
```

### 运行集成测试

```bash
# 设置环境变量
export XBOARD_BASE_URL="https://your-xboard-domain.com"
export XBOARD_TEST_EMAIL="test@example.com"
export XBOARD_TEST_PASSWORD="password123"

# 运行测试
flutter test test/integration_test.dart
```

---

## 🔧 关于对接“旧版”Xboard

新版Xboard在HTTP头中，使用标准的"authorization: Bearer $token"格式。
旧版Xboard缺少Bearer字符串，而是使用"authorization: $token"格式。有"Bearer"
存在时调用会认证失败，返回403错误。因此在对接旧版Xboard时需要去掉Bearer
字符串。

这个操作可以在nginx反向代理的配置中实现。例如，

``` nginx
location /api {
    set $auth_header "";
    # Check if the Authorization header exists and starts with "Bearer"
    if ($http_authorization ~* "^Bearer\s+(.+)") {
        set $auth_header $1;
    }
    # Set the modified Authorization header without "Bearer"
    proxy_set_header Authorization $auth_header;

    proxy_pass         http://127.0.0.1:7001/api;
}
```

---

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

---

## 📞 支持

遇到问题？欢迎：

- 📫 提交 [Issue](https://github.com/hakimi-x/flutter_xboard_sdk/issues)
- 💬 参与 [讨论](https://github.com/hakimi-x/flutter_xboard_sdk/discussions)

---

<div align="center">

**[⬆ 回到顶部](#flutter-xboard-sdk)**

Made with ❤️ by Hakimi-X

</div>
