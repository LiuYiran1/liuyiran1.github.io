# CJ_interpreter 实验报告（Lab1 / Lab2）

## 摘要
- 目的：总结在 CJ_interpreter 项目中实现的运行时作用域与活动记录、静态作用域扫描（access link）、函数值与调用、控制流（while/break/continue/return）以及语义检查与错误定位的工作。
- 范围：实现思路、关键数据结构、控制流处理、遇到的问题与解决方案、测试/运行建议与后续改进方向。
- 参考代码：项目关键文件位于 `src/`（示例： `src/executor/ActivationRecords.cj`, `src/executor/AccessLinkBuilder.cj`, `src/executor/Evaluator.cj`, `src/executor/Record.cj`, `src/executor/Value.cj`）。报告： `report1.md`, `report2.md`。

---

## 目标与设计原则

- 目标：
  - 提供清晰的运行时作用域表示，支持块作用域与函数栈帧。
  - 实现静态作用域（access link）分析以支持闭包/捕获语义。
  - 支持函数为一等公民（作为值传递）、参数绑定、返回值，以及严格的语义检查（类型、不可变性等）。
  - 正确处理控制流语义（while、break、continue、return），并在发生错误时能准确定位 AST 节点。

- 设计原则：
  - 抽象化栈帧（`ActivationRecords`）以统一块和函数的操作。
  - 区分静态链（`accessLink`）与动态链（`controlLink`）。
  - 将错误与位置绑定（抛出 `CjcjRuntimeErrorWithLocation`），便于精确报错。
  - 模板（静态描述）与运行时实例分离，避免栈帧共享状态造成污染。

---

## 关键组件与实现

### ActivationRecords（活动记录）
- 文件：`src/executor/ActivationRecords.cj`
- 描述：抽象类 `ActivationRecords` 封装作用域中的记录集合与常用操作（`addRecord`、`getRecord`、`setRecord` 等），并包含用于链式查找的 `accessLink` 与 `controlLink`。
- 子类：
  - `BlockActivationRecords`（块级作用域）
  - `FuncActivationRecords`（函数栈帧），增加 `return_addr`、`save_returnv_addr`、`arguments` 等字段，用于返回值与参数管理。
- 要点：在查找/赋值时区分静态和动态链；记录方法接收 `Node` 以便报错定位。

### Record（变量记录）
- 文件：`src/executor/Record.cj`
- 描述：每个变量/函数名由一条 `Record` 表示，包含 `name`、`value`、`recordType`、`keyword`（`var`/`let`）等。
- 行为：`updateValue` 对赋值进行保护（例如函数名不可被重新赋值），在违反语义时抛出带位置信息的错误。

### AccessLinkBuilder（静态作用域扫描）
- 文件：`src/executor/AccessLinkBuilder.cj`
- 作用：在执行前对 AST 进行静态扫描，构建每个 `Block` / `FuncDecl` 的静态父作用域模板（`accessLink` 映射），并标识空函数体等信息，产出 Node → `ActivationRecords` 的模板映射（`accessLinkMap`）。
- 要点：将扫描结果作为蓝图，在运行时基于蓝图按需实例化，避免共享实例。

### Value 与函数表示
- 文件：`src/executor/Value.cj`
- 描述：在 `Value` 枚举中新增 `VFunction(FuncValue)` 与 `VMain(MainValue)`。`FuncValue` 保存 `FuncDecl` 与可选 `capturedEnv`（捕获环境）。
- 目的：使函数成为一等值，支持闭包与传参。

### Evaluator（解释器核心）
- 文件：`src/executor/Evaluator.cj`
- 内容要点：
  - 程序启动时先注册全局函数，再初始化全局变量，保证函数不会捕获随后的未声明变量。
  - 变量声明通过创建新的嵌套 `BlockActivationRecords` 保持声明顺序，防止嵌套函数捕获到定义时不可见的变量。
  - `visit(CallExpr)` 实现参数检查、赋参、调用，并捕获 `ReturnException` 获取返回值。
  - 赋值与查找利用 `ActivationRecords` 的静/动链规则，在非本地修改时检查可变性与合法性。

### 控制流实现（while / break / continue / return）
- 实现策略：
  - `break` 与 `continue` 通过抛出特定 `CjcjRuntimeErrorWithLocation` 实现（携带 `ErrorCode.BREAK_OUTSIDE_LOOP` / `CONTINUE_OUTSIDE_LOOP`）。
  - `while` 捕获这些异常并在捕获路径中回退 `recordsStack`（使用进入时记录的栈深度）并恢复 `curRecords`，然后执行跳出或继续逻辑。
  - `return` 使用 `ReturnException` 向上冒泡，`visit(CallExpr)` 捕获并处理。
- 优点：异常传播简化跨层控制流跳转实现，并便于在捕获处统一恢复作用域状态。

### 特殊边界处理
- `Int64.min`（`-9223372036854775808`）处理：在 `UnaryExpr` 中对该字面量做特殊检测并抛出 `NEG_OVERFLOW`，避免解析时溢出。
- 非本地可变变量访问约束：通过 `ActivationRecords` 的静态查找/赋值接口加入 `isNonLocal` 标志与递归传递逻辑，实现内层函数对外层可变非局部变量访问的语义检查。

---

## 遇到的问题与解决方法（要点）

1. 模板复用导致栈帧污染
   - 问题：静态扫描阶段直接生成的 `ActivationRecords` 实例若在运行时复用，会导致递归时多栈帧共享同一实例，相互污染。
   - 解决：将扫描产物视为蓝图，在运行时复制或实例化独立栈帧；闭包捕获时记录必要变量而非引用模板实例。

2. 异常实现控制流导致作用域未退栈
   - 问题：异常提前退出 block 会跳过 block 的退出逻辑，导致作用域未退栈。
   - 解决：在 `while` 处记录进入时的栈深度，捕获 `break/continue` 后手动回退 `recordsStack` 并恢复 `curRecords`。

3. 字面量溢出（Int64.min）
   - 问题：解析极端字面量导致溢出。
   - 解决：在 `UnaryExpr` 节点检测并抛出明确语义错误。

---

## 测试与运行
- 仓库内包含测试及错误样例：`tests/`、`input1/`、`input2/`，以及 `final/lab3ref+project/tests/`（OOP 测试）。
- 本地运行示例（根据项目实际脚本调整）：
```bash
# 若仓库提供脚本
bash test.sh
# 或
bash run_test.sh
```
- 建议：先运行仓库自带测试脚本验证整体功能，再针对闭包、递归、异常路径编写单元测试。

---

## 已知问题与限制
- 访问链构建与实例化：当前实现仍存在“边解释边构建 accessLink”的情况；若不先做完整静态分析并在运行时严格实例化，复杂闭包场景可能出现边界行为。
- 模板实例化必须谨慎：`AccessLinkBuilder` 生成的模板不能直接复用于多个栈帧，需按需复制或实例化。

---

## Final 实现补充：面向对象特性与垃圾回收

这一部分对应 `final/lab3ref+project` 中的最终实现，是整个项目中很重要的一块。它在前面解释器与作用域系统的基础上，进一步补齐了类、接口、继承、子类型、动态派遣、`super`、对象初始化以及 tracing GC 的完整运行时链路。

### 1. 类型系统重构：类、接口、`Any` / `Nothing` / `Option`

- 文件：`final/lab3ref+project/src/executor/types/Types.cj`
- 核心变化：
  - 增加了 `TClass` 与 `TInterface` 两种引用类型。
  - `TClass` 维护 `superClass`、`interfaces`，并引入 `isAbstract`、`isOpen` 标记。
  - `TInterface` 维护 `superInterfaces`，形成接口继承树。
  - `TAny` 和 `TNothing` 作为子类型系统中的上界与下界，配合 `Option`、函数类型共同参与子类型判断。
- 子类型判定：
  - `TypeManager.isSubtype` 递归检查类继承链、接口继承链、函数类型协变/逆变关系，以及 `Option` 的协变关系。
  - 对函数类型采用参数逆变、返回值协变的规则，这使得函数可以更自然地参与赋值与传参检查。
- 实际意义：
  - 这部分决定了 `is` / `as`、函数参数检查、构造器调用、父类与接口赋值等所有 OOP 语义的合法性。

### 2. 继承与接口检查：分阶段静态检查

- 文件：`final/lab3ref+project/src/executor/types/TypeChecker.cj`
- 我做的事情：
  - 把类型检查拆成多个阶段：`Stage1Checker`、`InheritChecker`、`Stage2Checker`。
  - 在 `Stage1Checker` 中先收集类与接口的类型信息。
  - 在 `InheritChecker` 中专门处理继承关系、修饰符、父类/父接口绑定。
  - 在 `Stage2Checker` 中检查成员覆盖、冲突、抽象成员实现等更细粒度的语义。
- 关键检查点：
  - `SUPER_TYPE_NOT_EXTENSIBLE`：父类型不是可扩展类型时直接报错。
  - `MISPLACED_SUPERCLASS`：父类必须放在 `:<` 后的第一个位置。
  - `UNIMPLEMENTED_ABSTRACT_MEMBER`：非抽象类必须实现所有抽象成员。
  - `INHERITED_FUNC_TYPE_CONFLICT`：继承来的同名函数参数类型不一致。
  - `INHERITED_FUNC_MULTIPLE_IMPLEMENTATION`：多个父类/接口提供了同名具体实现，而当前类没有覆盖时直接报错。
- 设计价值：
  - 继承合法性、接口继承合法性和成员冲突不是在运行时补救，而是在静态检查阶段一次性拦截，这让后续动态派遣的实现更稳定。

### 3. 类型上下文与方法收集：为虚表和继承冲突服务

- 文件：`final/lab3ref+project/src/executor/types/TypeContext.cj`
- 关键设计：
  - `TypeContext` 保存类型层面的变量、函数、类型定义以及上下文层级。
  - `FuncDeclWithTypeContext` 记录函数声明及其可见外层变量集合，支持函数类型与作用域分析。
  - `TypeManager.getAvailableMethods` 递归收集父类与接口的方法，并在当前类型定义处进行覆盖，形成方法可见性与冲突判断的基础。
- 我在这里解决的问题：
  - 既要知道一个类“能看到哪些方法”，又要知道“哪些方法是继承来的、哪些是当前类声明的、哪些是抽象声明”。
  - 这套数据结构也为后续动态派遣建立了统一的类型视图。

### 4. 对象上下文、`this` / `super` 和动态派遣

- 文件：`final/lab3ref+project/src/executor/values/Context.cj`
- 对象上下文：
  - `Context` 不只是变量容器，还绑定了对应的 `TypeContext`，从而把“运行时对象状态”和“静态类型定义”连接起来。
  - 对象的变量值、函数值、`this` 引用、对象 ID、转发地址等都存储在 `Context` 中。
- 动态派遣：
  - `syncFuncsForObject()` 会根据类层级同步函数到对象上下文。
  - 同步顺序是“父类 -> 接口 -> 当前类”，因此子类同名方法会自然覆盖父类或接口的方法。
  - 解释执行时直接用已经建立好的函数表查找成员函数，不需要每次重新做复杂的继承搜索。
- `this` / `super`：
  - `this` 通过对象上下文回溯获取当前实例。
  - `super` 不能简单地按当前对象的虚表找，否则会错误地拿到子类覆盖后的实现；因此我在调用 `super` 时专门回退到定义当前 `init` 的类的父类上下文中取原始初始化函数。

### 5. 构造器初始化顺序与默认 `super()` 处理

- 文件：`final/lab3ref+project/src/executor/values/Evaluator.cj`
- 实现方式：
  - 在 `apply` 处理 `init` 时，先把 `this` 绑定到当前对象。
  - 如果构造函数体第一条语句不是显式 `super()` 或 `this()`，则自动查找父类的 `init` 并先执行它。
  - 这样可以兼容“用户省略 `super()`”的情况，同时满足构造链必须完整调用的约束。
- 初始化后检查：
  - 构造完对象后调用 `checkInitialized()`，确保成员变量都已经被正确初始化，否则报 `MEMBER_NOT_INITIALIZED_AFTER_INIT`。
- 这部分的价值：
  - 保证对象从构造到可用的生命周期是完整的，不会出现未初始化成员被读取的隐患。

### 6. `is` / `as` 与运行时类型判断

- 文件：`final/lab3ref+project/src/executor/values/Evaluator.cj`
- 实现思路：
  - `is`：取表达式的运行时类型，再用 `TypeManager.isSubtype` 判断目标类型是否为其父类型或兼容类型，返回布尔值。
  - `as`：如果可转换，则返回 `Option(Some(v))`，否则返回 `Option(None)`。
- 关键点：
  - 运行时类型判断不是只依赖静态注解，而是基于实际值的运行时类型完成，这样才能配合继承和接口使用。

### 7. 左值求值：支持成员赋值的精确定位

- 文件：`final/lab3ref+project/src/executor/types/LValue.cj`，以及运行时版本 `src/executor/values/LValue.cj`
- 问题：
  - 像 `a.b.c = 10` 这样的赋值，不能只求值成一个普通值，而必须定位到“最终要写入的对象上下文和字段名”。
- 解决：
  - 单独实现 `LValueEvaluator`，让它返回一个 `LValue`，其中包含 `Context` 与字段名 `name`。
  - 这样赋值时就能准确写回对象实例中的成员，而不是误把左值当普通表达式求值。

### 8. 垃圾回收：基于 tracing 的 Cheney 风格迁移

- 文件：`final/lab3ref+project/src/executor/values/GCManager.cj`
- 整体流程：
  1. 切换 from-space 与 to-space。
  2. 扫描根集，把可达对象迁移到新空间。
  3. 使用 worklist 迭代扫描对象图，持续搬移间接可达对象。
  4. 更新所有对象引用和转发地址。
  5. 调用底层 `memory.deallocateExcept` 清理旧空间无效对象。
- 具体实现：
  - `allocate` 在对象分配前检查空间，空间不足时触发 GC。
  - `countMembers` 统计对象实际占用大小，确保搬移时地址和容量正确。
  - `moveValue` 不只处理对象本身，也递归处理 `Option` 和闭包中捕获的环境。
- 关键点：
  - 对象和函数闭包都可能持有对堆对象的引用，所以 GC 扫描必须把函数值中的捕获环境也当作引用图的一部分。

### 9. 闭包与 GC 的结合

- 文件：`final/lab3ref+project/src/executor/values/GCManager.cj`
- 处理方式：
  - 当扫描到 `Value.VFunc` 且其携带闭包上下文时，需要递归迁移这个环境及其内部对象。
  - 这避免了闭包捕获的对象在 GC 后变成悬挂引用。
- 这部分很重要：
  - 说明你不仅实现了“对象的 GC”，还处理了“函数作为一等值时的堆引用安全性”，这是整个运行时设计里非常关键的一环。

### 10. 这部分实现的总体价值

- 把解释器从“基本表达式 + 作用域”扩展为“支持类、接口、继承、多态、对象实例化、类型检查和 GC 的完整运行时”。
- 通过 `TypeContext` / `Context` 的双层结构，把静态类型信息和运行时对象状态联结起来。
- 通过 `syncFuncsForObject` 和方法收集机制实现动态派遣，通过 `LValueEvaluator` 精确支持成员赋值，通过 `GCManager` 保证对象和闭包的内存安全。

---

## 后续改进建议
- 程序入口完整运行 `AccessLinkBuilder`，生成不可变静态元数据，运行时基于其创建隔离栈帧。
- 将类型与赋值规则进一步向 `Record.updateValue` 或 `ActivationRecords` 内聚合，减少 `Evaluator` 中重复校验逻辑。
- 为栈帧进入/退出引入统一接口（`enterFrame` / `exitFrame`），在异常路径确保执行以避免手动回退栈的脆弱性。
- 扩展测试覆盖：闭包捕获顺序、嵌套函数对外层变量读/写、异常路径下的作用域恢复、极端数值字面量等场景。
- 如果继续完善 final 版本，可以进一步补充更多边界测试：
  - `super.member` 和 `this.member` 的覆盖差异。
  - 多接口同名默认实现的冲突场景。
  - 闭包捕获对象在多轮 GC 后的稳定性。
  - 构造器省略 `super()` 时的初始化顺序。

---

## 答辩要点（可直接复述）
- 实现了以 `ActivationRecords` 为核心的运行时作用域体系，区分静态链与动态链，支持函数闭包与捕获。
- 支持函数为一等公民、参数绑定与返回机制；使用异常处理跨层控制流并在捕获处恢复作用域。
- 修复了极端字面量溢出问题，并为更严格的静态分析与闭包语义留出扩展点。

---

## 附：参考文件
- 报告文档： `report1.md`, `report2.md`
- 关键源码：
  - `src/executor/ActivationRecords.cj`
  - `src/executor/AccessLinkBuilder.cj`
  - `src/executor/Evaluator.cj`
  - `src/executor/Record.cj`
  - `src/executor/Value.cj`


---

若需要，我可以：
- 把此文档转换为更正式的 Markdown（已保存为 `REPORT_complete.md`），或导出为 Word/PDF；
- 基于此生成答辩 PPT 提纲（每页一节）；
- 将文中要点替换为你仓库中确切代码片段（我可从文件中摘取若干关键函数示例并插入文档）。
