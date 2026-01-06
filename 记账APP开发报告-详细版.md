# 记账APP开发报告

## 第2章 总体设计

### 2.1 总体结构设计

本记账APP采用分层架构设计，将系统分为表示层、业务逻辑层、数据访问层和数据持久层四个层次。表示层负责用户界面的展示和交互，包括闪屏页面、登录注册页面、主页面和添加编辑记录页面；业务逻辑层处理用户操作和业务规则，包括用户认证、记账记录的增删改查、数据统计等功能；数据访问层通过DAO接口提供数据操作抽象，隔离业务逻辑和数据库实现；数据持久层使用Room数据库框架实现数据的本地存储。各层之间通过接口和依赖注入实现松耦合，便于维护和扩展。总体结构图如图2-1所示。

**（此处应插入图2-1：总体结构图，包含以下内容）**
- 表示层（UI Layer）：SplashActivity、LoginActivity、RegisterActivity、MainActivity、AddEditRecordActivity
- 业务逻辑层（Business Layer）：用户认证逻辑、记账管理逻辑、统计计算逻辑
- 数据访问层（DAO Layer）：UserDao、AccountRecordDao
- 数据持久层（Database Layer）：AppDatabase、User实体、AccountRecord实体

图2-1 总体结构图

### 2.2 文件结构框架

本项目的文件结构采用标准的Android项目组织方式，按照功能模块和层次进行划分。Java源代码位于`app/src/main/java/com/example/code/`目录下，按照包结构组织：entity包包含实体类，dao包包含数据访问接口，database包包含数据库配置，adapter包包含RecyclerView适配器，util包包含工具类，根目录包含所有Activity类。资源文件位于`app/src/main/res/`目录下，包括布局文件、字符串资源、颜色资源、动画资源等。文件结构图如图2-2所示。

**（此处应插入图2-2：文件结构图，包含以下内容）**
```
app/src/main/
├── java/com/example/code/
│   ├── entity/          # 实体类
│   │   ├── User.java
│   │   └── AccountRecord.java
│   ├── dao/             # 数据访问接口
│   │   ├── UserDao.java
│   │   └── AccountRecordDao.java
│   ├── database/        # 数据库配置
│   │   └── AppDatabase.java
│   ├── adapter/         # RecyclerView适配器
│   │   └── RecordAdapter.java
│   ├── util/            # 工具类
│   │   ├── SharedPreferencesHelper.java
│   │   └── DataInitializer.java
│   ├── SplashActivity.java
│   ├── LoginActivity.java
│   ├── RegisterActivity.java
│   ├── MainActivity.java
│   └── AddEditRecordActivity.java
└── res/
    ├── layout/          # 布局文件
    ├── values/          # 资源文件
    ├── drawable/        # 图片和drawable资源
    └── anim/            # 动画资源
```

图2-2 文件结构图

## 第3章 详细设计

### 3.1 详细设计

#### 3.1.1 类的详细设计

**1. User实体类**

User类用于存储用户信息，包含id、username和password三个属性。id为主键，自动增长；username为用户名，唯一标识用户；password为密码，用于用户认证。

**主要方法：**
- `getId()`: 获取用户ID
- `setId(long id)`: 设置用户ID
- `getUsername()`: 获取用户名
- `setUsername(String username)`: 设置用户名
- `getPassword()`: 获取密码
- `setPassword(String password)`: 设置密码

**2. AccountRecord实体类**

AccountRecord类用于存储记账记录信息，包含id、userId、type、amount、category、item、paymentMethod、note、date等属性。id为主键，自动增长；userId为外键，关联到User表；type表示记录类型（收入或消费）；amount为金额；category为分类；item为项目；paymentMethod为支付方式；note为备注；date为日期。

**主要方法：**
- 标准的getter和setter方法
- `AccountRecord()`: 无参构造函数（Room使用）
- `AccountRecord(...)`: 带参构造函数（@Ignore注解，Room不使用）

**3. SplashActivity类**

SplashActivity是应用的启动页面，负责显示应用Logo和名称，并在2.5秒后根据用户登录状态跳转到相应页面。

**主要方法：**
- `onCreate(Bundle)`: 初始化界面，启动动画，设置延迟跳转
- `animateCircle(View, long)`: 实现背景圆圈的脉冲动画效果

**用户登录流程：**
**（此处应插入流程图图片，展示从启动到登录的完整流程）**
流程图应包含：启动应用 → 显示闪屏 → 检查登录状态 → 跳转到登录页/主页 → 用户输入 → 验证 → 登录成功/失败

**4. LoginActivity类**

LoginActivity负责用户登录功能，验证用户名和密码，登录成功后保存用户信息并跳转到主页面。

**主要方法：**
- `onCreate(Bundle)`: 初始化界面，设置点击事件监听器，启动进入动画
- `login()`: 验证用户输入，查询数据库，执行登录逻辑
- `animatePageEntry()`: 实现页面元素的淡入动画效果

**登录验证流程：**
**（此处应插入流程图图片，展示登录验证的详细步骤）**
流程图应包含：用户输入用户名密码 → 验证输入是否为空 → 查询数据库 → 验证用户是否存在 → 保存登录状态 → 跳转主页面

**5. RegisterActivity类**

RegisterActivity负责用户注册功能，验证输入信息，检查用户名是否已存在，创建新用户。

**主要方法：**
- `onCreate(Bundle)`: 初始化界面，设置注册按钮点击事件
- `register()`: 验证输入，检查用户名唯一性，创建新用户

**用户注册流程：**
**（此处应插入流程图图片，展示注册的详细步骤）**
流程图应包含：用户填写注册信息 → 验证输入 → 检查用户名是否存在 → 验证密码一致性 → 创建用户 → 注册成功

**6. MainActivity类**

MainActivity是应用的主页面，显示财务统计信息和记账记录列表，提供添加、编辑、删除记录的功能。

**主要方法：**
- `onCreate(Bundle)`: 初始化界面，设置RecyclerView，加载数据
- `setupRecyclerView()`: 创建适配器，设置布局管理器
- `loadRecords()`: 从数据库加载记录，更新列表显示，更新统计信息
- `updateStatistics()`: 计算总收入、总支出和余额，更新UI显示
- `editRecord(AccountRecord)`: 启动编辑页面
- `deleteRecord(AccountRecord)`: 显示确认对话框，删除记录
- `animatePageEntry()`: 实现页面进入动画

**数据加载和统计流程：**
**（此处应插入流程图图片，展示数据加载和统计计算的流程）**
流程图应包含：页面加载 → 查询数据库 → 获取所有记录 → 计算总收入 → 计算总支出 → 计算余额 → 更新UI显示

**7. AddEditRecordActivity类**

AddEditRecordActivity负责添加和编辑记账记录，支持输入金额、类型、分类、项目、支付方式、备注和日期。

**主要方法：**
- `onCreate(Bundle)`: 初始化界面，判断新增/编辑模式，加载现有数据
- `showDatePicker()`: 显示日期选择对话框
- `loadRecord()`: 从数据库加载记录数据，填充表单
- `saveRecord()`: 验证输入，保存或更新记录

**保存记录流程：**
**（此处应插入流程图图片，展示保存记录的详细步骤）**
流程图应包含：用户填写表单 → 验证输入 → 判断新增/编辑模式 → 创建/更新记录对象 → 保存到数据库 → 返回主页面

**8. AppDatabase类**

AppDatabase是Room数据库的配置类，使用单例模式确保整个应用只有一个数据库实例。

**主要方法：**
- `getInstance(Context)`: 获取数据库单例实例
- `userDao()`: 获取UserDao接口实例
- `accountRecordDao()`: 获取AccountRecordDao接口实例

**9. RecordAdapter类**

RecordAdapter是RecyclerView的适配器，负责将记账记录数据绑定到列表项视图。

**主要方法：**
- `onCreateViewHolder()`: 创建ViewHolder实例
- `onBindViewHolder()`: 绑定数据到ViewHolder
- `updateRecords(List)`: 更新记录列表
- `RecordViewHolder.bind()`: 绑定单条记录数据，设置颜色和样式

### 3.2 数据库设计

本应用使用Room数据库框架，共设计两个数据表：users表和account_records表。users表用于存储用户信息，account_records表用于存储记账记录，两个表通过外键关联，实现一对多的关系。

#### 3.2.1 users表

users表用于存储用户的基本信息，包括用户ID、用户名和密码。用户ID作为主键，自动增长，确保每个用户都有唯一标识。用户名用于登录认证，密码用于验证用户身份。

| 字段名 | 数据类型 | 约束 | 说明 |
|--------|---------|------|------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | 用户ID，主键，自增 |
| username | TEXT | NOT NULL | 用户名，唯一标识 |
| password | TEXT | NOT NULL | 用户密码 |

#### 3.2.2 account_records表

account_records表用于存储用户的记账记录，包含记录的详细信息。每条记录都关联到一个用户，通过userId外键实现关联。当用户被删除时，相关的记账记录也会被级联删除。

| 字段名 | 数据类型 | 约束 | 说明 |
|--------|---------|------|------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | 记录ID，主键，自增 |
| userId | INTEGER | FOREIGN KEY, NOT NULL | 用户ID，外键关联users表 |
| type | TEXT | NOT NULL | 记录类型：收入或消费 |
| amount | REAL | NOT NULL | 金额 |
| category | TEXT | NOT NULL | 分类 |
| item | TEXT | | 项目 |
| paymentMethod | TEXT | | 支付方式 |
| note | TEXT | | 备注 |
| date | TEXT | NOT NULL | 日期，格式：yyyy-MM-dd |

## 第4章 实现及结果分析

### 4.1 实现

#### 1、UI界面

本应用的UI界面采用Material Design设计规范，使用清淡典雅的主题色，界面美观现代。主要包含以下界面：

- **闪屏页面**：显示应用Logo和名称，带有丰富的动画效果
- **登录页面**：简洁的登录表单，支持用户名密码登录
- **注册页面**：用户注册表单，包含用户名、密码和确认密码
- **主页面**：显示财务统计卡片和记账记录列表
- **添加/编辑记录页面**：完整的记账表单，支持所有字段输入

#### 2、各个文件核心代码

**⑴SplashActivity.java**

SplashActivity的核心代码实现了闪屏页面的动画效果和页面跳转逻辑：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_splash);
    
    // 获取视图引用
    CardView iconCard = findViewById(R.id.iconCard);
    ImageView iconWallet = findViewById(R.id.iconWallet);
    TextView tvTitle = findViewById(R.id.tvTitle);
    
    // 图标卡片动画：缩放+淡入
    ObjectAnimator scaleX = ObjectAnimator.ofFloat(iconCard, "scaleX", 0f, 1f);
    ObjectAnimator scaleY = ObjectAnimator.ofFloat(iconCard, "scaleY", 0f, 1f);
    ObjectAnimator alpha = ObjectAnimator.ofFloat(iconCard, "alpha", 0f, 1f);
    AnimatorSet iconAnim = new AnimatorSet();
    iconAnim.playTogether(scaleX, scaleY, alpha);
    iconAnim.setDuration(800);
    iconAnim.setInterpolator(new OvershootInterpolator(1.5f));
    iconAnim.start();
    
    // 延迟跳转
    new Handler(Looper.getMainLooper()).postDelayed(() -> {
        SharedPreferencesHelper prefsHelper = new SharedPreferencesHelper(this);
        Intent intent;
        if (prefsHelper.isLoggedIn()) {
            intent = new Intent(SplashActivity.this, MainActivity.class);
        } else {
            intent = new Intent(SplashActivity.this, LoginActivity.class);
        }
        startActivity(intent);
        finish();
    }, SPLASH_DELAY);
}
```

**⑵LoginActivity.java**

LoginActivity的核心代码实现了用户登录验证和页面动画：

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

**⑶MainActivity.java**

MainActivity的核心代码实现了数据加载、统计计算和记录管理：

```java
private void loadRecords() {
    List<AccountRecord> records = recordDao.getAllRecordsByUser(userId);
    
    // 更新记录数量显示
    binding.tvRecordCount.setText(records.size() + " 条");
    
    if (records.isEmpty()) {
        binding.rvRecords.setVisibility(View.GONE);
        binding.tvEmpty.setVisibility(View.VISIBLE);
    } else {
        binding.rvRecords.setVisibility(View.VISIBLE);
        binding.tvEmpty.setVisibility(View.GONE);
        adapter.updateRecords(records);
    }
    
    updateStatistics();
}

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

**⑷AddEditRecordActivity.java**

AddEditRecordActivity的核心代码实现了记录的保存和更新：

```java
private void saveRecord() {
    String amountStr = binding.etAmount.getText().toString().trim();
    String category = binding.etCategory.getText().toString().trim();
    String item = binding.etItem.getText().toString().trim();
    String paymentMethod = binding.etPaymentMethod.getText().toString().trim();
    String note = binding.etNote.getText().toString().trim();
    String date = binding.etDate.getText().toString().trim();
    
    // 验证输入
    if (amountStr.isEmpty() || category.isEmpty() || date.isEmpty()) {
        Toast.makeText(this, "请填写必填项", Toast.LENGTH_SHORT).show();
        return;
    }
    
    double amount = Double.parseDouble(amountStr);
    String type = binding.rbIncome.isChecked() ? "收入" : "消费";
    
    if (recordId == -1) {
        // 新增记录
        AccountRecord record = new AccountRecord(userId, type, amount, category, item, paymentMethod, note, date);
        long id = recordDao.insertRecord(record);
        if (id > 0) {
            Toast.makeText(this, "添加成功", Toast.LENGTH_SHORT).show();
            finish();
        }
    } else {
        // 更新记录
        AccountRecord record = recordDao.getRecordById(recordId);
        if (record != null) {
            record.setType(type);
            record.setAmount(amount);
            record.setCategory(category);
            record.setItem(item);
            record.setPaymentMethod(paymentMethod);
            record.setNote(note);
            record.setDate(date);
            recordDao.updateRecord(record);
            Toast.makeText(this, "更新成功", Toast.LENGTH_SHORT).show();
            finish();
        }
    }
}
```

**⑸AppDatabase.java**

AppDatabase的核心代码实现了数据库的单例模式：

```java
@Database(entities = {User.class, AccountRecord.class}, version = 2, exportSchema = false)
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
             .fallbackToDestructiveMigration()
             .build();
        }
        return instance;
    }
}
```

### 4.2 运行结果及分析

#### ⑴主界面如图4-1所示

**（此处应插入图4-1：主界面截图）**

主界面展示了财务概览卡片和记账记录列表。财务概览卡片包含总收入、总消费和余额三个统计指标，使用不同颜色的卡片区分收入和消费。记账记录列表使用RecyclerView展示，每条记录显示分类、项目、支付方式、备注、日期和金额等信息，收入和消费使用不同的颜色标识，便于用户快速识别。

图4-1 主界面

#### ⑵用户登录界面如图4-2所示

**（此处应插入图4-2：用户登录界面截图）**

登录界面采用渐变背景，界面简洁美观。包含用户名和密码输入框，以及登录和注册按钮。输入框采用Material Design风格的OutlinedBox样式，按钮采用圆角设计，整体视觉效果现代优雅。

图4-2 用户登录界面

#### ⑶闪屏界面如图4-3所示

**（此处应插入图4-3：闪屏界面截图）**

闪屏界面显示应用Logo和名称，带有丰富的动画效果。图标卡片采用缩放和淡入动画，图标本身带有旋转动画，标题和副标题采用淡入和上移动画，背景圆圈采用脉冲动画，整体动画效果流畅自然。

图4-3 闪屏界面

#### ⑷添加记录界面如图4-4所示

**（此处应插入图4-4：添加记录界面截图）**

添加记录界面包含完整的记账表单，支持输入金额、类型（收入/消费）、分类、项目、支付方式、备注和日期。表单采用Material Design设计规范，输入框采用OutlinedBox样式，日期选择使用DatePickerDialog，用户体验良好。

图4-4 添加记录界面

#### ⑸记账记录列表如图4-5所示

**（此处应插入图4-5：记账记录列表截图）**

记账记录列表使用卡片式设计，每条记录显示完整信息。左侧使用颜色条区分收入和消费，分类和类型标签突出显示，金额使用大字体显示，支付方式和日期使用图标标识，整体布局清晰易读。

图4-5 记账记录列表

## 第5章 测试

### 5.1 测试方法

软件测试是确保软件质量的重要手段，常用的测试方法包括黑盒测试和白盒测试。

#### 1、黑盒测试

黑盒测试也称为功能测试或行为测试，是一种测试方法，测试人员不需要了解程序内部的代码结构和实现细节，只需要根据软件的功能需求规格说明，设计测试用例，验证软件的功能是否符合预期。黑盒测试关注的是输入和输出，测试人员将程序视为一个黑盒子，只关心程序对输入的处理结果是否正确。

在本项目中，黑盒测试主要用于测试用户界面的功能，如用户注册、登录、添加记录、编辑记录、删除记录等功能是否正常工作。

#### 2、白盒测试

白盒测试也称为结构测试或逻辑驱动测试，是一种测试方法，测试人员需要了解程序内部的代码结构、逻辑流程和实现细节，根据程序的内部结构设计测试用例，验证程序的逻辑是否正确，代码路径是否被正确执行。白盒测试关注的是程序内部的逻辑，测试人员需要了解代码的实现细节。

在本项目中，白盒测试主要用于测试数据库操作、业务逻辑处理、数据统计计算等核心功能的正确性。

### 5.2 测试用例分析

#### 1、Room数据库构造函数警告测试

**⑴问题**

在开发过程中，遇到了Room数据库的构造函数警告问题。当实体类（User和AccountRecord）同时存在无参构造函数和带参构造函数时，Room编译器会发出警告："There are multiple good constructors and Room will pick the no-arg constructor. You can use the @Ignore annotation to eliminate unwanted constructors."

**问题截图：**
**（此处应插入问题截图，显示编译警告信息）**

**⑵修改方案**

为了解决这个问题，需要在带参构造函数上添加`@Ignore`注解，告诉Room这个构造函数不需要使用，只使用无参构造函数即可。

**修改前代码：**
```java
public class User {
    public User() {
    }
    
    public User(String username, String password) {  // 没有@Ignore注解
        this.username = username;
        this.password = password;
    }
}
```

**修改后代码：**
```java
public class User {
    public User() {
    }
    
    @Ignore  // 添加@Ignore注解
    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }
}
```

**修改成功截图：**
**（此处应插入修改后的截图，显示警告消失）**

使用白盒测试方法，通过查看代码结构和Room框架的要求，识别出问题原因并成功解决。修改后，编译警告消失，程序可以正常编译运行。

#### 2、数据库版本升级测试

**⑴问题**

在开发过程中，需要为AccountRecord实体类添加新字段（item和paymentMethod），这需要升级数据库版本。如果不正确处理数据库版本升级，会导致应用崩溃或数据丢失。

**问题描述：**
- 数据库版本从1升级到2
- 新增了item和paymentMethod字段
- 如果不处理版本迁移，旧版本数据库会无法使用

**⑵修改方案**

在AppDatabase类中，将数据库版本从1升级到2，并添加`fallbackToDestructiveMigration()`方法，在开发阶段允许Room删除旧数据库并重建新数据库。

**修改代码：**
```java
@Database(entities = {User.class, AccountRecord.class}, version = 2, exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    public static synchronized AppDatabase getInstance(Context context) {
        if (instance == null) {
            instance = Room.databaseBuilder(
                context.getApplicationContext(),
                AppDatabase.class,
                "accounting_database"
            ).allowMainThreadQueries()
             .fallbackToDestructiveMigration()  // 添加此方法
             .build();
        }
        return instance;
    }
}
```

**测试结果：**
**（此处应插入测试结果截图，显示应用正常运行）**

使用白盒测试方法，通过分析数据库版本升级机制，正确配置了版本迁移策略。修改后，应用可以正常升级数据库版本，新字段可以正常使用，旧数据会被清除（开发阶段可接受）。

#### 3、用户登录功能测试

**⑴问题**

在测试用户登录功能时，发现如果用户输入错误的用户名或密码，应用会显示"用户名或密码错误"的提示，但如果用户名为空或密码为空，应用也会显示相同的错误信息，用户体验不够友好。

**问题描述：**
- 输入验证不够详细
- 错误提示信息不够明确

**⑵修改方案**

在login()方法中添加详细的输入验证，分别检查用户名和密码是否为空，并显示相应的错误提示。

**修改代码：**
```java
private void login() {
    String username = binding.etUsername.getText().toString().trim();
    String password = binding.etPassword.getText().toString().trim();
    
    // 添加详细的输入验证
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
        // 登录成功
    } else {
        Toast.makeText(this, getString(R.string.login_failed), Toast.LENGTH_SHORT).show();
    }
}
```

**测试结果：**
**（此处应插入测试结果截图，显示不同错误情况下的提示信息）**

使用黑盒测试方法，通过测试不同的输入情况，验证了输入验证功能的正确性。修改后，用户可以清楚地知道是哪个字段输入有误，用户体验得到改善。

---

**报告完成日期**：2024年

**开发者**：记账APP开发团队

