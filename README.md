# Flutter-ReduRx [![Build Status](https://travis-ci.org/leocavalcante/Flutter-ReduRx.svg?branch=master)](https://travis-ci.org/leocavalcante/Flutter-ReduRx)

Flutter bindings for [ReduRx](https://github.com/leocavalcante/ReduRx).

It provides a `Provider` as an `InheritedWidget` so you can reach the ReduRx Store wherever you have a `BuildContext`.<br>
It also has a `Connect` Widget so you can grab a few parts of the State (sub-state/props) and it will automatically recall `builder` whenever **only this little connected part of the State has changed** preventing unnecessary rebuilds.

## Getting started
* [Tutorial: Handling State in Flutter with ReduRx](https://medium.com/@leocavalcante/tutorial-handling-state-in-flutter-with-redurx-b4d50c647e4a)

## API

### Connect
* `convert: State -> Props` - A function used by `map` to transform the state in a sub-state/props.
* `where: Props oldProps, Props newProps -> bool` - A function to explicitly filter when the `builder` should be called.
* `builder: Props newProps -> Widget` - A function that receives the sub-state/props and returns a `Widget`.

```dart
Connect<State, String>(
  convert: (state) => state.title,
  where: (oldTitle, newTitle) => newTitle != oldTitle,
  builder: (title) => Text(title),
)
```

### Provider
* `store` - A ReduRx `Store`.
* `child` - Your application's `Widget`.
* `static dispatch<S>(BuildContext, ActionType)` - Helper to call `dispatch` on the Provider's context Store.

```dart
Provider(
  store: Store<State>(State('Initial title')),
  child: App(),
)
```

```dart
Provider.dispatch<State>(context, ChangeTitle('New title'));
```

For a better understanding of `dispatch` and `ActionType` head to [ReduRx](https://github.com/leocavalcante/ReduRx).

## Example
```dart
import 'package:flutter/material.dart' hide State;
import 'package:flutter_redurx/flutter_redurx.dart';

class State {
  State({this.title, this.count});
  final String title;
  final int count;
}

class Increment extends Action<State> {
  @override
  State reduce(State state) =>
      State(title: state.title, count: state.count + 1);
}

void main() {
  final store =
      Store<State>(State(title: 'Flutter-ReduRx Demo Title', count: 0));
  runApp(Provider(store: store, child: MyApp()));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter-ReduRx Demo',
      theme: ThemeData(
        primarySwatch: Colors.pink,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Connect<State, String>(
            convert: (state) => state.title,
            where: (prev, next) => next != prev,
            builder: (title) => Text(title ?? ''),
          ),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text('You have pushed the button this many times:'),
              Connect<State, String>(
                convert: (state) => state.count.toString(),
                where: (prev, next) => next != prev,
                builder: (count) => Text(count ?? '',
                    style: Theme.of(context).textTheme.display1),
              ),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => Provider.dispatch<State>(context, Increment()),
          tooltip: 'Increment',
          child: Icon(Icons.add),
        ));
  }
}
```
