# 2.3 Status Management

## 2.3.1 Introduction

Responsive programming frameworks will always have a theme of "State management". Whether it is React/Vue (both Web development frameworks that support reactive programming) or Flutter, the issues discussed and the workarounds are the same. So, if you are familiar with React/Vue state management, you can skip this section. Anyway, let's ask ourselves a question: Who should manage the state of `StatefulWidget`? The Widget itself? Its Parent Widget? Both of them? Or another object? The answer is it depends! The following are the most common ways to manage state:

- Widgets manage their own state.
- The Widget manages the state of child widgets.
- Mixed management (both parent and child widgets manage state).

How do you decide which management method to use? Here are some official rules to help you make your decision:

- If the state is user data, such as the checked state of a check box or the location of a slider, then the state is best managed by the parent Widget.
- If the state is about the appearance of the UI, such as color or animation, then the state is best managed by the Widget itself.
- If a state is shared by different widgets, it is best managed by their common parent Widget.

Managing states inside the widget has better encapsulation, while managing states in the parent widget is more flexible. In some cases, if you're not sure how to manage state, the preferred option is to manage states in the parent Widget (flexibility is more important).

Next, we'll illustrate the different ways to manage state by creating three simple examples: TapboxA, TapboxB, and TapboxC. These examples have similar function — creating a box, and when clicked, the box background switches between green and gray. The state `_active` determines the color. Green is true and gray is false, as shown in Figure 2-8:

![图2-8](https://raw.githubusercontent.com/happylee1/PublicPicBed/main/img/2-8-1683559434546-3.png)

The following example uses a `GestureDetector` to identify click events. We'll cover the details of this `GestureDetector` later in the "Event Handling" chapter.

## 2.3.2 The Widget manages its own state

We implement a TapboxA in its corresponding `_TapboxAState` class:

- Manages the state of TapboxA.
- Definition of `_active`: A Boolean value Determining the current color of the box.
- Definition of `_handleTap()` Function, which updates `_active` when the box is clicked, and call `setState()` to update the UI.
- Implements all the interactive behavior of a widget.

```dart
// TapboxA manages its own state.

//------------------------- TapboxA ----------------------------------

class TapboxA extends StatefulWidget {
  TapboxA({Key? key}) : super(key: key);

  @override
  _TapboxAState createState(a)= > _TapboxAState(a); }class _TapboxAState extends State<TapboxA> {
  bool _active = false;

  void _handleTap() {
    setState(() {
      _active = !_active;
    });
  }

  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _handleTap,
      child: Container(
        child: Center(
          child: Text(
            _active ? 'Active' : 'Inactive',
            style: TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: BoxDecoration(
          color: _active ? Colors.lightGreen[seven hundred] : Colors.grey[600],),),); }}
```

## 2.3.3 Parent Widget manages states of child Widgets

It is often a good way for the parent Widget to manage states and tell its children when to update. For example, an `IconButton` is an icon button, but it is a stateless `Widget` because we assume that the parent Widget needs to know if the button has been clicked in order to act accordingly.

In the following example, TapboxB exports its state to its parent widget via a callback, which is managed by the parent widget, so its parent widget is the `StatefulWidget`. But TapboxB does not manage any state, so TapboxB is a `StatelessWidget`.

The `ParentWidgetState` class:

- Manages the `_active` State for TapboxB.
- Implements `_handleTapboxChanged()`, which is called when the box is clicked.
- Calls `setState()` to update the UI when the state changes.

`TapboxB` class:

- Subclasses `StatelessWidget` Class, because all state is handled by its parent widget.
- It notifies the parent widget when a click is detected.

```dart
// ParentWidget indicates the TapboxB management status.

//------------------------ ParentWidget --------------------------------

class ParentWidget extends StatefulWidget {
  @override
  _ParentWidgetState createState(a)= > _ParentWidgetState(a); }class _ParentWidgetState extends State<ParentWidget> {
  bool _active = false;

  void _handleTapboxChanged(bool newValue) {
    setState(() {
      _active = newValue;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      child: TapboxB(
        active: _active,
        onChanged:_handleTapboxChanged, ), ); }}
//------------------------- TapboxB ----------------------------------

class TapboxB extends StatelessWidget {
  TapboxB({Key? key, this.active: false, required this.onChanged})
      : super(key: key);

  final bool active;
  final ValueChanged<bool> onChanged;

  void _handleTap() {
    onChanged(!active);
  }

  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _handleTap,
      child: Container(
        child: Center(
          child: Text(
            active ? 'Active' : 'Inactive',
            style: TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: BoxDecoration(
          color: active ? Colors.lightGreen[seven hundred] : Colors.grey[600],),),); }}
```

## 2.3.4 Mixed Status Management

For some widgets, a mixed management approach can be very useful. In this case, the widget itself manages some internal state, and the parent widget manages some other external state.

In the TapboxC example below, a dark green border appears around the box when the finger is pressed down, and disappears when it is lifted. After clicking finish, the color of the box changes. TapboxC exports its _active state to its parent widget, but manages its _highlight state internally. This example has two state objects _ParentWidgetState and _TapboxCState.

The _ParentWidgetStateC class:

- management`_active`State.
- realization`_handleTapboxChanged()`, called when the box is clicked.
- When clicking on the box and`_active`Called when the state changes`setState()`Update the UI.

_TapboxCState object:

- management`_highlight`State.
- `GestureDetector`Listen for all tap events. When the user clicks down, it adds highlights (dark green border); When the user releases, the highlight is removed.
- Update when you press, lift, or unclick`_highlight`State, call`setState()`Update the UI.
- When clicked, the change of state is passed to the parent widget.

```dart
//---------------------------- ParentWidget ----------------------------

class ParentWidgetC extends StatefulWidget {
  @override
  _ParentWidgetCState createState(a)= > _ParentWidgetCState(a); }class _ParentWidgetCState extends State<ParentWidgetC> {
  bool _active = false;

  void _handleTapboxChanged(bool newValue) {
    setState(() {
      _active = newValue;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      child: TapboxC(
        active: _active,
        onChanged:_handleTapboxChanged, ), ); }}
//----------------------------- TapboxC ------------------------------

class TapboxC extends StatefulWidget {
  TapboxC({Key? key, this.active: false, required this.onChanged})
      : super(key: key);

  final bool active;
  final ValueChanged<bool> onChanged;
  
  @override
  _TapboxCState createState(a)= > _TapboxCState(a); }class _TapboxCState extends State<TapboxC> {
  bool _highlight = false;

  void _handleTapDown(TapDownDetails details) {
    setState(() {
      _highlight = true;
    });
  }

  void _handleTapUp(TapUpDetails details) {
    setState(() {
      _highlight = false;
    });
  }

  void _handleTapCancel() {
    setState(() {
      _highlight = false;
    });
  }

  void _handleTap() {
    widget.onChanged(!widget.active);
  }

  @override
  Widget build(BuildContext context) {
    // Add a green border when pressed and unhighlight when lifted
    return GestureDetector(
      onTapDown: _handleTapDown, // Handle the press event
      onTapUp: _handleTapUp, // Handle lift events
      onTap: _handleTap,
      onTapCancel: _handleTapCancel,
      child: Container(
        child: Center(
          child: Text(
            widget.active ? 'Active' : 'Inactive',
            style: TextStyle(fontSize: 32.0, color: Colors.white),
          ),
        ),
        width: 200.0,
        height: 200.0,
        decoration: BoxDecoration(
          color: widget.active ? Colors.lightGreen[seven hundred] : Colors.grey[600],
          border: _highlight
              ? Border.all(
                  color: Colors.teal[seven hundred],
                  width: 10.0,): null),),); }}
```

Another implementation might export the highlighting state to the parent widget while keeping the _active state internal, which might not make sense if you were to use the TapBox for someone else. Developers only care if the box is Active, not how the highlighting is managed, so let TapBox handle these details internally.

## 2.3.5 Global Status Management

When the application requires some state synchronization across widgets (including across routes), the above approach is not adequate. For example, we have a Settings page where we can set the language of the App. In order for the Settings to take effect in real time, we expect the language-dependent widgets of the app to be rebuilt when the language state changes, but these language-dependent widgets and the Settings page are not together, so this situation is difficult to manage with the above method. In this case, the right thing to do is to handle this communication between distant widgets through a global state manager. At present, there are two main approaches:

1. Implement a global event bus, corresponding language state changes to an event, and then rely on the application language widgets in the APP`initState`Method to subscribe to language change events. When the user switches languages in the Settings page, we publish the language change event, and the widget that subscribed to the event is notified and called upon receiving the notification`setState(...)`Method reversion`build`Just by itself.
2. Use some of the packages dedicated to state management, such as Provider and Redux, which you can view in detail at pub.

Provider packages are implemented and used in the "Functional Widgets" chapter, and a global event bus is implemented in the "Event Handling and Notification" chapter, which readers can flip through as needed.