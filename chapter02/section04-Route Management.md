# 2.4 Route Management

Route in mobile development usually refers to a Page, which is the same as the concept of Route in Web development for single-page application. Route in Android usually refers to an Activity, and in iOS refers to a ViewController. The so-called route management, is to manage how to jump between pages, usually also known as navigation management. Route management in Flutter is similar to native development in that navigation management maintains a routing stack for both Android and iOS, push for opening a new page, pop for closing a page, and route management is about how the routing stack is managed.

## 2.4.1 A simple example

Based on the "counter" example in Section 2.1, we make the following modifications:

1. Create a new route and name it "NewRoute"

    ```dart
    class NewRoute extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(
            title: Text("New route"),
          ),
          body: Center(
            child: Text("This is new route"),),); }}
    ```

    The new route inherits from the StatelessWidget and has a simple interface that says "This is new route" in the middle of the page.

2. Add a button (TextButton) to the widget for Column in the _MyHomePageState.build method:

    ```dart
    Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[...// Omit irrelevant code
        TextButton(
          child: Text("open new route"),
          onPressed: () {
            // Navigate to the new route
            Navigator.push( 
              context,
              MaterialPageRoute(builder: (context) {
                return NewRoute(a); }),); },),],)
    ```

    We added a button to open a new route. Clicking this button will open a new route page, as shown in Figure 2-9 and 2-10.

    ![图2-9](./assets/2-9.png)

    ![图2-10](./assets/2-10.png)

## 2.4.2 MaterialPageRoute

MaterialPageRoute inherits from the PageRoute class, which is an abstract class representing a modal routing page that occupies an entire screen space. The MaterialPageroute class also defines interfaces and attributes associated with transition animations during route building and switching. MaterialPageRoute is a component provided by the Material component library that enables routing switching animations consistent with the platform page switching animation style for different platforms:

- For Android, when a new page is opened, the new page slides from the bottom of the screen to the top of the screen; When you close the page, the current page slides from the top of the screen to the bottom of the screen and disappears, while the previous page appears on the screen.
- For iOS, when a page is opened, the new page will slide from the right edge of the screen to the left of the screen until the new page is fully displayed on the screen, while the previous page will slide from the current screen to the left of the screen and disappear; When you close a page, the reverse is true: the current page slides out from the right side of the screen, while the previous page slides in from the left.

Here is what the MaterialPageRoute constructor means:

```
  MaterialPageRoute({
    WidgetBuilder builder,
    RouteSettings settings,
    bool maintainState = true,
    bool fullscreenDialog = false,})
```

- `builder`Is a callback function of type WidgetBuilder that builds the specific content of the routed page and returns a widget. We usually implement this callback to return an instance of the new route.
- `settings`Contains route configuration information, such as route name and initial route (home page).
- `maintainState`By default, when a new route is added to the stack, the original route is still stored in memory. If you want to release all resources occupied by the route when it is useless, you can set this parameter`maintainState`for`false`.
- `fullscreenDialog`Indicates whether the new routing page is a full screen modal dialog box, in iOS if`fullscreenDialog`for`true`The new page will slide in from the bottom of the screen (instead of horizontally).

> If you want to customize the route switching animation, you can do it yourself by inheriting PageRoute. We will implement a custom routing component later when we introduce the animation.

## 2.4.3 Navigator

Navigator is a routing management component that provides ways to open and exit routing pages. The Navigator manages the collection of active routes through a stack. Usually, the page displayed on the current screen is the route at the top of the stack. Navigator provides a number of ways to manage the routing stack, and we'll just cover two of its most common methods:

### 1. `Future push(BuildContext context, Route route)`

When a given route is pushed onto the stack (that is, a new page is opened), the return value is a Future object to receive the data returned when the new route is pushed off the stack (that is, closed).

### 2. `bool pop(BuildContext context, [ result ])`

Routing the top of the stack off the stack results in data returned to the previous page when the page was closed.

Navigator has many other methods, such as navigator.replace, navigator.popuntil, etc. Please refer to the API documentation or SDK source notes for more details. We also need to introduce another concept related to routing, named routes.

### 3. Example method

Every static method in the Navigator class that takes context as the first parameter corresponds to an instance Navigator method, For example, Navigator.push(BuildContext context, Route route) is equivalent to Navigator.of(context).push(Route route).

## 2.4.4 Route Value Transmission

In many cases, we need to bring some parameters when routing jump. For example, we need to bring a commodity id when opening the commodity details page, so that the commodity details page will know which commodity information to display. Another example is that we need to select the delivery address when filling in the order. After opening the address selection page and selecting the address, the address selected by the user can be returned to the order page and so on. Let's use a simple example to show how the old and new routes pass parameters.

Here is an example of creating a TipRoute route that takes a prompt text parameter and displays the text that is passed into it on the page. In addition to TipRoute, we added a "back" button that, when clicked, will return to the previous route with a return parameter. Let's look at the implementation code.

TipRoute implementation code:

```dart
class TipRoute extends StatelessWidget {
  TipRoute({
    Key key,
    required this.text,  // Accepts a text parameter
  }) : super(key: key);
  final String text;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Hint"),
      ),
      body: Padding(
        padding: EdgeInsets.all(18),
        child: Center(
          child: Column(
            children: <Widget>[
              Text(text),
              ElevatedButton(
                onPressed:(a)= > Navigator.pop(context, "I'm the return value."),
                child: Text("Return"),)],),),),); }}
```

Here is the code to open the new Routing TipRoute:

```dart
class RouterTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: ElevatedButton(
        onPressed:(a)async {
          // Open TipRoute and wait for the result to return
          var result = await Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) {
                return TipRoute(
                  // Route parameters
                  text: "This is prompt xxxx.",); },),);// Output 'TipRoute' route return result
          print("Route return value: $result");
        },
        child: Text("Open the prompt page"),),); }}
```

Run the above code and click "Open Tip" on the RouterTestRoute page to bring up TipRoute, as shown in Figure 2-11:

![图2-11](./assets/2-11.png)

Need to note:

1. The prompt copy "I am Prompt xxxx" is passed to the new routing page through the text parameter of TipRoute. We can do this by waiting for Navigator.push(...) Returns the Future to get the return data of the new route.

2. There are two ways to go back to the previous page in the TipRoute page; The first way is to click the back arrow directly in the navigation bar, and the second way is to click the "back" button in the page. The difference between these two returns is that the former does not return data to the previous route, while the latter does. Here's what the print method on the RouterTestRoute page outputs in the console after clicking the back button on the page and the navigation back arrow, respectively:

    ```bash
    I/flutter (27896): Route return value: I am the return value I/flutter (27896): Route return value: null
    ```

The above describes the value passing mode of the non-named route. The value passing mode of the named route is different, which will be explained in the following section.

## 2.4.5 Naming Routes

The so-called "Named Route" is a route with a name. We can give a name to the route first, and then we can directly open a new route by the route name. This brings an intuitive and simple way for route management.

### 1. Routing table

To use named routes, we must first provide and register a routing table so that the application knows which name corresponds to which routing component. In fact, registering a routing table is to name a route. The routing table is defined as follows:

```dart
Map<String, WidgetBuilder> routes;
```

It's a Map, key is the name of the route, it's a string; value is a builder callback function that generates the corresponding routing widget. When we open a new route through the route name, the application will find the corresponding WidgetBuilder callback function in the routing table according to the route name, and then call the callback function to generate the route widget and return.

### 2. Register the routing table

The routing table is simply registered. We go back to the previous example of a "counter", then find MaterialApp in the build method of the MyApp class and add the routes property as follows:

```dart
MaterialApp(
  title: 'Flutter Demo',
  theme: ThemeData(
    primarySwatch: Colors.blue,
  ),
  // Register the routing table
  routes:{
   "new_page":(context) = > NewRoute(),...// Other route registration information is omitted
  } ,
  home: MyHomePage(title: 'Flutter Demo Home Page'),);
```

We are now finished registering the routing table. The home route in the above code does not use named routes. What if we want to register home as a named route as well? It's really simple. Just look at the code:

```dart
MaterialApp(
  title: 'Flutter Demo',
  initialRoute:"/", // The route named "/" is the home page of the application.
  theme: ThemeData(
    primarySwatch: Colors.blue,
  ),
  // Register the routing table
  routes:{
   "new_page":(context) = > NewRoute(),
   "/":(context) = > MyHomePage(title: 'Flutter Demo Home Page'), // Register the home page route});
```

As you can see, all we have to do is register the MyHomePage route in the routing table and use its name as the value of the MaterialApp initialRoute attribute, which determines which named route the initial routing page of the application will be.

### 3. Open the new route page using the route name

To open a new route by route name, use Navigator's pushNamed method:

```dart
Future pushNamed(BuildContext context, String routeName,{Object arguments})
```

Navigator has other methods to manage named routes, such as pushReplacementNamed, in addition to the pushNamed method, and readers can check the API documentation for themselves. Next we open the new routing page with the route name and change the TextButton onPressed callback code to:

```dart
onPressed: () {
  Navigator.pushNamed(context, "new_page");
  //Navigator.push(context,
  // MaterialPageRoute(builder: (context) {
  // return NewRoute();
  / /}));
},
```

Hot reload application, click the "open new route" button again, still can open a new route page.

### 4. Pass named route parameters

In the original Flutter, named routes were not able to pass parameters, which was later supported; Here's how a named route passes and gets route parameters:

Let's register a route first:

```dart
 routes:{
   "new_page":(context) = > EchoRoute(),},
```

Obtain routing parameters through the RouteSetting object on the routing page:

```dart
class EchoRoute extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    // Obtain route parameters
    var args=ModalRoute.of(context).settings.arguments;
    / /... Omit irrelevant code}}
```

Parameters are passed when the route is opened

```dart
Navigator.of(context).pushNamed("new_page", arguments: "hi");
```

### Step 5 Fit

Suppose we also want to register the TipRoute routing page from the routing parameter passing example above into the routing table so that we can also open it by route name. However, since TipRoute accepts a text parameter, how can we accommodate this situation without changing TipRoute's source code? It's pretty simple:

```dart
MaterialApp(...)// Omit irrelevant code
  routes: {
   "tip2": (context){
     return TipRoute(text: ModalRoute.of(context)!.settings.arguments); },},);
```

## 2.4.6 Route Generation hook

Suppose we want to develop an e-commerce App. When the user is not logged in, he can see the store and commodity information, but the transaction record, shopping cart, user's personal information and other pages can only be seen after login. To do this, we need to determine the user login status before opening each routing page! If it's going to be a hassle to have to judge a route every time we open it, what better way? The answer is yes!

MaterialApp has an onGenerateRoute property which may be called when opening the named route, possibly because when calling Navigator.pushNamed(...) When opening the named route, if the specified route name has been registered in the routing table, the builder function in the routing table is called to generate the routing component. onGenerateRoute is called to generate a route only if it is not registered in the routing table. The onGenerateRoute callback signature is as follows:

```dart
Route<dynamic> Function(RouteSettings settings)
```

With the onGenerateRoute callback, it is very easy to implement the above function of controlling page permissions: instead of using the routing table, we provide an onGenerateRoute callback and then have uniform permission control within the callback, such as:

```dart
MaterialApp(...)// Omit irrelevant code
  onGenerateRoute:(RouteSettings settings){
	  return MaterialPageRoute(builder: (context){
		   String routeName = settings.name;
       // If the accessed routing page requires login but is not currently logged in, then directly return to the login page route.
       // Guide the user to log in; In other cases, the route is enabled normally.}); });
```

> Note that onGenerateRoute only works for named routes.

## 2.4.7 Summary

This chapter first introduces routing management and parameter passing in Flutter, and then focuses on named routing. It is important to note here that because named routes are only an optional route management method, readers may be hesitant about which route management method to use in real development. Here, according to the author's experience, it is suggested that readers better use the named route management mode, which will bring the following benefits:

1. More explicit semantics.
2. The code is better maintained; If anonymous routing is used, it must be invoked`Navigator.push`Not only would you need to import the dart file for the new routing page, but the code would be very fragmented.
3. Can pass`onGenerateRoute`Do some global route jump preprocessing logic.

To sum up, I recommend using named routing, of course, this is not a golden rule, readers can decide according to their own preferences or actual situation.

In addition, there are some elements of route management that we haven't covered, such as navigatorObservers, which listens for all route jumps, and onUnknownRoute, which is called when open a non-existent named route, in the MaterialApp. Since these features are not commonly used and are relatively simple, we won't cover them in detail, leaving the reader to consult the API documentation.