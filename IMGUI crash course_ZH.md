Unity的IMGUI系统非常简单，但功能非常强大。主要部分有：

- [OnGUI callback](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.OnGUI.html)。此回调用于处理与GUI有关的所有事情。
- 与OnGUI关系密切的[Event class](http://docs.unity3d.com/Documentation/ScriptReference/Event.html)。
- [GUIStyle class](http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle.html)定义一个控件的外观。它实际上负责任何GUI的绘制工作。
- [GUISkin class](http://docs.unity3d.com/Documentation/ScriptReference/GUISkin.html)这基本上只是一个预定义的控件样式集合，并且拥有一个自定义style数组，可以自由拓展。
- [GUIUtility](http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility.html)与[EditorGUIUtility](http://docs.unity3d.com/Documentation/ScriptReference/EditorGUIUtility.html)。当使用GUILayout class时，不要忘记不起眼但依旧重要的[GUILayoutUtility class](http://docs.unity3d.com/Documentation/ScriptReference/GUILayoutUtility.html)。

# OnGUI

OnGUI是一个回调，引擎会出于多种目的自动调用它。它通常每帧至少调用两次，但是当一些事件发生时可能会调用更多次。

我认为OnGUI是一个坏名字。当然，它的主要目的是“绘制GUI”，但这实际上是事件处理器的回调。我将在下一节中对此进行解释。

# The Event class

Event Class保存有关OnGUI中当前正在处理的事件的信息。即其静态成员[current](http://docs.unity3d.com/Documentation/ScriptReference/Event-current.html)。它保存Event类的实例，并且仅在OnGUI中有效。

该[type property](http://docs.unity3d.com/Documentation/ScriptReference/Event-type.html)持有当前被处理的[EventType](http://docs.unity3d.com/Documentation/ScriptReference/Event-type.html)。

以下是不同事件的概述：

- **Mouse events** 例如MouseDown，MouseUp，MouseMove（**仅限Editor**），MouseDrag，ScrollWheel。我应该补充一点，Unity将在移动触摸设备上模拟这些事件。它通过获取所有触摸的算术平均值来计算鼠标位置。
- **Keyboard events** 例如KeyDown和KeyUp。注意：与Input.GetKeyDown/Up不同，这些事件映射操作系统的键盘事件。因此，当您按住某个键时，系统将一直触发按键事件。对于文本编辑类的GUI，这很重要。
- **Layout event** 这是一个特殊事件，始终在所有其他事件之前执行，并用于确定自动布局控件的大小和位置。注意：可以使用[useGUILayout](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour-useGUILayout.html)禁用此事件，但请记住，在这种情况下，所有GUILayout内容均无法使用。
- **Repaint event** 该事件实际上将绘制GUI元素。
- **Used event** 任何在处理事件时不希望其他人获得此事件的控件都可以使用[Use function](http://docs.unity3d.com/Documentation/ScriptReference/Event.Use.html)来“吃掉”该事件。调用Use时，事件类型将设置为Used，并且每个控件都应该忽略此事件。

除了事件类型之外，它还包含事件的其他信息。

- [mousePosition](http://docs.unity3d.com/Documentation/ScriptReference/Event-mousePosition.html) **GUI坐标系中**的鼠标位置。GUI坐标系不同于屏幕坐标。该GUI的原点在**左上角**，屏幕坐标的原点在**左下角**。[GUIUtility class](http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility.html)提供这些坐标转换的功能。
- [button](http://docs.unity3d.com/Documentation/ScriptReference/Event-button.html) 仅对鼠标事件有效，并包含鼠标按钮索引（0-左按钮，1-右按钮，2-中间按钮）
- [modifier](http://docs.unity3d.com/Documentation/ScriptReference/Event-modifiers.html) 有关Shift，Alt和Ctrl键的信息
- [keyCode](http://docs.unity3d.com/Documentation/ScriptReference/Event-keyCode.html) 仅对键盘事件有效。持有按下/释放的键的[KeyCode](http://docs.unity3d.com/Documentation/ScriptReference/KeyCode.html)。
- 其他，例如修饰键的快捷键。

# GUIStyle

如前所述，该类负责向屏幕渲染控件。当“检测到”repaint事件时，GUI控件函数将调用[Draw functions](http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle.Draw.html)之一。一个GUIStyle可以包含3个[GUIStyleStates](http://docs.unity3d.com/Documentation/ScriptReference/GUIStyleState.html)（截至Unity 2020.3 LTS），但在同一时间，只有一个是“有效的”。使用哪种状态取决于调用什么Draw函数，传递哪些参数以及控件当前是否具有[keyboard focus](http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility-keyboardControl.html)。

绘制单个GUIStyle时，它可以产生0到3个draw call。这就是为什么非常复杂的GUI被认为不利于移动开发的原因，但是我们在Android和iOS上大量使用了GUI系统，在很多的现代设备上也没有太多问题。

GUIStyle可以绘制的内容是：

- GUIStyleState的background。通常用于定义按钮/文本字段/ ...的大致外观。如果设置了background，则始终先绘制background。
- 在GUIContent class中定义的内容图像。
- 在GUIContent class中定义的内容文本。

在GUIStyle中定义的[ImagePosition](http://docs.unity3d.com/Documentation/ScriptReference/ImagePosition.html)，可以用于标识图片相对于文本的位置，或者只绘制图片而忽略文本。

GUIStyle具有8个[GUIStyleState](http://docs.unity3d.com/ScriptReference/GUIStyleState.html)，其中每个状态都允许您指定background以及textcolor。注意：仅当状态具有background时，才能使用该State。因此，不幸的是，仅设置没有background的textcolor是不可能的。

不同的状态是：

- normal
- hover
- active
- focused
- onNormal
- onHover
- onActive
- onFocused

**normal** 是大多数情况下使用的样式状态。例如，Label将仅使用此状态。
**hover** 可被buttons，toggles以及绝大多数可以互动的控件，当你将鼠标悬停在控件上时会使用此State
**active** 这是按下按钮的状态。当你鼠标单击并按住元素时就会使用此State
**focused** 当编辑器处于活动状态时并且您可以编辑文本时，TextFields和TextAreas将使用此State

所有的**onXXX**状态与上面的其他4个功能具有相同的功能，但他们时用于具有“on”状态的控件（如Toggle）的。如果Toggle处于未选中状态，然后单击它，您将看到*active*状态。当Toggle已被选中时，您将看到*onActive*状态。

如上所述，没有background的state将被忽略。因此，如果您的按钮样式具有带有纹理的“normal”和“active”状态，但没有hover状态，则当您将鼠标悬停在按钮上时，Unity将使用normal状态。如果只想在悬停时更改textcolor，则必须为“hover”使用与“normal”相同的background，否则hover state将被忽略。

最后一点：Unity实现了一个隐式转换运算符，它将使用当前GUISkin的[GetStyle](http://docs.unity3d.com/Documentation/ScriptReference/GUISkin.GetStyle.html)函数自动将字符串转换为GUIStyle。

# GUISkin

如上所述，GUISkin只是GUIStyles的集合。与GUIStyles不同，GUISkin可以*轻松*地作为资产存储在项目中。您可以在“Assets/Create”菜单中创建一个。

要使用GUISkin，只需在绘制控件之前将其赋值给GUI.skin。通常的方法是将赋值放在OnGUI的顶部。每次调用OnGUI时都必须对它赋值。该赋值仅对OnGUI的当前执行有效。如果还要在其他脚本中使用这个skin，则也必须将赋值也放置在这些脚本中。

大多数控件都有可选参数来指定应用于绘制控件的GUIStyle。在这里，您可以简单地使用一个字符串（由于隐式转换），并使用要使用的样式的名称。如果未传递任何样式，则每个内置控件均具有其自己的默认样式。

# Control functions

上述类定义了GUI系统的基础。它们可以用于创建自定义控件，也可以以某种方式直接使用。

但是，Unity提供了预制的[控制功能](http://docs.unity3d.com/Documentation/ScriptReference/GUI.html)，大大简化了用法。例如[button](http://docs.unity3d.com/Documentation/ScriptReference/GUI.Button.html)功能。它使用默认样式“button”将其自身绘制到屏幕上。此函数还处理MouseDown和MouseUp事件。它使用控件Rect来确定鼠标的位置是否在Rect内，并使其自身成为[hotControl](http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility-hotControl.html)。该函数返回一个布尔值，该值始终为false，除了在MouseUp事件中其作为hotControl并且鼠标在其Rect范围中释放的时候。

这就是将click事件传递回调用方的方式。

某些控件（例如，scrollbars和sliders）使用多个GUIStyle。它们通常具有一个主要样式和其他“子样式”，这些样式是通过在主要样式的名称中添加一些内容来确定的。horizontalScrollbar实际上是在内部执行此操作：

```csharp
GUI.skin.GetStyle(style.name + "thumb")
GUI.skin.GetStyle(style.name + "leftbutton")
GUI.skin.GetStyle(style.name + "rightbutton")
```

获得其他样式。这是次优的方式，因为您没有scrollbars功能（或[反射器](http://ilspy.net/)）的源代码。

# GUI与GUILayout

有些讨厌GUILayout，有些喜欢它。就个人而言，即使有时候有些棘手，我还是非常喜欢它，从长远来看，它要容易得多。

基本上，GUILayout只是GUI控件的扩展。大多数GUILayout函数仅调用等效的GUI函数，但在其周围添加layout。“正常” GUI功能完全忽略Layout事件。GUILayout函数使用内部系统来构建布局组，这些布局组在Layout事件期间以及调用Repaint事件之前将所有控件收集到一个组中，Unity根据每个控件所请求的空间来分配可用空间。

以下是有关GUILayout的基本信息：

- 基本上，GUI中存在的所有控件的确也存在于GUILayout中，但是它们缺少通常在其中指定其位置和大小的position参数。
- 所有内置的GUILayout控件只是使用了许多[GUILayoutUtility.GetRect](http://docs.unity3d.com/ScriptReference/GUILayoutUtility.GetRect.html)的重载来为控件保留一个Rect。layout控件只是从layout系统中获取一个rect，然后调用与该rect等效的GUI.XXX。
- GUI的position对位GUILayout函数末尾的变长参数数组，可让您添加一个或多个“[GUILayoutOptions](http://docs.unity3d.com/Documentation/ScriptReference/GUILayoutOption.html)”。这些选项“对象”不能直接创建。Unity已实现了初始化此类实例并返回它的函数。这些“选项”用于覆盖某些GUIStyle设置，例如拉伸，固定尺寸，最小和最大尺寸。
- GUI的[GUI Groups](http://docs.unity3d.com/Documentation/ScriptReference/GUI.BeginGroup.html)对位GUILayout的[Areas](http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginArea.html)。这是唯一必须指定Rect的东西。areas无法嵌套（比如两个Area嵌套，则只渲染第一个Area的内容），因此它的Rect始终相对于屏幕。当您在一个area内时，整个布局将同样限于该屏幕区域。
- 最重要的两个函数对是[Begin/EndHorizontal](http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginHorizontal.html)和[Begin/EndVertical](http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginVertical.html) 这些**可以**嵌套。因此，实际布局将通过将这些组嵌套在一起来定义。
- 接下来的两个助手是[Space](http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.Space.html)和[FlexibleSpace](http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.FlexibleSpace.html)。空间就像控件一样，但无需绘制任何内容。它只是在当前组中保留固定数量的像素。FlexibleSpace吞噬了所有剩余的空间。通常有两种控件，“固定大小”和“[拉伸](http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle-stretchWidth.html)”控件。Fixedsize并不意味着他们无法更改大小。他们只是占用所需的空间。所有拉伸控件（如FlexibleSpace）将共享剩余空间并根据其内容大小进行分配。

一些注意事项：
horizontal和vertical布局组通常没有GUIStyle，而是纯分组元素，但是您只需在参数中指定一种样式即可将其绘制为box/button/customstyle。

您可以通过指定其他样式来更改元素的外观。这将显示一个类似于普通按钮的toggle：

```csharp
togglestate = GUILayout.Toggle(togglestate, "Toggle Text", "button");
```

***更新（2016.02.18）***

# GUI（Layout）vs EditorGUI（Layout）

[EditorGUI](http://docs.unity3d.com/ScriptReference/EditorGUI.html)和[EditorGUILayout](http://docs.unity3d.com/ScriptReference/EditorGUILayout.html)只是GUI和GUILayout类的“扩展”，它们仅在editor脚本中可用，因为它们位于“ UnityEditor.dll”中。这两个类不能替代GUI和GUILayout。它们只是提供了其他控件，这些控件在编辑器脚本（如[custom inspector](http://docs.unity3d.com/ScriptReference/Editor.html)或[EditorWindows）中很有用](http://docs.unity3d.com/ScriptReference/EditorWindow.html)。在编辑器脚本中，普通的GUI和GUILayout控件也可以正常工作。

`您无法在运行时使用EditorGUI或EditorGUILayout中的任何控件，因为它们在引擎中不存在。它们属于编辑器。`

### Event更新

自从[新的UGUI系统](http://docs.unity3d.com/Manual/UISystem.html)发布以来，Event类现在具有两个附加的静态方法：

- [Event.GetEventCount](http://docs.unity3d.com/ScriptReference/Event.GetEventCount.html)
- [Event.PopEvent](http://docs.unity3d.com/ScriptReference/Event.PopEvent.html)

这些与旧的GUI系统并没有真正的关系，但是一些新的GUI行为脚本使用它们来处理输入事件。这些方法使您可以处理事件队列中等待当前帧的所有事件。它将返回与发送到OnGUI的事件相同的事件，只是它仅返回输入事件。因此，不会生成布局或重绘事件。

您自己使用它们时必须小心。一旦从队列中“弹出”事件，该事件就消失了。每个事件在队列中仅存在一次。因此，如果两个脚本试图以这种方式读取事件，则它将无法正常工作，因为第一个脚本会将事件从队列中删除。

***更新（2016.11.28）***

# 幕后花絮

### Control IDs

即使不是每个GUI控件都需要它们，但是它们中的大多数在内部分配了一个所谓的“Control ID”。此ID用于在多个OnGUI/OnInspectorGUI调用之间标识某个控件。控件可以使用[GUIUtility.GetControlID](https://docs.unity3d.com/ScriptReference/GUIUtility.GetControlID.html)为其自身分配一个或多个Control ID 。该方法采用一些应该有帮助的参数，以便每个控件始终获得相同的ID。为此，可以传递一个任意的“Hash”值（和int值），该值应表示控件的“类型”。通常，大多数控件使用基于某些字符串常量的缓存的哈希码：

```csharp
static int m_MyControlHash = "MyControlTypeNameHere".GetHashCode();
```

使用此方法标识控件可以让我们几乎不会意外的选择到另一个控件。在过去的GUI.Button中，此哈希存储在内部静态“ buttonHash”变量中：

```csharp
GUI.buttonHash = "Button".GetHashCode();
```

这些ID的实际目的是跟踪控件的状态（如果需要）。

注意：虽然可以有条件地调用某些控件，但**不要**在Layout事件和后续事件之间更改控件的数量和顺序非常重要。如上所述，OnGUI处理的每个事件始终与一直在实际事件之前出现的Layout事件配对。如果您在两者之间进行更改，则会使布局系统混乱，并可能“破坏”controlID堆栈。请记住，controlID是在每个OnGUI调用中生成的。因此，每次调用OnGUI时，Unity都会从一个空堆栈开始，它会根据已传递的不同“hints”，尝试将生成的ID与旧堆栈中的ID进行匹配。

### hotControl和keyboardControl

[GUIUtility.hotControl](https://docs.unity3d.com/ScriptReference/GUIUtility-hotControl.html)和[GUIUtility.keyboardControl](https://docs.unity3d.com/ScriptReference/GUIUtility-keyboardControl.html)可以标记控件的特殊状态。两者都是公共静态int字段，任何控件都可以读取或写入。两者都用于保存当前“hot”或当前具有键盘焦点的控件的controlID。大多数按钮，textField等可点击控件都使用hotControl。该过程通常如下：

- 当收到MouseDown事件，并且hotControl当前为“0”时，控件可能要检查鼠标是否在其Rect/clickable区域上方，如果是这种情况，它将hotControl为它的ControlID。这样可以防止其他控件对某些事件做出反应，因为它们还会根据hotControl变量检查它们的ControlID。
- 稍后收到MouseUp事件时，控件将检查自身当前是否为hotControl。在这种情况下，它将把hotControl重置为“0”。这很重要，因为仅当hotControl为0时才“允许”控件将hotControl值设置为自身的ControlID（译者注，也就是说允许自身或其他控件成为hotControl）。按钮通常会重复进行区域检查，并且如果两者都为true，则按钮为hotControl的并且鼠标仍位于按钮上方，控件将返回“true”，这将导致执行使用按钮的if块。

类似的过程用于keyboardControl。虽然通常不会在MouseUp上重置它，但是当鼠标不再位于控件区域内时在MouseDown上重置它。

### GetTypeForControl

[Event.GetTypeForControl](https://docs.unity3d.com/ScriptReference/Event.GetTypeForControl.html)基本上与[Event.type](https://docs.unity3d.com/ScriptReference/Event-type.html)属性相同，但是会基于实际发生的事件以及hotControl和keyboardControl的状态应用一些过滤。如果没有控件处于hotControl，则它基本上返回未经过滤的当前事件。这使每个控件都有机会使其首先变为hotControl。设置hotControl后，“GetTypeForControl”仅在传递的控件ID当前为hotControl或具有键盘焦点时才返回事件。hotControl主要过滤鼠标事件，而KeyboardControl过滤按键事件。好吧，这很明显，对^^。

### GetStateObject

[GUIUtility.GetStateObject](https://docs.unity3d.com/ScriptReference/GUIUtility.GetStateObject.html)方法的工作方式有点奇怪。尚不清楚Unity如何真正跟踪所涉及的对象以及是否在一段时间后将其删除，但是在某些情况下它还是很有用的。

那么它是做什么的。此方法能够为您创建一个任意的类实例，并将该对象链接到给定的控件ID。该类实例将继续存在。请注意，该类必须是普通类，即不是MonoBehaviour或ScriptableObject派生类。它还需要有一个无参数的构造函数，以便GetStateObject可以创建一个实例。您可能还记得，只要不声明任何参数化的构造函数，每个类默认都会自动具有一个无参数的构造函数。

这些“状态类”用于保存每个控制实例的自定义状态信息，这些信息需要传递到下一个OnGUI调用。在示例中，我记得是GUI.DragWindow控件。是的，它也是一个控件并分配控件ID，即使它不是“可视”控件也是如此。过去，DragWindow使用一个小的内部类（GUI.DragWindowState）来保存初始窗口rect和一个偏移量Vector2，以便可以正确计算拖动运动。

TextField还使用TextEditor类的实例，该实例比状态类仅多一点。实际上，TextEditor类内部实现了很多内容，比如我们输入的文本，编辑位置等。因此，它可以处理按键，使用箭头键移动光标，处理文本选择，...

### SceneView

对于编辑器编程，SceneView是功能最强大的工具之一，但是正确处理它也有些棘手。通常，有两种方法可以在SceneView中绘制或处理GUI事件：

- 活动的CustomEditor（custom inspector）可以实现一种称为[OnSceneGUI](https://docs.unity3d.com/ScriptReference/Editor.OnSceneGUI.html)的方法。
- 任何其他编辑器类（例如EditorWindows或标记有InitializeOnLoad的类）都可以将方法委托给静态`SceneView.onSceneGUIDelegate`。

Unity编辑器的每个实际window都有其自己的图形上下文，并且所有内容都由IMGUI系统驱动。Unity主编辑器窗口中只有很少的内容是用native代码（例如主菜单）实现的。

因此，一旦有了SceneView的实际回调，您就可以做任何您想做的事情。与普通的EditorWindows（inspector, hierarchy, project面板）不同，SceneView通常使用3d投影并使用相机绘制大多数内容。由于这种2d /3d混合状态，因此场景视图中的工作方式与其他EditorWindows有所不同。

通常，您在SceneView内部处理3d坐标，因此对于大多数事情而言，所有GUI控件都非常无用。这就是为什么有[Handles class](https://docs.unity3d.com/ScriptReference/Handles.html)的原因。SceneView和Handles类属于彼此（译者注，难听点叫耦合）。它提供了一些3d控件，例如PositionHandle，FreeMoveHandle，ScaleSlider和RadiusHandle等。

即使场景视图主要是3d上下文，您仍然可以在场景视图中使用普通的2d GUI，但是您必须在Handles.BeginGUI（）和Handles.EndGUI（）之间包装2d gui内容。这使您可以直接在SceneView内部显示和GUI控件。一个非常重要的方法位于HandleUtility类中，称为WorldToGUIPoint。它一次完成所有讨厌的转换（3d-> screenspace-> guispace->当前cliparea）。GUIPointToWorldRay与之相反。那里还有许多其他有用的方法，但这当然取决于需求。即使绘制2d控件也是可能的，您也应该尽量避免使用它。设置和选项应显示在inpector中或单独的EditorWindow中。

### Tools

当您想实现自己的工具以某种方式编辑场景时，这是一个非常重要的小类。它提供的功能不多，但控制着一项重要功能。静态`Tools.current`属性是类型为“工具”的枚举。工具目前定义了这些值：`View, Move, Rotate, Scale, Rect, None`。这些应该是熟悉的，因为它们是编辑器中的标准工具，可以使用左上方的工具按钮进行切换。对我们而言，唯一真正重要的价值是“None”。将Tools.current设置为Tool.None会禁用所有默认工具，使您可以实施自己的工具。地形Inspector也是如此。当您在Inspector中选择一种“painting mode”时，您可能会注意到，左上方的所有工具按钮都将被切换为“关闭”。这意味着当前工具为“None”。

实现自己的工具时，您最可能需要的另一重要事项是防止取消选择当前对象。这是通过分配一个被动控件id并将其传递给 `HandleUtility.AddDefaultControl`：

```csharp
int myID = GUIUtility.GetControlID(FocusType.Passive);
HandleUtility.AddDefaultControl(myID);
```

待续 ...

# 译者小彩蛋

都看到这里了，让我装个杯不过分吧，来看看美容前和美容后的编辑器对比吧

![image-20210418212147425](https://myfirstblog.oss-cn-hangzhou.aliyuncs.com/typoraImages_2/20210418212309.png)

![QQ截图20210418211819](https://myfirstblog.oss-cn-hangzhou.aliyuncs.com/typoraImages_2/20210418211925.png)
