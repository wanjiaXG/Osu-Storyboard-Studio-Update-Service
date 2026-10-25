# 编写模板

在模板库选择「管理模板 → 新建模板」，即可得到可运行的源码目录、动画素材和 IDE 工程。编辑器监视这个目录；保存 `.cs`、素材或 `module.json` 后约 300 ms 开始更新。源码编译使用 C# 14 / .NET 10 与固定 Roslyn 5.0.0，不执行 MSBuild、NuGet 还原、分析器或源码生成器。使用 IDE 时可以自行安装 .NET 10 SDK。

发布的模板与源码模板实现同一接口：

```csharp
using SbStudio.Authoring;
using SbStudio.Authoring.Templates;

public sealed class MyTemplate : ITemplateModule
{
    private ParameterKey<int> count = null!;
    private ParameterKey<AssetReference> image = null!;

    public void Define(TemplateDefinitionBuilder d)
    {
        d.Name("My particles").Category("Particles").AdjustableDuration(5000);
        count = d.Integer("count").Default(20).Range(1, 1000).Label("Count").Key;
        image = d.Image("image").Default(AssetReference.Package("assets/particle.png")).Key;
    }

    public void Generate(TemplateContext ctx, StoryboardBuilder b)
    {
        // This constant is deliberately not exposed in the editor.
        const double speed = 50;
        var random = ctx.Random(123);
        for (int i = 0; i < ctx.Parameters.Get(count); i++)
        {
            ctx.Cancellation.ThrowIfCancellationRequested();
            double x = random.Between(0, 640);
            b.Sprite(ctx.Parameters.Get(image))
                .Move(0, ctx.Duration, x, 0, x, speed * ctx.Duration / 1000)
                .Scale(0, .3)
                .Fade(0, ctx.Duration, 1);
        }
    }
}
```

目录中的 `module.json` 指定稳定的包 ID、模板 ID 和入口类型，不扫描所有类来猜入口：

```json
{
  "formatVersion": 2,
  "packageId": "example.particles",
  "packageVersion": "1.0.0",
  "sdkMajor": 2,
  "minHostVersion": "2.0.0",
  "entryAssembly": "lib/Template.dll",
  "kind": "template",
  "entries": [{ "moduleId": "example.particles.snow", "entryType": "MyTemplate" }],
  "dependencies": []
}
```

把图片放在 `assets/particle.png`。可将代码拆为多个 `.cs`，包括自己的扩展方法；`bin/`、`obj/`、`.git/`、`.build/`、`.vs/` 不参与源码编译。纯托管依赖放在 `lib/` 并显式列入 `dependencies`；不要携带另一份 SDK。一个包可声明多个模板入口。

`Define` 只描述参数和输入，必须保持确定性。使用稳定参数 ID，不用显示名称作为 ID。可声明 Integer、Number、Time、Toggle、Choice、Text、Multiline、Point2、Color、Image、Animation、File，配合范围、默认值、单位、分组、条件显示。只有声明的参数才可以在软件中修改；未声明种子/速度不会自动出现控件。Choice/Toggle 可作为显示条件。类型与参数 ID 应保持版本兼容，未知参数会保留以便后续恢复。

`Generate` 每次收到新的模块实例、参数快照和独立 Builder；不能依赖上次运行的字段或全局生成列表。用 `ctx.Random(seed)` 获得可复现的随机序列。检查 `ctx.Cancellation` 可快速响应用户取消；不合作的代码会由宿主结束工作进程。`ctx.Log(...)` 的输出有长度上限，可在实例属性中查看。

Builder 保留链式 API：Sprite/Animation、嵌套 Group、Move/MoveX/MoveY、Fade、Scale/ScaleXY、Rotate/RotateRad、Color、FlipX/FlipY/Additive、Loop、Trigger 及编组深度命令。Loop/Trigger 的回调只能添加自己的命令，不允许嵌套命令块；封存后的 Builder 不能继续修改。默认图层为 Background。输入的时间单位为毫秒，转入编辑器时四舍五入到整数毫秒，半数远离零；Rotate 使用度，RotateRad 使用弧度。

全部命令使用模板局部时间，编辑器物化时只加一次放置时间。只有调用 `Uses(TemplateInputs.AbsoluteTime)` 的模板才应读取绝对时间；它们随实例时间移动重新生成。Timing/HitObjects/Metadata 也需要在 `Define` 用 `Uses(...)` 声明。输入难度在添加时绑定，切换预览难度不会更换这个输入。标准物件坐标用 `ctx.StoryboardPosition(hit)` 转成故事板坐标（+64,+56）；其他模式保留原坐标，由作者解释。

`.AdjustableDuration(defaultMs)` 允许用户调整时长；固定内容模板不提供通用的时长编辑。用户加在模板编组外层的运动与生成内容独立，刷新不会覆盖。

资源用 `AssetReference.Project(...)`、`.Package(...)` 或 `ctx.Assets.Write(...)` 返回的生成引用。只使用相对路径，不使用绝对路径或 `..`。`ctx.Assets.Read` 与 `ImageSize` 可以读取资源；输出所引用的图片和动画帧会收集到工程的 `assets/generated/templates/`，因此解除绑定后也能使用。动画路径是基名，例如 `assets/frame.png`，实际帧为 `frame0.png`、`frame1.png`。

在模板库右键开发模板，选择「导出模板包」。生成的 `.sbtemplate` 包含 DLL、静态参数清单、声明依赖、资源和文件哈希，安装时校验并固定版本。修改内容发布时应递增模板包版本号。同版本的不同内容按内容哈希分别保存，旧副本不会被覆盖。更新安装包不会自动升级旧工程实例。源码、DLL 与运行的输入相同时应得到相同结果。

### 单 DLL 分发

模板和编组附加效果也可以只发一个 DLL：放进程序目录的 `Templates/`（模板）或 `Effects/`（效果）即可，下次刷新模板库/效果库时自动出现，无需安装或另行信任——放进程序目录本身就表示信任它。DLL 内嵌以下资源，资源名就是包内路径：

- `module.json`：同上文清单，不需要写 `entryAssembly`（取 DLL 文件名），单 DLL 模块不能声明 `dependencies`；
- `assets/...`、`presets/...`：图片、动画帧和随包预设。

用 MSBuild 构建时在工程末尾导入 `SbStudio.Authoring.Sdk/build/SbStudio.Module.targets`，它会把工程目录下的 `module.json`、`assets/**`、`presets/**` 按上述名称嵌入。素材也可以不嵌入，而放在 DLL 旁边与它同名的文件夹里，例如 `Templates/MySnow.dll` 与 `Templates/MySnow/assets/a.png`；同一路径不能既嵌入又外置。编辑器首次见到某个 DLL（或其外置文件有变化）时，用独立进程读取定义并展开为普通的已校验包，缓存在 `%AppData%\OsuStoryboardStudio\authoring\modules\`；DLL 放错文件夹（例如效果放进 `Templates/`）会在库的错误列表中提示，不会加载。添加到工程时与其他包一样固定副本，之后删掉或更换 DLL 不会改变已有实例。

内置的飘雪、背景推拉、节拍闪光与震动各自是一个工程、一个 DLL。清单可用 `"replaces": ["旧包 ID"]` 声明它接替的旧包：旧包中同一 `moduleId` 的实例会把它当作新版本，仍需在界面上显式升级。

模板身份由 `PackageId` 与 `ModuleId` 共同确定，显示名称可以重复或修改；升级时保留原 ID。新建开发模板会生成包含完整 UUID 的包 ID，请勿复制该 ID 用于另一个独立模板。`InstanceGuid` 仅表示工程中的一个模板实例，内容哈希表示包的具体修订，均不代替模板身份。

模板库将同一身份的版本合并展示，项目继续使用保存的版本。有新版时可选择仅供后续添加使用，或明确升级某个已有实例；这些操作支持撤销。其他版本可从右键「版本列表」添加，原包仍保留。

“仅供后续添加使用”不会改变已有实例。即使模板库已选择新版，只要还有可升级的旧实例，仍会保留“升级已有实例”入口；升级窗口会优先选中失败的旧实例，确认后重新生成并切换该实例。当前 SDK 的程序集身份为 `2.0.0.0`，旧 SDK 0.1 与 1.x 的包均不能直接加载；内置模板可通过升级已有实例切换到新版包，第三方模板需使用当前 SDK 重新构建。编辑器会在执行前检查依赖并提示处理方法。

执行故障不会替换上次成功输出。保存可以同时保留当前参数和旧画面，重开后会重新生成。转换普通编组保留整个生成动画与外层运动；转换后编辑的内容不会反推成模板参数。分享完整工程时包含 `templates/` 与 `assets/`；游戏导出只输出故事板和所需素材。

第三方代码运行在独立进程，但拥有当前用户的文件和网络权限。请只安装和授权可信来源。

模板包可以附带 `presets/*.sbpreset`，用户在实例的「管理模板 → 随包预设」中选择。先在软件中保存一份参数预设，再放入源码目录的 `presets/` 后打包即可；预设只保存公开参数与可迁移素材，不携带放置时间、外层运动或信任授权。内置模板各附一套示例预设。

SDK V2 的程序集身份固定为 `SbStudio.Authoring.Sdk, Version=2.0.0.0`，与编辑器产品版本独立。托管依赖的文件名须匹配程序集简单名称，同包内不得声明重名依赖，也不得私带 SDK DLL。早期试验版 SDK 0.x 编译的模板请重新打包。

源码修改会使用新工作进程；同一版本的工作进程也会定期回收，不能依赖静态变量跨次持久化。编译结果按源码、编译器和 SDK 身份缓存并校验完整性。请求临时目录在资源收集完成后清理；工程历史、保存文件和视频导出引用的结果会保留。缓存清理不会删除已物化的素材或工程固定模板包。

## 迁移到 SDK V2

旧 SDK 编译的二进制包不能在 SDK V2 中运行。内置模板的已有实例请在实例上明确升级到当前内置包（现为各自独立的 3.0.0 包）；第三方包请联系作者用新 SDK 重新打包。旧开发目录提供「迁移源码副本」：选择一个新的目标目录（默认是同级的 `<名称>-v2`），原目录保持不变。宿主在副本中改写 C# 语法和 SDK 工程引用，转换清单并显示编译诊断；字符串和注释不会改动。

模板 API 除下表的改名外保持原有成员与签名。GroupBuilder 只提供 M/MX/MY/S/R/Z/RX/RY 命令；编组上没有不透明度、颜色、参数、矢量缩放、循环和触发器。曲线端点必须为 (0,0) 和 (1,1)。

SDK V2 同时提供模板与编组附加效果；`EffectGroup` 的接口与可编译示例见下文「编组附加效果 SDK」。

下表由 SDK 内的 `SourceMigrationMap` 生成，源码迁移使用同一张表。

| V1 | V2 |
| --- | --- |
| `SbStudio.Effects.Sdk` | `SbStudio.Authoring.Sdk` |
| `SbStudio.Effects.BuiltIn` | `SbStudio.Templates.BuiltIn` |
| `SbStudio.Effects` | `SbStudio.Authoring` |
| `IEffectModule` | `ITemplateModule` |
| `EffectContext` | `TemplateContext` |
| `EffectDefinitionBuilder` | `TemplateDefinitionBuilder` |
| `EffectDefinition` | `ModuleDefinition` |
| `EffectInputs` | `TemplateInputs` |
| `IEffectAssets` | `IModuleAssets` |
| `EffectJson` | `ModuleJson` |
| `EffectEntry` | `ModuleEntry` |
| `EffectRequestIdentity` | `ModuleRequestIdentity` |
| `EffectDiagnostic` | `ModuleDiagnostic` |
| `EffectId` | `ModuleId` |

模板 Builder 与模板类型位于 `SbStudio.Authoring.Templates`；共用的参数、资源、JSON 与协议位于 `SbStudio.Authoring`。

## 编组附加效果 SDK

效果实现 `SbStudio.Authoring.GroupEffects.IGroupEffectModule`，只为传入的 SDK `EffectGroup` 追加组命令，不创建或替换内容。每次请求得到新的输出列表，名称、中性点和已声明的目标摘要/成员树均只读。以下例子可作为包的 `Example` 入口：

```csharp
using SbStudio.Authoring;
using SbStudio.Authoring.GroupEffects;

public sealed class Example : IGroupEffectModule
{
    private ParameterKey<double> amplitude = null!;
    public void Define(GroupEffectDefinitionBuilder definition)
    {
        amplitude = definition.Number("amplitude").Default(8).SuggestedRange(0, 50).Key;
        definition.Seed("seed");
    }
    public void Apply(GroupEffectContext context, EffectGroup group)
    {
        int middle = GroupEffectRules.Milliseconds(context.Duration / 2);
        if (middle <= 0 || middle >= context.Duration) return;
        double value = context.Parameters.Get(amplitude);
        group.Move(0, middle, 0, 0, value, 0, Ease.SineInOut)
            .Move(middle, context.Duration, value, 0, 0, 0, Ease.SineInOut);
    }
}
```

包清单使用 `formatVersion: 2`、`sdkMajor: 2`、`kind: "groupEffect"`；入口字段仍为 `moduleId`/`entryType`，导出扩展名为 `.sbfx`。一个包只包含一种 kind。效果与模板库分别按清单种类过滤，改扩展名不会改变种类。

链式 API 只有 M/MX/MY/S/R/Z/RX/RY，以及附在上一条命令上的 `Curve(...)`；`Append(OutputCommand)` 同样执行校验。Move 是相对中性点的位移；Scale 中性值为 1，其他通道为 0。Rotate 使用度，RotateRad 使用弧度。Z/RX/RY 会使整棵树进入三维烘焙，导出缩放为 V，F/C 也可能重新拟合。

局部时间为 `[0, Duration]`，统一半数远离零取整。每个输出通道首条的起始值与末条的结束值必须中性；相邻段可共享端点但不可重叠，M 与 MX/MY 不可冲突；重复同毫秒零长度点被拒绝，普通边界点排在随后区段之前。曲线必须包含 `(0,0)` 与 `(1,1)` 两端。最多 100,000 条命令、3,600,000 ms，超限或不合法时整次失败，不补间、不截断。

`Seed(id)` 是实例种子：创建实例时随机分配完整 32 位整数并持久化，预设排除该参数；普通 `Integer` 仍是普通参数。范围表示常用滑块范围，不限制效果数值自由输入；模块须检查真正不合法的值并报错。`GroupEffectDefinitionBuilder.CreateInstanceParameters` 和 `PresetParameters` 提供相应规则。

效果的参数与作用时间编辑会先用候选输入调用模块并校验完整输出，成功后才替换该实例的绑定和命令。模块拒绝输入时，原参数、范围、结果和历史保持，界面显示拒绝原因；拖动时同样适用。这个校验会等待模块执行，复杂模块的编辑可能停顿；应在分配大量数据前检查输入并及时响应取消。源码热更新失败仍显示最后完整结果及错误。

效果库右键开发条目可导出 `.sbfx`；安装后的包首次使用仍须单独信任，开发目录授权不会自动授权它的二进制包。库按包和模块身份合并版本，工程固定包优先；选择新版可仅用于后续添加，或显式升级已有实例。升级只替换该效果的绑定与命令，失败保留原身份、内容和结构。实例属性提供重新生成和参数预设；预设包含公开参数及所需素材，不包含实例种子、作用起点或内外层运动。

`Uses(GroupEffectInputs.Timing/HitObjects/Metadata/AbsoluteTime/TargetSummary/TargetStructure)` 声明输入依赖。未声明的谱面输入不传给模块；目标成员树只通过受控暂存文件及 SHA256 传入，最多 64 MiB、深度 32。树不包含当前效果、其他效果和整体外层运动。`context.AbsoluteTime(u)` 使用 PlacementTime 与 TimeScale 的仿射映射。`context.Random(seed)` 使用固定版本 StableRandom，结果不依赖机器与线程。

效果结果为 `GroupEffectOutput { FormatVersion, Commands }`。原始局部结果随工程保存，映射后的组命令负责显示；缺包仍可显示已保存动作。损坏、未知格式或不一致的所有权标记会保留已保存数据并锁定自动写入。隐藏 Host 用于故障隔离，仍具有当前用户的权限。
