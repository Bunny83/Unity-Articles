
Unity's IMGUI system is quite simple but very powerful. The main parts are:

 - The [OnGUI callback][3]. This callback is used to handle everything that has to do with GUI.
 - The [Event class][4] which is tightly connected to OnGUI.
 - The GUI controls defined in [GUI][5], [GUILayout][6] and *additional* controls which are only available in the editor in [EditorGUI][7] and [EditorGUILayout][8].
 - The [GUIStyle class][9] which defines how a single control looks like. It's actually responsible for any GUI drawing.
 - The [GUISkin class][10] which is basically just a collection of predefined default styles for all built in controls and has an array for unlimited custom styles at the end.
 - The utility classes [GUIUtility][11] and in the editor the [EditorGUIUtility][12]. Not to forget the tiny but important [GUILayoutUtility class][13] when it comes to using GUILayout classes.

# OnGUI


OnGUI is a callback that is called automatically by the engine for several purposes. It's *usually* called at least two times per frame but might be called several times a frame when other events happens.

OnGUI is, in my opinion, a bad name. Of course it's main purpose is to "do GUI stuff", but it's actually an event processor callback. I will explain that in the next section.


# The Event class


The static [Event.current][14] holds the current [EventType][16] that describes why the `OnGUI/OnInspectorGUI` callbacks are being called. 

Here's a quick overview of the different event types:

 - **Mouse events** like MouseDown, MouseUp, MouseMove(**editor only**), MouseDrag, ScrollWheel. I should add that Unity will emulate those events on mobile touch devices. It calculates the mouseposition by taking the arithmetic mean of all touches.
 - **Keyboard events** like KeyDown and KeyUp. Note: unlike Input.GetKeyDown / Up those events map the system's OS keyboard events. So when you hold down a key, the keydown event is repeated by the systems repeat rate. This is of course important for text editor GUIs.
 - **The Layout event**. This is a special event that is always executed before all other eventa and is used to determine the size and position for automatic layouted controls. Note: This event can be disabled with [useGUILayout][17] but keep in mind that all GUILayout stuff won't work in this case.
 - **The Repaint event**. This event will actually draw the GUI elements.
 - The **Used** event type. Any control that processes an event and don't want others to get this event can use the [Use function][18] to "eat" the event. When Use is called the event type will be set to Used and every control should just ignore this event.

Beside the type of event, it holds additional information for the event.

 - [mousePosition][19] holds the mouse position in **GUI coordinates**. GUI coordinates are different from screen coordinates. The GUI has it's origin at the **top left** corner, screen coordinates has it's origin at the **bottom left** corner. The [GUIUtility class][20] has functions to convert those coordinates.
 - [button][21]. Only valid for mouse events and contains the mouse button index (0 - left button, 1 - right button, 2 - middle button)
 - [modifiers][22] hold information about Shift, Alt, Ctrl keys
 - [keyCode][23]. Only valid for keyboard events. Holds the [KeyCode][24] of the key that has been pressed / released.
 - Some additional things like shortcuts for modifier keys.

# GUIStyle


As already mentioned this class is responsible for rendering a control to the screen. A GUI control function will call one of the [Draw functions][25] when the repaint event is "detected". An GUIStyle consists of different [GUIStyleStates][26] where only one is "active" at a time. Which state is used is determined by what Draw function is called, which parameters are passed and if the control has currently the [keyboard focus][27] or not.

When a single GUIStyle is drawn it can produce 0 to 3 draw calls. That's why very complicated GUIs are considered bad for mobile development, however we used the GUI system a lot on Android and iOS and hadn't much problems on more or less modern devices.

The things a GUIStyle might draw are:

 - The background image from the GUIStyleState. This is usually used to define how the button / textfield / ... generally looks like. If a background image is set it's always drawn first.
 - An content image defined in the GUIContent class that is passed to Dram.
 - The content's Text also defined in the GUIContent.

The [ImagePosition][28] in the GUIStyle defines where the image is displayed in relation to the text or to only draw the image and ignore the text or the other way round.

A GUIStyle has 8 <a href="http://docs.unity3d.com/ScriptReference/GUIStyleState.html">GUIStyleState</a>s where each state allows you to specify a background image as well as a text color. Note: A state is only used when it has a Background image. So unfortunately it's not possible to just setup text colors without an background image.

The different states are:

 - normal
 - hover
 - active
 - focused
 - onNormal
 - onHover
 - onSctive
 - onFocused

**normal** is the stylestate that is used in most cases. A label for example will only use this state.  
**hover** is used by buttons, toggles and most other interactive elements when you hover over the element.  
**active** is used for interactive elements while you click-and-hold on the element. It's the button down state.  
**focused** is used by TextFields and TextAreas when the editor is active and you can edit the text.

All the **onXXX** states have the same function as the other 4 above, but are used for elements that have an "on" state like a Toggle. So if the toggle is unchecked and you click on it you will see the *active* state. When the toggle is already checked you will see the *onActive* state.

As mentioned above states without a background image are ignored. So if your button style has a "normal" and "active" state with a texture but no hover state, Unity will use the normal state when you hover over your button. If you want to just change the text color when you hover you have to use the same background image for "hover" as you used for "normal" otherwise the hover state will be ignored.

As last note: Unity has implemented an implicit casting operator (sounds complicated, but it isn't :D) that will automatically convert a string to a GUIStyle by using the [GetStyle][29] function of the current GUISkin.

# The GUISkin class


The GUISkin is, like mentioned above, just a collection of GUIStyles. Unlike GUIStyles a GUISkin can *easily* be stored as Asset in your project. You can create one in the Assets/Create menu (same as context menu in the project window).

To use a GUISkin it just has to be assigned to GUI.skin before you draw a control. The usual way is to place the assignment at the top of OnGUI. It has to be assigned everytime OnGUI is called. The assignment is only valid for the current execution of OnGUI. If you want to use the skin in other scripts as well you have to place the assignment in those as well.

Most controls have optional parameters to specify a GUIStyle which should be used to draw the control. At this place you can simply use a string (thanks to the implicit cast) with the name of the style you want to use. If no style is passed each builtin control has it's own default style.

# The Control functions


The above mentioned classes define the basis of the GUI system. They can be used to create custom controls or can be directly used in some way.

Unity however provides premade [control functions][30] which simplifies the usage a lot. For example the [Button][31] function. It uses the default style "button" to draw itself to the screen. This function also processes MouseDown and MouseUp events. It uses the controls Rect to determine if the mouse position is within it's Rect and make itself the [hotControl][32]. The function returns a boolean value which will be always false except in the MouseUp event when it's the hotcontrol and released inside it's Rect.

That's how the click event is passed back to the caller.

Some controls, like scrollbars and sliders use more than one GUIStyle. They usually have one main style and additional "child styles" which are determined by adding something to the name of the main style. The horizontalScrollbar literally does this internally:

    GUI.skin.GetStyle(style.name + "thumb")
    GUI.skin.GetStyle(style.name + "leftbutton")
    GUI.skin.GetStyle(style.name + "rightbutton")

to get the other styles. this is a suboptimal way since if you don't have the source code of the scrollbar function (or [a reflector][33]) you don't know that.

# GUI vs GUILayout


Some hate GUILayout some like it. Personally i really like it even when it's sometimes a bit tricky it's much easier in the long run.

Basically GUILayout is just an extension of the GUI controls. Most GUILayout functions just call the equivalent GUI function but adds the layouting around it. The "normal" GUI functions completely ignores the Layout event. The GUILayout functions use an internal system that builds up layout groups which collect all controls in a group during the Layout event and before the Repaint event is invoked Unity distributes the available space according to how much space each single control has requested.

Here are the basic things about GUILayout:

 - Basically all controls that exists in GUI does also exist in GUILayout, but they lack of an position parameter where you usually specify it's position and size.
 - All the built-in control functions inside GUILayout simply uses one of the many overloads of [GUILayoutUtility.GetRect][34] to reserve a Rect for the control. The layout controls just get a rect from the layout system and in turn call the GUI.XXX equivalent with that rect.
 - Instead of a position they have an additional parameter array at the end which allows you to add no, one or more "[GUILayoutOptions][35]". Those option "objects" can't be directly created. Unity has implemented functions that initializes such an instance and returns it. Those "Options" are used to override certains GUIStyle settings like stretching, fixedsize, min and max size.
 - Instead of the [GUI Groups][36] there are [Areas][37]. This is the only thing where you have to specify a Rect. Areas can't be cascaded, so the Rect is always relative to the screen. While you are inside an area the whole layouting will be restricted to this screen area.
 - The most important two function pairs are [Begin / EndHorizontal][38] and [Begin / EndVertical][39]. Those **can** be cascaded. So the actual layout will be defined with stacking these groups together.
 - The next two helpers are [Space][40] and [FlexibleSpace][41]. Space acts like a control without drawing anything. It just reserves a fix amount of pixels in the current group. FlexibleSpace eats up all left over space. Generally there are two types of controls, "fixedsized" and "[stretching][42]" controls. Fixedsized doesn't mean they aren't able to change size. They just take as much space they need. All stretching controls (like FlexibleSpace) will share the left over space and distribute it according to their content sizes.

Some notes:  
The horizontal and vertical layout groups usually don't have a GUIStyle and are pure grouping elements, however you can simply specify a style in the parameters to draw it as box / button / customstyle.

You can change the appearance of an element by just specify a different style. This will display a toggle which looks like a normal button:

    togglestate = GUILayout.Toggle(togglestate, "Toggle Text", "button");

***update(2016.02.18)***  

# EditorGUI(Layout) vs GUI(Layout)

[EditorGUI][43] and [EditorGUILayout][44] are simply "extensions" of the GUI and GUILayout classes which are only available inside editor script as those are located in the "UnityEditor.dll". Those two classes aren't ment as a replacement of GUI and GUILayout. They simply provide additional controls which are useful in editor script like [custom inspectors][45] or [EditorWindows][46]. In editor script the normal GUI and GUILayout controls work just fine.

Before someone asks, no, you can't use any controls from EditorGUI or EditorGUILayout at runtime as they don't exist in the engine. They belong to the editor.

### Recent additions to the Event class

Since the release of the [new UGUI system][47] the Event class has now two additional static methods:

 - [Event.GetEventCount][48]
 - [Event.PopEvent][49]

Those are not really related to the old GUI system but are used by some of the new GUI behaviour scripts to process input events. Those methods allow you to process all events that are waiting in the event queue for the current frame. It will return the same events as those send to OnGUI except it only returns input events. So no layout or repaint events are generated.

Though you have to be careful when using those yourself. Once you "popped" an event from the queue, it's gone. Each event exists only once in the queue. So if two scripts try to read events that way it won't work since the first script will remove the event from the queue.

***update(2016.11.28)***  

# Behind the scenes
### Control IDs

Even though they are not required for every GUI control, most of them internally allocate a so called "control ID". This ID is ment to identify a certain control across multiple calls of OnGUI / OnInspectorGUI. A control can allocate itself one or multiple control IDs by using the function [GUIUtility.GetControlID][50]. That method takes some parameters which  should help so each control gets always the same ID. For this purpose one can pass an arbitrary "hash" value (and int value) that should represent your control's "type". Usually most controls use a cached hashcode based on some string constant:

    static int m_MyControlHash = "MyControlTypeNameHere".GetHashCode();

Using this method makes it unlikely to accidentally pick the same int as another control type. In the past GUI.Button used this hash stored in the internal static "buttonHash" variable:

    GUI.buttonHash = "Button".GetHashCode();

The actual purpose of those IDs is to track the state for a control if it needs it.

Note: while it's possible to call some controls conditionally, it's very important to **not** change the count and order of controls between the Layout event and the following event. As mentioned above every event that is processed by OnGUI is always paired with a Layout event that always comes before the actual event. If you change something in between it will mess up the layouting system and could "corrupt" the controlID stack. Keep in mind the control IDs are generated every OnGUI call. So each time OnGUI is called Unity starts with an empty stack It tries to match the generated IDs with the IDs from the old stack based on the different "hints" that has been passed.

### hotControl and keyboardControl

[GUIUtility.hotControl][51] as well as [GUIUtility.keyboardControl][52] can mark a special state for a control. Both are public static int fields which any control might read or write to. Both are ment to hold the control ID of a control which is currently "hot" or currently has the keyboard focus. hotControl is used by most clickable controls like Button / Toggle. The process is usually the following:

 - When a MouseDown event is received, and hotControl is currently "0" a control might want to check if the mouse is over it's Rect / clickable area and if that's the case it will set the hotControl to it's control ID. That prevents other control from reacting to certain events since they also check their ID against the hotControl variable.
 - When later a MouseUp event is received a control will check if itself is currently the hot control. In that case it will reset hotControl back to "0". This is important as controls are only "allowed" to put themselfs up as hotControl when it's currently 0. A button usually repeats the area check and if both is true, so the button is hot and the mouse was still over the button, the control will return "true" which will cause the if block, in which the button is used, to execute.

A similar process is used for keyboardControl. Though it's usually not reset on MouseUp but on MouseDown when the mouse is no longer inside the controls area.

### GetTypeForControl

[Event.GetTypeForControl][53] basically does the same as the [Event.type][54] property, but applies some filtering based on what event actualy happened and based on the state of hotControl and keyboardControl. If no control is hot it basically returns the current event unfiltered. This gives each control the chance to make them hot in the first place. Once a hotControl is set, "GetTypeForControl" will only return the event when the passed control id is currently hot or has the keyboard focus. hotControl mainly filters mouse events while keyboardControl filters key events. Well, that's kinda obvious, right ^^.

### GetStateObject
The [GUIUtility.GetStateObject][55] method is a bit strange in how it works. It's unclear how Unity actually keeds track of the objects involved and if they are actually removed after some time or not, but nontheless it's a quite useful in some situations.

So what does it do. This method basicalls is capable of creating an arbitrary class instance for your and link that object to the given control id. The class instance will persist. Note that the class must be a normal class, so no MonoBehaviour or ScriptableObject derived class. It also need to have a parameterless constructor so GetStateObject can actually create an instance. As you might remember as long as you don't declare any parameterized constructor every class automatically has a parameter-less constructor by default.

Those "state classes" are ment to hold custom state information per control instance that need to be carried over to the next OnGUI calls. On example i remember was the GUI.DragWindow control. Yes, it's also a control and allocates a control ID even though it's not a "visual" control. In the past DragWindow used a small internal class (GUI.DragWindowState) to hold the initial window rect and an offset Vector2 so the drag movement can be calculated correctly.

The TextField also uses an instance of the TextEditor class which is a bit more then only a state class. The TextEditor class actually implements the whole editing functionality that you know from any inputfield. So it handles keypresses, moves the cursor with the arrow keys, handles the text selection, ...

#Editor and SceneView programming
### The SceneView

When it comes to editor programming, the SceneView is one of the most powerful one but can also be a bit tricky to handle it right. Generally there are two ways how to draw or handle GUI events in the SceneView:

 - An active CustomEditor (custom inspector) can implement a method called [OnSceneGUI][56]
 - Any other editor class (like EditorWindows or classes marked with InitializeOnLoad) can subscribe a method to the static `SceneView.onSceneGUIDelegate` event.

In case it wasn't clear yet, every actual window of the Unity editor has it's own graphics context and everything is driven by the IMGUI system. Only a few things in the Unity main editor window are implemented in native code (such as the main menu).

So once you have an actual callback of the SceneView you can do whatever you want. Unlike normal EditorWindows (which includes the inspector, hierarchy, project panel) the SceneView usually works with a 3d projection and uses a camera to draw most things. Due to this 2d / 3d hybrid state things work a bit different in the sceneview as opposed to other EditorWindows.

Usually you deal with 3d coordinates inside the SceneView therefore for most things all the GUI controls are pretty useless. That's why there is [the Handles class][57]. SceneView and the Handles class belong to each other. It provides some 3d controls like the PositionHandle, FreeMoveHandle, ScaleSlider and RadiusHandle just to name a few.

Even the sceneview is mainly a 3d context you can still use plain 2d GUI inside the sceneview, but you have to wrap 2d gui stuff between Handles.BeginGUI() and Handles.EndGUI(). This allows you to directly display and GUI controls inside the sceneview. A quite important method is located in the HandleUtility class and is called WorldToGUIPoint. It does all the nasty conversions at once (3d->screenspace->guispace->current cliparea). Kind of the opposite does GUIPointToWorldRay. There are many other useful methods in there but it of course depends on the needs. Even drawing 2d controls is possible you should use it sparsely. Settings and options should be displayed either in the inspector or in a seperate EditorWindow.

### The Tools class

When you want to implement your own tool to somehow edit the scene this is a quite important little class. It doesn't provide much but controls an important feature. The static `Tools.current` property is an enum of type "Tool". Tool defines currently those values: `View, Move, Rotate, Scale, Rect, None`. Those should be familiar as they are the standard tools you have in the editor which can be switched with the tool buttons at the top left. The only really important value here for us is "None". Setting Tools.current to Tool.None disables all default tools which allows you to implement your own tool. The same does the Terrain inspector. When you select one of the "painting modes" in the inspector you might notice that all tool buttons at the top left will be toggled "off". That means the current tool is "None".

Another important thing you most likely need when implementing your own tool is to prevent the deselection of the current object. This is done by allocating a passive control id and pass it to  `HandleUtility.AddDefaultControl`:

    int myID = GUIUtility.GetControlID(FocusType.Passive);
    HandleUtility.AddDefaultControl(myID);

That's all for now about sceneview, handles and custom tools.

To be continued ...

... omg 57 links ^^


  [3]: http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.OnGUI.html
  [4]: http://docs.unity3d.com/Documentation/ScriptReference/Event.html
  [5]: http://docs.unity3d.com/Documentation/ScriptReference/GUI.html
  [6]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.html
  [7]: http://docs.unity3d.com/Documentation/ScriptReference/EditorGUI.html
  [8]: http://docs.unity3d.com/Documentation/ScriptReference/EditorGUILayout.html
  [9]: http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle.html
  [10]: http://docs.unity3d.com/Documentation/ScriptReference/GUISkin.html
  [11]: http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility.html
  [12]: http://docs.unity3d.com/Documentation/ScriptReference/EditorGUIUtility.html
  [13]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayoutUtility.html
  [14]: http://docs.unity3d.com/Documentation/ScriptReference/Event-current.html
  [15]: http://docs.unity3d.com/Documentation/ScriptReference/Event-type.html
  [16]: http://docs.unity3d.com/Documentation/ScriptReference/Event-type.html
  [17]: http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour-useGUILayout.html
  [18]: http://docs.unity3d.com/Documentation/ScriptReference/Event.Use.html
  [19]: http://docs.unity3d.com/Documentation/ScriptReference/Event-mousePosition.html
  [20]: http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility.html
  [21]: http://docs.unity3d.com/Documentation/ScriptReference/Event-button.html
  [22]: http://docs.unity3d.com/Documentation/ScriptReference/Event-modifiers.html
  [23]: http://docs.unity3d.com/Documentation/ScriptReference/Event-keyCode.html
  [24]: http://docs.unity3d.com/Documentation/ScriptReference/KeyCode.html
  [25]: http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle.Draw.html
  [26]: http://docs.unity3d.com/Documentation/ScriptReference/GUIStyleState.html
  [27]: http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility-keyboardControl.html
  [28]: http://docs.unity3d.com/Documentation/ScriptReference/ImagePosition.html
  [29]: http://docs.unity3d.com/Documentation/ScriptReference/GUISkin.GetStyle.html
  [30]: http://docs.unity3d.com/Documentation/ScriptReference/GUI.html
  [31]: http://docs.unity3d.com/Documentation/ScriptReference/GUI.Button.html
  [32]: http://docs.unity3d.com/Documentation/ScriptReference/GUIUtility-hotControl.html
  [33]: http://ilspy.net/
  [34]: http://docs.unity3d.com/ScriptReference/GUILayoutUtility.GetRect.html
  [35]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayoutOption.html
  [36]: http://docs.unity3d.com/Documentation/ScriptReference/GUI.BeginGroup.html
  [37]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginArea.html
  [38]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginHorizontal.html
  [39]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.BeginVertical.html
  [40]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.Space.html
  [41]: http://docs.unity3d.com/Documentation/ScriptReference/GUILayout.FlexibleSpace.html
  [42]: http://docs.unity3d.com/Documentation/ScriptReference/GUIStyle-stretchWidth.html
  [43]: http://docs.unity3d.com/ScriptReference/EditorGUI.html
  [44]: http://docs.unity3d.com/ScriptReference/EditorGUILayout.html
  [45]: http://docs.unity3d.com/ScriptReference/Editor.html
  [46]: http://docs.unity3d.com/ScriptReference/EditorWindow.html
  [47]: http://docs.unity3d.com/Manual/UISystem.html
  [48]: http://docs.unity3d.com/ScriptReference/Event.GetEventCount.html
  [49]: http://docs.unity3d.com/ScriptReference/Event.PopEvent.html
  [50]: https://docs.unity3d.com/ScriptReference/GUIUtility.GetControlID.html
  [51]: https://docs.unity3d.com/ScriptReference/GUIUtility-hotControl.html
  [52]: https://docs.unity3d.com/ScriptReference/GUIUtility-keyboardControl.html
  [53]: https://docs.unity3d.com/ScriptReference/Event.GetTypeForControl.html
  [54]: https://docs.unity3d.com/ScriptReference/Event-type.html
  [55]: https://docs.unity3d.com/ScriptReference/GUIUtility.GetStateObject.html
  [56]: https://docs.unity3d.com/ScriptReference/Editor.OnSceneGUI.html
  [57]: https://docs.unity3d.com/ScriptReference/Handles.html
