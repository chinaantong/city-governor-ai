# Phase 2 启动指令 - 给Trae

## 🎯 任务：实现政府部门管理系统

**Phase**：Phase 2  
**预计工时**：2周（10个工作日）  
**目标代码行数**：3000-4000行  
**截止日期**：第12-13天

---

## 📋 快速开始

### 第1步：阅读任务文档 (1小时)
```
已上传的详细任务文档：Docs/PHASE_2_TASKS.md
- 包含所有代码模板
- 详细的实现步骤
- 单元测试框架
```

### 第2步：创建开发分支
```bash
git checkout develop
git pull origin develop
git checkout -b feature/phase-2
```

### 第3步：开始开发

按照以下顺序实现所有模块：

---

## 🔨 实现顺序

### 任务1：部门系统 (3天)
**完成项**：
- [ ] `Assets/Scripts/Government/Department.cs` - 部门基类
- [ ] `Assets/Scripts/Government/DepartmentManager.cs` - 部门管理器
- [ ] `Assets/Scripts/Tests/DepartmentTests.cs` - 部门单元测试

**验收标准**：
- 代码编译无误
- 7个基础部门正确创建
- 单元测试通过率 100%

**Git提交**：
```
[Phase-2] feat(Government): Implement department system
```

---

### 任务2：副市长系统 (4天)
**完成项**：
- [ ] `Assets/Scripts/Government/ViceMayorPersonality.cs` - 性格系统
- [ ] `Assets/Scripts/Government/ViceMayor.cs` - 副市长类
- [ ] `Assets/Scripts/Government/ViceMayorManager.cs` - 副市长管理器

**验收标准**：
- 3个初始副市长正确创建
- 性格参数可正确调整
- 副市长可分配多个部门

**Git提交**：
```
[Phase-2] feat(Government): Implement vice mayor system
```

---

### 任务3：预算系统 (3天)
**完成项**：
- [ ] `Assets/Scripts/Government/BudgetSystem.cs` - 预算管理系统
- [ ] `Assets/Scripts/Tests/GovernmentSystemTests.cs` - 政府系统测试

**验收标准**：
- 预算正确分配给各部门
- 预算权重可调整
- 债务系统正常运作

**Git提交**：
```
[Phase-2] feat(Government): Implement budget system
```

---

### 任务4：市议会系统 (2天)
**完成项**：
- [ ] `Assets/Scripts/Government/CouncilSystem.cs` - 市议会系统
- [ ] `Assets/Scripts/UI/GovernmentPanelController.cs` - 政府UI控制器

**验收标准**：
- 20个议员正确创建
- 投票逻辑正确运作
- UI面板可显示部门信息

**Git提交**：
```
[Phase-2] feat(Government): Implement council and UI system
```

---

## ⚠️ 重要注意事项

### 1. 代码规范
- 遵循 `Docs/CODING_STANDARDS.md` 中的规范
- 所有方法添加XML文档注释
- 命名必须使用PascalCase或camelCase

### 2. 测试覆盖率
- 每个主要类需要单元测试
- 测试覆盖率需要 > 80%
- 所有测试必须通过

### 3. Git工作流
- 每个任务完成后提交一次
- 提交消息格式：`[Phase-2] feat(Component): Description`
- 定期push到远程分支

### 4. 编译检查
- 每次提交前检查编译是否无误
- 不允许有编译警告
- 使用 `#pragma warning disable` 需要有充分理由

---

## 📝 提交模板

```bash
# 部门系统完成后
git add Assets/Scripts/Government/Department*.cs Assets/Scripts/Tests/DepartmentTests.cs
git commit -m "[Phase-2] feat(Government): Implement department system

- Add Department class with budget and efficiency
- Add DepartmentManager with 7 basic departments
- Add unit tests for department functionality
- All tests passing with > 95% coverage"

# 副市长系统完成后
git add Assets/Scripts/Government/ViceMayor*.cs
git commit -m "[Phase-2] feat(Government): Implement vice mayor system

- Add ViceMayorPersonality for character traits
- Add ViceMayor class with department assignment
- Add ViceMayorManager with initial hiring
- Support for personality learning and adaptation"

# 预算系统完成后
git add Assets/Scripts/Government/BudgetSystem.cs Assets/Scripts/Tests/GovernmentSystemTests.cs
git commit -m "[Phase-2] feat(Government): Implement budget system

- Add BudgetSystem with budget allocation
- Add budget weight adjustment
- Add debt tracking system
- All integration tests passing"

# 市议会系统完成后
git add Assets/Scripts/Government/CouncilSystem.cs Assets/Scripts/UI/GovernmentPanelController.cs
git commit -m "[Phase-2] feat(Government): Implement council and UI system

- Add CouncilSystem with 20 members
- Add voting session and vote result tracking
- Add GovernmentPanelController UI
- Complete Phase 2 implementation"

# Phase 2完全完成
git push origin feature/phase-2
```

---

## 🎯 完成检查清单

完成时需要检查：

- [ ] 所有4个任务完成
- [ ] 代码编译无误无警告
- [ ] 所有单元测试通过
- [ ] 代码覆盖率 > 80%
- [ ] 遵循编码规范
- [ ] 所有改动已提交到Git
- [ ] PR准备好合并

---

## 📞 遇到问题？

1. **检查PHASE_2_TASKS.md** - 有详细的代码模板
2. **查看Architecture.md** - 了解系统设计
3. **参考CODING_STANDARDS.md** - 编码规范
4. **查看Phase 1代码** - 参考实现模式

---

## 🚀 开始开发！

现在您可以开始Phase 2的开发了！

```bash
# 切换到feature/phase-2分支
git checkout feature/phase-2

# 开始编码
# ... 实现部门系统 ...
```

**祝您开发顺利！** 🎮

---

**文档版本**：v1.0  
**生成日期**：2025年6月8日  
**目标完成**：第12-13天
