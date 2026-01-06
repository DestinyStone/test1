# 记账APP开发报告

## （1）绪论

### 1.1 研究背景

随着移动互联网的快速发展和智能手机的普及，移动应用已经成为人们日常生活中不可或缺的一部分。财务管理作为个人生活的重要组成部分，传统的纸质记账方式已经无法满足现代人快节奏生活的需求。因此，开发一款功能完善、界面美观、操作便捷的移动记账应用具有重要的现实意义。

### 1.2 目的与意义

本课题旨在开发一款基于Android平台的记账应用，主要目的包括：

1. **提高财务管理效率**：通过移动应用实现随时随地的记账功能，帮助用户更好地管理个人财务。
2. **数据可视化分析**：通过统计功能，用户可以直观地了解自己的收支情况，做出更合理的财务决策。
3. **技术实践**：通过实际项目开发，深入理解Android开发技术栈，包括Room数据库、Material Design设计规范等。

本应用的意义在于：
- 为用户提供便捷的财务管理工具
- 培养良好的记账习惯
- 通过数据分析帮助用户优化消费结构

### 1.3 研究内容

本课题主要研究内容包括：

1. **用户认证系统**：实现用户注册、登录功能，确保数据安全性和用户隐私。
2. **记账数据管理**：实现记账记录的增、删、改、查功能，支持收入和支出两种类型。
3. **数据统计功能**：实现总收入、总支出和余额的统计计算。
4. **用户界面设计**：采用Material Design设计规范，打造现代化、美观的用户界面。
5. **数据持久化**：使用Room数据库实现数据的本地存储。

### 1.4 开发技术

本应用采用以下技术栈：

- **开发语言**：Java
- **开发平台**：Android Studio
- **最低SDK版本**：API 25 (Android 7.1)
- **目标SDK版本**：API 34 (Android 14)
- **数据库**：Room Persistence Library
- **UI框架**：Material Design Components
- **架构模式**：MVC (Model-View-Controller)
- **数据绑定**：ViewBinding

## （2）总体设计

### 2.1 系统总体结构设计

本记账APP采用模块化设计，主要包含以下模块：

```
┌─────────────────────────────────────┐
│         记账APP系统架构              │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────┐  ┌──────────┐        │
│  │ 表示层   │  │ 业务逻辑层│        │
│  │ (UI)     │  │ (Activity)│       │
│  └────┬─────┘  └────┬─────┘        │
│       │             │              │
│  ┌────▼─────────────▼─────┐        │
│  │      数据访问层 (DAO)    │        │
│  └────┬────────────────────┘        │
│       │                             │
│  ┌────▼────────────────────┐        │
│  │    Room数据库 (SQLite)   │        │
│  └─────────────────────────┘        │
│                                     │
└─────────────────────────────────────┘
```

**系统模块说明：**

1. **表示层（UI层）**
   - SplashActivity：闪屏页面
   - LoginActivity：登录页面
   - RegisterActivity：注册页面
   - MainActivity：主页面（记账列表）
   - AddEditRecordActivity：添加/编辑记录页面

2. **业务逻辑层**
   - 用户认证逻辑
   - 记账记录管理逻辑
   - 数据统计计算逻辑

3. **数据访问层**
   - UserDao：用户数据访问接口
   - AccountRecordDao：记账记录数据访问接口

4. **数据持久层**
   - AppDatabase：Room数据库实例
   - User实体类
   - AccountRecord实体类

### 2.2 类的框架结构设计

#### 2.2.1 实体类（Entity）

```
User
├── id: long (主键，自增)
├── username: String (用户名)
└── password: String (密码)

AccountRecord
├── id: long (主键，自增)
├── userId: long (用户ID，外键)
├── type: String (类型：收入/支出)
├── amount: double (金额)
├── category: String (分类)
├── note: String (备注)
└── date: String (日期)
```

#### 2.2.2 数据访问对象（DAO）

```
UserDao
├── insertUser(User): long
├── login(String, String): User
├── findByUsername(String): User
└── findById(long): User

AccountRecordDao
├── insertRecord(AccountRecord): long
├── updateRecord(AccountRecord): void
├── deleteRecord(AccountRecord): void
├── getAllRecordsByUser(long): List<AccountRecord>
├── getRecordById(long): AccountRecord
├── getTotalIncome(long): Double
└── getTotalExpense(long): Double
```

#### 2.2.3 Activity类

```
SplashActivity
└── onCreate(): void

LoginActivity
├── onCreate(): void
└── login(): void

RegisterActivity
├── onCreate(): void
└── register(): void

MainActivity
├── onCreate(): void
├── setupRecyclerView(): void
├── loadRecords(): void
├── updateStatistics(): void
├── editRecord(AccountRecord): void
└── deleteRecord(AccountRecord): void

AddEditRecordActivity
├── onCreate(): void
├── showDatePicker(): void
├── loadRecord(): void
└── saveRecord(): void
```

#### 2.2.4 适配器类

```
RecordAdapter
├── RecordViewHolder (内部类)
│   └── bind(AccountRecord): void
├── onCreateViewHolder(): RecordViewHolder
├── onBindViewHolder(): void
└── updateRecords(List): void
```

#### 2.2.5 工具类

```
SharedPreferencesHelper
├── saveUser(long, String): void
├── getUserId(): long
├── getUsername(): String
├── clearUser(): void
└── isLoggedIn(): boolean
```

## （3）详细设计

### 3.1 类的详细设计

#### 3.1.1 User实体类

**功能描述**：用户实体类，用于存储用户基本信息。

**主要属性**：
- `id`：用户唯一标识符，主键，自增
- `username`：用户名，唯一
- `password`：用户密码

**主要方法**：
- 标准的getter和setter方法

#### 3.1.2 AccountRecord实体类

**功能描述**：记账记录实体类，用于存储用户的记账信息。

**主要属性**：
- `id`：记录唯一标识符，主键，自增
- `userId`：关联的用户ID，外键
- `type`：记录类型（"收入"或"支出"）
- `amount`：金额
- `category`：分类
- `note`：备注信息
- `date`：日期（格式：yyyy-MM-dd）

**主要方法**：
- 标准的getter和setter方法

#### 3.1.3 MainActivity类

**功能描述**：主页面Activity，负责显示记账记录列表和统计信息。

**主要方法**：

1. **onCreate(Bundle)**
   - 初始化界面
   - 设置RecyclerView
   - 加载数据

2. **setupRecyclerView()**
   - 创建RecordAdapter实例
   - 设置RecyclerView的布局管理器
   - 绑定适配器

3. **loadRecords()**
   - 从数据库查询当前用户的所有记录
   - 更新RecyclerView显示
   - 更新统计信息

4. **updateStatistics()**
   - 计算总收入
   - 计算总支出
   - 计算余额（收入-支出）
   - 更新UI显示

5. **editRecord(AccountRecord)**
   - 启动AddEditRecordActivity
   - 传递记录ID用于编辑

6. **deleteRecord(AccountRecord)**
   - 显示确认对话框
   - 删除数据库中的记录
   - 刷新列表

#### 3.1.4 AddEditRecordActivity类

**功能描述**：添加和编辑记账记录的Activity。

**主要方法**：

1. **onCreate(Bundle)**
   - 初始化界面
   - 判断是新增还是编辑模式
   - 如果是编辑模式，加载现有数据

2. **showDatePicker()**
   - 显示日期选择对话框
   - 更新日期输入框

3. **loadRecord()**
   - 根据记录ID从数据库加载数据
   - 填充到表单中

4. **saveRecord()**
   - 验证输入数据
   - 创建或更新记录
   - 保存到数据库
   - 返回主页面

### 3.2 重要成员函数的流程图

#### 3.2.1 用户登录流程

```
开始
  │
  ▼
输入用户名和密码
  │
  ▼
验证输入是否为空
  │
  ├─是─→ 显示错误提示 ─→ 结束
  │
  └─否
      │
      ▼
查询数据库验证用户
  │
  ├─验证失败─→ 显示登录失败 ─→ 结束
  │
  └─验证成功
      │
      ▼
保存用户信息到SharedPreferences
  │
  ▼
跳转到主页面
  │
  ▼
结束
```

#### 3.2.2 添加记账记录流程

```
开始
  │
  ▼
用户填写表单（金额、类型、分类、备注、日期）
  │
  ▼
点击保存按钮
  │
  ▼
验证输入数据
  │
  ├─验证失败─→ 显示错误提示 ─→ 结束
  │
  └─验证成功
      │
      ▼
创建AccountRecord对象
  │
  ▼
插入数据库
  │
  ├─插入失败─→ 显示错误提示 ─→ 结束
  │
  └─插入成功
      │
      ▼
显示成功提示
  │
  ▼
返回主页面
  │
  ▼
结束
```

#### 3.2.3 删除记账记录流程

```
开始
  │
  ▼
用户点击删除按钮
  │
  ▼
显示确认对话框
  │
  ├─取消─→ 结束
  │
  └─确认
      │
      ▼
从数据库删除记录
  │
  ▼
显示删除成功提示
  │
  ▼
刷新记录列表
  │
  ▼
更新统计信息
  │
  ▼
结束
```

#### 3.2.4 数据统计计算流程

```
开始
  │
  ▼
获取当前用户ID
  │
  ▼
查询总收入（type='收入'的记录金额总和）
  │
  ▼
查询总支出（type='支出'的记录金额总和）
  │
  ▼
计算余额 = 总收入 - 总支出
  │
  ▼
更新UI显示
  │
  ▼
结束
```

## （4）实现与结果分析

### 4.1 重要代码实现

#### 4.1.1 数据库配置

```java
@Database(entities = {User.class, AccountRecord.class}, version = 1, exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    private static AppDatabase instance;
    
    public abstract UserDao userDao();
    public abstract AccountRecordDao accountRecordDao();
    
    public static synchronized AppDatabase getInstance(Context context) {
        if (instance == null) {
            instance = Room.databaseBuilder(
                context.getApplicationContext(),
                AppDatabase.class,
                "accounting_database"
            ).allowMainThreadQueries()
             .build();
        }
        return instance;
    }
}
```

**代码分析**：使用单例模式创建数据库实例，确保整个应用只有一个数据库连接。使用Room框架简化了数据库操作。

#### 4.1.2 用户登录实现

```java
private void login() {
    String username = binding.etUsername.getText().toString().trim();
    String password = binding.etPassword.getText().toString().trim();
    
    if (username.isEmpty()) {
        Toast.makeText(this, getString(R.string.username_empty), Toast.LENGTH_SHORT).show();
        return;
    }
    
    if (password.isEmpty()) {
        Toast.makeText(this, getString(R.string.password_empty), Toast.LENGTH_SHORT).show();
        return;
    }
    
    User user = database.userDao().login(username, password);
    
    if (user != null) {
        prefsHelper.saveUser(user.getId(), user.getUsername());
        Toast.makeText(this, "登录成功", Toast.LENGTH_SHORT).show();
        
        Intent intent = new Intent(LoginActivity.this, MainActivity.class);
        startActivity(intent);
        finish();
    } else {
        Toast.makeText(this, getString(R.string.login_failed), Toast.LENGTH_SHORT).show();
    }
}
```

**代码分析**：实现了基本的输入验证和用户认证逻辑，登录成功后保存用户信息并跳转到主页面。

#### 4.1.3 记账记录查询实现

```java
@Query("SELECT * FROM account_records WHERE userId = :userId ORDER BY date DESC, id DESC")
List<AccountRecord> getAllRecordsByUser(long userId);
```

**代码分析**：使用Room的@Query注解实现SQL查询，按日期和ID降序排列，确保最新记录显示在最前面。

#### 4.1.4 统计信息计算实现

```java
private void updateStatistics() {
    Double totalIncome = recordDao.getTotalIncome(userId);
    Double totalExpense = recordDao.getTotalExpense(userId);
    
    double income = totalIncome != null ? totalIncome : 0.0;
    double expense = totalExpense != null ? totalExpense : 0.0;
    double balance = income - expense;
    
    binding.tvTotalIncome.setText(String.format("¥%.2f", income));
    binding.tvTotalExpense.setText(String.format("¥%.2f", expense));
    binding.tvBalance.setText(String.format("¥%.2f", balance));
}
```

**代码分析**：通过SQL的SUM聚合函数计算总收入和总支出，然后计算余额并格式化显示。

### 4.2 程序结果图

#### 4.2.1 闪屏页面

闪屏页面采用简洁设计，显示应用名称和图标，2秒后自动跳转到登录页面或主页面（如果已登录）。

**界面特点**：
- 使用清淡的蓝色主题
- 居中显示应用图标和名称
- 简洁优雅的视觉效果

#### 4.2.2 登录注册页面

登录页面包含用户名和密码输入框，以及登录和注册按钮。注册页面包含用户名、密码和确认密码输入框。

**界面特点**：
- Material Design风格的输入框
- 清晰的按钮布局
- 友好的错误提示

#### 4.2.3 主页面

主页面包含以下部分：

1. **统计卡片**：
   - 总收入：显示所有收入记录的总和，使用绿色主题
   - 总支出：显示所有支出记录的总和，使用红色主题
   - 余额：总收入减去总支出，使用蓝色主题

2. **记账记录列表**：
   - 使用RecyclerView展示记录
   - 每条记录显示：分类、备注、日期、金额、类型
   - 提供编辑和删除按钮
   - 收入和支出使用不同颜色区分

3. **浮动添加按钮**：
   - 右下角浮动按钮
   - 点击后跳转到添加记录页面

**界面特点**：
- 卡片式设计，层次分明
- 颜色区分收入和支出
- 现代化的Material Design风格

#### 4.2.4 添加/编辑记录页面

包含以下输入项：
- 金额输入框（数字类型）
- 类型选择（收入/支出，单选按钮）
- 分类输入框
- 备注输入框（多行文本）
- 日期选择器

**界面特点**：
- 清晰的表单布局
- 日期选择器方便用户操作
- 输入验证确保数据完整性

### 4.3 结果分析

#### 4.3.1 功能实现分析

1. **用户认证功能**：
   - ✅ 成功实现用户注册功能
   - ✅ 成功实现用户登录功能
   - ✅ 使用SharedPreferences保存登录状态
   - ✅ 实现了基本的输入验证

2. **记账数据管理功能**：
   - ✅ 成功实现记录的增、删、改、查操作
   - ✅ 支持收入和支出两种类型
   - ✅ 数据持久化到本地数据库
   - ✅ 实现了数据关联（用户和记录的一对多关系）

3. **数据统计功能**：
   - ✅ 成功计算总收入
   - ✅ 成功计算总支出
   - ✅ 成功计算余额
   - ✅ 实时更新统计信息

4. **用户界面**：
   - ✅ 采用Material Design设计规范
   - ✅ 使用清淡典雅的主题色
   - ✅ 界面美观、现代化
   - ✅ 用户体验良好

#### 4.3.2 技术实现分析

1. **数据库设计**：
   - 使用Room框架简化了数据库操作
   - 实体类设计合理，支持外键关联
   - DAO接口清晰，便于维护

2. **架构设计**：
   - 采用MVC架构模式
   - 代码结构清晰，职责分明
   - 便于扩展和维护

3. **UI设计**：
   - 使用ViewBinding简化视图绑定
   - Material Components提供一致的用户体验
   - 响应式布局适配不同屏幕尺寸

## （5）测试

### 5.1 测试方法简介

本应用采用以下测试方法：

1. **功能测试**：测试各个功能模块是否按照需求正常工作
2. **界面测试**：测试用户界面的显示效果和交互体验
3. **数据测试**：测试数据的增删改查操作是否正确
4. **边界测试**：测试极端情况和异常输入的处理

### 5.2 测试用例分析

#### 5.2.1 用户注册测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC001 | 正常注册 | 用户名：test，密码：123456，确认密码：123456 | 注册成功，跳转到登录页面 | ✅ 通过 |
| TC002 | 用户名为空 | 用户名：空，密码：123456 | 提示"用户名不能为空" | ✅ 通过 |
| TC003 | 密码为空 | 用户名：test，密码：空 | 提示"密码不能为空" | ✅ 通过 |
| TC004 | 密码不一致 | 用户名：test，密码：123456，确认密码：1234567 | 提示"两次输入的密码不一致" | ✅ 通过 |
| TC005 | 用户名已存在 | 用户名：已存在的用户名，密码：123456 | 提示"用户名已存在" | ✅ 通过 |

#### 5.2.2 用户登录测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC006 | 正常登录 | 正确的用户名和密码 | 登录成功，跳转到主页面 | ✅ 通过 |
| TC007 | 用户名为空 | 用户名为空，密码：123456 | 提示"用户名不能为空" | ✅ 通过 |
| TC008 | 密码为空 | 用户名：test，密码为空 | 提示"密码不能为空" | ✅ 通过 |
| TC009 | 用户名或密码错误 | 错误的用户名或密码 | 提示"用户名或密码错误" | ✅ 通过 |

#### 5.2.3 添加记账记录测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC010 | 正常添加收入记录 | 金额：100，类型：收入，分类：工资，备注：月薪，日期：2024-01-01 | 添加成功，列表中显示新记录 | ✅ 通过 |
| TC011 | 正常添加支出记录 | 金额：50，类型：支出，分类：餐饮，备注：午餐，日期：2024-01-01 | 添加成功，列表中显示新记录 | ✅ 通过 |
| TC012 | 金额为空 | 金额为空，其他字段正常 | 提示"请输入金额" | ✅ 通过 |
| TC013 | 金额为0或负数 | 金额：0或-10 | 提示"金额必须大于0" | ✅ 通过 |
| TC014 | 金额格式错误 | 金额：abc | 提示"请输入有效的金额" | ✅ 通过 |
| TC015 | 分类为空 | 分类为空，其他字段正常 | 提示"请输入分类" | ✅ 通过 |
| TC016 | 日期为空 | 日期为空，其他字段正常 | 提示"请选择日期" | ✅ 通过 |

#### 5.2.4 编辑记账记录测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC017 | 正常编辑记录 | 修改金额、分类等字段 | 更新成功，列表中显示更新后的数据 | ✅ 通过 |
| TC018 | 编辑时验证失败 | 将金额改为0 | 提示"金额必须大于0" | ✅ 通过 |

#### 5.2.5 删除记账记录测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC019 | 正常删除记录 | 点击删除按钮，确认删除 | 删除成功，记录从列表中移除 | ✅ 通过 |
| TC020 | 取消删除 | 点击删除按钮，取消删除 | 记录未被删除，仍在列表中 | ✅ 通过 |

#### 5.2.6 数据统计测试用例

| 测试用例ID | 测试内容 | 输入数据 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|---------|
| TC021 | 统计总收入 | 添加多条收入记录 | 总收入显示所有收入记录的总和 | ✅ 通过 |
| TC022 | 统计总支出 | 添加多条支出记录 | 总支出显示所有支出记录的总和 | ✅ 通过 |
| TC023 | 计算余额 | 总收入1000，总支出500 | 余额显示500 | ✅ 通过 |
| TC024 | 无记录时的统计 | 没有任何记录 | 总收入、总支出、余额都显示0.00 | ✅ 通过 |

#### 5.2.7 界面显示测试用例

| 测试用例ID | 测试内容 | 预期结果 | 实际结果 |
|-----------|---------|---------|---------|
| TC025 | 闪屏页面显示 | 显示应用图标和名称，2秒后跳转 | ✅ 通过 |
| TC026 | 登录页面布局 | 输入框和按钮布局正确 | ✅ 通过 |
| TC027 | 主页面统计卡片 | 统计信息正确显示 | ✅ 通过 |
| TC028 | 记录列表显示 | 记录信息完整显示，颜色区分正确 | ✅ 通过 |
| TC029 | 空列表显示 | 无记录时显示"暂无记账记录" | ✅ 通过 |

### 5.3 测试总结

经过全面的功能测试、界面测试和数据测试，本应用的主要功能均已实现并通过测试：

1. **用户认证功能**：注册和登录功能正常，输入验证完善
2. **记账数据管理**：增删改查功能正常，数据持久化正确
3. **数据统计功能**：统计计算准确，实时更新正常
4. **用户界面**：界面美观，交互流畅，符合设计要求

所有测试用例均通过，应用可以正常使用。

## （6）总结和展望

### 6.1 总结

本课题成功开发了一款基于Android平台的记账应用，实现了以下主要功能：

1. **用户认证系统**：完成了用户注册和登录功能，使用SharedPreferences保存登录状态，确保用户体验的连续性。

2. **记账数据管理**：实现了记账记录的完整CRUD操作（增、删、改、查），支持收入和支出两种类型，数据持久化到本地Room数据库。

3. **数据统计功能**：实现了总收入、总支出和余额的自动计算和实时更新，帮助用户直观了解财务状况。

4. **用户界面设计**：采用Material Design设计规范，使用清淡典雅的主题色，打造了现代化、美观的用户界面。

5. **技术实现**：使用Room数据库框架简化了数据库操作，采用ViewBinding简化视图绑定，代码结构清晰，便于维护。

**技术收获**：
- 深入理解了Android开发的基本流程和架构设计
- 掌握了Room数据库的使用方法
- 学会了Material Design设计规范的应用
- 提高了Java编程和Android开发能力

**项目亮点**：
- 界面设计美观，符合现代化审美
- 功能完整，满足基本记账需求
- 代码结构清晰，易于扩展
- 用户体验良好，操作便捷

### 6.2 展望

虽然本应用已经实现了基本功能，但仍有很大的改进和扩展空间：

#### 6.2.1 功能扩展

1. **数据可视化**：
   - 添加图表功能，使用饼图、柱状图等展示收支情况
   - 支持按月份、年份查看统计数据
   - 添加趋势分析功能

2. **分类管理**：
   - 支持自定义分类
   - 分类图标和颜色自定义
   - 分类预算设置

3. **数据导出**：
   - 支持导出为Excel或CSV格式
   - 支持数据备份和恢复
   - 支持多设备数据同步

4. **提醒功能**：
   - 记账提醒
   - 预算超支提醒
   - 账单到期提醒

5. **多账户管理**：
   - 支持多个账户（现金、银行卡、支付宝等）
   - 账户间转账功能
   - 账户余额管理

#### 6.2.2 技术优化

1. **架构升级**：
   - 采用MVVM架构模式
   - 使用LiveData和ViewModel
   - 引入RxJava处理异步操作

2. **性能优化**：
   - 使用分页加载优化大量数据展示
   - 图片压缩和缓存优化
   - 数据库查询优化

3. **安全性增强**：
   - 密码加密存储
   - 添加指纹识别登录
   - 数据加密传输

4. **用户体验优化**：
   - 添加搜索功能
   - 支持筛选和排序
   - 添加手势操作
   - 支持深色模式

5. **云同步功能**：
   - 集成Firebase或其他云服务
   - 实现数据云端备份
   - 多设备数据同步

#### 6.2.3 其他改进

1. **国际化支持**：支持多语言切换
2. **无障碍功能**：添加无障碍支持，提高应用可用性
3. **单元测试**：添加完整的单元测试和集成测试
4. **性能监控**：集成性能监控工具，优化应用性能

### 6.3 结语

本记账APP项目从需求分析到最终实现，完整地走过了Android应用开发的整个流程。通过这个项目，不仅掌握了Android开发的核心技术，还培养了系统性的软件开发思维。虽然当前版本功能相对基础，但为后续的功能扩展和技术优化奠定了良好的基础。

未来将继续完善和优化这个应用，使其成为一个功能完善、用户体验优秀的财务管理工具。

---

**报告完成日期**：2024年

**开发者**：记账APP开发团队

