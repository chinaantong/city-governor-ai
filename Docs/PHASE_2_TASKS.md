# Phase 2 详细开发任务 - 政府部门管理系统

## 阶段目标

实现政府部门、副市长和预算系统，为后续的AI决策和市议会系统奠定基础。

**预计工时**：2周（10个工作日）  
**目标完成日期**：第12-13天

---

## 任务1：部门系统 (3天)

### 任务1.1：Department类（第1天）

**文件**：`Assets/Scripts/Government/Department.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class Department
{
    [SerializeField] private string departmentName;
    [SerializeField] private string description;
    [SerializeField] private DepartmentType departmentType;
    [SerializeField] private float monthlyBudget;
    [SerializeField] private float efficiency = 1.0f;
    [SerializeField] private float influenceLevel = 0.5f;
    
    private ViceMayor assignedViceMayor;
    private Dictionary<string, float> metrics = new Dictionary<string, float>();
    private float performanceScore = 5.0f; // 0-10 scale
    private int monthsInOperation = 0;
    
    // 事件
    public event Action<float> OnBudgetChanged;
    public event Action<float> OnEfficiencyChanged;
    
    public Department(string name, DepartmentType type, string desc = "")
    {
        departmentName = name;
        departmentType = type;
        description = desc;
        InitializeMetrics();
    }
    
    private void InitializeMetrics()
    {
        // 根据部门类型初始化指标
        switch (departmentType)
        {
            case DepartmentType.Transportation:
                metrics["CongestionRate"] = 50f;
                metrics["AverageCommute"] = 30f;
                metrics["PublicTransitUsage"] = 30f;
                break;
            case DepartmentType.Environment:
                metrics["AirQuality"] = 70f;
                metrics["GreenCoverage"] = 20f;
                metrics["PollutionLevel"] = 30f;
                break;
            case DepartmentType.Education:
                metrics["EducationLevel"] = 50f;
                metrics["SchoolCoverage"] = 40f;
                metrics["StudentSatisfaction"] = 60f;
                break;
            case DepartmentType.Health:
                metrics["HealthIndex"] = 70f;
                metrics["MedicalCoverage"] = 50f;
                metrics["HospitalQuality"] = 60f;
                break;
            case DepartmentType.Infrastructure:
                metrics["InfrastructureCompletion"] = 60f;
                metrics["PowerAvailability"] = 95f;
                metrics["WaterSupply"] = 95f;
                break;
            case DepartmentType.Police:
                metrics["CrimeRate"] = 40f;
                metrics["PublicSafety"] = 65f;
                metrics["PoliceResponseTime"] = 5f;
                break;
            case DepartmentType.Finance:
                metrics["BudgetBalance"] = 100f;
                metrics["TaxRevenue"] = 1000000f;
                metrics["DebtLevel"] = 0f;
                break;
        }
    }
    
    // 属性
    public string DepartmentName => departmentName;
    public DepartmentType Type => departmentType;
    public float MonthlyBudget => monthlyBudget;
    public float Efficiency => efficiency;
    public float InfluenceLevel => influenceLevel;
    public ViceMayor AssignedViceMayor => assignedViceMayor;
    public float PerformanceScore => performanceScore;
    public Dictionary<string, float> Metrics => new Dictionary<string, float>(metrics);
    
    // 方法
    public void SetBudget(float newBudget)
    {
        float oldBudget = monthlyBudget;
        monthlyBudget = newBudget;
        OnBudgetChanged?.Invoke(newBudget);
        
        // 预算变化影响效率
        if (newBudget > oldBudget)
        {
            efficiency = Mathf.Clamp01(efficiency + 0.1f);
        }
        else if (newBudget < oldBudget * 0.5f)
        {
            efficiency = Mathf.Clamp01(efficiency - 0.15f);
        }
    }
    
    public void AssignViceMayor(ViceMayor viceMayor)
    {
        assignedViceMayor = viceMayor;
    }
    
    public void RemoveViceMayor()
    {
        assignedViceMayor = null;
    }
    
    public void Update()
    {
        monthsInOperation++;
        UpdateMetrics();
        CalculatePerformanceScore();
    }
    
    private void UpdateMetrics()
    {
        // 根据预算和效率更新各项指标
        // 这是一个简化版本，实际应该根据具体逻辑更新
        foreach (var metric in metrics.Keys)
        {
            // 示例：效率高则指标改善
            float change = efficiency * monthlyBudget / 1000000f;
            metrics[metric] = Mathf.Clamp(metrics[metric] + change, 0f, 100f);
        }
    }
    
    private void CalculatePerformanceScore()
    {
        // 计算部门表现评分 (0-10)
        float scoreSum = 0f;
        foreach (var value in metrics.Values)
        {
            scoreSum += value / 10f; // 归一化到0-10
        }
        performanceScore = scoreSum / metrics.Count;
        
        // 根据副市长调整评分
        if (assignedViceMayor != null)
        {
            performanceScore += assignedViceMayor.GetDepartmentBonus(departmentType);
        }
    }
    
    public float GetMetric(string metricName)
    {
        return metrics.ContainsKey(metricName) ? metrics[metricName] : 0f;
    }
    
    public void SetMetric(string metricName, float value)
    {
        if (metrics.ContainsKey(metricName))
        {
            metrics[metricName] = Mathf.Clamp(value, 0f, 100f);
        }
    }
    
    public void ProduceOutput()
    {
        // 部门生成输出（建筑升级、服务提升等）
        if (monthlyBudget > 0)
        {
            // 根据部门类型执行相应操作
            EventManager.Instance.RaiseEvent($"Department_{departmentType}_ProducedOutput", this);
        }
    }
}
```

**检查清单**：
- [ ] Department类编译成功
- [ ] 所有指标正确初始化
- [ ] 预算和效率联动正确

---

### 任务1.2：DepartmentManager类（第2天）

**文件**：`Assets/Scripts/Government/DepartmentManager.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class DepartmentManager : Singleton<DepartmentManager>
{
    private Dictionary<DepartmentType, Department> departments;
    private List<Department> departmentList;
    
    public event Action<Department> OnDepartmentAdded;
    public event Action<Department> OnDepartmentRemoved;
    
    public override void Awake()
    {
        base.Awake();
        Initialize();
    }
    
    private void Initialize()
    {
        departments = new Dictionary<DepartmentType, Department>();
        departmentList = new List<Department>();
        
        // 创建7个基础部门
        CreateBasicDepartments();
    }
    
    private void CreateBasicDepartments()
    {
        var basicDepts = new (DepartmentType, string)[]
        {
            (DepartmentType.Transportation, "交通部"),
            (DepartmentType.Environment, "环保部"),
            (DepartmentType.Education, "教育部"),
            (DepartmentType.Health, "卫生部"),
            (DepartmentType.Infrastructure, "城建部"),
            (DepartmentType.Police, "警察部"),
            (DepartmentType.Finance, "财政部")
        };
        
        foreach (var (type, name) in basicDepts)
        {
            AddDepartment(new Department(name, type));
        }
    }
    
    public void AddDepartment(Department department)
    {
        if (department == null) return;
        
        if (!departments.ContainsKey(department.Type))
        {
            departments[department.Type] = department;
            departmentList.Add(department);
            OnDepartmentAdded?.Invoke(department);
            EventManager.Instance.RaiseEvent("DepartmentAdded", department);
        }
    }
    
    public void RemoveDepartment(DepartmentType type)
    {
        if (departments.TryGetValue(type, out var dept))
        {
            departments.Remove(type);
            departmentList.Remove(dept);
            OnDepartmentRemoved?.Invoke(dept);
            EventManager.Instance.RaiseEvent("DepartmentRemoved", dept);
        }
    }
    
    public Department GetDepartment(DepartmentType type)
    {
        return departments.ContainsKey(type) ? departments[type] : null;
    }
    
    public Department GetDepartmentByName(string name)
    {
        return departmentList.Find(d => d.DepartmentName == name);
    }
    
    public List<Department> GetAllDepartments()
    {
        return new List<Department>(departmentList);
    }
    
    public void Update()
    {
        foreach (var dept in departmentList)
        {
            dept.Update();
            dept.ProduceOutput();
        }
    }
    
    public float GetTotalBudgetUsed()
    {
        float total = 0f;
        foreach (var dept in departmentList)
        {
            total += dept.MonthlyBudget;
        }
        return total;
    }
}
```

**检查清单**：
- [ ] DepartmentManager编译成功
- [ ] 7个基础部门正确创建
- [ ] Add/Remove部门工作正确
- [ ] GetDepartment方法工作正确

---

### 任务1.3：单元测试（第3天）

**文件**：`Assets/Scripts/Tests/DepartmentTests.cs`

```csharp
using NUnit.Framework;
using UnityEngine;

public class DepartmentTests
{
    private Department department;
    private DepartmentManager deptManager;
    
    [SetUp]
    public void Setup()
    {
        department = new Department("交通部", DepartmentType.Transportation);
        
        var managerGO = new GameObject("DepartmentManager");
        deptManager = managerGO.AddComponent<DepartmentManager>();
    }
    
    [Test]
    public void TestDepartmentCreation()
    {
        Assert.AreEqual("交通部", department.DepartmentName);
        Assert.AreEqual(DepartmentType.Transportation, department.Type);
    }
    
    [Test]
    public void TestSetBudget()
    {
        department.SetBudget(500000);
        Assert.AreEqual(500000, department.MonthlyBudget);
    }
    
    [Test]
    public void TestBudgetAffectsEfficiency()
    {
        var initialEfficiency = department.Efficiency;
        department.SetBudget(1000000);
        // 预期效率提升
        Assert.Greater(department.Efficiency, initialEfficiency);
    }
    
    [Test]
    public void TestDepartmentManagerCreatesBasicDepartments()
    {
        var depts = deptManager.GetAllDepartments();
        Assert.AreEqual(7, depts.Count);
    }
    
    [Test]
    public void TestGetDepartment()
    {
        var dept = deptManager.GetDepartment(DepartmentType.Transportation);
        Assert.IsNotNull(dept);
        Assert.AreEqual(DepartmentType.Transportation, dept.Type);
    }
    
    [TearDown]
    public void Cleanup()
    {
        Object.Destroy(GameObject.Find("DepartmentManager"));
    }
}
```

---

## 任务2：副市长系统 (4天)

### 任务2.1：ViceMayorPersonality类（第1天）

**文件**：`Assets/Scripts/Government/ViceMayorPersonality.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class ViceMayorPersonality
{
    [SerializeField] public string name;
    [SerializeField] public int age;
    [SerializeField] public string background;
    
    // 性格偏好 (0-1)
    [SerializeField] public float developmentBias = 0.5f;      // 发展 vs 保守
    [SerializeField] public float environmentBias = 0.5f;      // 环保 vs 工业
    [SerializeField] public float welfareBias = 0.5f;          // 福利 vs 效率
    
    // 沟通风格
    [SerializeField] public string[] communicationStyle;        // 急进、温和、现实等
    [SerializeField] public Dictionary<ProposalType, float> proposalWeights;
    
    // AI学习参数
    private Dictionary<ProposalType, float> agreementRates = new Dictionary<ProposalType, float>();
    private float personalityVariance = 0.1f;  // 性格变异度
    
    public ViceMayorPersonality(string personName, int personAge, string personBg)
    {
        name = personName;
        age = personAge;
        background = personBg;
        InitializeProposalWeights();
    }
    
    private void InitializeProposalWeights()
    {
        proposalWeights = new Dictionary<ProposalType, float>
        {
            { ProposalType.Infrastructure, 0.5f + developmentBias * 0.3f },
            { ProposalType.Environment, 0.5f + environmentBias * 0.3f },
            { ProposalType.Welfare, 0.5f + welfareBias * 0.3f },
            { ProposalType.TaxPolicy, 0.5f },
            { ProposalType.Security, 0.5f }
        };
    }
    
    public float EvaluateProposal(ProposalType type)
    {
        if (proposalWeights.ContainsKey(type))
        {
            return proposalWeights[type];
        }
        return 0.5f;
    }
    
    public void AdjustPersonality(ProposalType type, bool accepted, float impact)
    {
        // 根据提案是否被接受调整性格倾向
        float adjustment = accepted ? impact * 0.05f : -impact * 0.05f;
        
        switch (type)
        {
            case ProposalType.Infrastructure:
                developmentBias = Mathf.Clamp01(developmentBias + adjustment);
                break;
            case ProposalType.Environment:
                environmentBias = Mathf.Clamp01(environmentBias + adjustment);
                break;
            case ProposalType.Welfare:
                welfareBias = Mathf.Clamp01(welfareBias + adjustment);
                break;
        }
        
        InitializeProposalWeights();
    }
    
    public string GenerateOpinion(ProposalType type, bool supportive)
    {
        string opinion = "";
        
        if (communicationStyle.Contains("Direct"))
            opinion = supportive ? "这是必要的！" : "我反对！";
        else if (communicationStyle.Contains("Gentle"))
            opinion = supportive ? "我建议考虑这个方案。" : "我有些顾虑...";
        else
            opinion = supportive ? "从数据看这是最优选择。" : "这不符合经济原理。";
        
        return opinion;
    }
}

public enum ProposalType
{
    Infrastructure,
    Environment,
    Welfare,
    TaxPolicy,
    Security,
    Education,
    Healthcare
}
```

---

### 任务2.2：ViceMayor类（第2-3天）

**文件**：`Assets/Scripts/Government/ViceMayor.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class ViceMayor
{
    [SerializeField] private string viceMayorId;
    [SerializeField] private ViceMayorPersonality personality;
    [SerializeField] private List<DepartmentType> managedDepartments;
    
    private float supportLevel = 0.7f;              // 市议会支持度
    private float satisfactionWithGov = 0.65f;     // 对政府的满意度
    private float loyaltyToMayor = 0.5f;           // 对市长的忠诚度
    private Dictionary<ProposalType, int> proposalHistory;
    private int monthsInOffice = 0;
    
    // 事件
    public event Action<float> OnSupportLevelChanged;
    public event Action<string> OnStatementMade;
    
    public ViceMayor(ViceMayorPersonality pers)
    {
        personality = pers;
        viceMayorId = System.Guid.NewGuid().ToString();
        managedDepartments = new List<DepartmentType>();
        proposalHistory = new Dictionary<ProposalType, int>();
    }
    
    // 属性
    public string Name => personality.name;
    public int Age => personality.age;
    public float SupportLevel => supportLevel;
    public float SatisfactionWithGov => satisfactionWithGov;
    public float LoyaltyToMayor => loyaltyToMayor;
    public ViceMayorPersonality Personality => personality;
    public List<DepartmentType> ManagedDepartments => new List<DepartmentType>(managedDepartments);
    
    // 方法
    public void AssignDepartment(Department dept)
    {
        if (dept != null && !managedDepartments.Contains(dept.Type))
        {
            managedDepartments.Add(dept.Type);
            dept.AssignViceMayor(this);
        }
    }
    
    public void RemoveDepartment(DepartmentType type)
    {
        managedDepartments.Remove(type);
    }
    
    public bool ManagesDepartment(DepartmentType type)
    {
        return managedDepartments.Contains(type);
    }
    
    public float GetDepartmentBonus(DepartmentType type)
    {
        // 副市长对其管理部门的加成
        if (ManagesDepartment(type))
        {
            return (supportLevel + loyaltyToMayor) * 0.5f;
        }
        return 0f;
    }
    
    public void RespondToProposal(Proposal proposal, bool accepted)
    {
        // 更新忠诚度
        float impactOnLoyalty = accepted ? 0.05f : -0.03f;
        loyaltyToMayor = Mathf.Clamp01(loyaltyToMayor + impactOnLoyalty);
        
        // 更新性格
        personality.AdjustPersonality(proposal.Type, accepted, 1.0f);
        
        // 记录历史
        if (!proposalHistory.ContainsKey(proposal.Type))
            proposalHistory[proposal.Type] = 0;
        proposalHistory[proposal.Type]++;
    }
    
    public void Update()
    {
        monthsInOffice++;
        
        // 每个月自然衰减支持度（如果不满意）
        if (satisfactionWithGov < 0.5f)
        {
            supportLevel = Mathf.Clamp01(supportLevel - 0.02f);
        }
    }
    
    public float EvaluateProposal(Proposal proposal)
    {
        // 评估副市长对提案的态度
        float evaluation = personality.EvaluateProposal(proposal.Type);
        
        // 加入个人利益因素
        if (managedDepartments.Contains(proposal.AffectedDepartment))
        {
            evaluation += 0.2f;  // 有利于自己的部门
        }
        
        return Mathf.Clamp01(evaluation);
    }
    
    public void MakePressureStatement(string topic)
    {
        string statement = $"{personality.name}发表声明：";
        statement += personality.GenerateOpinion(ProposalType.Infrastructure, UnityEngine.Random.value > 0.5f);
        OnStatementMade?.Invoke(statement);
    }
}
```

---

### 任务2.3：ViceMayorManager类（第4天）

**文件**：`Assets/Scripts/Government/ViceMayorManager.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class ViceMayorManager : Singleton<ViceMayorManager>
{
    private List<ViceMayor> viceMayors;
    private int maxViceMayors = 5;
    private int currentViceMayorCount = 0;
    
    public event Action<ViceMayor> OnViceMayorHired;
    public event Action<ViceMayor> OnViceMayorFired;
    
    public override void Awake()
    {
        base.Awake();
        Initialize();
    }
    
    private void Initialize()
    {
        viceMayors = new List<ViceMayor>();
        HireInitialViceMayors();
    }
    
    private void HireInitialViceMayors()
    {
        // 创建3个初始副市长
        var initialVMs = new[]
        {
            CreateViceMayor("张强", 40, "交通专家", new[] { "Direct", "DataDriven" }, 0.8f, 0.3f, 0.5f),
            CreateViceMayor("李慧", 35, "环保学者", new[] { "Gentle", "Persuasive" }, 0.3f, 0.9f, 0.7f),
            CreateViceMayor("王健", 45, "财政官僚", new[] { "Pragmatic", "Analytical" }, 0.5f, 0.4f, 0.8f)
        };
        
        foreach (var vm in initialVMs)
        {
            HireViceMayor(vm);
        }
    }
    
    private ViceMayor CreateViceMayor(string name, int age, string bg, string[] style, 
        float devBias, float envBias, float welfBias)
    {
        var personality = new ViceMayorPersonality(name, age, bg)
        {
            communicationStyle = style,
            developmentBias = devBias,
            environmentBias = envBias,
            welfareBias = welfBias
        };
        
        return new ViceMayor(personality);
    }
    
    public void HireViceMayor(ViceMayor viceMayor)
    {
        if (currentViceMayorCount >= maxViceMayors)
        {
            Debug.LogWarning("已达到副市长上限！");
            return;
        }
        
        viceMayors.Add(viceMayor);
        currentViceMayorCount++;
        OnViceMayorHired?.Invoke(viceMayor);
        EventManager.Instance.RaiseEvent("ViceMayorHired", viceMayor);
    }
    
    public void FireViceMayor(ViceMayor viceMayor)
    {
        if (viceMayors.Remove(viceMayor))
        {
            currentViceMayorCount--;
            OnViceMayorFired?.Invoke(viceMayor);
            EventManager.Instance.RaiseEvent("ViceMayorFired", viceMayor);
        }
    }
    
    public List<ViceMayor> GetAllViceMayors()
    {
        return new List<ViceMayor>(viceMayors);
    }
    
    public ViceMayor GetViceMayorByName(string name)
    {
        return viceMayors.Find(vm => vm.Name == name);
    }
    
    public int GetCurrentCount()
    {
        return currentViceMayorCount;
    }
    
    public void Update()
    {
        foreach (var vm in viceMayors)
        {
            vm.Update();
        }
    }
}
```

---

## 任务3：预算系统 (3天)

### 任务3.1：BudgetSystem类（第1-2天）

**文件**：`Assets/Scripts/Government/BudgetSystem.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class BudgetSystem : Singleton<BudgetSystem>
{
    private float totalMonthlyBudget = 5000000f;
    private Dictionary<DepartmentType, float> departmentBudgets;
    private Dictionary<DepartmentType, float> budgetWeights;
    private float totalSpent = 0f;
    private float debt = 0f;
    
    public event Action<float> OnBudgetAllocated;
    public event Action<float> OnDebtChanged;
    
    public override void Awake()
    {
        base.Awake();
        Initialize();
    }
    
    private void Initialize()
    {
        departmentBudgets = new Dictionary<DepartmentType, float>();
        budgetWeights = new Dictionary<DepartmentType, float>();
        
        // 初始预算权重 (百分比)
        budgetWeights[DepartmentType.Transportation] = 0.20f;
        budgetWeights[DepartmentType.Environment] = 0.15f;
        budgetWeights[DepartmentType.Education] = 0.15f;
        budgetWeights[DepartmentType.Health] = 0.15f;
        budgetWeights[DepartmentType.Infrastructure] = 0.20f;
        budgetWeights[DepartmentType.Police] = 0.10f;
        budgetWeights[DepartmentType.Finance] = 0.05f;
        
        AllocateBudget();
    }
    
    public void AllocateBudget()
    {
        departmentBudgets.Clear();
        
        foreach (var weight in budgetWeights)
        {
            float budget = totalMonthlyBudget * weight.Value;
            departmentBudgets[weight.Key] = budget;
            
            var dept = DepartmentManager.Instance.GetDepartment(weight.Key);
            if (dept != null)
            {
                dept.SetBudget(budget);
            }
        }
        
        OnBudgetAllocated?.Invoke(totalMonthlyBudget);
        EventManager.Instance.RaiseEvent("BudgetAllocated", this);
    }
    
    public void SetBudgetWeight(DepartmentType type, float weight)
    {
        weight = Mathf.Clamp01(weight);
        budgetWeights[type] = weight;
        
        // 重新分配
        AllocateBudget();
    }
    
    public float GetDepartmentBudget(DepartmentType type)
    {
        return departmentBudgets.ContainsKey(type) ? departmentBudgets[type] : 0f;
    }
    
    public float GetBudgetWeight(DepartmentType type)
    {
        return budgetWeights.ContainsKey(type) ? budgetWeights[type] : 0f;
    }
    
    public float GetTotalBudget()
    {
        return totalMonthlyBudget;
    }
    
    public bool RequestBudgetAdjustment(DepartmentType type, float newAmount)
    {
        // 检查总预算是否足够
        float currentBudgetUsed = 0f;
        foreach (var budget in departmentBudgets.Values)
        {
            currentBudgetUsed += budget;
        }
        
        float adjustment = newAmount - departmentBudgets[type];
        
        if (currentBudgetUsed + adjustment <= totalMonthlyBudget)
        {
            departmentBudgets[type] = newAmount;
            return true;
        }
        
        return false;
    }
    
    public void AddDebt(float amount)
    {
        debt += amount;
        OnDebtChanged?.Invoke(debt);
    }
    
    public void ReduceDebt(float amount)
    {
        debt = Mathf.Max(0, debt - amount);
        OnDebtChanged?.Invoke(debt);
    }
    
    public float GetDebt()
    {
        return debt;
    }
    
    public float GetDebtToIncomeRatio()
    {
        return totalMonthlyBudget > 0 ? debt / (totalMonthlyBudget * 12f) : 0f;
    }
}
```

---

### 任务3.2：单元测试（第3天）

**文件**：`Assets/Scripts/Tests/GovernmentSystemTests.cs`

```csharp
using NUnit.Framework;
using UnityEngine;

public class GovernmentSystemTests
{
    private DepartmentManager deptManager;
    private ViceMayorManager vmManager;
    private BudgetSystem budgetSystem;
    
    [SetUp]
    public void Setup()
    {
        // 创建必要的GameObject和Manager
        var deptManagerGO = new GameObject("DepartmentManager");
        deptManager = deptManagerGO.AddComponent<DepartmentManager>();
        
        var vmManagerGO = new GameObject("ViceMayorManager");
        vmManager = vmManagerGO.AddComponent<ViceMayorManager>();
        
        var budgetGO = new GameObject("BudgetSystem");
        budgetSystem = budgetGO.AddComponent<BudgetSystem>();
    }
    
    [Test]
    public void TestViceMayorPersonality()
    {
        var personality = new ViceMayorPersonality("张强", 40, "交通专家");
        Assert.AreEqual("张强", personality.name);
        Assert.AreEqual(40, personality.age);
    }
    
    [Test]
    public void TestViceMayorCreation()
    {
        var personality = new ViceMayorPersonality("测试", 30, "背景");
        var vm = new ViceMayor(personality);
        Assert.AreEqual("测试", vm.Name);
    }
    
    [Test]
    public void TestViceMayorAssignDepartment()
    {
        var vm = vmManager.GetAllViceMayors()[0];
        var dept = deptManager.GetDepartment(DepartmentType.Transportation);
        
        vm.AssignDepartment(dept);
        Assert.IsTrue(vm.ManagesDepartment(DepartmentType.Transportation));
    }
    
    [Test]
    public void TestBudgetAllocation()
    {
        float totalBudget = budgetSystem.GetTotalBudget();
        Assert.Greater(totalBudget, 0);
        
        float transportBudget = budgetSystem.GetDepartmentBudget(DepartmentType.Transportation);
        Assert.Greater(transportBudget, 0);
    }
    
    [Test]
    public void TestBudgetWeightAdjustment()
    {
        budgetSystem.SetBudgetWeight(DepartmentType.Transportation, 0.3f);
        float weight = budgetSystem.GetBudgetWeight(DepartmentType.Transportation);
        Assert.AreEqual(0.3f, weight, 0.01f);
    }
    
    [Test]
    public void TestDebtSystem()
    {
        budgetSystem.AddDebt(1000000);
        Assert.AreEqual(1000000, budgetSystem.GetDebt());
        
        budgetSystem.ReduceDebt(500000);
        Assert.AreEqual(500000, budgetSystem.GetDebt());
    }
    
    [TearDown]
    public void Cleanup()
    {
        Object.Destroy(GameObject.Find("DepartmentManager"));
        Object.Destroy(GameObject.Find("ViceMayorManager"));
        Object.Destroy(GameObject.Find("BudgetSystem"));
    }
}
```

---

## 任务4：市议会系统基础 (2天)

### 任务4.1：CouncilSystem和VotingSystem（第1-2天）

**文件**：`Assets/Scripts/Government/CouncilSystem.cs`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class CouncilMember
{
    public string name;
    public float supportForMayor;
    public Dictionary<ProposalType, float> preferences;
    
    public CouncilMember(string memberName)
    {
        name = memberName;
        supportForMayor = 0.5f;
        preferences = new Dictionary<ProposalType, float>();
    }
}

public class CouncilSystem : Singleton<CouncilSystem>
{
    private List<CouncilMember> members;
    private int memberCount = 20;
    private VotingSession currentVote;
    private float averageSupportLevel = 0.6f;
    
    public event Action<VotingSession> OnVoteStarted;
    public event Action<VoteResult> OnVoteCompleted;
    
    public override void Awake()
    {
        base.Awake();
        Initialize();
    }
    
    private void Initialize()
    {
        members = new List<CouncilMember>();
        CreateCouncilMembers();
    }
    
    private void CreateCouncilMembers()
    {
        for (int i = 0; i < memberCount; i++)
        {
            var member = new CouncilMember($"议员_{i+1}");
            member.supportForMayor = Random.Range(0.3f, 0.8f);
            members.Add(member);
        }
    }
    
    public void StartVote(Proposal proposal, float mayorSatisfactionLevel)
    {
        currentVote = new VotingSession(proposal, mayorSatisfactionLevel);
        OnVoteStarted?.Invoke(currentVote);
    }
    
    public VoteResult GetVoteResult()
    {
        if (currentVote == null) return null;
        
        int votesFor = 0;
        int votesAgainst = 0;
        
        foreach (var member in members)
        {
            float voteThreshold = member.supportForMayor + (Random.Range(-0.2f, 0.2f));
            
            if (voteThreshold > 0.5f)
                votesFor++;
            else
                votesAgainst++;
        }
        
        bool passed = votesFor > votesAgainst;
        var result = new VoteResult(passed, votesFor, votesAgainst, memberCount);
        
        OnVoteCompleted?.Invoke(result);
        currentVote = null;
        
        return result;
    }
    
    public float GetAverageSupportLevel()
    {
        float total = 0f;
        foreach (var member in members)
        {
            total += member.supportForMayor;
        }
        return total / members.Count;
    }
    
    public void UpdateMemberSupport(float satisfactionChange)
    {
        foreach (var member in members)
        {
            member.supportForMayor = Mathf.Clamp01(member.supportForMayor + satisfactionChange);
        }
    }
}

[System.Serializable]
public class VotingSession
{
    public Proposal proposal;
    public float mayorSatisfactionAtVote;
    public System.DateTime voteTime;
    
    public VotingSession(Proposal prop, float satisfaction)
    {
        proposal = prop;
        mayorSatisfactionAtVote = satisfaction;
        voteTime = System.DateTime.Now;
    }
}

[System.Serializable]
public class VoteResult
{
    public bool passed;
    public int votesFor;
    public int votesAgainst;
    public int totalVotes;
    
    public VoteResult(bool p, int for_votes, int against_votes, int total)
    {
        passed = p;
        votesFor = for_votes;
        votesAgainst = against_votes;
        totalVotes = total;
    }
}

[System.Serializable]
public class Proposal
{
    public string title;
    public string description;
    public DepartmentType affectedDepartment;
    public ProposalType type;
    public float estimatedCost;
    public int estimatedDuration;
    public Dictionary<string, float> expectedImpacts;
    
    public DepartmentType AffectedDepartment => affectedDepartment;
}
```

---

## 提交和集成

### 任务4.2：创建UI面板（第2天）

**文件**：`Assets/Scripts/UI/GovernmentPanelController.cs`

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class GovernmentPanelController : MonoBehaviour
{
    [SerializeField] private GameObject departmentPrefab;
    [SerializeField] private Transform departmentContainer;
    [SerializeField] private Text totalBudgetText;
    [SerializeField] private Text debtText;
    [SerializeField] private Slider budgetSlider;
    
    private DepartmentManager deptManager;
    private ViceMayorManager vmManager;
    private BudgetSystem budgetSystem;
    
    private void Start()
    {
        deptManager = DepartmentManager.Instance;
        vmManager = ViceMayorManager.Instance;
        budgetSystem = BudgetSystem.Instance;
        
        RefreshUI();
    }
    
    public void RefreshUI()
    {
        UpdateBudgetDisplay();
        UpdateDepartmentList();
        UpdateViceMayorList();
    }
    
    private void UpdateBudgetDisplay()
    {
        totalBudgetText.text = $"预算: ¥{budgetSystem.GetTotalBudget():N0}";
        debtText.text = $"债务: ¥{budgetSystem.GetDebt():N0}";
    }
    
    private void UpdateDepartmentList()
    {
        // 清空列表
        foreach (Transform child in departmentContainer)
        {
            Destroy(child.gameObject);
        }
        
        // 添加各部门
        var depts = deptManager.GetAllDepartments();
        foreach (var dept in depts)
        {
            var item = Instantiate(departmentPrefab, departmentContainer);
            // 设置部门信息
        }
    }
    
    private void UpdateViceMayorList()
    {
        var viceMayors = vmManager.GetAllViceMayors();
        // 显示副市长列表
    }
}
```

---

## Phase 2 完成检查清单

完成所有以下任务：

- [ ] **部门系统**
  - [ ] Department类创建和功能完整
  - [ ] DepartmentManager管理7个基础部门
  - [ ] 部门单元测试通过

- [ ] **副市长系统**
  - [ ] ViceMayorPersonality性格系统实现
  - [ ] ViceMayor类和管理系统完整
  - [ ] 副市长可分配部门

- [ ] **预算系统**
  - [ ] BudgetSystem预算分配和调整
  - [ ] 债务系统实现
  - [ ] 预算单元测试通过

- [ ] **市议会系统**
  - [ ] CouncilSystem投票系统实现
  - [ ] VotingSession和VoteResult数据结构
  - [ ] 投票逻辑正确

- [ ] **UI和集成**
  - [ ] GovernmentPanelController UI控制器
  - [ ] 所有系统集成测试通过
  - [ ] 代码提交到Git

- [ ] **代码质量**
  - [ ] 遵循编码规范
  - [ ] 编译无错误无警告
  - [ ] 单元测试覆盖 > 80%

---

## 提交指南

```bash
git checkout feature/phase-2
git add Assets/Scripts/Government/ Assets/Scripts/Tests/ Assets/Scripts/UI/
git commit -m "[Phase-2] feat: Complete government department management system"
git push origin feature/phase-2
```

---

**预计完成时间**：2周（10个工作日）  
**目标代码行数**：3000-4000行  
**单元测试数**：8-10个

---

祝Trae开发顺利！如有问题随时反馈。

