# 编码规范 - City Governor AI

本文档定义了项目的编码标准，所有开发者必须遵守。

---

## 1. C# 编码规范

### 1.1 命名规范

#### 类名、接口名、结构体名 - PascalCase
```csharp
public class GameManager { }
public interface IUpdatable { }
public struct GridPosition { }
```

#### 方法名 - PascalCase
```csharp
public void UpdateCityState() { }
private void CalculateSatisfaction() { }
public bool TryPlaceBuilding() { }
```

#### 属性名 - PascalCase (公开) / camelCase (私有)
```csharp
public float TotalBudget { get; set; }
private float currentSatisfaction;
public float CurrentSatisfaction => currentSatisfaction;
```

#### 常量 - UPPER_SNAKE_CASE
```csharp
private const float MAX_BUDGET = 10000000f;
private const int DEFAULT_MAP_SIZE = 100;
public const string GAME_VERSION = "1.0.0";
```

#### 局部变量和参数 - camelCase
```csharp
void ProcessBuilding(Building building, float cost)
{
    float remainingBudget = totalBudget - cost;
    // ...
}
```

#### 事件名 - PascalCase，On前缀
```csharp
public event Action OnCityUpdated;
public event Action<Building> OnBuildingPlaced;
```

#### 委托名 - PascalCase，Callback或Handler后缀
```csharp
public delegate void OnDecisionCallback(Decision decision);
public delegate void UpdateHandler();
```

### 1.2 访问修饰符规范

```csharp
// ✅ 推荐：明确指定访问级别
public class CityManager
{
    // 字���
    private List<Building> buildings;
    public float TotalBudget { get; private set; }
    
    // 方法
    public void Initialize() { }
    private void UpdateBuildings() { }
}

// ❌ 不推荐：省略访问修饰符
class CityManager
{
    List<Building> buildings;
    void Initialize() { }
}
```

### 1.3 代码结构

#### 类的顺序规范
```csharp
public class CityManager : MonoBehaviour
{
    // 1. 常量
    private const float MAX_POPULATION = 1000000f;
    
    // 2. 静态字段和属性
    private static CityManager instance;
    public static CityManager Instance { get; }
    
    // 3. 序列化字段
    [SerializeField]
    private float initialBudget = 5000000f;
    
    // 4. 私有字段
    private float currentBudget;
    private List<Building> buildings;
    
    // 5. 公开属性
    public float CurrentBudget { get; private set; }
    
    // 6. 事件
    public event Action OnBudgetChanged;
    
    // 7. Unity生命周期方法
    private void Awake() { }
    private void Start() { }
    private void Update() { }
    
    // 8. 公开方法
    public void Initialize() { }
    
    // 9. 私有方法
    private void UpdateCityState() { }
}
```

### 1.4 括号和缩进

```csharp
// ✅ 推荐：Allman风格
if (condition)
{
    DoSomething();
}
else
{
    DoOtherThing();
}

// ❌ 不推荐：Egyptian风格
if (condition) {
    DoSomething();
} else {
    DoOtherThing();
}
```

### 1.5 使用using语句

```csharp
// ✅ 推荐
using System;
using System.Collections.Generic;
using UnityEngine;

// ❌ 不推荐：不要使用通配符
using System.*;
```

---

## 2. Unity 特定规范

### 2.1 MonoBehaviour脚本

```csharp
using UnityEngine;

public class CityViewController : MonoBehaviour
{
    [SerializeField]
    private Camera mainCamera;
    
    [SerializeField]
    private float zoomSpeed = 10f;
    
    private CityManager cityManager;
    
    private void Awake()
    {
        // 初始化引用
        cityManager = CityManager.Instance;
    }
    
    private void Start()
    {
        // 场景启动时调用
    }
    
    private void Update()
    {
        // 每帧更新
        HandleInput();
    }
    
    private void HandleInput()
    {
        // 处理输入
    }
}
```

### 2.2 Prefab命名

```
Prefabs/
├── Buildings/
│   ├── House_Prefab
│   ├── Factory_Prefab
│   └── School_Prefab
├── UI/
│   ├── MainMenu_Panel
│   └── ProposalDialog_Panel
└── Managers/
    └── GameManager_Prefab
```

### 2.3 Scene命名

```
Scenes/
├── MainMenu.unity
├── CityGame.unity
├── Settings.unity
└── Test.unity
```

### 2.4 Layer和Tag

**Layer命名**：
- Default
- UI
- Building
- Terrain
- Water

**Tag命名**：
- Player
- Building
- Interactive
- Interactable

---

## 3. 代码质量标准

### 3.1 方法长度

- ✅ **理想**：20-30行以内
- ⚠️ **可接受**：30-50行
- ❌ **过长**：超过100行（应该分解）

### 3.2 循环复杂度

- ✅ **理想**：< 10
- ⚠️ **可接受**：10-15
- ❌ **过高**：> 15（应该重构）

### 3.3 避免的做法

```csharp
// ❌ 避免：魔法数字
if (satisfaction < 30) { }  // 为什么是30？

// ✅ 推荐：使用命名常量
if (satisfaction < DISMISSAL_THRESHOLD) { }

// ❌ 避免：嵌套过深
if (condition1)
{
    if (condition2)
    {
        if (condition3)
        {
            // 3层嵌套过深
        }
    }
}

// ✅ 推荐：使用提前返回
if (!condition1) return;
if (!condition2) return;
if (!condition3) return;
// 执行主逻辑

// ❌ 避免：在循环中new对象
for (int i = 0; i < buildings.Count; i++)
{
    var data = new BuildingData();  // 频繁分配
}

// ✅ 推荐：重用对象
var data = new BuildingData();
for (int i = 0; i < buildings.Count; i++)
{
    data = buildings[i].GetData();
}
```

---

## 4. 注释规范

### 4.1 XML文档注释

```csharp
/// <summary>
/// 更新城市状态，每帧调用一次
/// </summary>
public void UpdateCityState()
{
    // ...
}

/// <summary>
/// 在指定位置放置建筑
/// </summary>
/// <param name="type">建筑类型</param>
/// <param name="position">放置位置</param>
/// <returns>放置成功返回新建筑实例，失败返回null</returns>
public Building PlaceBuilding(BuildingType type, Vector3 position)
{
    // ...
}

/// <summary>
/// 获取部门的效率评分
/// </summary>
/// <exception cref="ArgumentNullException">当部门为null时抛出</exception>
public float GetDepartmentEfficiency(Department department)
{
    if (department == null)
        throw new ArgumentNullException(nameof(department));
    
    return department.Efficiency;
}
```

### 4.2 行注释

```csharp
// 仅在需要解释"为什么"时使用
// 不要注释"做什么"（代码本身应该清楚）

// ✅ 好的注释
// 需要缓存结果以避免每帧重新计算
private Dictionary<Department, float> efficiencyCache;

// ❌ 坏的注释
float x = 10;  // 设置x为10
```

---

## 5. 错误处理

### 5.1 异常处理

```csharp
// ✅ 推荐：具体的异常处理
try
{
    var data = LoadGameData();
}
catch (FileNotFoundException ex)
{
    Debug.LogError($"Game file not found: {ex.Message}");
}
catch (JsonSerializationException ex)
{
    Debug.LogError($"Failed to deserialize game data: {ex.Message}");
}

// ❌ 避免：过于宽泛的异常捕获
try
{
    var data = LoadGameData();
}
catch (Exception ex)
{
    Debug.LogError("Error");
}
```

### 5.2 空值检查

```csharp
// ✅ 推荐：使用null-coalescing操作符
public float GetBudget()
{
    return budget ?? 0f;
}

// ✅ 推荐：空值检查
if (building == null)
{
    return false;
}

// ❌ 避免：不必要的复杂检查
if (building != null && building.GetType() != null)
{
    // ...
}
```

---

## 6. 性能最佳实践

### 6.1 避免在Update中分配内存

```csharp
// ❌ 不好：每帧分配新List
private void Update()
{
    List<Building> buildings = GetAllBuildings();  // 每帧新建
}

// ✅ 推荐：缓存结果
private List<Building> cachedBuildings;

private void Start()
{
    cachedBuildings = GetAllBuildings();
}

private void OnBuildingChanged()
{
    cachedBuildings = GetAllBuildings();  // 仅当数据改变时更新
}
```

### 6.2 使用对象池

```csharp
public class BuildingPool : Singleton<BuildingPool>
{
    private Queue<Building> pool;
    
    public Building Get()
    {
        return pool.Count > 0 ? pool.Dequeue() : CreateNewBuilding();
    }
    
    public void Return(Building building)
    {
        building.gameObject.SetActive(false);
        pool.Enqueue(building);
    }
}
```

---

## 7. 测试规范

### 7.1 单元测试

```csharp
using NUnit.Framework;

public class CityManagerTests
{
    private CityManager cityManager;
    
    [SetUp]
    public void Setup()
    {
        cityManager = new CityManager();
        cityManager.Initialize();
    }
    
    [Test]
    public void TestBuildingPlacement_ValidPosition_ReturnsBuilding()
    {
        // Arrange
        var type = BuildingType.House;
        var position = Vector3.zero;
        
        // Act
        var building = cityManager.PlaceBuilding(type, position);
        
        // Assert
        Assert.IsNotNull(building);
        Assert.AreEqual(type, building.Type);
    }
    
    [TearDown]
    public void Cleanup()
    {
        cityManager.Cleanup();
    }
}
```

---

## 8. Git提交规范

### 8.1 提交消息格式

```
[Phase-X] type(component): subject

body

footer
```

**Type**：
- `feat` - 新功能
- `fix` - Bug修复
- `refactor` - 代码重构
- `docs` - 文档更新
- `test` - 测试相关
- `chore` - 其他杂务

**Component**：
- CitySystem
- GovernmentSystem
- AISystem
- UISystem
- EventSystem
- DataSystem

**示例**：
```
[Phase-1] feat(CitySystem): Implement basic building placement

- Add Building class with placement logic
- Implement GridCell for map validation
- Add unit tests for placement validation

Closes #1
```

---

## 9. 代码审查清单

提交PR前，确保：

- [ ] 代码遵循命名规范
- [ ] 没有编译警告
- [ ] 添加了必要的注释和文档
- [ ] 编写了单元测试
- [ ] 测试通过率 > 95%
- [ ] 没有使用魔法数字
- [ ] 没有过深的嵌套
- [ ] 没有未使用的变量或方法
- [ ] 代码可读性良好

---

**文档版本**：v1.0  
**最后更新**：2025年6月8日
