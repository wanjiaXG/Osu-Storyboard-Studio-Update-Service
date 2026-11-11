# 创作 SDK V4：模板与效果

用几行 C# 就能做出可调的故事板模板（比如下雪、文字）和可以套在任何编组上的效果（比如震动、模糊）。只会变量、`for` 循环和 `if` 就够了。

## 五分钟上手

1. 在模板库点「管理模板 → 新建模板」，输入一个名字。编辑器会建好一个能直接运行的源码文件夹，并在文件夹中选中 `Example.code-workspace`。用 VS Code 打开这个工作区，或用 Visual Studio 打开 `Example.sln`。
2. 把模板拖到时间轴上，就能看到预览。
3. 改代码并保存。大约 0.3 秒后预览自动更新；开发中模板成功编译和生成后会同步参数列表，源码中已删除的参数会从实例移除，仍存在的参数保留面板设置。编译或生成失败时保留上一次有效参数和画面，属性面板会指出错误位置及原因。

不需要手写 JSON 或配置项目。模块文件夹中的 `Example.csproj` 已引用 SDK、契约程序集和代码诊断器；`Example.sln` 用于加载项目，`Example.code-workspace` 用于在 VS Code 中打开整个模块文件夹。库和项目源码管理列表的右键「编辑源码」使用「首选项 → 编辑 → 源码编辑器」设置，可选择自动、Visual Studio Code、Visual Studio、系统默认或自定义 .exe。下拉列表会检测并分别列出已安装的Visual Studio版本、VS Code／Insiders、Cursor、Windsurf、VSCodium、Rider、Sublime Text、Notepad++；可刷新检测结果，悬停查看具体路径，选择后固定使用该安装位置。自动优先VS Code、最新Visual Studio，其次其他检测项，最后使用系统默认。自定义路径仅选择自定义时显示。界面说明实际打开的类型：Visual Studio／Rider／系统默认优先.sln→.slnx→.csproj，VS Code等打开含.csproj的模块文件夹，Notepad++优先.cs文件；诊断定位打开.cs。Cursor 会显式进入编辑器模式，并优先打开模块已有的 `Example.code-workspace`；没有此文件时打开模块文件夹，诊断定位沿用相同工作区。不会新建或改写已有模块的工作区文件。便携版或未检测到的程序可自定义选择。VS Code及同类编辑器的源码打开和诊断定位为不同模块使用独立窗口，保留已经打开的其他项目；同一模块文件夹再次打开时由编辑器复用对应窗口。自定义程序接收模块文件夹或源码文件路径；选择 Code.exe／Cursor.exe／Windsurf.exe／VSCodium.exe／devenv.exe 时也支持对应的项目和错误定位方式。

VS Code 代码提示需要 C# 扩展（推荐同时使用 C# Dev Kit）和 .NET 10 SDK。新建模块已提供扩展推荐，打开工作区后按提示安装；安装后等待项目加载完成，再编辑 `Example.cs`。仅用 VS Code 打开一个 `.cs` 文件可能无法加载项目及 SDK 引用，语言状态会显示「杂项文件／无项目」。受限模式也会限制语言服务；仅在你确认源码可信时，由你在 VS Code 中确认工作区信任。程序自带的运行时负责预览执行，不能替代 IDE 所需的开发 SDK。已有模块也可以通过「编辑源码」或 VS Code 的「打开文件夹」打开含 `Example.csproj` 的模块目录。若仍无提示，检查 C# / C# Dev Kit 的输出窗口是否显示项目加载或 SDK 错误。

官方说明：[C# 项目加载](https://code.visualstudio.com/docs/csharp/project-management)、[C# Dev Kit 环境排查](https://code.visualstudio.com/docs/csharp/cs-dev-kit-faq)。

## 第一个模板

```csharp
using SbStudio.Authoring;

public class 下雪 : Template
{
    public override TemplateInfo Info => new()
    {
        Name = "下雪",
        Id = "user.3f9c…",                 // 新建时自动生成，不要修改
        Category = TemplateCategory.Particles,
    };

    // ↓ 带 public 的每一行都会出现在设置面板里
    public Image 图片 = "particle.png";
    public int 数量 = 200;
    public double 下落距离 = 400;
    public Color 颜色 = Color.White;

    public override void Generate()
    {
        for (int i = 0; i < 数量; i++)
        {
            var 雪花 = AddSprite(图片);
            double x = RandomNumber(0, 640);
            double y = RandomNumber(-100, 0);

            雪花.At(0).Position(x, y).Opacity(0).Color(颜色);
            雪花.At(300).Opacity(1);
            雪花.At(Duration).Position(x, y + 下落距离).Opacity(0);
        }
    }
}
```

读法很直接：“在第 0 毫秒，位置在 (x, y)，看不见；第 300 毫秒完全显现；结束时落到下面并消失。”

## 设置面板：哪些能调

**规则只有一条：带 `public` 而且能改的字段，就会出现在设置面板上。**

| 写法 | 面板上 |
| --- | --- |
| `public int 数量 = 200;` | 出现，默认 200 |
| `int 风力 = 5;`、`private double 重力 = 9.8;` | 不出现，写死在代码里 |
| `public const`、`public readonly`、`public static` | 不出现 |

- 字段名就是面板上的名字，初始值就是默认值；字段上方的 `///` 注释是鼠标悬停时显示的说明。
- 每次生成前，软件会把面板上的值填进字段。在 `Generate` 里改字段只影响这一次。
- 写成 `public`、但面板不支持的类型（比如 `List<int>`）会得到一条黄色提示，模板照常运行，只是这个字段不出现在面板上。

### 类型决定控件

| 字段类型 | 控件 |
| --- | --- |
| `int`、`double` | 数字框 |
| `BeatSpan` | 时长数字框，拍 / 毫秒切换及常用拍数 |
| `Angle` | 顺时针角度旋钮与数字框 |
| `NumberRange` | 最小值与最大值数字框 |
| `Gradient` | 2–64 色标的渐变条，可拖动、增删和改色 |
| `bool` | 复选框 |
| 自己定义的 `enum` | 下拉框 |
| `string` | 单行文本 |
| `LongText` | 多行文本 |
| `Font` | 可搜索的字体选择器，带实时预览 |
| `Difficulty` | 工程难度下拉框（SDK 4.5，只用于模板，每个模板最多一个）。写 `public Difficulty 难度 = "";`，选中后 `Beatmap` 读取这个难度；没选或该难度已不在工程里时 `Beatmap` 没有物件，用 `难度.Chosen` 判断 |
| `Color` | 颜色选择器（`Color.White` 或 `"#RRGGBB"`） |
| `RgbCurves` | RGB总曲线与R/G/B独立曲线编辑器（SDK 4.8），默认恒等映射；`LookupTables()`返回三张256级查找表，总曲线先应用，再应用分通道曲线，Alpha不变 |
| `Color[]` | 可增删、排序的色板，支持1–64色与一键彩虹（SDK 4.2及以上） |
| `Point` | 两个坐标框（直接写 `(320, 240)`） |
| `Time` | 毫秒时间框 |
| `Image` | 图片选择按钮 |
| `Animation` | 帧序列选择按钮（`name0.png`、`name1.png`… 写基名 `"name.png"`） |
| `FilePath` | 文件选择按钮，用 `ReadFile(字段)` 读取内容 |
| `List<类>` 或 `类[]` | 列表（SDK 4.11）：每项是一个普通类的一组字段，可添加、复制、上下移动和删除，默认不限项数；见下方“列表参数” |
| `TextStyle` | 一组文字样式：字体、字号、颜色、描边、阴影颜色及偏移、粗体、行距、对齐、宽度，并带预览；`ShadowColor` 从 SDK 4.3 起提供，默认为黑色 |

SDK 4.9 中，`Image`、`Animation` 直接写文件名时，优先读取故事板工程的 `assets/`，不存在时读取模块源码项目（或安装包）自身的 `assets/`。`Image.Project(path)` / `Animation.Project(path)` 明确指定工程素材；`Image.Local(path)` / `Animation.Local(path)` 明确指定模块素材。显式指定来源时不回退。所有路径都相对于对应的 `assets/`，子目录使用 `/`。`ImageSize`、`OpenImage`、`OpenGif` 与图片参数使用同样的查找规则。用户在面板上点“选择…”时打开程序内的素材选择窗口，只能浏览和选择工程 assets 内的文件（不能跳出 assets），选中的文件原地使用、不复制；窗口打开时从外部拖入的文件会像素材面板一样导入到窗口当前显示的文件夹（ASCII 文件名规则相同）并选中，原文件不动；assets/generated 不接受拖入。参数始终明确使用工程来源。`FilePath` 的字符串仍表示模块文件，工程文件使用 `FilePath.Project(path)`。

模板和编组均提供以下入口，返回 `Visual`，可以继续写命令或关键帧：

通过素材导入或参数选择新导入图片、GIF、音频、视频时，文件名中的非 ASCII 字符替换为 `_`，不同内容的重名副本使用 `_2`、`_3` 等后缀；同名同内容仍复用。参数引用导入后的工程文件名，外部原文件保留。编号动画参数使用原有内容命名机制，帧基名和编号同样为 ASCII。已有工程文件、目录及导入谱面的原始路径不批量改名；显式重新选择已有非 ASCII 文件时会创建兼容文件名副本。

内置模板和效果的自带素材必须明确指定来源。固定素材使用 `AddSpriteFromLocal` / `AddAnimationFromLocal`；可供用户替换的素材参数默认值使用 `Image.Local(...)` / `Animation.Local(...)`，读取尺寸、GIF或像素时也传递同一个明确来源的引用。不要用裸字符串的自动查找读取自带素材，避免故事板工程中的同名文件覆盖它。`AddSprite(image)` / `AddAnimation(animation)` 可以接收已明确来源的引用、用户选择的素材参数或运行时生成的素材；明确使用工程素材时指定 Project/FromProject。

| 默认自动查找 | 仅工程素材 | 仅模块素材 |
| --- | --- | --- |
| `AddSprite(image, layer, origin)` | `AddSpriteFromProject(path, layer, origin)` | `AddSpriteFromLocal(path, layer, origin)` |
| `AddAnimation(animation, frameCount, frameDelay, loopMode, layer, origin)` | `AddAnimationFromProject(path, frameCount, frameDelay, loopMode, layer, origin)` | `AddAnimationFromLocal(path, frameCount, frameDelay, loopMode, layer, origin)` |

动画传入基名（例如 `walk.png` 对应 `walk0.png`、`walk1.png`…），以第 0 帧是否在工程中存在决定整套帧的来源。缺少后续帧会报错，不跨来源拼帧。`frameCount` 是帧数，`frameDelay` 是每帧毫秒数；`loopMode` 默认 `AnimationLoop.LoopForever`，`layer` 默认 `Layer.Background`，`origin` 默认 `Origin.Centre`。生成的图片/动画保留生成来源，不参与文件名查找。

色板直接声明为 `public Color[] 配色 = ["#FF0000", "#00FF00", "#0000FF"];`。数组顺序就是面板上的颜色顺序，模板自行决定如何使用这些颜色。将旧的单颜色参数合并为色板时，可在 `RenamedParameters` 中按顺序把多个旧Id映射到同一个色板Id；已有新色板值优先，未通过验证的旧值按原规则报告。

### 列表参数

需要“任意多个同类东西”时（例如拼接的每张图片），先写一个普通的类，再把字段声明为它的 `List` 或数组（SDK 4.11）：

```csharp
public sealed class 图层项
{
    /// <summary>这一层的图片。</summary>
    [ParameterId("image")] public Image 图片 = Image.Local("layer.png");
    [ParameterId("offset")] public Point 偏移 = (0, 0);
    public bool 加亮 = false;
    [ShowWhen(nameof(加亮), true), Range(0, 1)] public double 强度 = .5;   // 只能依赖同一项里的 bool 或 enum
}

[ItemName("图层 {0}"), ItemCount(0, 100)] public List<图层项> 图层 = [new()];
```

- 类必须有无参构造函数；`new 图层项()` 就是用户点“添加”时得到的一项。字段可用上表除列表、`TextStyle`、`Difficulty` 以外的类型，以及 `Range`、`Step`、`Unit`、`ShowWhen` 等特性；`FoundNextTo` 不能用在列表项里。
- `ItemCount(最少, 最多)` 可省略，省略表示 0 项起、不限数量；`ItemName` 是每项的名字，`{0}` 为从 1 开始的序号。
- 保存的值是一个数组，每项按字段 Id 记录；后来新增的字段在旧值里取默认值，删掉的字段被忽略。列表项里选的图片同样受素材引用保护、改名同步、预设和复制粘贴支持。
- 翻译键：`列表Id`、`列表Id.tip`、`列表Id.item`（每项名字）、`列表Id.字段Id`、`列表Id.字段Id.tip`。

参数面板的颜色框由色块和 HEX 文本框组成，包括单个 `Color`、色板每一项和 `Control(ColorPicker)`。HEX 支持带或不带 `#` 的三位、六位写法，Enter 或失焦提交，Esc 取消，无效内容不改原值；框内重置恢复整项默认颜色。点击色块打开统一的确认／取消选择器，细调在弹窗内暂存，确认后提交；取消、Esc、点击外部或关闭入口时保留原值。

曲线声明为 `public RgbCurves 曲线 = new();`，也可用 `new RgbCurves(rgb: [new(0, 0), new(.5, .7), new(1, 1)])` 设置默认值。坐标归一到0–1，每条曲线2–64点，输入从0到1严格递增，输出可非单调，端点输出可调。点击“编辑曲线”打开完整浮窗，Ctrl＋单击加点、拖动改点、右键删除内部点、Alt取消边缘吸附；输入/输出框使用0–255值。切换通道只切换编辑视图，重置可作用当前通道或全部。浮窗修改暂存本地草稿，确认后作为一步提交；取消、Esc、点击外部或编辑对象/参数变化会丢弃未确认修改。浮窗内撤销重做只作用草稿。长时间逐行处理图片时可调用受保护的 `CheckCancellation()` 响应取消。

### 需要时才写的特性

```csharp
[Range(1, 5000)] public int 数量 = 200;              // 超出范围的输入会被拒绝，并说明原因
[SuggestedRange(0, 50), Unit("px")] public double 幅度 = 8;  // 滑块的常用区间和单位
[Step(0.1)] public double 速度 = 1;
[Section("外观")] public Color 颜色 = Color.White;     // 面板分组
[ShowWhen(nameof(动画), 动画方式.上升)] public double 上升距离 = 16;  // 条件显示
[FoundNextTo(nameof(图片), "{name}-mask.png")] public Image 遮罩 = "sample-mask.png";  // 选好“图片”时自动找同目录的文件（只用于模板）
[Control(ParameterControl.Drag)] public double 角度 = 0;
[Range(-100, 100), SliderColors("#28753E", "#666666", "#993D89")] public double 色调 = 0; // SDK 4.8：数值输入与常驻渐变滑块，2–8种等距颜色
[ParameterId("count")] public int 雪花数量 = 200;      // 改字段名时写上旧名，已保存的值不会丢
```

## 动画

### 关键帧

`对象.At(时间)` 跳到某一毫秒，然后写这一刻它是什么样：

| 方法 | 含义 |
| --- | --- |
| `Position(x, y)`、`X(v)`、`Y(v)` | 位置 |
| `Scale(v)`、`Scale(x, y)` | 缩放（1 为原尺寸） |
| `Rotation(度)` | 旋转，顺时针为正 |
| `Opacity(v)` | 不透明度：0 看不见，1 完全显示 |
| `Color(颜色)` | 颜色叠加 |
| `FlipH(true)`、`FlipV(true)`、`Additive(true)` | 从这一刻起翻转 / 加法混合 |

每个属性分开计算：只在两个关键帧之间有变化的属性才会动。只写了一帧的属性会在那一刻直接设成这个值。

### 缓动

缓动写在**到达**的那一帧上，表示“从上一帧变到这一帧时怎么变”。不写就是匀速。

```csharp
雪花.At(0).Opacity(0);
雪花.At(300, Ease.Out).Opacity(1);            // 先快后慢地显现
雪花.At(1000).Position(x, y, Ease.SineIn);    // 只给这一个属性换缓动
```

`Ease.` 后面列出全部可选缓动。`Ease.Step` 表示到达时瞬间跳变；`Ease.Curve((0, 0), (0.3, 0.9), (1, 1))` 是自绘曲线（只用于编组运动）。

### osb 命令写法（进阶）

`Template.AddSprite(...)` 和 `Group.AddSprite(...)` 用图片素材创建故事板精灵，返回 `Visual`，可继续写命令或关键帧。图片素材仍使用 `Image`，帧动画仍用 `AddAnimation(...)`。

熟悉 osb 的话，也可以直接写命令，和关键帧混用：

```csharp
var 光 = AddSprite("glow.png", Layer.Foreground, Origin.Centre);
光.Move(0, 1000, startPosition: (0, 240), endPosition: (640, 240), ease: Ease.Out)
  .Fade(0, 1000, 1, 0)
  .Loop(0, 4, loop => loop.Scale(0, 250, 1, 1.2).Scale(250, 500, 1.2, 1));
光.Trigger("HitSoundClap", 0, Duration, t => t.Fade(0, 100, 1, 0));
```

SDK 4.9 统一使用具体参数名称，不保留旧命名重载：区间时间为 `startTime/endTime`，缩放为 `startScale/endScale`，位移为 `startX/startY/endX/endY` 或 `startPosition/endPosition`，透明度为 `startOpacity/endOpacity`，旋转为 `startAngle/endAngle`（度），颜色为 `startColor/endColor`，深度为 `startDepth/endDepth`。固定值重载使用 `scale/opacity/angle/color/depth`，二维缩放使用 `scaleX/scaleY`。效果运动命令与关键帧遵循相同命名。

```csharp
AddSprite("bg.png").Scale(startTime: 0, endTime: 1000, startScale: 0.5, endScale: 1, ease: Ease.Out);
AddSpriteFromLocal("pixel.png").Fade(startTime: 0, endTime: Duration, opacity: 1);
AddAnimation("walk.png", frameCount: 8, frameDelay: 80, loopMode: AnimationLoop.LoopForever);
```

`Loop(startTime, loopCount, commands)`、`Trigger(triggerName, startTime, endTime, commands, triggerGroup)` 同样使用具体名称。读取效果目标时使用 `ContentStartTime/ContentEndTime`、`AuthoringCommand.StartTime/EndTime/StartValues/EndValues`、`ImageSegment.StartTime/EndTime`；GIF 帧时长为 `DelayMilliseconds`，文字排版原文索引为 `StartIndex`。旧自定义源码需要按新名称修改后重新编译。

有 `Move/MoveX/MoveY/Scale/ScaleXY/Rotate/Fade/Color/FlipX/FlipY/Additive`，编组还有 `Z/RotX/RotY`。角度一律用度。`Loop`、`Trigger` 块里的时间相对块开始，块里也可以写关键帧。

### 编组

编组的 `Fade`、`Color`、`ScaleXY`、`FlipX`、`FlipY`、`Additive` 作用于组内每张图：不透明度、颜色和尺寸与图片自身的值相乘，重叠部分不合成为一张图。编组内只有一张图时沿用替换该图对应属性的行为。使用者可在编组属性里改为替换。

```csharp
var 组 = AddGroup("花朵");
组.Pivot(320, 240);
var 花 = 组.AddSprite("flower.png");
组.At(0).Rotation(0).At(Duration).Rotation(360);   // 整组一起转
```

### 复制对象（SDK 4.17）

精灵、帧动画和编组都可以 `Clone(timeOffset)`，效果类似在时间轴上复制粘贴：到调用时为止写好的命令、关键帧、循环、触发器、名称和位置都会复制，`timeOffset` 把副本整体平移若干毫秒（可为负，四舍五入到整毫秒）。复制编组时，组内的精灵、动画、子编组和 3D 网格一起复制。副本和原对象在同一个编组里，之后各写各的，互不影响；稳定键 `Key` 不复制，需要时给副本另设。

```csharp
var 星 = AddSprite("star.png");
星.At(0).Opacity(0).At(200).Opacity(1);
for (int i = 1; i < 8; i++)
    星.Clone(timeOffset: i * 150).Position(80 * i, 240);   // 每隔 150 毫秒再出现一颗
组.Clone(timeOffset: 2000);                                // 整个编组 2 秒后再来一次
```

## 3D 网格（SDK 4.13）

编组上调用 `Mesh` 可以输出一个 3D 网格模型，网格内嵌在编组里，不需要素材文件。顶点是 x、y、z 依次排列的坐标（X 向右、Y 向下、Z 远离观众），面是每三个一组的顶点序号，从模型外面看去逆时针绕行，正面朝向观众一侧。

```csharp
var 组 = AddGroup("盒子");
组.Pivot(320, 240);
组.Mesh(
    vertices: [-80,-80,-80, 80,-80,-80, 80,80,-80, -80,80,-80,
               -80,-80, 80, 80,-80, 80, 80,80, 80, -80,80, 80],
    faces: [0,2,1, 0,3,2,  4,5,6, 4,6,7,  4,3,0, 4,7,3,
            1,6,5, 1,2,6,  4,0,1, 4,1,5,  3,6,2, 3,7,6],
    startTime: 0, endTime: Duration).Color("#3A7BFF").DoubleSided();
组.At(0).Rotation(0).At(Duration).Rotation(360);   // 模型跟着编组一起转
```

- `Mesh(vertices, faces, startTime, endTime, layer)`：显示时间和图层在这里给出（`layer` 默认 `Layer.Background`），它们决定模型何时出现在哪一层；模型随编组整体移动、旋转、缩放。
- 链式设置：`Color(颜色)` 整体颜色（不写保持默认）；`FaceColors(颜色…)` 每个面一种颜色，数量必须与面数一致；`Unlit()` 不受光照直接显示原色；`DoubleSided()` 背面也画；`Wireframe()` 只画线框；`SolidAndWireframe()` 面和线框一起。默认受光照、单面、只画面。
- 编组照常可以写命令和关键帧（旋转、缩放、`Z`/`RotX`/`RotY` 等），模型跟着一起动。
- 顶点不是有限数、面序号越界或面数为 0 时，生成停止并说明原因。顶点和面的数量没有上限，由参数和模块超时决定。
- SDK 4.15：`Points()` 用许多发光小点拼出模型（点云），点均匀铺在表面上，背向观众的一面变暗；`PointCount(数量)` 设点数（大于 0），`PointSize(像素)` 设每个点在屏幕上的宽度（大于 0）。不写 `Points()` 时仍只画面。例如 `组.Mesh(顶点, 面, 0, Duration).Points().PointCount(3000).PointSize(5);`。

### 现成的形状与模型文件（SDK 4.20）

不想自己写顶点时，用 `MeshShapes` 里的基本形状，或者读一个 OBJ 模型文件：

```csharp
public FilePath 模型 = "model.obj";

public override void Generate()
{
    var 组 = AddGroup("形状");
    组.Pivot(320, 240);
    var 形状 = MeshShapes.Torus(radius: 90, tubeRadius: 30).RotateX(60);
    // var 形状 = LoadMesh(模型).FitInside(200);        // 读 OBJ，并缩放到最长边 200 像素
    组.Mesh(形状, 0, Duration).Color("#3A7BFF");
    组.At(0).Rotation(0).At(Duration).Rotation(360);
}
```

| 成员 | 作用 |
| --- | --- |
| `MeshShapes.Box/Plane/Sphere/Cylinder/Cone/Torus/Pyramid/Prism` | 立方体、平面、球、圆柱、圆锥、圆环、棱锥、棱柱，中心都在原点 |
| `MeshShapes.Extrude(轮廓点, 厚度)` | 把一个平面轮廓（可以是凹的）拉出厚度 |
| `LoadMesh(文件)`、`ObjFile.Parse(文字)` | 读取 OBJ 模型；只读形状，不读贴图和材质 |
| `形状.Translate/Scale/RotateX/RotateY/RotateZ` | 移动、缩放、旋转，返回新形状 |
| `形状.Centered()`、`形状.FitInside(尺寸)` | 把形状挪到原点；等比缩放到最长边为指定尺寸 |
| `MeshData.Merge(形状…)` | 把几个形状合成一个 |
| `组.Mesh(形状, 开始, 结束, 图层)` | 放进编组，之后照常链式设置颜色、点云等 |

## 常用工具

| 成员 | 作用 |
| --- | --- |
| `Duration` | 这个实例的时长（毫秒），所有时间都从 0 开始 |
| `RandomNumber(min, max)`、`RandomInt(min, max)`、`RandomPoint()`、`Chance(p)` | 随机数。结果可复现，面板上的“换一批”会换一组 |
| `Random(seed)` | 一个独立的随机序列 |
| `MoveToward(点, 角度, 距离)` | 沿某个方向移动后的位置（0° 向右，90° 向下） |
| `Screen.Left/Right/Top/Bottom/Center` | 屏幕范围（宽屏 -107 到 747） |
| `Beats(细分)` | 实例范围内每一拍（或每 1/n 拍）的时间 |
| `Point`、`Vector3`、`Maths` | 位移运算、距离、插值、旋转和区间换算 |
| `Noise(x)`、`Noise(x, y)` | 由实例种子决定的平滑噪声，范围 -1 到 1 |
| `Beat(index)`、`Bar(index)`、`BeatLengthAt(t)`、`BpmAt(t)`、`Snap(t, divisor)` | 拍子、小节、拍长、BPM 与拍点吸附，时间均为实例内毫秒 |
| `IsKiai(t)`、`KiaiSections`、`OnBeat(divisor, action)` | Kiai 判断、时间段和逐拍回调 |
| `FadeIn`、`FadeOut`、`Show`、`Pop`、`Pulse`、`Blink`、`Spin` | 淡入淡出、弹出、强调、闪烁与旋转 |
| `At(t).MoveBy`、`ScaleBy`、`RotateBy` | 以上一属性关键帧为基准写相对变化 |
| `AnimatePosition/Scale/Rotation/Opacity/Color`、`Animate`、`FollowPath` | 公式驱动或沿路径生成动画 |
| `AddSprites`、`Group.Members`、`VisualSet`、`Layouts` | 批量对象、错开播放、筛选、洗牌和布局 |
| `Color.FromHsl`、`ToHsl`、`Rainbow`、命名色 | HSL 换算、等距色环与常用颜色，字符串也支持 `#RGB` |
| `Beatmap.Timing`、`Beatmap.HitObjects`、`Beatmap.Title` | 谱面信息；`Beatmap.StoryboardPosition(物件)` 换算坐标；`物件.Track` 是物件实际经过的路线（故事板坐标、歌曲时间，含叠放偏移，滑条沿线往返） |
| `Beatmap.GetHitObjects(sliderPrecision)` | 按指定滑条采样精度读取物件；精度是相对谱面原始折线的最大位置误差（osu! 像素，默认0.5），越小路线点越多；0保留完整精度。端点及每次往返时刻始终保留 |
| `AbsoluteTime(t)`、`LocalTime(t)` | 实例时间与歌曲时间互换 |
| `RenderText(文字, 样式)`、`RenderTextLayout(...)` | 把文字画成图片 |
| `AddText`、`TextBlock.Chars/Words/Lines`、`StaggerWords/Lines` | 保留整段排版的逐字精灵与分组动画 |
| `Particles`、`Area`、`ParticleEmitter.Run` | 按区域发射粒子，自动拟合运动与复用精灵 |
| `ImageFrame.Map/Resize/Scale/Crop/Trim/Pad/FlipHorizontal/FlipVertical` | 像素变换与几何处理 |
| `ImageFrame.Tint/Opacity/Grayscale/Invert/Adjust/Composite/Mask` | 颜色处理、透明合成与蒙版 |
| `ImageFrame.Glow/Shadow/Outline/Slice`、`AddSlices` | 图片样式与网格切片 |
| `Canvas.DrawImage/FillRadial`、`PixelImage/CircleImage/GlowImage` | 绘制图片、径向渐变与缓存的白色小图 |
| `ReadFile(文件)`、`ImageSize(图片)`、`OpenImage(图片)` | 读取素材 |
| `OpenGif(图片)` | 逐帧读取 GIF，每帧是合成后的完整画面并带显示时长；不是 GIF 时返回空列表 |
| `WriteAnimation(名称, 帧列表)` | 把同一格式的PNG或JPG写成编号帧序列，保留编码内容；JPG帧可用`帧.ToJpeg(质量)`获得。结果交给`AddAnimation`，不可混用两种格式 |
| `Fail(...)`、`Warn(...)`、`Log(...)` | 报错、黄色提示、日志 |

SDK 4.6 起，`RenderTextLayout(...)` 返回的 `Lines` 提供完整排版后的行位置、宽度、基线与 `Elements`（文字元素的原文范围、位置和尺寸）。坐标相对逻辑画布左上角，不含描边/阴影留白；索引按 UTF-16 计算，组合字符属于同一个元素。可以据此生成逐字动画，同时保留自动换行与对齐。拆开重新绘制文字元素时，连写文字及跨元素字形塑形不保证与整行绘制完全一致。

用到谱面或歌曲时间的模板，谱面变化或实例移动时会自动重新生成，不需要另外声明。

## 图像处理（SDK 4.19）

所有 `ImageFrame` 变换生成新图。颜色处理、合成和蒙版保留逻辑画布与偏移。`Pad`、`Glow`、`Shadow`、`Outline` 向外扩边，偏移同步增加，原内容相对逻辑画布的落点保持。`Resize` 和 `Scale` 同时缩放像素与逻辑画布边界，边界向外取整。缩小使用面积平均，放大使用双线性插值，均按预乘透明度计算。

```csharp
var frame = OpenImage(Image.Local("picture.png"));
var tinted = frame.Tint(Color.Cyan).Opacity(.8);
var reduced = tinted.Resize(320, 240);
var styled = reduced.Outline(2, Color.White).Glow(8, Color.Blue);
var sprite = AddSprite(styled.Image).Position(Screen.Center);
sprite.Show(0, Duration);
var pieces = AddSlices(frame, 4, 3, Screen.Center, .5);
pieces.Stagger(40, (piece, i) => piece.FadeIn(0, 300));
```

`Crop` 建立裁出区域大小的新逻辑画布，偏移归零。`Trim` 保留原逻辑画布，仅当透明边完全在逻辑画布外时可以裁去。当前图片格式要求逻辑画布包含在像素图片内，裁边切入逻辑画布或全透明时会明确报错。翻转同时镜像像素和逻辑画布的位置。`Mask` 用蒙版亮度乘以蒙版透明度替换原 alpha。`Adjust` 的三个参数取 -1 到 1，颜色运算超出色域时截到有效颜色。

`Glow` 和 `Shadow` 用三次滑动平均柔化 alpha，半径是总扩散范围。`Outline` 使用欧氏距离产生圆角外描边。`Slice` 保留全部网格片，不能整除时各片最多相差一个像素。片的 `Offset` 相对原像素图片中心。`AddSlices` 以传入的 `center` 拼好并跳过全透明片，每片在 0 毫秒写入 `scale` 基准缩放。`Canvas.DrawImage(frame.ReadPixels(), x, y)` 以左上角定位，可用小数坐标。`FillRadial` 到指定半径时达到渐变末色。需要取消绘制时可使用 `new Canvas(width, height, cancellation)`。

## 逐字文字（SDK 4.19）

`position` 是整段逻辑文字画布的中心，默认 `Screen.Center`。左右对齐影响段内各行的排布。每个可见文字元素生成一个精灵，组合字符保持完整，空白不创建精灵，重复元素在同一文字块中只渲染一次。`Words` 中连续拉丁字母、数字及撇号合为一个词，汉字和假名分别成词，标点独立成词。`Lines` 保留空行和自动换行，`CharInfo(i)` 返回原字符串的 UTF-16 范围及行、词序号。

```csharp
var style = new TextStyle { Size = 96, MaxWidth = 1800 };
var text = AddText("Hello world\n第二行", style, scale: 480.0 / 1440);
text.Chars.Stagger(40, (letter, i) => letter.FadeIn(0, 200));
text.StaggerWords(120, (word, i) =>
    word.Each((letter, j) => letter.Pop(0, 250, text.Scale)));
text.StaggerLines(300, (line, i) =>
    line.Each((letter, j) => letter.FadeOut(2000, 300)));
```

文字按 `style.Size` 像素渲染，`scale` 是显示倍数。每个字都先写一条 `Scale(0, scale)`，之后的位置或透明度命令会继续使用该缩放。后续缩放动作显式传入 `text.Scale`。`Width`、`Height` 已乘显示倍数。词和行的错开回调期间，该组所有精灵共享时间偏移，嵌套偏移相加，回调结束后恢复。连写文字及跨元素字形塑形仍受逐字独立渲染的限制。

## 歌词文件（SDK 4.19）

`ReadLyrics(文件)` 读取 LRC、SRT 或 ASS 歌词，自动识别格式和编码，返回按时间排好的每一行。时间是歌曲里的毫秒，用 `LocalTime` 换成实例内的时间。

```csharp
public FilePath 歌词 = "lyrics.lrc";
public TextStyle 样式 = new();

public override void Generate()
{
    foreach (var 行 in ReadLyrics(歌词))
    {
        double 开始 = LocalTime(行.StartTime), 结束 = LocalTime(行.EndTime);
        if (结束 <= 0 || 开始 >= Duration) continue;          // 不在这个实例的时间范围里
        var 文字 = AddText(行.Text, 样式, position: (320, 400), scale: 0.3);
        文字.Chars.Each((字, i) => 字.Show(开始, 结束, fadeIn: 200, fadeOut: 200));
    }
}
```

- 每行有 `StartTime`、`EndTime`、`Text`；带逐字时间的歌词（增强 LRC 的 `<mm:ss.xx>`、ASS 的 `\k`）还有 `Syllables`，每个音节有自己的起止时间和文字，可以用来做卡拉 OK 式的逐字变色。
- LRC 一行的结束时间取下一行的开始；空行只用来结束上一行。看不懂的行会被跳过，不会报错。
- 已经拿到文字内容时，可以直接用 `LyricsFile.Parse(文字)`。

## 粒子（SDK 4.19）

用 `Run(start, end)` 才真正生成，每次调用继续使用实例随机序列。所有时间为实例内毫秒，速度为 px/s，加速度为 px/s²，角度为度。`Emit` 的速率可叠加，`Burst` 在实例内的绝对局部时间发生。`Prewarm` 将连续发射提前一个最大寿命。默认出生在屏幕中心，向下运动。

```csharp
var particles = Particles(GlowImage(8))
    .From(Area.Line((Screen.Left, -20), (Screen.Right, -20)))
    .Emit(40).Life(3000, 5000)
    .Speed(60, 120).Direction(90, spread: 15)
    .Gravity(0, 30).Drag(.2).Sway(24, period: 1800)
    .Scale(.1, .4).Spin(-40, 40)
    .Fade(fadeIn: 300, fadeOut: 500).Opacity(.6, 1)
    .Colors(Color.White, Color.Cyan).Additive().Prewarm()
    .Run(0, Duration);
```

`Area` 支持点、线、矩形、圆和圆环，面积形状均匀取点。阻力按 `dv/dt = Gravity - Drag × v` 计算，摆动垂直初速度。`ScaleOverLife` 的首尾倍数乘在随机基准缩放上。`ColorOverLife` 用整个寿命的进度取渐变颜色。淡入淡出重叠时取较低透明度，提前截断时直接消失。

匀速运动只写一条移动命令，其余轨迹默认误差不超过 0.5 像素（逐毫秒采样）。离屏判断使用图片对角线乘最大缩放留边，覆盖旋转及非中心锚点。结束的精灵复用于稍后出生的粒子，各生命间保留透明间隔并重置位置、缩放、旋转、透明度和颜色。`Run` 返回实际生成的精灵集合，复用后精灵数可能小于粒子出生总数。

## 数学与噪声（SDK 4.18）

点可以直接加减和乘除倍数。角度以度为单位，向右为 0，向下为 90。`Noise` 连续变化，相同种子下图案相同，面板上的“换一批”会改变它。

```csharp
Point center = Screen.Center;
Point right = center + (100, 0);
Point down = right.Rotate(90, center);
Point halfway = Point.Lerp(center, down, .5);
double amount = Maths.Map(Noise(.3, .8), -1, 1, 0, 1);
AddSprite(Image.Project("dot.png")).Position(Point.Lerp(center, halfway, amount));
```

`Vector3` 使用 `(x, y, z)`，提供加减、倍数、长度、归一化、点积与叉积。`Maths` 还提供 `Clamp`、`Clamp01`、`Smooth`、`Wrap` 与 `AngleDifference`。

## 拍子时间（SDK 4.18）

所有输入和返回的时间均为实例内毫秒。`Beat(0)` 是 `Beats()` 的第一个拍点，可用小数拍，也可继续数到实例末尾以外。每个 Timing 点按自己的拍号重新计算小节线。没有 Timing 时使用 500 毫秒一拍、4/4 拍。`BeatLengthAt` 已按实例伸缩换算，`BpmAt` 保持歌曲原来的 BPM。

```csharp
var dot = AddSprite(Image.Project("dot.png"));
double start = Beat(0);
double end = Beat(4.5);
dot.Show(start, end);
OnBeat(1, (time, index) => {
    if (IsKiai(time)) dot.Pulse(time, BeatLengthAt(time) / 2);
});
```

`Snap(time, 4)` 吸附到最近的四分之一拍，距离相同取较早的实例内时间。`KiaiSections` 是裁到 `[0, Duration]` 的时间段，`TimeRange.Contains` 包含起点且不包含终点。负倍率保留 `Beats` 的反向边界约定，交界处可能有重合拍点。

## 常用动作（SDK 4.18）

动作可用于精灵、编组和命令块，返回原对象以便继续写。时长以毫秒计，`Show` 的淡入淡出总长超出显示时间段时会按比例缩短。

```csharp
var dot = AddSprite(Image.Project("dot.png"));
dot.Show(0, 2000, fadeIn: 200, fadeOut: 300);
dot.Pop(0, 300, scale: 1);
dot.Pulse(500, 200, scale: 1, peak: 1.2);
dot.Spin(0, 2000, turns: 2);
dot.Blink(2200, 3000, interval: 100, low: 0, high: 1);
```

`FadeIn` 与 `FadeOut` 可单独指定缓动。`Pulse` 从 peak 回落到 scale，`Pop` 使用回弹缓动。效果目标默认遵守首尾中性规则；入场、出场的单侧规则见效果章节。

## 相对关键帧（SDK 4.18）

相对值在关键帧按时间排序后计算，基准是同一属性的上一帧。位置首帧从对象的初始位置开始，编组也是 `Group.Position` 指定的绝对位置。缩放首帧从 1、旋转首帧从 0 开始。命令块内需要先写该属性的绝对帧。

```csharp
var dot = AddSprite(Image.Project("dot.png")).Position(100, 200);
dot.At(1000).MoveBy(30, 0).ScaleBy(2).RotateBy(90);
dot.At(0).Position(100, 200).Scale(1).Rotation(0);
dot.At(500).MoveBy(20, 10, Ease.SineOut);
// 1000 毫秒时的位置为 (150, 210)。
dot.Clone(1200);
```

位置与 `X/Y` 混用时会拆轴，相对缩放与双轴缩放混用时分别乘到两个轴。复制后，相对帧继续独立计算。

## 公式驱动与沿路径（SDK 4.18）

回调中的 `t` 从 0 到 1。SDK 自动采样并化简为整毫秒的线性命令，`precision` 越小越贴合、命令通常越多，必须为正的有限数。默认容差为位置 0.5 像素、旋转 0.5 度、缩放 0.005、不透明度 0.01、颜色每通道 2 级。

```csharp
var dot = AddSprite(Image.Project("dot.png"));
dot.AnimatePosition(0, 1000, t => Screen.Center + Point.FromAngle(t * 360, 100));
dot.AnimateScale(0, 1000, t => 1 + .2 * Math.Sin(t * Math.PI));
dot.Animate(1000, 2000, t => new Pose { Rotation = t * 720, Opacity = 1 - t });
var path = Spline.Through((100, 100), (320, 300), (500, 100));
dot.FollowPath(path, 2000, 4000, Ease.SineInOut, faceForward: true, precision: .5);
```

`Pose` 只填写需要的属性，同一段内应保持填写一致。`FollowPath` 也接收 `DrawnPath`，支持 `Ease.Curve`，拒绝 `Ease.Step`，朝向角度沿路径连续展开。公式需要返回有限数，并应保持相同进度下结果相同。常量只生成一条瞬时命令，零时长读取 `t = 0`。效果目标提供 `AnimateOffset`、`AnimateScale`、`AnimateRotation`，首尾校验与手写命令相同。

## 批量与布局（SDK 4.18）

`VisualSet` 是一批对象的快照。`Stagger` 按每个对象间隔错开，`StaggerOver` 按首尾总间隔错开。回调的序号始终是对象在集合中的原序号，回调内命令、关键帧和循环／触发器的外层时间都会平移。

```csharp
var dots = AddSprites(Image.Project("dot.png"), 12);
dots.Layout(Layouts.Circle(Screen.Center, 150));
dots.Stagger(60, (dot, index) => dot.Pop(0, 300), StaggerOrder.FromCenter);
var palette = Color.Rainbow(dots.Count);
dots.Each((dot, index) => dot.Color(0, palette[index]));
var even = dots.Where((dot, index) => index % 2 == 0);
even.Shuffled().StaggerOver(1000, (dot, index) => dot.Pulse(1500, 200));
```

顺序可选 `Forward`、`Reverse`、`FromCenter`、`FromEdges`，同距离按原序号排列。`Layouts` 还提供 `Grid`、`Line`、`AlongPath`、`Points`，布局可以单独用 `At(index, count)` 求位置。闭合路径和整圈排列不重复起点。编组可用 `AddSprites` 和 `Members`，`Reversed` 返回倒序快照。洗牌使用对象所属实例的随机序列，集合应属于同一个实例。

## 颜色补充（SDK 4.18）

颜色字符串接受 `#RGB` 与 `#RRGGBB`。命名色采用常用网页颜色的同名值，例如 `Green` 是深绿色，鲜绿色使用 `Lime`。HSL 的色相单位为度，饱和度和亮度范围为 0 到 1。

```csharp
Color accent = "#F80";
Color soft = Color.FromHsl(30, .7, .65);
var (hue, saturation, lightness) = soft.ToHsl();
var palette = Color.Rainbow(8);
var dots = AddSprites(Image.Project("dot.png"), palette.Length);
dots.Each((dot, i) => dot.Color(0, palette[i]));
```

## 画形状（SDK 4.12）

`Canvas` 是一块透明画布（单位为像素），画好后用 `CreateImage(画布.ToPixels())` 变成图片。所有边缘自带抗锯齿，`softness` 让边缘在指定宽度内柔和淡出（做光晕、光斑）。

```csharp
var canvas = new Canvas(300, 300);
canvas.Fill(Shape.Circle((150, 150), 120), Color.White, softness: 8)
      .Stroke(Shape.Star((150, 150), 100, 45, 5), 6, "#FFD040", StrokePosition.Inside);
var image = CreateImage(canvas.ToPixels()).Image;
AddSprite(image).Color(0, Duration, "#3A7BFF");
```

| 成员 | 作用 |
| --- | --- |
| `Shape.Rectangle(x, y, 宽, 高, 圆角)`、`Shape.Ellipse(中心, 横半径, 纵半径)`、`Shape.Circle(中心, 半径)` | 矩形（可圆角）、椭圆、圆 |
| `Shape.Polygon(点…)`、`Shape.RegularPolygon(中心, 半径, 边数)`、`Shape.Star(中心, 外半径, 内半径, 角数)` | 任意多边形（可凹、可自交）、正多边形、星形 |
| `Shape.Line(点…)` | 线条，只能描边，两端为圆头 |
| `Shape.FromSpline(曲线)` | 由曲线得到形状：闭合的可填充，不闭合的只能描边 |
| `形状.Expand(距离)` | 向外扩大（负数向内收缩），拐角随之变圆 |
| `画布.Fill(形状, 颜色或渐变, …)` | 填充；渐变的 `angle` 为方向，0 从左到右，90 从上到下 |
| `画布.Stroke(形状, 宽度, 颜色, 位置)` | 描边，位置可选轮廓居中、内侧或外侧 |
| `画布.Clear()`、`画布.Clear(颜色)` | 清空或整块涂色 |

想让同一张图换不同颜色时，画成白色再用 `Color` 命令着色：颜色怎么改都只需要这一张图，生成更快，导出的谱面也更小。纯色的直角矩形甚至不用画，一个白色像素用 `ScaleXY` 拉伸到需要的大小即可。

## 颜色与渐变（SDK 4.12）

| 成员 | 作用 |
| --- | --- |
| `Color.FromHsv(色相, 饱和度, 明度)`、`颜色.ToHsv()` | 色相用度（0 红、120 绿、240 蓝），饱和度和明度为 0 到 1 |
| `颜色.RotateHue(度)` | 转动色相 |
| `Color.Mix(a, b, 比例)` | 在两色之间取色；按人眼感受均匀混合，中间色不会像直接混合 RGB 那样发灰 |
| `颜色.Lighten(量)`、`颜色.Darken(量)` | 变亮或变暗，量为 0 到 1，1 为纯白或纯黑 |
| `颜色.ToHex()` | 写成 `#RRGGBB` |
| `new Gradient(颜色…)`、`new Gradient((位置, 颜色)…)` | 多色渐变；`渐变.At(位置)` 取色，`渐变.Steps(个数)` 等距取一组颜色 |

## 曲线与路径（SDK 4.12）

`Spline` 是一条平面曲线。位置按长度计算：`PointAt(0.5)` 总在曲线的正中间，所以让比例随时间均匀增加就是匀速运动。

```csharp
var path = Spline.Through((100, 300), (250, 120), (420, 260), (600, 140));
for (int i = 0; i <= 40; i++)
{
    double t = i / 40.0;
    var p = path.PointAt(t);
    // 沿曲线放一排图片，并让每张朝向前进方向
    AddSprite(Image.Local("dot.png")).Position(p.X, p.Y).Rotate(0, Duration, path.DirectionAt(t)).Scale(0, Duration, 0.5);
}
```

| 成员 | 作用 |
| --- | --- |
| `Spline.Lines(点…)`、`Spline.LinesClosed(点…)` | 折线；Closed 版本回到起点 |
| `Spline.Through(点…)`、`Spline.ThroughClosed(点…)` | 平滑地依次经过每一个点，拐角处不会冲过头或打结 |
| `Spline.Bezier(控制点…)` | 贝塞尔曲线：从第一个点到最后一个点，中间的点只牵引方向；点数不限 |
| `Spline.Arc(中心, 半径, 起始角, 转过角度)`、`Spline.Ellipse(中心, 横半径, 纵半径)` | 圆弧、椭圆（角度 0 向右，90 向下，正数顺时针） |
| `曲线.Length`、`曲线.Closed` | 总长度、是否闭合 |
| `曲线.PointAt(比例)`、`曲线.DirectionAt(比例)` | 位于全长某个比例处的点和前进方向（度，可直接用于 `Rotate`）；闭合曲线的比例超出 0 到 1 时绕回 |
| `曲线.Points(个数)` | 沿曲线等距取点 |
| `曲线.Flatten(容差)` | 变成尽量少的直线段，每段偏离曲线不超过容差（默认 0.25 像素）；直线部分不会被多余地切开。用它写 `Move` 命令时命令数最少 |

## 音乐频谱（SDK 4.1）

模板里调用 `AnalyzeSpectrum(new SpectrumOptions { Bands = 64, FramesPerSecond = 30 })`，即可读取输入难度的真实音乐频谱。
软件自动推断音乐及绝对时间依赖；移动、伸缩、反向或替换音乐后重新分析对应的歌曲时间。
`SpectrumData.FrameCount/BandCount` 给出帧数和频段数，`Time(frame)` 是实例内毫秒，`Frequency(band)` 是频段中心频率，`Level(frame, band)` 是线性幅度，静音为 0。
`MinFrequency`、`MaxFrequency` 与 `Logarithmic` 控制频段分布，最高频率自动限制在音乐采样率的一半。
数据不包含布局，可以分别映射到直线、圆环、弧线或其他自定义造型。
服务使用居中的 4096 点 Hann 窗 FFT，按各声道能量合成，反相立体声不会相互抵消；音乐范围之外按静音处理。
分析支持取消和内容缓存，最多保留两份结果。没有音乐时报告错误并保留上次成功的模板结果。

内置「Music spectrum」提供 Straight、Mirrored、Circular 三种布局，以及频段、帧率、增益、噪声下限、上升/回落响应、柱宽、高度、颜色渐变等参数。
Circular 的 Arc angle 可设置开放弧线，完整圆环不重复首尾柱。输出为普通图片与 OSB 命令，不需要运行时音频分析。

## 效果

效果套在一个已有的编组上，给它追加运动或替换图片：

```csharp
public class 震动 : Effect
{
    public override EffectInfo Info => new() { Name = "震动", Id = "user.…", Category = EffectCategory.Motion };

    [SuggestedRange(0, 50), Unit("px")] public double 幅度 = 8;
    public double 每秒次数 = 12;

    public override void Apply()
    {
        int 次数 = (int)Math.Max(2, Duration * 每秒次数 / 1000);
        Target.At(0).Offset(0, 0);
        for (int i = 1; i < 次数; i++)
            Target.At(i * Duration / 次数, Ease.SineInOut).Offset(RandomNumber(-幅度, 幅度), RandomNumber(-幅度, 幅度));
        Target.At(Duration, Ease.SineInOut).Offset(0, 0);
    }
}
```

- `Target` 的关键帧有 `Offset`、`OffsetX/OffsetY`、`Scale`、`Rotation`、`Depth`、`RotationX/RotationY`、`Opacity`、`Color`。它们叠加在编组原有动画之上，默认开头和结尾都须中性。
- `Target` 支持 `Fade` 命令和 `At(...).Opacity(...)`、`Color(...)`，分别与原不透明度和颜色相乘。中性值为 1 和白色。
- 转成 JPG：`OpenImage(图片).ToJpeg(90)` 返回尺寸不变的 JPG，透明处变黑，半透明的颜色按透明度压暗。
- 图片效果：`foreach (var image in Images) image.Replace(frame => frame.GaussianBlur(4));`。`Visuals` 列出编组里的每个对象，`Target.Members` 是成员结构。

## 模块信息 `Info`

只有 `Name` 和 `Id` 必填，其余都有默认值，用不到可以不写。

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `Name` | 必填 | 库里显示的名称 |
| `Id` | 必填 | 稳定身份，新建时生成；复制别人的模板时，编辑器会提示生成新身份 |
| `Version` | `"1.0.0"` | 导出时内容变了但版本没变，编辑器会提示一键升版 |
| `Category` | `Other` | 模板：Particles、Text、Background、Rhythm、Transition、Shapes、Light、Other、Custom（自定义模板）；效果：Motion、Blur、Color、Distort、Stylize、Transition、Time、Other、Custom（自定义效果）；界面新建模块默认使用 Custom，未指定 Category 的源码仍默认 Other |
| `Description` | 类上方的 `///` 注释 | 说明 |
| `Author`、`Website` | 空 | 作者信息 |
| `SourceLanguage` | 新建时的界面语言 | 代码里文字所用的语言 |
| `Duration`、`FixedDuration`、`MinDuration`、`MaxDuration` | 5000、false、1、1073741823 | 模板默认时长与可调范围（毫秒）；最长默认不设实际限制，只受 osb 整数毫秒范围约束 |
| `Cacheable` | true | 相同输入总是得到相同结果 |
| `Replaces` | 空 | 接替的旧模块 Id，旧实例可以升级到它 |
| `RenamedParameters` | 空 | 旧参数 Id → 新 Id，升级实例和读取预设时迁移取值；拆成多个时用逗号分隔 |

新版本 SDK 只会给 `Info` 加带默认值的新字段，旧模块不用改。需要更新版本软件的模块装到旧软件上时，会提示先更新软件。

## 多语言

在模块文件夹里放 `lang/<语言代码>.json`，例如 `lang/en.json`、`lang/ja.json`。语言代码和软件界面一致（`en`、`zh-Hans`、`zh-Hant`、`ja`、`ko`、`ru`、`es`、`th`、`pt-BR`、`it`、`nl`、`fr`、`de`、`lt`、`id`、`vi`）。

```json
{
  "$name": "Snow",
  "$description": "Drifting snowflakes.",
  "数量": "Count",
  "数量.tip": "How many flakes fall at once.",
  "动画方式.淡入": "Fade in",
  "section.外观": "Appearance",
  "最多支持 {0} 行": "At most {0} lines are supported."
}
```

- 键：`$name`、`$description`、参数 Id、`参数Id.tip`（`///` 注释）、`枚举类型.成员`、`section.分组名`，列表参数的 `列表Id.item`、`列表Id.字段Id`（及 `.tip`），以及 `Fail/Warn/Log` 里写的文字。
- 显示时按这个顺序找：当前界面语言 → 当前语言就是 `SourceLanguage` 时用原文 → 英文 → 原文。**不写翻译也能正常显示。**
- 右键开发中的模块选「导出翻译表」，得到列好全部键和原文的骨架，只需填空。
- 翻译不需要重新编译：把 `lang/ja.json` 放进 DLL 旁边与它同名的文件夹即可生效。改翻译不会重新生成画面。
- 翻译只用于界面文字和提示，生成的画面不会随界面语言变化。

## 报错与提示

```csharp
if (行数 > 500) Fail("最多支持 {0} 行", 500);   // 停止生成，属性面板显示这条错误和行号
Warn("图片很大，生成可能较慢");                   // 黄色提示，照常生成
Log("生成了 {0} 个对象", 数量);
```

保存时，编辑器（以及用 IDE 打开时）会检查这些问题：

| 代码 | 级别 | 内容 |
| --- | --- | --- |
| SB1001 | 错误 | 没有、或有不止一个 `Template`/`Effect` 类（一个文件夹只放一个模块） |
| SB1002 | 错误 | `Id` 格式不对 |
| SB1003 | 错误 | `Version` 不是“主.次.修订” |
| SB1004 | 错误 | `Name` 为空 |
| SB1101 | 警告 | public 字段的类型面板不支持 |
| SB1102 | 提示 | public readonly / static / const 字段不会出现在面板 |
| SB1103 | 错误 | 特性用错，例如 `Range` 用在文字上、最小值大于最大值、`ShowWhen` 指向不存在的参数 |
| SB1104 | 警告 | 默认值超出 `Range` |
| SB1201 | 警告 | 翻译文件里有代码中已经没有的键 |
| SB1202 | 警告 | 翻译的 `{0}` 占位符和原文对不上 |
| SB1203 | 提示 | 某个语言还缺哪些条目 |

运行时出错（比如除以零、数组越界）同样会指出是第几行。取消生成和超时都由 SDK 处理，不需要自己检查。

SDK 4.14 起不再限制命令、对象、关键帧、节拍点、图片替换或网格的数量，生成规模由参数和首选项里的模块超时决定；模板与编组效果的时长也不再限制在 1 小时以内（上限 1073741823 毫秒，只为让开始时间加时长不超出 osb 的整数范围）；数据本身不合法（非有限数、索引越界、时间区间无效等）时仍然会停止并说明原因。

## 发布

SDK 4.10 的图片效果可用 `image.ReplaceTimed(frameDuration, (frame, time) => ...)` 在效果局部时间格上处理图片；`frameDuration` 为至少 1 毫秒的有限间隔，回调返回一个 `ImageFrame`。它保留原动画帧数及播放时钟，为静态图片也能添加时变处理；相同输入和相位的生成图片应由效果自行缓存复用。不支持触发器控制的图片。内置“图片描边”已迁移到效果库，支持保留原图或只显示描边，以及流动蚂蚁线；目标的位置、时长和变换由原对象控制。

通过当前工程中的“新建模板 / 新建效果”创建的源码，软件在该工程中固定归入“自定义模板 / 自定义效果”；修改 `Info.Category` 不会改变其在库、搜索或项目源码管理列表中的分类。创建来源由工程单独记录，源码仍保留作者声明的分类。独立开发、外部加载及安装包使用声明分类；旧源码没有创建来源记录时也保持原分类。导出包或单独构建 DLL 不携带这项工程分类限制。工程整体移动会保留记录，单独复制源码则不携带记录。

- **导出包**：在库里右键开发中的模块，选「导出模板包…」或「导出效果包…」，得到 `.sbtemplate` / `.sbfx`。别人在库里安装即可。
- **单个 DLL**：用 IDE 构建 `Example.csproj`，得到一个 DLL，`assets/`、`presets/`、`lang/` 和说明注释都已嵌在里面。放进程序目录的 `Templates/`（模板）或 `Effects/`（效果）就会出现在库里。素材也可以放在 DLL 旁边与它同名的文件夹里。
- **随包预设**：把在软件里保存的 `.sbpreset` 放进 `presets/` 再导出。
- **依赖**：纯托管的 DLL 放进 `lib/` 即可，不要带另一份 SDK。

添加到工程的实例会固定当时的版本。有新版本时，用户可以在库里选择升级已有实例：参数按 Id 保留，`RenamedParameters` 里列出的会自动迁移。

第三方代码运行在独立进程中，但拥有当前用户的文件和网络权限，请只安装可信来源的模块。

## 从旧 SDK 迁移

旧 SDK（V3 及更早）的模块不能在 V4 中运行，需要按下表改写：

| 旧写法 | V4 写法 |
| --- | --- |
| `module.json` | 删除，改写 `Info` |
| `ITemplateModule` / `IGroupEffectModule` / `IEffectModule` | 继承 `Template` / `Effect` |
| `Define` 里的 `d.Integer("count").Default(200).Range(1, 50000)` | `[Range(1, 50000)] public int Count = 200;` |
| `c.Parameters.Get(count)` | 直接用字段 `Count` |
| `b.Sprite(asset)` | `AddSprite(image)` |
| `.Move(0, 1000, x1, y1, x2, y2)` | `.At(0).Position(x1, y1).At(1000).Position(x2, y2)`（旧写法也仍然可用） |
| `c.Random(seed).Between(a, b)` | `RandomNumber(a, b)` |
| `c.Cancellation.ThrowIfCancellationRequested()` | 删除 |
| `d.Uses(TemplateInputs.Timing)` | 删除，用到时自动提供 |
| `throw new ArgumentException("...")` | `Fail("...")` |
| `RotateRad` | `Rotate`（度） |

想保留旧实例的参数值，就给字段加 `[ParameterId("旧 id")]`，并在 `Info.Replaces` 里写上旧包 Id。

## 谱面数据（SDK 4.20）

`HitObject.Type` 区分 `Circle/Slider/Spinner/Hold`，`Kind` 保留原名称。
物件提供 `NewCombo`、从 1 起的 `ComboIndex`、从 0 起且计入跳色的 `ComboNumber`、`ComboColor`、
`HitSound` 位标志、`SampleSet/AdditionSet` 和歌曲绝对时间的 `SliderRepeats/SliderTicks`。
未指定连击色时 `ComboColor` 为白色。`Normal` 音效标志为零，其他标志可组合。

`Beatmap` 提供 CS、AR、OD、HP、滑条倍速与点密度，以及 Unicode 标题/作者、制图者、来源、标签、预览点、书签、休息段和连击色。
`CircleRadius` 是标准模式的故事板像素半径，`ApproachTime` 是歌曲毫秒。
`Beatmap.Breaks/Bookmarks` 使用歌曲绝对时间，`Module.Breaks` 则转换到局部时间并裁到实例范围。
`ObjectsIn(startLocal, endLocal)` 按开始时间筛选局部半开区间，返回的物件仍保留歌曲绝对时间。

```csharp
var route = Beatmap.CursorRoute(sliderPrecision: .5);
var position = route.PointAt(AbsoluteTime(500));
foreach (var hit in ObjectsIn(0, Duration))
    Log("物件 {0}，音效 {1}", hit.Type, hit.HitSound);
```

`Route.Points/StartTime/EndTime` 使用歌曲绝对时间，后开始的物件接管重叠部分，首尾外保持端点。
同一时刻的首点优先。空路线的位置为屏幕中心。每个模板仍可选择一个难度。
元数据和物件分别登记输入依赖，单独读取元数据时，物件编辑不触发重新生成。

## 音乐分析（SDK 4.20）

模板和效果均可使用 `AnalyzeSpectrum`、`AnalyzeLoudness(fps = 60)`、`AnalyzeBands(fps = 60)` 与
`DetectOnsets(band = OnsetBand.All, sensitivity = 1)`。移动、伸缩和反向实例会重新映射歌曲时间。
独立离线解码流负责分析，结果按音乐内容及请求缓存，支持取消。

```csharp
var loudness = AnalyzeLoudness();
var bands = AnalyzeBands();
Log("响度 {0}，低频 {1}", loudness.LevelAt(500), bands.LowAt(500));
foreach (double time in DetectOnsets(OnsetBand.Low))
    Log("起音 {0}", time);
```

`LoudnessData.FrameCount/Time(i)/Level(i)/LevelAt(t)/Peak` 提供 0–1 线性 RMS。
`BandData` 提供 `FrameCount/Time(i)`、`Low/Mid/High(i)` 与 `LowAt/MidAt/HighAt(t)`，
频段为 20–250、250–4000、4000 Hz 以上，幅度来自 2048 点 Hann 频谱的分段能量。
时间均为局部毫秒，任意时间读取线性插值，范围外保持端点。
起音检测使用 5 ms 步长的频谱通量、自适应阈值、峰值筛选及 PCM 攻击起点回溯，歌曲中的最小起音间隔为 50 ms。
灵敏度为正有限数，增大时降低检测阈值。

## 拍数时长（SDK 4.20）

```csharp
public BeatSpan 时长 = BeatSpan.Beats(2);
// Generate 或 Apply 中
var milliseconds = Length(时长, atLocalTime: 500);
```

面板可切换拍与毫秒，并提供 1/4、1/2、1、2、4 拍。
切换单位保留数值，作者可用 `BeatSpan.Milliseconds(500)` 指定局部毫秒。
拍数在指定时刻换算，并计入实例伸缩。读取 `Length` 会登记 Timing 与绝对时间依赖。
负数与非有限数会报告错误。

## 角度（SDK 4.20）

```csharp
[Range(-180, 180)] public Angle 方向 = 45;
// Angle 与 double 可隐式转换，单位都是度。
```

面板提供旋钮与数字输入。0° 向右，正角度顺时针旋转。数字输入遵守 `Range`。

## 数值区间（SDK 4.20）

```csharp
[SuggestedRange(0, 300)] public NumberRange 速度 = new(60, 120);
// Generate 中
var speed = RandomIn(速度);
Particles(Image.Local("particle.png")).Life(new NumberRange(1000, 2000))
    .Speed(速度).Scale(new NumberRange(.1, .4)).Run(0, Duration);
```

`range.Min/Max/Lerp(t)/Contains(value)` 提供闭区间运算。最小值大于最大值或端点非有限时拒绝。
`Range` 约束两端。粒子的 `Life/Speed/Scale/Spin/Opacity/InitialRotation` 接受单个区间，原双数值入口保持可用。

## 渐变参数（SDK 4.20）

```csharp
public Gradient 配色 = new((0d, Color.Red), (.4, Color.Lime), (1d, Color.Blue));
// Generate 或 Apply 中
var color = 配色.At(.5);
```

面板渐变参数包含 2–64 色标，可拖动、输入 0–1 位置、添加和删除色标，颜色通过统一确认/取消选择器编辑。
列表项同样支持上述四种类型，保留 `ShowWhen`、重置默认值、预设、复制和参数重命名迁移。
`Color[]` 色板与渐变参数可同时使用。

## 读回状态与时间工具（SDK 4.21）

```csharp
var card = AddSprite(Image.Project("card.png"));
card.At(0).Position(100, 240).Scale(1).At(1000, Ease.SineInOut).Position(540, 240).Scale(2);
VisualState state = card.StateAt(500);
TimeRange? lifetime = card.Lifetime;
VisualBounds bounds = card.Bounds(500, 128, 64);
card.Shift(200).Stretch(2, pivotTime: 200);
card.Reverse(200, 2200);
```

`StateAt` 读取调用时已写入的命令和关键帧，循环参与求值。触发器由游戏事件决定，读回时跳过。状态包含位置、横纵缩放、度数旋转、不透明度、颜色、翻转、加法混合与寿命内的 `Active`。同通道重叠时先开始且仍在运行的命令优先，开始前取首命令起值，结束后保持末值。颜色舍入到字节。

状态与包围盒使用对象自身的二维坐标。`Bounds(time)` 自动读取图片或动画首帧尺寸。`Group.StateAt` 读取编组自身状态。`Group` 的三个时间工具递归作用于调用时已有成员，之后新写的命令保持作者给出的时间。`Stretch` 倍数须为正。`Reverse` 要求区间完整包含受影响命令，存在循环、触发器或边界截断时拒绝整次操作。反向缓动与自绘曲线会镜像，两个非对称弹性缓动使用逐毫秒折线表示。

## 故事板音效（SDK 4.21）

```csharp
public FilePath 音效 = FilePath.Project("click.wav");
// 在 Generate 中
AddSample(音效, time: 300, volume: 80, layer: Layer.Background);
AddSample(FilePath.Local("accent.ogg"), 600);
AddSample(FilePath.Auto("click.wav"), 900);
```

音量为 0 到 100，时间为实例内毫秒。`FilePath.Project` 指定工程素材，`Local` 指定模块自带文件，`Auto` 优先工程再查模块。文件参数既有的字符串默认值继续指定模块自带文件。音效随实例移动、伸缩、隐藏、删除和重新生成，伸缩只改变触发时刻。它们经现有故事板音效播放和标准 Sample 事件导出。效果的入口仍为 `Apply`，音效由模板 `Generate` 输出。

## 摄像机与三维场景（SDK 4.21）

```csharp
var scene = AddScene3D(focalLength: 1000);
scene.Camera.Target(new Vector3(320, 240, 0)).Distance(1000);
scene.Camera.At(0).Orbit(0).Zoom(1000)
    .At(Duration, Ease.SineInOut).Orbit(20).Zoom(1200);
scene.Add(Image.Project("card.png"), new Vector3(420, 240, 500), .8)
    .Fade(0, Duration, 1);
scene.AddMesh(MeshShapes.Box(100, 100, 100), new Vector3(320, 240, 200), 0, Duration);
```

`Group.Camera(focalLength)` 也可直接为编组创建摄像机。摄像机提供目标点 `Target`、距离 `Distance`、水平环绕 `Orbit`、俯仰 `Tilt`、滚转 `Roll` 和投影焦距 `Zoom`，均可在 `At` 上写关键帧。角度为度，焦距和距离须为正。`Projection(CameraProjection.Orthographic)` 使用正交投影。X 向右、Y 向下、Z 远离观众，图片默认面向镜头。

一个最外层编组共用一个摄像机。模板输出根编组的摄像机会成为实例容器的摄像机，可用已有摄像机面板编辑，重新生成时由模板覆盖。模板删除该摄像机时一并移除。多个输出根摄像机会报错，嵌套编组的摄像机保留但停用，模板嵌入其它编组时使用最外层摄像机。`Scene3D.Add` 的显示寿命由图片命令决定，`AddMesh` 显式接收显示时间。

## 效果颜色、入场与出场（SDK 4.21）

```csharp
public sealed class 淡入上移 : Effect
{
    public override EffectInfo Info => new()
    {
        Name = "淡入上移", Id = "user.fade-up", Kind = EffectKind.Entrance,
    };
    public override void Apply() => Target.At(0).Opacity(0).Offset(0, 24)
        .At(Duration, Ease.SineOut).Opacity(1).Offset(0, 0);
}

public sealed class 淡出缩小 : Effect
{
    public override EffectInfo Info => new()
    {
        Name = "淡出缩小", Id = "user.fade-shrink", Kind = EffectKind.Exit,
    };
    public override void Apply() => Target.At(0).Opacity(1).Scale(1)
        .At(Duration, Ease.SineIn).Opacity(0).Scale(.8);
}
```

`Overlay` 为默认，首尾须中性。`Entrance` 只要求结尾中性，开始前保持起值。`Exit` 只要求开头中性，结束后保持末值。`HoldBefore()` 和 `HoldAfter()` 可分别放宽单侧约束，既有 `HoldEnds()` 放宽两侧。移动与角度的中性值为 0，缩放与不透明度为 1，颜色为白色。不透明度系数范围为 0 到 1。

效果颜色按 RGB 通道相乘，可与图片替换、多层效果一起使用，沿用效果条的添加、拖动、伸缩、开关和转普通编组。`Target.Members` 与 `Visuals` 继续提供成员信息及图片处理入口。逐成员运动、逐成员依次弹出以及 `Echo` 残影尚未提供。
