# Trae 开发指南 - City Governor AI

欢迎 Trae！这是专为您准备的完整开发指南。请按照这份文档逐步完成项目开发。

---

## 📋 项目概况

### 快速概览
- **项目名称**：City Governor AI (城市决策者)
- **项目类型**：城市模拟策略游戏
- **引擎**：Unity 2022 LTS
- **编程语言**：C#
- **美术风格**：写实
- **目标玩家**：策略游戏爱好者、城市规划迷
- **预计总工期**：12-14周

### 核心特色
1. **城市建筑系统** - 类似天际线2的城市管理
2. **政府部门系统** - 动态的部门结构和副市长管理
3. **AI对话引擎** - 有性格的AI副市长
4. **决策系统** - 市议会投票和政治博弈
5. **危机事件系统** - 城市危机和应对机制

---

## 🎯 您的任务

您的职责是：
- ✅ 完整实现所有游戏系统
- ✅ 编写所有核心代码和脚本
- ✅ 进行单元测试和集成测试
- ✅ 优化性能和用户体验
- ✅ 按照开发计划按时交付各个Phase

---

## 📂 项目结构

```
city-governor-ai/
├── Assets/
│   ├── Scripts/          ← 您的主要工作地点
│   ├── Scenes/
│   ├── Prefabs/
│   ├── Materials/
│   ├── Textures/
│   └── Audio/
├── Docs/
│   ├── GDD.md           ← 游戏设计文档
│   ├── Architecture.md  ← 技术架构 (请必读！)
│   └── Development_Plan.md ← 开发计划
└── ProjectSettings/
```

---

## 🚀 开发流程指南

### 1. 项目启动前准备

#### 步骤1：阅读文档 (2小时)
- [ ] 完整阅读 `Docs/GDD.md` - 理解游戏设计
- [ ] 完整阅读 `Docs/Architecture.md` - 理解系统架构
- [ ] 完整阅读 `Docs/Development_Plan.md` - 理解时间规划

#### 步骤2：环境准备
- [ ] 安装 Unity 2022 LTS
- [ ] Clone 项目到本地
- [ ] 打开项目，验证没有编译错误
- [ ] 配置IDE (Visual Studio / VSCode)

#### 步骤3：创建开发分支
```bash
git checkout -b develop
git checkout -b feature/phase-1
```

### 2. 编码规范

#### 命名规范
```csharp
// 类名：PascalCase
public class GameManager { }

// 方法名：PascalCase
public void UpdateCityState() { }

// 属性名：camelCase (私有) 或 PascalCase (公开)
private float cityPopulation;
public float CityPopulation { get; set; }

// 常量：UPPER_SNAKE_CASE
private const float MAX_BUDGET = 10000000f;

// 接口：I + PascalCase
public interface IUpdatable { }
```

#### 代码风格
```csharp
// ✅ 推荐
public class CityManager : Singleton<CityManager>
{
    private float totalBudget;
    private List<Building> buildings;
    
    public void Update()
    {
        // 具体实现
    }
}

// ❌ 不推荐
public class cityManager : MonoBehaviour
{
    public float totalbudget;
    public List<Building> buildings;
    
    void update() { }
}
```

#### 注释规范
```csharp
/// <summary>
/// 更新城市状态，每帧调用
/// </summary>
public void UpdateCityState()
{
    // 更新建筑状态
    foreach (var building in buildings)
    {
        building.Update();
    }
}
```

### 3. Git工作流

#### 提交规范
```
[Phase-1] feat: Implement basic city system

Type: feat/fix/refactor/docs
Component: CitySystem/AISystem/UISystem

Description of changes...
```

#### 分支管理
- `main` - 稳定发布版本
- `develop` - 开发主分支
- `feature/phase-X` - 各Phase特性分支
- `bugfix/issue-description` - Bug修复分支

#### 提交流程
```bash
# 创建特性分支
git checkout -b feature/phase-1

# 进行开发，定期提交
git add .
git commit -m "[Phase-1] feat: Description"

# 完成时创建Pull Request
git push origin feature/phase-1
```

---

## 📝 Phase 1：项目框架与基础系统 (2周)

### 任务清单

#### 任务1.1：项目初始化 (完成时间：第1-2天)
- [ ] 创建Unity项目基础结构
- [ ] 创建所有需要的文件夹
- [ ] 配置 .gitignore 文件
- [ ] 设置项目编译选项

**检查清单**：
- [ ] 项目能正常编译
- [ ] 没有编译警告
- [ ] Git能正确追踪文件

---

#### 任务1.2：核心框架 (完成时间：第2-3天)

**需要实现的类**：

1. **Singleton.cs** - 单例基类
```csharp
public abstract class Singleton<T> : MonoBehaviour where T : Singleton<T>
{
    public static T Instance { get; private set; }
    
    protected virtual void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
        }
        else
        {
            Instance = this as T;
        }
    }
}
```

2. **GameManager.cs** - 游戏主管理器
3. **EventManager.cs** - 事件系统
4. **DataManager.cs** - 数据管理系统
5. **Constants.cs** - 常量定义
6. **Enums.cs** - 枚举定义

**关键接口**：
```csharp
public interface IInitializable
{
    void Initialize();
}

public interface IUpdatable
{
    void Update();
}
```

**检查清单**：
- [ ] 所有类都能正确编译
- [ ] Singleton正确实现了单例模式
- [ ] 事件系统能正确发布和订阅事件
- [ ] 编写单元测试验证功能

---

#### 任务1.3：城市系统基础 (完成时间：第4-8天)

**需要实现的类**：

1. **CityManager.cs** - 城市管理器
2. **Map.cs** - 地图系统
3. **GridCell.cs** - 网格单元格
4. **Building.cs** - 建筑基类
5. **BuildingData.cs** - 建筑数据
6. **ResourceSystem.cs** - 资源系统
7. **CityMetrics.cs** - 城市指标

**关键功能**：
- 创建可放置建筑的地图网格
- 实现建筑放置和移除
- 实现基础资源生产和消耗
- 计算城市实时指标

**测试用例**：
```csharp
[Test]
public void TestBuildingPlacement()
{
    var building = CityManager.Instance.PlaceBuilding(BuildingType.House, Vector3.zero);
    Assert.IsNotNull(building);
}

[Test]
public void TestResourceProduction()
{
    // 测试资源生产
}
```

**检查清单**：
- [ ] 能正确创建地图
- [ ] 能正确放置和移除建筑
- [ ] 资源系统正常运作
- [ ] 城市指标正确计算
- [ ] 编写了单元测试

---

#### 任务1.4：基础UI框架 (完成时间：第8-9天)

**需要实现的类**：

1. **UIManager.cs** - UI管理器
2. **CityViewController.cs** - 城市视图控制器
3. **HUDController.cs** - HUD控制器

**需要创建的UI**：
- Canvas根节点
- 主菜单面板
- 游戏进行中HUD
- 暂停菜单

**检查清单**：
- [ ] UI能正确显示
- [ ] 基础交互能工作
- [ ] 没有布局问题

---

#### 任务1.5：测试与集成 (完成时间：第9-10天)

**需要进行的测试**：
- [ ] 所有类的单元测试
- [ ] 系统集成测试
- [ ] 性能基准测试
- [ ] 编译和运行测试

**检查清单**：
- [ ] 代码覆盖率 > 80%
- [ ] 没有编译错误或警告
- [ ] 游戏能正常启动和运行

---

### Phase 1 交付物

```
✅ 完整的项目架构
✅ 工作的城市系统基础
✅ 资源系统
✅ 基础UI框架
✅ 单元测试和文档
```

**验收标准**：
- [ ] 代码质量 > 80分
- [ ] 功能完整性 100%
- [ ] 性能达标 (60+ FPS)
- [ ] 所有单元测试通过

---

## 🔧 常用工具和库

### Unity内置工具
- Unity Physics
- UI Toolkit / uGUI
- Animator
- SceneManager

### 推荐第三方库
- **DOTween** - 动画补间
- **Newtonsoft.Json** - JSON序列化
- **NUnit** - 单元测试框架

### 版本控制
```bash
# 初始化仓库
git init

# 添加远程仓库
git remote add origin https://github.com/chinaantong/city-governor-ai.git

# 拉取最新代码
git pull origin main

# 创建开发分支
git checkout -b develop
```

---

## 📞 沟通与反馈

### 遇到问题时
1. 检查文档和架构设计
2. 查看代码注释
3. 查看相关单元测试
4. 如果仍需帮助，提出问题

### 定期检查点
- 每天上午：检查是否按计划进行
- 每周：进行代码审查和集成测试
- 每个Phase：验收检查

---

## ⚡ 快速开始

### 第1天的任务

```
1. 克隆项目
   git clone https://github.com/chinaantong/city-governor-ai.git
   cd city-governor-ai

2. 创建开发分支
   git checkout -b develop
   git checkout -b feature/phase-1

3. 打开Unity项目
   - 选择 Open Project
   - 选择项目文件夹

4. 阅读文档 (2小时)
   - Docs/GDD.md
   - Docs/Architecture.md

5. 开始创建项目结构
   - Assets/Scripts/Core/
   - Assets/Scripts/City/
   - Assets/Scripts/Government/
   - 等等...

6. 创建Singleton基类
   Assets/Scripts/Core/Singleton.cs

7. 提交第一个commit
   git add .
   git commit -m "[Phase-1] Initial project setup"
   git push origin feature/phase-1
```

---

## 📊 进度跟踪

### 周进度目标

**第1-2周 (Phase 1)**：
- [ ] Day 1-2: 项目初始化
- [ ] Day 3-4: 核心框架
- [ ] Day 5-8: 城市系统
- [ ] Day 9: 基础UI
- [ ] Day 10: 测试和优化

---

## 🎓 学习资源推荐

### Unity教程
- Unity官方文档
- C# 编程基础

### 游戏开发
- 游戏架构设计模式
- 事件驱动系统

### 代码质量
- Clean Code原则
- Design Patterns

---

## ✅ 最终检查清单

在开始编码前，确认您：
- [ ] 已经完整阅读所有文档
- [ ] 理解了项目架构
- [ ] 理解了开发时间规划
- [ ] 明白了编码规范
- [ ] 知道如何使用Git
- [ ] 能够编译和运行Unity项目
- [ ] 准备好开始开发

---

## 🚀 开始开发！

一切准备就绪！现在您可以开始开发这个激动人心的项目了。

**祝您开发愉快！** 🎮

---

**文档版本**：v1.0  
**最后更新**：2025年6月8日  
**目标群体**：Trae AI编码代理
