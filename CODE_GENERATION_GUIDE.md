# Guide to Web Scraping and Code Generation for Dart

This document provides a guide for AI agents on how to reuse the web scraping and code generation techniques found in this repository. The `tool/generate_lib.dart` script is a powerful example of how to create a Dart wrapper for a JavaScript library by scraping its documentation.

## Overview

The code generation process in this repository involves the following steps:

1.  **Fetching HTML:** The script downloads the HTML content of the Google Maps JavaScript API documentation.
2.  **Parsing HTML:** It then parses the HTML to extract information about the API, such as classes, methods, properties, and events.
3.  **Parsing Types:** The script uses a custom parser to understand the JavaScript type definitions and translate them into Dart types.
4.  **Generating Dart Code:** Finally, it generates the Dart source code for the library based on the extracted information.

## Technologies Used

The following Dart packages are essential for this process:

*   **`package:http`**: This package is used to make HTTP requests and fetch the HTML content from the web.
*   **`package:html`**: This is a robust HTML parser that allows you to traverse and manipulate the HTML document tree using CSS selectors. This is crucial for finding the specific pieces of information you need in the documentation.
*   **`package:petitparser`**: This package provides a framework for building custom parsers. In this project, it's used to parse the JavaScript type signatures (e.g., `Array<string>`, `function(number): string`) and convert them into a structured representation that can be used to generate Dart types.
*   **`dart:io`**: Used for file system operations like creating and writing the generated Dart files.

## Step-by-Step Process

Here's a breakdown of the process implemented in `tool/generate_lib.dart`:

1.  **Identify the Source:** The script starts by defining the base URL of the documentation to be scraped.
2.  **Fetch the Main Page:** It fetches the main reference page to get a list of all the available libraries.
3.  **Iterate Through Libraries:** For each library, it fetches the corresponding documentation page.
4.  **Extract API Elements:** On each library page, it uses `querySelectorAll` with specific CSS selectors to find the HTML elements that define the classes, methods, properties, and events of the API.
5.  **Parse Type Signatures:** For each extracted element, it uses the `petitparser`-based parser defined in `tool/types.dart` to analyze the JavaScript type signatures.
6.  **Generate Code:** The script then constructs the Dart code as strings and writes them to the appropriate files in the `lib/src/generated` directory. It creates extension types, methods, and properties that mirror the JavaScript API.
7.  **Format the Code:** After generating the code, it runs `dart format` to ensure the generated code is well-formatted.

## How to Reuse this Technique

AI agents can adapt this web scraping and code generation technique for other purposes. Here’s a general guide on how to do it:

1.  **Analyze the Target Website:** Before you start, thoroughly inspect the HTML structure of the documentation website you want to scrape. Use your browser's developer tools to identify the HTML tags and CSS selectors that uniquely identify the information you need (e.g., class names, method signatures, descriptions).
2.  **Adapt the Scraper:**
    *   Change the base URL in the script to point to the new documentation website.
    *   Update the CSS selectors in the `querySelectorAll` calls to match the structure of the new website. You will need to be very specific to get the right information.
3.  **Customize the Type Parser:** If the target website uses a different format for its type signatures, you will need to modify the grammar in `tool/types.dart`. The `petitparser` package is very flexible and allows you to define complex grammars.
4.  **Modify the Code Generation Logic:** The code generation part of the script will need to be adapted to produce the desired output format. For example, if you are generating code for a different language, you will need to change the syntax of the generated code.
5.  **Handle Custom Cases:** As seen in the `generateExtensionType` function in `tool/generate_lib.dart`, you might encounter special cases or inconsistencies in the documentation. Be prepared to add custom logic to handle these situations.

By following these steps, you can create a powerful tool to automate the creation of libraries or other resources from web documentation.

## Effective JavaScript Interoperability with `dart:js_interop`

The code generation script in this repository heavily relies on `dart:js_interop` to create a bridge between Dart and JavaScript. This section provides a more detailed guide on how to use `dart:js_interop` effectively, based on the analysis of `tool/generate_lib.dart` and best practices.

### Why `dart:js_interop`?

The `dart:js_interop` library is the modern way to interact with JavaScript from Dart. It supersedes the older `dart:js` and `package:js` libraries, offering several advantages:

*   **Type Safety:** `dart:js_interop` uses `extension type` to create static wrappers around JavaScript objects. This allows the Dart compiler to catch type errors at compile time, leading to more robust code.
*   **Performance:** `extension type` is a zero-cost abstraction. It doesn't create new objects at runtime, so there is no performance overhead compared to calling JavaScript directly.
*   **Code Conciseness:** The new interop mechanism is more declarative and requires less boilerplate code.

### Using `@JS()` and `external`

The `@JS()` annotation and the `external` keyword are the core components of `dart:js_interop`.

*   **`@JS()`**: This annotation tells the Dart compiler that a class, function, or property is defined in JavaScript. You can also provide a name to the annotation (e.g., `@JS('my_js_function')`) if the Dart name is different from the JavaScript name.
*   **`external`**: This keyword marks a function or property as having its implementation provided elsewhere (in this case, in JavaScript).

Here's a simple example of how to use them together to call a JavaScript function:

```dart
// In your Dart code
import 'dart:js_interop';

@JS('alert')
external void alert(String message);

void main() {
  alert('Hello from Dart!');
}
```

### The Power of `extension type`

`extension type` is the key feature that makes `dart:js_interop` so powerful. It allows you to define a new Dart type that "wraps" an existing JavaScript type, giving it a new, type-safe interface.

In `tool/generate_lib.dart`, you can see that an `extension type` is created for each JavaScript class in the Google Maps API. This provides a Dart-friendly way to interact with the JavaScript objects.

Here is a simplified example of how to create an `extension type` for a JavaScript object:

```dart
// In your JavaScript file (e.g., web/my_lib.js)
class MyJsClass {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, ${this.name}!`;
  }
}
```

```dart
// In your Dart code
import 'dart:js_interop';

@JS('MyJsClass')
extension type MyJsClass._(JSObject _) implements JSObject {
  external MyJsClass(String name);

  external String get name;
  external String greet();
}

void main() {
  final myJsObject = MyJsClass('World');
  print(myJsObject.greet()); // Prints "Hello, World!"
}
```

### Handling Asynchronous Operations with `JSPromise`

JavaScript's asynchronous operations are based on `Promise`s. `dart:js_interop` provides the `JSPromise` extension type to work with them.

To convert a `JSPromise` to a Dart `Future`, you can use the `toDart` getter, which is an extension on `JSPromise`.

Here's how you can call an asynchronous JavaScript function that returns a `Promise`:

```javascript
// In your JavaScript file
async function myAsyncFunction() {
  return 'Hello from a Promise!';
}
```

```dart
// In your Dart code
import 'dart:js_interop';

@JS()
external JSPromise myAsyncFunction();

void main() async {
  final result = await myAsyncFunction().toDart;
  print(result); // Prints "Hello from a Promise!"
}
```

This approach is much cleaner and more type-safe than the old `promiseToFuture` function from `dart:js_util`.

By leveraging `dart:js_interop` and its features like `extension type` and `JSPromise`, you can create robust, performant, and maintainable Dart wrappers for any JavaScript library. The `tool/generate_lib.dart` script in this repository serves as an excellent, real-world example of these concepts in action.

### More Examples: From HTML to Dart

To make the process more concrete, let's walk through more examples of how the `generate_lib.dart` script converts the HTML documentation into Dart code.

#### Example 1: A Method

Let's start with a simple method: `getLabel()` from the `google.maps.Marker` class.

**1. The HTML Source**

The script first fetches the HTML for the [Marker class documentation](https://developers.google.com/maps/documentation/javascript/reference/marker#Marker). A simplified snippet of the relevant HTML for the `getLabel()` method looks like this:

```html
<table class="methods" summary="Show a summary of methods from the class Marker">
  <tbody>
    <tr>
      <td class="jd-link">
        <code><a href="...">getLabel</a>()</code>
      </td>
      <td class="jd-returns--descriptions">
        <div class="jd-returns">
          <code><a href="...">MarkerLabel</a>|string</code>
        </div>
        <div class="jd-description">
          <p>Returns the label of the marker.</p>
        </div>
      </td>
    </tr>
  </tbody>
</table>
```

**2. Parsing the HTML**

The script uses the `package:html` to parse this HTML. The core logic is in the `extractMethods` function. It finds all the `<table>` elements with the class `methods` and then iterates through the rows (`<tr>`) to extract the information for each method.

Here's how it extracts the data for `getLabel()`:

*   **Method Name:** It gets the text from the first `<code>` tag inside the first `<td>`, which is "getLabel".
*   **Return Type:** It finds the `<code>` tag inside the `div` with the class `jd-returns`. The text content is `MarkerLabel|string`.
*   **Type Parsing:** The `translateType` function, which uses the `petitparser`-based grammar from `tool/types.dart`, is then used to parse `MarkerLabel|string` into a Dart-compatible type. The parser recognizes the `|` as a union type and converts it accordingly.

**3. Generating the Dart Code**

Finally, the `generateExtensionType` function takes the extracted information and generates the Dart code. For the `getLabel()` method, it generates the following code within the `Marker` extension type:

```dart
@JS('getLabel')
external MarkerLabelOrString? _getLabel();

MarkerLabelOrString? get label => _getLabel();
```

As you can see, the script intelligently creates a getter named `label` from the `getLabel` method name. It also creates a type `MarkerLabelOrString` to represent the union type `MarkerLabel|string`.

#### Example 2: Constructor with Optional Parameters

Now let's look at the constructor for the `google.maps.LatLng` class, which has optional parameters.

**1. The HTML Source**

Here is a simplified snippet of the HTML for the `LatLng` constructor:

```html
<table class="constructors" summary="A summary of constructors for LatLng">
  <tbody>
    <tr>
      <td>
        <code>new LatLng(lat, lng[, noClampNoWrap])</code>
      </td>
    </tr>
  </tbody>
</table>
```

**2. Parsing the HTML**

The `extractConstructor` function is responsible for parsing this. It finds the `<table>` with the class `constructors` and extracts the signature from the `<code>` tag. The square brackets `[]` indicate an optional parameter.

**3. Generating the Dart Code**

The script generates the following `external` constructor in the `LatLng` extension type:

```dart
external LatLng(num lat, num lng, [bool? noClampNoWrap]);
```

The `[bool? noClampNoWrap]` part of the generated code correctly represents the optional parameter in Dart.

#### Example 3: An Event

Finally, let's examine how the script handles events, using the `click` event on the `Map` class as an example.

**1. The HTML Source**

The HTML for the `click` event looks like this:

```html
<table class="details" summary="Show a summary of events from the class Map">
  <tbody>
    <tr>
      <td class="jd-link" itemprop="property">
        <code><a href="...">click</a></code>
      </td>
      <td class="jd-descr" itemprop="description">
        <div class="jd-descr-Ì´" >
          <p>This event is fired when the user clicks on the map.</p>
          <p><strong>Arguments:</strong></p>
          <ul class="jd- μπορούν">
            <li>
              <code>event</code>:
              <code><a href="...">MapMouseEvent</a></code>
              <p>The event object.</p>
            </li>
          </ul>
        </div>
      </td>
    </tr>
  </tbody>
</table>
```

**2. Parsing the HTML**

The `extractEvents` function parses this structure. It looks for a `<table>` with the class `details` and then extracts the event name from the `<code>` tag. It also extracts the event arguments from the `<ul>` element.

**3. Generating the Dart Code**

The script then generates a stream getter for the event. For the `click` event, it generates the following code:

```dart
Stream<MapMouseEvent> get onClick {
  late StreamController<MapMouseEvent> sc;
  late MapsEventListener mapsEventListener;
  void start() => mapsEventListener = event.addListener(
    this,
    'click',
    ((MapMouseEvent e) => sc.add(e)).toJS,
  );
  void stop() => mapsEventListener.remove();
  sc = StreamController<MapMouseEvent>(
    onListen: start,
    onCancel: stop,
    onResume: start,
    onPause: stop,
  );
  return sc.stream;
}
```

This code creates a `Stream` that fires a `MapMouseEvent` whenever the JavaScript `click` event is triggered on the map. This is a very idiomatic way to handle events in Dart.

These examples demonstrate the power of combining web scraping with code generation. By carefully analyzing the structure of the source documentation, you can automate the creation of a complete and type-safe Dart library for any JavaScript API.
