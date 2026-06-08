# 技术架构设计 - City Governor AI

## 系统架构总览

```
┌─────────────────────────────────────────────────────────┐
│                     UI Layer                             │
│  (主菜单、游戏界面、对话、决策等UI系统)                  │
└─────────────────────────────────────────────────────────┘
                            ↑↓
┌─────────────────────────────────────────────────────────┐
│                   Business Logic Layer                   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐    │
│  │ CitySystem   │ │ Government   │ │   AISystem   │    │
│  │              │ │   System     │ │              │    │
│  └──────────────┘ └──────────────┘ └──────────────┘    │
└─────────────────────────────────────────────────────────┘
                            ↑↓
┌─────────────────────────────────────────────────────────┐
│                   Core Systems Layer                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │ Game     │ │ Event    │ │ Data     │                │
│  │ Manager  │ │ System   │ │ Manager  │                │
│  └──────────┘ └──────────┘ └──────────┘                │
└─────────────────────────────────────────────────────────┘
                            ↑↓
┌──���──────────────────────────────────────────────────────┐
│                  Data Layer                              │
│  (SaveSystem、数据序列化、本地存储)                      │
└─────────────────────────────────────────────────────────┘
```

---

## 核心模块设计

### 1. City System (城市系统)

#### 1.1 CityManager (城市管理器)
```csharp
class CityManager : Singleton<CityManager>
{
    // 基础数据
    private CityData cityData;
    private Map gameMap;
    private List<Building> buildings;
    private ResourceSystem resourceSystem;
    
    // 方法
    public void Initialize();
    public void Update();
    public Building PlaceBuilding(BuildingType type, Vector3 position);
    public void RemoveBuilding(Building building);
    public CityMetrics GetCurrentMetrics();
}
```

#### 1.2 Map (地图系统)
```csharp
class Map
{
    private Terrain terrain;
    private int width, height;
    private GridCell[,] grid;
    
    public void Initialize(int w, int h);
    public GridCell GetCell(Vector3 position);
    public bool IsValidPlacement(Vector3 position, BuildingType type);
    public List<GridCell> GetAreaAroundPoint(Vector3 center, int radius);
}
```

#### 1.3 Building (建筑)
```csharp
class Building
{
    public int id;
    public BuildingType type;
    public Vector3 position;
    public int level;
    public float health;
    public Department managedBy;
    public float maintenanceCost;
    
    public void Upgrade();
    public void TakeDamage(float damage);
    public void Maintain();
    public Dictionary<string, float> GetProduction();
    public Dictionary<string, float> GetConsumption();
}
```

#### 1.4 ResourceSystem (资源系统)
```csharp
class ResourceSystem
{
    private Dictionary<ResourceType, float> resources;
    private Dictionary<ResourceType, float> production;
    private Dictionary<ResourceType, float> consumption;
    
    public void Update();
    public void AddResource(ResourceType type, float amount);
    public void ConsumeResource(ResourceType type, float amount);
    public float GetResourceAmount(ResourceType type);
    public bool CanConsume(ResourceType type, float amount);
}
```

---

### 2. Government System (政府系统)

#### 2.1 GovernmentManager (政府管理器)
```csharp
class GovernmentManager : Singleton<GovernmentManager>
{
    private List<Department> departments;
    private List<ViceMayor> viceMayors;
    private BudgetSystem budgetSystem;
    private CouncilSystem councilSystem;
    private SatisfactionSystem satisfactionSystem;
    
    public void Initialize();
    public void AddDepartment(Department dept);
    public void RemoveDepartment(Department dept);
    public Department GetDepartment(string name);
    public List<Department> GetAllDepartments();
}
```

#### 2.2 Department (部门)
```csharp
class Department
{
    public string name;
    public string description;
    public float budget;
    public float efficiency;
    public ViceMayor viceMayor;
    public Dictionary<string, float> metrics;
    public float influence;
    
    public void Update();
    public void SetBudget(float amount);
    public float GetPerformanceScore();
    public void ProduceOutput();
}
```

#### 2.3 ViceMayor (副市长)
```csharp
class ViceMayor
{
    public string name;
    public int age;
    public ViceMayorPersonality personality;
    public List<Department> managedDepartments;
    public float supportLevel;
    public float satisfactionWithCurrentGov;
    
    public void AssignDepartment(Department dept);
    public void RemoveDepartment(Department dept);
    public void Update();
    public float EvaluateProposal(Proposal proposal);
}
```

#### 2.4 ViceMayorPersonality (副市长性格)
```csharp
class ViceMayorPersonality
{
    public float developmentBias;        // 发展偏好 (0-1)
    public float environmentBias;       // 环保偏好 (0-1)
    public float welfareBias;           // 福利偏好 (0-1)
    
    public string[] communicationStyle; // 沟通风格标签
    public Dictionary<ProposalType, float> proposalWeights;
    
    public void AdjustBias(ProposalType type, float impact);
}
```

#### 2.5 BudgetSystem (预算系统)
```csharp
class BudgetSystem
{
    private float totalMonthlyBudget;
    private Dictionary<Department, float> departmentBudgets;
    private Dictionary<Department, float> budgetWeights;
    
    public void Update();
    public void SetBudgetWeight(Department dept, float weight);
    public void AllocateBudget();
    public float GetDepartmentBudget(Department dept);
    public bool RequestBudgetAdjustment(Department dept, float newAmount);
}
```

#### 2.6 CouncilSystem (市议会系统)
```csharp
class CouncilSystem
{
    private List<CouncilMember> members;
    private VotingSession currentVote;
    
    public void Initialize();
    public void StartVote(Proposal proposal);
    public VoteResult GetVoteResult();
    public float GetSupportLevel();
}

class CouncilMember
{
    public string name;
    public float supportForMayor;
    public Dictionary<string, float> preferences;
}
```

#### 2.7 SatisfactionSystem (满意度系统)
```csharp
class SatisfactionSystem
{
    private float baseSatisfaction;
    private Dictionary<string, float> satisfactionFactors;
    
    public void Update();
    public float CalculateSatisfaction();
    public void AddFactor(string factorName, float value, float weight);
    public void RemoveFactor(string factorName);
    public float GetCurrentSatisfaction();
    public bool IsRemovalTriggered();
}
```

---

### 3. AI System (AI系统)

#### 3.1 AIManager (AI管理器)
```csharp
class AIManager : Singleton<AIManager>
{
    private CityAnalyzer analyzer;
    private ProposalGenerator proposalGenerator;
    private DialogueEngine dialogueEngine;
    private PlayerPreferenceTracker preferenceTracker;
    
    public void Update();
    public List<Proposal> GenerateProposals();
    public void ProcessPlayerDecision(Decision decision);
}
```

#### 3.2 CityAnalyzer (城市分析器)
```csharp
class CityAnalyzer
{
    public CityAnalysisResult AnalyzeCity();
    public List<string> IdentifyProblems();
    public Dictionary<string, float> GetMetricsForDepartment(Department dept);
    public string GenerateProblemDescription(string problemType);
}
```

#### 3.3 ProposalGenerator (建议生成器)
```csharp
class ProposalGenerator
{
    private Dictionary<string, List<Proposal>> proposalTemplates;
    
    public List<Proposal> GenerateProposals(CityAnalysisResult analysis);
    public Proposal GenerateProposal(string problemType, Department dept);
    public void AdjustProposalByPlayerPreference(Proposal prop, 
        PlayerPreferences prefs);
}

class Proposal
{
    public string title;
    public string description;
    public Department proposedBy;
    public float estimatedCost;
    public int estimatedDuration;
    public List<ProposalOption> options;
    public Dictionary<string, float> expectedImpacts;
    public int priority;
}

class ProposalOption
{
    public string title;
    public string description;
    public float cost;
    public int duration;
    public Dictionary<string, float> impacts;
}
```

#### 3.4 DialogueEngine (对话引擎)
```csharp
class DialogueEngine
{
    private DialogueNode currentNode;
    private DialogueTree dialogueTree;
    
    public void StartDialogue(ViceMayor viceMayor, Proposal proposal);
    public DialogueResponse GetResponse(string playerInput);
    public List<DialogueOption> GetCurrentOptions();
    public void EndDialogue();
}

class DialogueNode
{
    public string text;
    public string speakerName;
    public DialogueOption[] options;
    public DialogueNode[] nextNodes;
}
```

#### 3.5 PlayerPreferenceTracker (玩家偏好追踪)
```csharp
class PlayerPreferenceTracker
{
    private Dictionary<ProposalType, int> proposalAgreements;
    private Dictionary<ProposalType, int> proposalRejections;
    private Dictionary<string, float> departmentAffinities;
    
    public void RecordDecision(Decision decision, bool accepted);
    public PlayerPreferences GetCurrentPreferences();
    public void AdjustAIBehavior();
    public float GetPredictedAcceptance(Proposal proposal);
}

class PlayerPreferences
{
    public Dictionary<string, float> typeWeights;
    public Dictionary<string, float> departmentBiases;
    public bool prefersFastDecisions;
    public bool prefersConservativeApproach;
}
```

---

### 4. Event System (事件系统)

#### 4.1 CrisisSystem (危机系统)
```csharp
class CrisisSystem : Singleton<CrisisSystem>
{
    private List<Crisis> activeCrises;
    private List<CrisisTemplate> crisisTemplates;
    
    public void Update();
    public void CheckTriggerConditions();
    public Crisis TriggerCrisis(CrisisType type);
    public void ResolveCrisis(Crisis crisis, CrisisResponse response);
}

class Crisis
{
    public CrisisType type;
    public string title;
    public string description;
    public float severity;
    public int durationDays;
    public CrisisOption[] options;
    public Dictionary<string, float> impacts;
    public int timeRemaining;
}
```

#### 4.2 EventManager (事件管理器)
```csharp
class EventManager : Singleton<EventManager>
{
    private Dictionary<string, Action> eventHandlers;
    
    public void Subscribe(string eventName, Action handler);
    public void Unsubscribe(string eventName, Action handler);
    public void RaiseEvent(string eventName);
    public void RaiseEvent<T>(string eventName, T data);
}
```

---

### 5. Data System (数据系统)

#### 5.1 GameData (游戏数据)
```csharp
class GameData
{
    public int gameDay;
    public int gameMonth;
    public int gameYear;
    
    public CityData cityData;
    public GovernmentData governmentData;
    public List<Decision> decisionHistory;
    public PlayerPreferences playerPreferences;
    
    public void Serialize();
    public void Deserialize();
}
```

#### 5.2 SaveSystem (存档系统)
```csharp
class SaveSystem : Singleton<SaveSystem>
{
    public void SaveGame(string saveName);
    public void LoadGame(string saveName);
    public List<SaveFile> GetAllSaves();
    public void DeleteSave(string saveName);
    
    private string SerializeData(GameData data);
    private GameData DeserializeData(string json);
}
```

#### 5.3 DataManager (数据管理器)
```csharp
class DataManager : Singleton<DataManager>
{
    private GameData currentGameData;
    
    public void Initialize();
    public GameData GetGameData();
    public void UpdateGameData();
    public void ResetGameData();
}
```

---

## 脚本文件结构

```
Assets/Scripts/
│
├── Core/
│   ├── GameManager.cs
│   ├── Singleton.cs
│   ├── Constants.cs
│   └── Enums.cs
│
├── City/
│   ├── CityManager.cs
│   ├── Map.cs
│   ├── GridCell.cs
│   ├── Building.cs
│   ├── BuildingType.cs
│   ├── ResourceSystem.cs
│   └── CityMetrics.cs
│
├── Government/
│   ├── GovernmentManager.cs
│   ├── Department.cs
│   ├── ViceMayor.cs
│   ├── ViceMayorPersonality.cs
│   ├── BudgetSystem.cs
│   ├── CouncilSystem.cs
│   ├── CouncilMember.cs
│   ├── SatisfactionSystem.cs
│   └── Proposal.cs
│
├── AI/
│   ├── AIManager.cs
│   ├── CityAnalyzer.cs
│   ├── ProposalGenerator.cs
│   ├── DialogueEngine.cs
│   ├── DialogueNode.cs
│   ├── DialogueTree.cs
│   ├── PlayerPreferenceTracker.cs
│   └── BehaviorTree.cs
│
├── Events/
│   ├── EventManager.cs
│   ├── CrisisSystem.cs
│   ├── Crisis.cs
│   ├── CrisisTemplate.cs
│   └── EventTrigger.cs
│
├── Data/
│   ├── GameData.cs
│   ├── CityData.cs
│   ├── GovernmentData.cs
│   ├── SaveSystem.cs
│   ├── DataManager.cs
│   ├── Decision.cs
│   └── Serializer.cs
│
└── UI/
    ├── UIManager.cs
    ├── CityViewController.cs
    ├── GovernmentPanelController.cs
    ├── DialogueUIController.cs
    ├── ProposalUIController.cs
    ├── DecisionHistoryUIController.cs
    ├── SettingsUIController.cs
    └── HUDController.cs
```

---

## 通信模式

### 事件驱动
```csharp
// 发布
EventManager.Instance.RaiseEvent("BuildingPlaced", building);

// 订阅
EventManager.Instance.Subscribe("BuildingPlaced", OnBuildingPlaced);
```

### 直接调用
```csharp
// 对于关键系统，允许直接调用
CityManager.Instance.PlaceBuilding(type, position);
```

---

## 数据流示例

### 玩家做出决策的流程

```
1. UI获取玩家输入
   ↓
2. UIManager 调用 GovernmentManager.ProcessDecision()
   ↓
3. GovernmentManager 更新 Department 和 BudgetSystem
   ↓
4. EventManager 触发 "DecisionMade" 事件
   ↓
5. 各系统监听事件并更新：
   - CityManager 更新城市
   - SatisfactionSystem 更新满意度
   - CrisisSystem 检查危机触发
   ↓
6. UIManager 更新界面显示
   ↓
7. DataManager 记录决策到历史
```

---

## 性能考虑

### 更新频率
- **高频** (每帧)：输入处理、UI更新
- **中频** (每秒)：AI分析、危机检查
- **低频** (每个游戏月)：部门评估、满意度计算、自动存档

### 优化策略
- 使用对象池复用建筑实例
- AI分析使用缓存，避免重复计算
- UI使用Canvas优化渲染
- 数据序列化使用增量存档

---

## 扩展性设计

### 模块化设计
- 每个系统独立，通过事件通信
- 易于添加新部门、新危机类型
- 易于自定义AI行为和对话

### 易于扩展的地方
1. 新部门：继承Department，配置参数
2. 新危机：CrisisTemplate中添加模板
3. 新建筑：BuildingType中添加类型，配置数据
4. 新对话：DialogueTree中添加分支

---

**文档版本**：v1.0  
**最后更新**：2025年6月8日
