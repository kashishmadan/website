---
title: An introduction to widget testing
short-title: Introduction
prev:
  title: Mock dependencies using Mockito
  path: /docs/cookbook/testing/unit/mocking
next:
  title: Find widgets
  path: /docs/cookbook/testing/widget/finders
---

{% assign api = site.api | append: '/flutter' -%}

In the [introduction to unit
testing](/docs/cookbook/testing/unit/introduction) recipe, you
learned how to test Dart classes using the `test` package. To test
widget classes, you need a few additional tools provided by the
[`flutter_test`]({{api}}/flutter_test/flutter_test-library.html)
package, which ships with the Flutter SDK.

The `flutter_test` package provides the following tools for testing widgets:

  * The [`WidgetTester`]({{api}}/flutter_test/WidgetTester-class.html),
    which allows building and interacting with widgets in a test
    environment.
  * The [`testWidgets()`]({{api}}/flutter_test/testWidgets.html)
    function, which automatically creates a new `WidgetTester` for
    each test case, and is used in place of the normal `test()` function.
  * The [`Finder`]({{api}}/flutter_test/Finder-class.html)
    classes, which allow searching for widgets in the test environment.
  * Widget-specific
    [`Matcher`]({{api}}/package-matcher_matcher/Matcher-class.html)
    constants, which help verify whether a `Finder` locates a widget or
    multiple widgets in the test environment.

If this sounds overwhelming, don't worry. Learn how all of these pieces fit
together throughout this recipe, which uses the following steps:

  1. Add the `flutter_test` dependency.
  2. Create a widget to test.
  3. Create a `testWidgets` test.
  4. Build the widget using the `WidgetTester`.
  5. Search for the widget using a `Finder`.
  6. Verify that the widget uses a `Matcher`.

### 1. Add the `flutter_test` dependency

Before writing tests, include the `flutter_test`
dependency in the `dev_dependencies` section of the `pubspec.yaml` file.
If creating a new Flutter project with the command line tools or
a code editor, this dependency should already be in place.

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
```

### 2. Create a widget to test

Next, create a widget for testing. For this recipe,
create a widget that displays a `title` and `message`.

```dart
class MyWidget extends StatelessWidget {
  final String title;
  final String message;

  const MyWidget({
    Key key,
    @required this.title,
    @required this.message,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: Center(
          child: Text(message),
        ),
      ),
    );
  }
}
```

### 3. Create a `testWidgets` test

With a widget to test, begin by writing your first test.
Use the
[`testWidgets()`]({{api}}/flutter_test/testWidgets.html)
function provided by the `flutter_test` package to define a test.
The `testWidgets` function allows you to define a widget test and creates a
`WidgetTester` to work with.

This test verifies that `MyWidget` displays a given title and message.

<!-- skip -->
```dart
void main() {
  // Define a test. The TestWidgets function also provides a WidgetTester
  // to work with. The WidgetTester allows you to build and interact
  // with widgets in the test environment.
  testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
    // Test code goes here.
  });
}
```

### 4. Build the widget using the `WidgetTester`

Next, build `MyWidget` inside the test environment by using the
[`pumpWidget()`]({{api}}/flutter_test/WidgetTester/pumpWidget.html)
method provided by `WidgetTester`. The `pumpWidget` method builds and
renders the provided widget.

Create a `MyWidget` instance that displays "T" as the title
and "M" as the message.

<!-- skip -->
```dart
void main() {
  testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
    // Create the widget by telling the tester to build it.
    await tester.pumpWidget(MyWidget(title: 'T', message: 'M'));
  });
}
```

#### Note

After the initial call to `pumpWidget()`, the `WidgetTester` provides
additional ways to rebuild the same widget. This is useful if you're
working with a `StatefulWidget` or animations.

For example, tapping a button calls `setState()`, but Flutter won't
automatically rebuild your widget in the test environment.
Use one of the following methods to ask Flutter to rebuild the widget.

  - [tester.pump()]({{api}}/flutter_test/TestWidgetsFlutterBinding/pump.html)
  : Triggers a rebuild of the widget after a given duration.
  - [tester.pumpAndSettle()]({{api}}/flutter_test/WidgetTester/pumpAndSettle.html)
  : Repeatedly calls pump with the given duration until there are no longer any frames scheduled. This essentially waits for all animations to complete.

These methods provide fine-grained control over the build lifecycle,
which is particularly useful while testing.

### 5. Search for our widget using a `Finder`

With a widget in the test environment, search
through the widget tree for the `title` and `message`
Text widgets using a `Finder`. This allows verification that
the widgets are being displayed correctly.

For this purpose, use the top-level
[`find()`]({{api}}/flutter_test/find-constant.html)
method provided by the `flutter_test` package to create the `Finders`.
Since you know you're looking for `Text` widgets, use the
[`find.text()`]({{api}}/flutter_test/CommonFinders-class.html)
method.

For more information about `Finder` classes, see the
[Finding widgets in a widget test](/docs/cookbook/testing/widget/finders)
recipe.

<!-- skip -->
```dart
void main() {
  testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
    await tester.pumpWidget(MyWidget(title: 'T', message: 'M'));

    // Create the Finders.
    final titleFinder = find.text('T');
    final messageFinder = find.text('M');
  });
}
```

### 6. Verify that the widget uses a `Matcher`

Finally, verify the title and message `Text` widgets appear on screen
using the `Matcher` constants provided by `flutter_test`.
`Matcher` classes are a core part of the `test` package,
and provide a common way to verify a given
value meets expectations.

Ensure that the widgets appear on screen exactly one time.
For this purpose, use the
[`findsOneWidget`]({{api}}/flutter_test/findsOneWidget-constant.html)
`Matcher`.

<!-- skip -->
```dart
void main() {
  testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
    await tester.pumpWidget(MyWidget(title: 'T', message: 'M'));
    final titleFinder = find.text('T');
    final messageFinder = find.text('M');

    // Use the `findsOneWidget` matcher provided by flutter_test to verify
    // that the Text widgets appear exactly once in the widget tree.
    expect(titleFinder, findsOneWidget);
    expect(messageFinder, findsOneWidget);
  });
}
```

#### Additional Matchers

In addition to `findsOneWidget`, `flutter_test` provides additional
matchers for common cases.

  * [findsNothing]({{api}}/flutter_test/findsNothing-constant.html)
  : verifies that no widgets are found
  * [findsWidgets]({{api}}/flutter_test/findsWidgets-constant.html)
  : verifies that one or more widgets are found
  * [findsNWidgets]({{api}}/flutter_test/findsNWidgets.html)
  : verifies that a specific number of widgets are found

### Complete example

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  // Define a test. The TestWidgets function also provides a WidgetTester
  // to work with. The WidgetTester allows building and interacting
  // with widgets in the test environment.
  testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
    // Create the widget by telling the tester to build it.
    await tester.pumpWidget(MyWidget(title: 'T', message: 'M'));

    // Create the Finders.
    final titleFinder = find.text('T');
    final messageFinder = find.text('M');

    // Use the `findsOneWidget` matcher provided by flutter_test to
    // verify that the Text widgets appear exactly once in the widget tree.
    expect(titleFinder, findsOneWidget);
    expect(messageFinder, findsOneWidget);
  });
}

class MyWidget extends StatelessWidget {
  final String title;
  final String message;

  const MyWidget({
    Key key,
    @required this.title,
    @required this.message,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: Center(
          child: Text(message),
        ),
      ),
    );
  }
}
```
