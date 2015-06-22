GRMustache.swift
================

GRMustache.swift is a [Mustache](http://mustache.github.io) template engine written in Swift 2, from the author of the Objective-C [GRMustache](https://github.com/groue/GRMustache).

It ships with built-in goodies and extensibility hooks that let you avoid the strict minimalism of the genuine Mustache language when you need it.

**June 9, 2015: GRMustache.swift 0.9.3 is out.** [Release notes](RELEASE_NOTES.md)

Get release announcements and usage tips: follow [@GRMustache on Twitter](http://twitter.com/GRMustache).

Jump to:

- [Usage](#usage)
- [Installation](#installation)
- [Documentation](#documentation)

Features
--------

- Support for the full [Mustache syntax](http://mustache.github.io/mustache.5.html)
- Filters, as `{{ uppercase(name) }}`
- Template inheritance, as in [hogan.js](http://twitter.github.com/hogan.js/), [mustache.java](https://github.com/spullara/mustache.java) and [mustache.php](https://github.com/bobthecow/mustache.php).
- Built-in [goodies](Docs/Guides/goodies.md)
- Unlike many Swift template engines, GRMustache.swift does not rely on the Objective-C runtime. It lets you feed your templates with ad-hoc values or your existing models, without forcing you to refactor your Swift code into Objective-C objects.


Requirements
------------

- iOS 7.0+ / OSX 10.9+
- Xcode 7


Usage
-----

`document.mustache`:

```mustache
Hello {{name}}
Your beard trimmer will arrive on {{format(date)}}.
{{#late}}
Well, on {{format(real_date)}} because of a Martian attack.
{{/late}}
```

```swift
import Mustache

// Load the `document.mustache` resource of the main bundle
let template = try! Template(named: "document")

// Let template format dates with `{{format(...)}}`
let dateFormatter = NSDateFormatter()
dateFormatter.dateStyle = .MediumStyle
template.registerInBaseContext("format", Box(dateFormatter))

// The rendered data
let data = [
    "name": "Arthur",
    "date": NSDate(),
    "real_date": NSDate().dateByAddingTimeInterval(60*60*24*3),
    "late": true
]

// The rendering: "Hello Arthur..."
let rendering = try! template.render(Box(data))
```


Installation
------------

### iOS7

To use GRMustache.swift in a project targetting iOS7, you must include the source files directly in your project.


### CocoaPods

[CocoaPods](http://cocoapods.org/) is a dependency manager for Xcode projects.

To use GRMustache.swift with Cocoapods, specify in your Podfile:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
use_frameworks!

pod 'GRMustache.swift', '0.9.3'
```


### Carthage

[Carthage](https://github.com/Carthage/Carthage) is another dependency manager for Xcode projects.

To use GRMustache.swift with Carthage, specify in your Cartfile:

```
github "groue/GRMustache.swift" == 0.9.3
```


### Manually

Download a copy of GRMustache.swift, embed the `Mustache.xcodeproj` project in your own project, and add the `MustacheOSX` or `MustacheiOS` target as a dependency of your own target.


Documentation
=============

To fiddle with the library, open the `Mustache.xcworkspace` workspace: it contains a Mustache-enabled Playground at the top of the files list.

External links:

- [The Mustache Language](http://mustache.github.io/mustache.5.html): the Mustache language itself. You should start here.
- [GRMustache.swift Reference](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Classes/Template.html) on cocoadocs.org

Rendering templates:

- [Loading Templates](#loading-templates)
- [Errors](#errors)
- [Mustache Tags Reference](#mustache-tags-reference)
    - [Variable Tags](#variable-tags) `{{value}}`
    - [Section Tags](#section-tags) `{{#value}}...{{/value}}`
    - [Inverted Section Tags](#inverted-section-tags) `{{^value}}...{{/value}}`
    - [Partial Tags](#partial-tags) `{{>partial}}`
    - [Partial Override Tags](#partial-override-tags) `{{<layout}}...{{/layout}}` (Template Inheritance)
    - [Set Delimiters Tags](#set-delimiters-tags) `{{=<% %>=}}`
    - [Comment Tags](#comment-tags) `{{! Wow. Such comment. }}`
    - [Pragma Tags](#pragma-tags) `{{% CONTENT_TYPE:TEXT }}`
- [The Context Stack and Expressions](#the-context-stack-and-expressions)

Feeding templates:

- [Templates Eat Boxed Values](#templates-eat-boxed-values)
- [Standard Swift Types Reference](#standard-swift-types-reference)
    - [Bool](#bool)
    - [Numeric Types](#numeric-types): Int, UInt and Double
    - [String](#string)
    - [Set](#set) (and similar collections)
    - [Array](#array) (and similar collections)
    - [Dictionary](#dictionary)
    - [NSObject](#nsobject)
    - [AnyObject and Any](#anyobject-and-any)
- [Custom Types](#custom-types)
- [Lambdas](#lambdas)
- [Filters](#filters)
- [Advanced Boxes](#advanced-boxes)

Misc:

- [Built-in goodies](#built-in-goodies)


Loading Templates
-----------------

Templates may come from various sources:

- Raw Swift strings:

    ```swift
    let template = try! Template(string: "Hello {{name}}")
    ```

- Bundle resources:

    ```swift
    // Loads the "document.mustache" resource of the main bundle:
    let template = try! Template(named: "document")
    ```

- Files and URLs:

    ```swift
    let template = try! Template(path: "/path/to/document.mustache")
    let template = try! Template(URL: templateURL)
    ```

- A *repository of templates*:
    
    These repositories represent a group of templates. They can be configured independently, and provide a few neat features like template caching. Check [TemplateRepository.swift](Mustache/Template/TemplateRepository.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Classes/TemplateRepository.html)). For example:
    
    ```swift
    // The repository of Bash templates, with extension ".sh":
    let repo = TemplateRepository(bundle: NSBundle.mainBundle(), templateExtension: "sh")
    
    // Disable HTML escaping for Bash scripts:
    repo.configuration.contentType = .Text
    
    // Load a template:
    let template = try! repo.template(named: "script")
    ```

For more information, check [Template.swift](Mustache/Template/Template.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Classes/Template.html)).


Errors
------

Not funny, but they happen. Standard NSErrors of domain NSCocoaErrorDomain, etc. may be thrown whenever the library needs to access the file system or other system resource. Mustache-specific errors are NSErrors of domain `GRMustacheErrorDomain`:

- Code `GRMustacheErrorCodeTemplateNotFound`:
    
    ```swift
    do {
        let template = try Template(named: "inexistant")
    } catch {
        // No such template: `inexistant`
    }
    ```
    
- Code `GRMustacheErrorCodeParseError`:
    
    ```swift
    do {
        let template = try Template(string: "Hello {{name")
    } catch {
        // Parse error at line 1: Unclosed Mustache tag
    }
    ```
    
- Code `GRMustacheErrorCodeRenderingError`:
    
    ```swift
    do {
        let template = try Template(string: "{{undefinedFilter(x)}}")
        let rendering = try template.render()
    } catch {
        // Error evaluating {{undefinedFilter(x)}} at line 1: Missing filter
    }
    ```

When you render trusted valid templates with trusted valid data, you can avoid error handling with the `try!` Swift construct:

```swift
// Assume valid parsing and rendering
let template = try! Template(string: "{{name}} has a Mustache.")
let rendering = try! template.render(Box(data))
```


Mustache Tags Reference
-----------------------

### Variable Tags

A *Variable tag* `{{value}}` renders the value associated with the key `value`, HTML-escaped. To avoid HTML-escaping, use triple mustache tags `{{{value}}}`:

```swift
let template = try! Template(string: "{{value}} - {{{value}}}")

// Mario &amp; Luigi - Mario & Luigi
let data = ["value": "Mario & Luigi"]
let rendering = try! template.render(Box(data))
```

### Section Tags

A *Section tag* `{{#value}}...{{/value}}` is a common syntax for three different usages:

- conditionally render a section.
- loop over a collection.
- dig inside an object.

Those behaviors are triggered by the value associated with `value`:


#### Falsey values

If the value is *falsey*, the section is not rendered. The only falsey values are:

- missing values
- false boolean
- zero numbers
- empty strings
- empty collections
- NSNull

For example:

```swift
let template = try! Template(string: "<{{#value}}Truthy{{/value}}>")

// "<Truthy>"
try! template.render(Box(["value": true]))
// "<>"
try! template.render(Box([:]))                  // missing value
try! template.render(Box(["value": false]))     // false boolean
```


#### Collections

If the value is a *collection*, the section is rendered as many times as there are elements in the collection. Each element on its turn in pushed on the top of the [context stack](#the-context-stack). This makes their keys available for tags inside the section.

Template:

```mustache
{{# friends }}
- {{# name }}
{{/ friends }}
```

Data:

```swift
[
  "friends": [
    [ "name": "Hulk Hogan" ],
    [ "name": "Albert Einstein" ],
    [ "name": "Tom Selleck" ],
  ]
]
```

Rendering:

```
- Hulk Hogan
- Albert Einstein
- Tom Selleck
```

Collections can be Swift arrays, ranges, sets, NSArray, NSSet, etc.


#### Objects

If the value is not falsey, and not a collection, the section is rendered once, and the value is pushed on the top of the [context stack](#the-context-stack). This makes its keys available for tags inside the section.

Template:

```mustache
{{# user }}
- {{ name }}
- {{ score }}
{{/ user }}
```

Data:

```swift
[
  "user": [
    "name": "Mario"
    "score": 1500
  ]
]
```

Rendering:

```
- Mario
- 1500
```


### Inverted Section Tags

An *Inverted section tag* `{{^value}}...{{/value}}` renders when a regular section `{{#value}}...{{/value}}` would not. You can think of it as the Mustache "else" or "unless".

Template:

```
{{# persons }}
- {{name}} is {{#alive}}alive{{/alive}}{{^alive}}dead{{/alive}}.
{{/ persons }}
{{^ persons }}
Nobody
{{/ persons }}
```

Data:

```swift
[
  "persons": []
]
```

Rendering:

```
Nobody
```

Data:

```swift
[
  "persons": [
    ["name": "Errol Flynn", "alive": false],
    ["name": "Sacha Baron Cohen", "alive": true]
  ]
]
```

Rendering:

```
- Errol Flynn is dead.
- Sacha Baron Cohen is alive.
```


### Partial Tags

A *Partial tag* includes another template inside a template. The included template is passed the current [context stack](#the-context-stack):

`document.mustache`:

```mustache
Guests:
{{# guests }}
  {{> person }}
{{/ guests }}
```

`person.mustache`:

```mustache
{{ name }}
```

Data:

```swift
[
  "guests": [
    ["name": "Frank Zappa"],
    ["name": "Lionel Richie"]
  ]
]
```

Rendering:

```
Guests:
- Frank Zappa
- Lionel Richie
```

Recursive partials are supported, but your data should avoid infinite loops.

Partial lookup depends on the origin of the main template:


#### File system

Partial names are relative paths when the template comes from the file system (via paths or URLs):

```swift
// Load /path/document.mustache
let template = Template(path: "/path/document.mustache")

// {{> partial }} includes /path/partial.mustache.
// {{> shared/partial }} includes /path/shared/partial.mustache.
```

Partials have the same file extension as the main template.

```swift
// Loads /path/document.html
let template = Template(path: "/path/document.html")

// {{> partial }} includes /path/partial.html.
```

When your templates are stored in a hierarchy of directories, you can use absolute paths to partials:

```swift
// The template repository defines the root of absolute partial paths:
let repository = TemplateRepository(directoryPath: "/path")

// Load /path/documents/profile.mustache
let template = repository.template(named: "documents/profile")

// {{> /shared/partial }} includes /path/shared/partial.mustache.
```


#### Bundle resources
    
Partial names are interpreted as resource names when the template is a bundle resource:

```swift
// Load the document.mustache resource from the main bundle
let template = Template(named: "document")

// {{> partial }} includes the partial.mustache resource.
```

Partials have the same file extension as the main template.

```swift
// Load the document.html resource from the main bundle
let template = Template(named: "document", templateExtension: "html")

// {{> partial }} includes the partial.html resource.
```


#### General case

Generally speaking, partial names are always interpreted by a *Template Repository*.

- `Template(named:...)` uses a bundle-based template repository: partial names are resource names.
- `Template(path:...)` uses a file-based template repository: partial names are relative paths.
- `Template(URL:...)` uses a URL-based template repository: partial names are relative URLs.
- `Template(string:...)` uses a template repository that can’t load any partial.
- `templateRepository.template(named:...)` uses the partial loading mechanism of the template repository.

Check [TemplateRepository.swift](Mustache/Template/TemplateRepository.swift) for more information ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Classes/TemplateRepository.html)).


#### Dynamic partials

A tag `{{> partial }}` includes a template, the one that is named "partial". One can say it is *statically* determined, since that partial has already been loaded before the template is rendered:

```swift
let repo = TemplateRepository(bundle: NSBundle.mainBundle())
let template = try! repo.template(string: "{{#user}}{{>partial}}{{/user}}")

// Now the `partial.mustache` resource has been loaded. It will be used when
// the template is rendered. Nothing can change that.
```

You can also include *dynamic partials*. To do so, use a regular variable tag `{{ partial }}`, and provide a template for the key "partial" in your rendered data:

```swift
// A template that delegates the rendering of a user to a partial.
// No partial has been loaded yet.
let template = try! Template(string: "{{#user}}{{partial}}{{/user}}")

// The user
let user = ["firstName": "Georges", "lastName": "Brassens", "occupation": "Singer"]

// Render with a first partial -> "Georges Brassens"
let partial1 = try! Template(string: "{{firstName}} {{lastName}}")
let data1 = ["user": Box(user), "partial": Box(partial1) ]
try! template.render(Box(data1))

// Render with a second partial -> "Singer"
let partial2 = try! Template(string: "{{occupation}}")
let data2 = ["user": Box(user), "partial": Box(partial2) ]
try! template.render(Box(data2))
```


### Partial Override Tags

GRMustache.swift supports *Template Inheritance*, like [hogan.js](http://twitter.github.com/hogan.js/), [mustache.java](https://github.com/spullara/mustache.java) and [mustache.php](https://github.com/bobthecow/mustache.php).

A *Partial Override Tag* `{{< layout }}...{{/ layout }}` includes another template inside the rendered template, just like a regular [partial tag](#partial-tags) `{{> partial}}`.

However, this time, the included template can contain *blocks*, and the rendered template can override them.

The included template `layout.mustache` below has `title` and `content` blocks that the rendered template can override:

```mustache
<html>
<head>
    <title>{{$ title }}Default title{{/ title }}</title>
</head>
<body>
    <h1>{{$ title }}Default title{{/ title }}</h1>
    {{$ content }}
        Default content
    {{/ content }}}
</body>
</html>
```

The rendered template `article.mustache`:

```mustache
{{< layout }}

    {{$ title }}{{ article.title }}{{/ title }}
    
    {{$ content }}
        {{{ article.html_body }}}
        <p>by {{ article.author }}</p>
    {{/ content }}
    
{{/ layout }}
```

```swift
let template = try! Template(named: "article")
let data = [
    "article": [
        "title": "The 10 most amazing handlebars",
        "html_body": "<p>...</p>",
        "author": "John Doe"
    ]
]
let rendering = try! template.render(Box(data))
```

The rendering is a full HTML page:

```HTML
<html>
<head>
    <title>The 10 most amazing handlebars</title>
</head>
<body>
    <h1>The 10 most amazing handlebars</h1>
    <p>...</p>
    <p>by John Doe</p>
</body>
</html>
```

A block `{{$ title }}...{{/ title }}` is always rendered, and rendered once. There is no boolean checks, no collection iteration. It is a name that allows other templates to override the block, not a key in your rendered data.

A template can override a partial which itself overrides another one. Recursion is possible, but your data should avoid infinite loops.

A template can contain several partial override tags.

Generally speaking, any part of a template can be refactored with partials and partial override tags. And this does not require modifications in other templates that depend on it.


#### Dynamic partial overrides

Like a regular partial tag, a partial override tag `{{< layout }}...{{/ layout }}` includes a statically determined template, the very one that is named "layout".

To override a *dynamic* partial, use a regular section tag `{{# layout }}...{{/ layout }}`, and provide a template for the key "layout" in your rendered data.


### Set Delimiters Tags

Mustache tags are generally enclosed by "mustaches" `{{` and `}}`. A *Set Delimiters Tag* can change that, right inside a template.

```
Default tags: {{ name }}
{{=<% %>=}}
ERB-styled tags: <% name %>
<%={{ }}=%>
Default tags again: {{ name }}
```

There are also APIs for setting those delimiters. Check `Configuration.tagDelimiterPair` in [Configuration.swift](Mustache/Configuration/Configuration.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Structs/Configuration.html)).


### Comment Tags

`{{! Comment tags }}` are simply not rendered at all.


### Pragma Tags

Several Mustache implementations use *Pragma tags*. They start with a percent `%` and are not rendered at all. Instead, they trigger implementation-specific features.

GRMustache.swift interprets two pragma tags that set the *content type* of the template:

- `{{% CONTENT_TYPE:TEXT }}`
- `{{% CONTENT_TYPE:HTML }}`

In a text template, there is no HTML-escaping, and both `{{name}}` and `{{{name}}}` have the same rendering. Text templates are globally HTML-escaped when included in HTML templates.

For a more complete discussion, see the documentation of  `Configuration.contentType` in [Configuration.swift](Mustache/Configuration/Configuration.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Structs/Configuration.html)).


The Context Stack and Expressions
---------------------------------

### The Context Stack

Variable and section tags fetch values in the data you feed your templates with: `{{name}}` looks for the key "name" in your input data, or, more precisely, in the *context stack*.

That context stack grows as the rendering engine enters sections, and shrinks when it leaves. The tag `{{name}}` looks for the "name" identifier in the value pushed by the last entered section. If this top value does not provide the key, the evaluation digs further down the stack, until it finds some value that has a "name". A key is considered missed only after the stack has been exhausted.

For example, given the template:

```mustache
{{#family}}
- {{firstName}} {{lastName}}
{{/family}}
```

Data:

```swift
[
    "lastName": "Johnson",
    "family": [
        ["firstName": "Peter"],
        ["firstName": "Barbara"],
        ["firstName": "Emily", "lastName": "Scott"],
    ]
]
```

The rendering is:

```
- Peter Johnson
- Barbara Johnson
- Emily Scott
```

The context stack is usually initialized with the data you render your template with:

```swift
// The rendering starts with a context stack containing `data`
template.render(Box(data))
```

Precisely speaking, a template has a *base context stack* on top of which the rendered data is added. This base context is always available whatever the rendered data. For example:

```swift
// The base context contains `baseData`
template.extendBaseContext(Box(baseData))

// The rendering starts with a context stack containing `baseData` and `data`
template.render(Box(data))
```

The base context is usually a good place to register [filters](#filters).

See [Template.swift](Mustache/Template/Template.swift) for more information on the base context ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Classes/Template.html)).


### Expressions

Variable and section tags contain *Expressions*. `name` is an expression, but also `article.title`, and `format(article.modificationDate)`. When a tag renders, it evaluates its expression, and renders the result.

There are four kinds of expressions:

- **The dot** `.` aka "Implicit Iterator" in the Mustache lingo:
    
    Dot evaluates to the top of the context stack. It lets you iterate over collection of strings, for example. `{{#items}}<{{.}}>{{/items}}` renders `<1><2><3>` when given [1,2,3].

- **Identifiers** like `name`:
    
    Evaluation of identifiers like `name` go through the context stack until a value provides the `name` key.
    
    Identifiers can not contain white space, dots, parentheses and commas. They can not start with any of those characters: `{}&$#^/<>`.

- **Compound expressions** like `article.title` and generally `<expression>.<identifier>`:
    
    This time there is no going through the context stack: `article.title` evaluates to the title of the article, regardless of `title` keys defined by enclosing contexts.
    
    `.title` (with a leading dot) is a compound expression based on the dot: it looks for `title` at the top of the context stack.
    
    Compare these three templates:
    
    - `...{{# article }}{{  title }}{{/ article }}...`
    - `...{{# article }}{{ .title }}{{/ article }}...`
    - `...{{ article.title }}...`
    
    The first will look for `title` anywhere in the context stack, starting with the `article` object.
    
    The two others are identical: they ensure the `title` key comes from the very `article` object.

- **Filter expressions** like `format(date)` and generally `<expression>(<expression>, ...)`:
    
    [Filters](#filters) are introduced below.


Templates Eat Boxed Values
--------------------------

In all examples above, all values rendered by templates were wrapped by the `Box()` function:

```swift
template.render(Box(["name": "Luigi"]))
template.render(Box(profile))
```

Some types can be boxed and rendered, some can not, and some require some help.


### Boxable Types

The types that can feed templates are:

- `NSObject` and all its subclasses.
- All types conforming to `MustacheBoxable` such as `String`, `Int`.
- Collections and dictionaries of such types: `[Int]`, `[String: String]`.
- A few function types such as `FilterFunction` (that we will see later).

Check the rendering of [Custom Types](#custom-types) below for more information about `MustacheBoxable`.


### Non Boxable Types

Some types can not be boxed, because the rendering engine does not know what do to with them. This is the case of tuples, most function types. When you try to box them, you get a compiler error.


### Imprecise Types

`Any`, `AnyObject`, `[String: Any]`, `[AnyObject]` et al. can not be directly boxed. There is no way for GRMustache.swift to render values it does not know anything about. Worse: those types may hide values that are not boxable at all.

They must be turned into a known boxable type before they can feed templates. And that should not be a problem for you, since you do not feed your templates with random data, do you? See [issue #8](https://github.com/groue/GRMustache.swift/issues/8) for some help.


Standard Swift Types Reference
------------------------------

GRMustache.swift comes with built-in support for the following standard Swift types:


### Bool

- `{{bool}}` renders "0" or "1".
- `{{#bool}}...{{/bool}}` renders if and only if *bool* is true.
- `{{^bool}}...{{/bool}}` renders if and only if *bool* is false.


### Numeric Types

GRMustache supports `Int`, `UInt` and `Double`:

- `{{number}}` renders the standard Swift string interpolation of *number*.
- `{{#number}}...{{/number}}` renders if and only if *number* is not 0 (zero).
- `{{^number}}...{{/number}}` renders if and only if *int* is 0 (zero).

The Swift types `Float`, `Int8`, `UInt8`, etc. have no built-in support: turn them into one of the three general types before injecting them into templates.

To format numbers, use `NSNumberFormatter`:

```swift
let percentFormatter = NSNumberFormatter()
percentFormatter.numberStyle = .PercentStyle

let template = try! Template(string: "{{ percent(x) }}")
template.registerInBaseContext("percent", Box(percentFormatter))

// Rendering: 50%
let data = ["x": 0.5]
let rendering = try! template.render(Box(data))
```

[More info on NSFormatter](Docs/Guides/goodies.md#nsformatter).


### String

- `{{string}}` renders *string*, HTML-escaped.
- `{{{string}}}` renders *string*, not HTML-escaped.
- `{{#string}}...{{/string}}` renders if and only if *string* is not empty.
- `{{^string}}...{{/string}}` renders if and only if *string* is empty.

Exposed keys:

- `string.length`: the length of the string.


### Set

A set can be rendered as long as its elements are boxable.

- `{{set}}` renders the concatenation of the renderings of set elements.
- `{{#set}}...{{/set}}` renders as many times as there are elements in the set, pushing them on top of the [context stack](#the-context-stack).
- `{{^set}}...{{/set}}` renders if and only if the set is empty.

Exposed keys:

- `set.first`: the first element.
- `set.count`: the number of elements in the set.

GRMustache.swift renders as `Set` all types conforming to `CollectionType where Generator.Element: MustacheBoxable, Index.Distance == Int`. This is the minimal type which allows iteration, access to the first element, and to the elements count.

If all you have is a set `Set<Any>` or `Set<AnyObject>`, you will get a compiler error when you try to box it. Check the [Templates Eat Boxed Values](#templates-eat-boxed-values) chapter for more information.


### Array

An array can be rendered as long as its elements are boxable.

- `{{array}}` renders the concatenation of the renderings of array elements.
- `{{#array}}...{{/array}}` renders as many times as there are elements in the array, pushing them on top of the [context stack](#the-context-stack).
- `{{^array}}...{{/array}}` renders if and only if the array is empty.

Exposed keys:

- `array.first`: the first element.
- `array.last`: the last element.
- `array.count`: the number of elements in the array.

GRMustache.swift renders as `Array` all types conforming to `CollectionType where Generator.Element: MustacheBoxable, Index: BidirectionalIndexType, Index.Distance == Int`. This is the minimal type which allows iteration, access to the first element, last element, and to the elements count.

Especially, `Range<T>` is supported:

```swift
// 123456789
let template = try! Template(string: "{{ numbers }}")
let rendering = try! template.render(Box(["numbers": Box(1..<10)]))
```

If all you have is an array `[Any]` or `[AnyObject]`, you will get a compiler error when you try to box it. Check the [Templates Eat Boxed Values](#templates-eat-boxed-values) chapter for more information.


### Dictionary

A dictionary can be rendered as long as its keys are String, and its values are boxable.

- `{{dictionary}}` renders the standard Swift string interpolation of *dictionary*.
- `{{#dictionary}}...{{/dictionary}}` renders once, pushing the dictionary on top of the [context stack](#the-context-stack).
- `{{^dictionary}}...{{/dictionary}}` does not render.

If all you have is a dictionary `[String: Any]` or `[String: AnyObject]`, you will get a compiler error when you try to box it. Check the [Templates Eat Boxed Values](#templates-eat-boxed-values) chapter for more information.


### NSObject

When an object conforms to the NSFastEnumeration protocol, like NSArray, it renders just like Swift [Array](#array). NSSet is an exception, rendered as a Swift [Set](#set). NSDictionary, another exception, renders as a Swift [Dictionary](#dictionary).

NSNumber is rendered as a Swift [Bool](#bool), [Int, UInt or Double](#numeric-types), depending on its value.

NSString is rendered as [String](#string)

NSNull renders as:

- `{{null}}` does not render.
- `{{#null}}...{{/null}}` does not render.
- `{{^null}}...{{/null}}` renders.

For other NSObject, those default rules apply:

- `{{object}}` renders the `description` method, HTML-escaped.
- `{{{object}}}` renders the `description` method, not HTML-escaped.
- `{{#object}}...{{/object}}` renders once, pushing the object on top of the [context stack](#the-context-stack). Keys exposed to templates are the names of properties.
- `{{^object}}...{{/object}}` does not render.

Subclasses can alter this behavior by overriding the `mustacheBox` method of the `MustacheBoxable` protocol. For more information, check the rendering of [Custom Types](#custom-types) below.


### AnyObject and Any

When you try to render a value of type `Any` or `AnyObject`, you get a compiler error:

```swift
let any: Any = ...
let anyObject: AnyObject = ...

let rendering = try! template.render(Box(any))
// Compiler Error

let rendering = try! template.render(Box(anyObject))
// Compiler Error
```

The solution is to convert the value to its actual type:

```swift
let json: AnyObject = ["name": "Lionel Richie"]

// Convert to Dictionary:
let dictionary = json as! [String: String]

// Lionel Richie has a Mustache.
let template = try! Template(string: "{{ name }} has a Mustache.")
let rendering = try! template.render(Box(dictionary))
```

The same kind of boxing trouble happens for collections of general types like `[String: Any]` or `[AnyObject]`. For more information, check the [Templates Eat Boxed Values](#templates-eat-boxed-values) chapter.


Custom Types
------------

### NSObject subclasses

Your NSObject subclass can trivially feed your templates:

```swift
// An NSObject subclass
class Person : NSObject {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

// Charlie Chaplin has a mustache.
let person = Person(name: "Charlie Chaplin")
let template = try! Template(string: "{{name}} has a mustache.")
let rendering = try! template.render(Box(person))
```

When extracting values from your NSObject subclasses, GRMustache.swift uses the [Key-Value Coding](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html) method `valueForKey:`, as long as the key is "safe" (safe keys are the names of declared properties, including NSManagedObject attributes).

Subclasses can alter this default behavior by overriding the `mustacheBox` method of the `MustacheBoxable` protocol, described below:


### Pure Swift Values and MustacheBoxable

Key-Value Coding is not available for Swift enums, structs and classes, regardless of eventual `@objc` or `dynamic` modifiers. Swift values can still feed templates, though, with a little help.

```swift
// Define a pure Swift object:
struct Person {
    let name: String
}
```

To let Mustache templates extract the `name` key out of a person so that they can render `{{ name }}` tags, we need to explicitly help the Mustache engine by conforming to the `MustacheBoxable` protocol:

```swift
extension Person : MustacheBoxable {
    
    // Here we simply feed templates with a dictionary:
    var mustacheBox: MustacheBox {
        return Box(["name": self.name])
    }
}
```

Now we can box and render users, arrays of users, dictionaries of users, etc:

```swift
// Freddy Mercury has a mustache.
let person = Person(name: "Freddy Mercury")
let template = try! Template(string: "{{name}} has a mustache.")
let rendering = try! template.render(Box(person))
```

For a more complete discussion, check the documentation of `MustacheBoxable` in [Box.swift](Mustache/Rendering/Box.swift).


Lambdas
-------

"Mustache lambdas" are functions that let you perform custom rendering. There are two kinds of Mustache lambdas: those that process section tags, and those that render variable tags.

```swift
// `{{fullName}}` renders just as `{{firstName}} {{lastName}}.`
let fullName = Lambda { "{{firstName}} {{lastName}}" }

// `{{#wrapped}}...{{/wrapped}}` renders the content of the section, wrapped in
// a <b> HTML tag.
let wrapped = Lambda { (string) in "<b>\(string)</b>" }

// <b>Frank Zappa is awesome.</b>
let templateString = "{{#wrapped}}{{fullName}} is awesome.{{/wrapped}}"
let template = try! Template(string: templateString)
let data = [
    "firstName": Box("Frank"),
    "lastName": Box("Zappa"),
    "fullName": Box(fullName),
    "wrapped": Box(wrapped)]
let rendering = try! template.render(Box(data))
```

Those "lambdas" are a special case of custom rendering functions. The raw `RenderFunction` type gives you extra flexibility when you need to perform custom rendering. See [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).


Filters
-------

Filters apply like functions, with parentheses: `{{ uppercase(name) }}`.

Generally speaking, using filters is a three-step process:

```swift
// 1. Define the filter using the `Filter()` function:
let uppercase = Filter(...)

// 2. Assign a name to your filter, and register it in a template:
template.registerInBaseContext("uppercase", Box(uppercase))

// 3. Render
template.render(...)
```

It helps thinking about four kinds of filters:

- [Value filters](#value-filters), as in `{{ square(radius) }}`
- [Pre-rendering filters](#pre-rendering-filters), as in `{{ uppercase(...) }}`
- [Custom rendering filters](#custom-rendering-filters), as in `{{# pluralize(cats.count) }}cat{{/}}`
- [Advanced filters](#advanced-filters)


### Value Filters

Value filters transform any type of input. They can return anything as well.

For example, here is a `square` filter which squares integers:

```swift
// Define the `square` filter.
//
// square(n) evaluates to the square of the provided integer.
let square = Filter { (n: Int?) in
    guard let n = n else {
        // No value, or not an integer: return the empty box.
        // We could throw an error as well.
        return Box()
    }
    
    // Results are returned boxed, as always:
    return Box(n * n)
}

// Register the square filter in our template:
let template = try! Template(string: "{{n}} × {{n}} = {{square(n)}}")
template.registerInBaseContext("square", Box(square))

// 10 × 10 = 100
let rendering = try! template.render(Box(["n": 10]))
```


Filters can accept a precisely typed argument as above. You may prefer managing the value type yourself:

```swift
// Define the `abs` filter.
//
// abs(x) evaluates to the absolute value of x (Int, UInt or Double):
let absFilter = Filter { (box: MustacheBox) in
    switch box.value {
    case let int as Int:
        return Box(abs(int))
    case let uint as UInt:
        return Box(uint)
    case let double as Double:
        return Box(abs(double))
    default:
        // GRMustache does not support any other numeric types: give up.
        return Box()
    }
}
```


You can process collections and dictionaries as well, and return new ones:

```swift
// Define the `oneEveryTwoItems` filter.
//
// oneEveryTwoItems(collection) returns the array of even items in the input
// collection.
let oneEveryTwoItems = Filter { (box: MustacheBox) in
    // `box.arrayValue` returns a `Array<MustacheBox>` whatever the boxed Swift
    // or Foundation collection (Array, Set, NSOrderedSet, etc.).
    guard let boxes = box.arrayValue else {
        // No value, or not a collection: return the empty box
        return Box()
    }
    
    // Rebuild another array with even indexes:
    var result: [MustacheBox] = []
    for case let (index, box) in boxes.enumerate() where index % 2 == 0 {
        result.append(box)
    }
    
    return Box(result)
}

// A template where the filter is used in a section, so that the items in the
// filtered array are iterated:
let templateString = "{{# oneEveryTwoItems(items) }}<{{.}}>{{/ oneEveryTwoItems(items) }}"
let template = try! Template(string: templateString)

// Register the oneEveryTwoItems filter in our template:
template.registerInBaseContext("oneEveryTwoItems", Box(oneEveryTwoItems))

// <1><3><5><7><9>
let rendering = try! template.render(Box(["items": Box(1..<10)]))
```


Multi-arguments filters are OK as well. but you use the `VariadicFilter()` function, this time:

```swift
// Define the `sum` filter.
//
// sum(x, ...) evaluates to the sum of provided integers
let sum = VariadicFilter { (boxes: [MustacheBox]) in
    var sum = 0
    for box in boxes {
        sum += (box.value as? Int) ?? 0
    }
    return Box(sum)
}

// Register the sum filter in our template:
let template = try! Template(string: "{{a}} + {{b}} + {{c}} = {{ sum(a,b,c) }}")
template.registerInBaseContext("sum", Box(sum))

// 1 + 2 + 3 = 6
let rendering = try! template.render(Box(["a": 1, "b": 2, "c": 3]))
```


Filters can chain and generally be part of more complex expressions:

    Circle area is {{ format(product(PI, circle.radius, circle.radius)) }} cm².


When you want to format values, just use NSNumberFormatter, NSDateFormatter, or any NSFormatter. They are ready-made filters:

```swift
let percentFormatter = NSNumberFormatter()
percentFormatter.numberStyle = .PercentStyle

let template = try! Template(string: "{{ percent(x) }}")
template.registerInBaseContext("percent", Box(percentFormatter))

// Rendering: 50%
let data = ["x": 0.5]
let rendering = try! template.render(Box(data))
```

[More info on NSFormatter](Docs/Guides/goodies.md#nsformatter).


### Pre-Rendering Filters

Value filters as seen above process input values, which may be of any type (bools, ints, collections, etc.). Pre-rendering filters always process strings, whatever the input value. They have the opportunity to alter those strings before they get actually included in the final template rendering.

You can, for example, reverse a rendering:

```swift
// Define the `reverse` filter.
//
// reverse(x) renders the reversed rendering of its argument:
let reverse = Filter { (rendering: Rendering) in
    let reversedString = String(rendering.string.characters.reverse())
    return Rendering(reversedString, rendering.contentType)
}

// Register the reverse filter in our template:
let template = try! Template(string: "{{reverse(value)}}")
template.registerInBaseContext("reverse", Box(reverse))

// ohcuorG
try! template.render(Box(["value": "Groucho"]))

// 321
try! template.render(Box(["value": 123]))
```

Such filter does not quite process a raw string, as you have seen. It processes a `Rendering`, which is a flavored string, a string with its contentType (text or HTML).

This rendering will usually be text: simple values (ints, strings, etc.) render as text. Our `reverse` filter preserves this content-type, and does not mangle HTML entities:

```swift
// &gt;lmth&lt;
try! template.render(Box(["value": "<html>"]))
```


### Custom Rendering Filters

An example will show how they can be used:

```swift
// Define the `pluralize` filter.
//
// {{# pluralize(count) }}...{{/ }} renders the plural form of the
// section content if the `count` argument is greater than 1.
let pluralize = Filter { (count: Int?, info: RenderingInfo) in
    
    // The inner content of the section tag:
    var string = info.tag.innerTemplateString
    
    // Pluralize if needed:
    if count > 1 {
        string += "s"  // naive
    }
    
    return Rendering(string)
}

// Register the pluralize filter in our template:
let templateString = "I have {{ cats.count }} {{# pluralize(cats.count) }}cat{{/ }}."
let template = try! Template(string: templateString)
template.registerInBaseContext("pluralize", Box(pluralize))

// I have 3 cats.
let data = ["cats": ["Kitty", "Pussy", "Melba"]]
let rendering = try! template.render(Box(data))
```

As those filters perform custom rendering, they are based on `RenderFunction`, just like [lambdas](#lambdas). Check the `RenderFunction` type in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) for more information about the `RenderingInfo` and `Rendering` types ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).


### Advanced Filters

All the filters seen above are particular cases of `FilterFunction`. "Value filters", "Pre-rendering filters" and "Custom rendering filters" are common use cases that are granted with specific APIs.

Yet the library ships with a few built-in filters that don't quite fit any of those categories. Go check their [documentation](Docs/Guides/goodies.md). And since they are all written with public GRMustache.swift APIs, check also their [source code](Mustache/Goodies), for inspiration. The general `FilterFunction` itself is detailed in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).


Advanced Boxes
--------------

The `MustacheBox` type that feeds templates is able to wrap many different behaviors. Let's review some of them:

- `Box(Bool)` returns a box that can trigger or prevent the rendering of sections:

    ```
    {{# verified }}VERIFIED{{/ verified }}
    {{^ verified }}NOT VERIFIED{{/ verified }}
    ```

- `Box(Array)` returns a box that is renders a section as many times as there are elements in the array, and which exposes the `count`, `first`, and `last` keys:
    
    ```
    You see {{ objects.count }} objects:
    {{# objects }}
    - {{ name }}
    {{/ objects }}
    ```

- `Box(Dictionary)` returns a box that exposes all the dictionary keys:
    
    ```
    {{# user }}
    - {{ name }}
    - {{ age }}
    {{/ user }}
    ```

- `Box(NSObject)` returns a box that exposes all the object properties:
    
    ```
    {{# user }}
    - {{ name }}
    - {{ age }}
    {{/ user }}
    ```

- `Box(NSFormatter)` returns a box that is able to format a single value, or all values inside a section ([more information](Docs/Guides/goodies.md#nsformatter)):
    
    ```
    {{ format(date) }}
    {{# format }}
    From {{ startDate }} to {{ endDate }}
    {{/ format }}
    ```

- `Box(StandardLibrary.each)` returns a box that is able to define some extra keys when iterating an array ([more information](Docs/Guides/goodies.md#localizer)):
    
    ```
    {{# each(items) }}
    - {{ @indexPlusOne }}: {{ name }}
    {{/}}
    ```

This variety of behaviors is available through public APIs. Before we dig into them, let's describe in detail the rendering of the `{{ F(A) }}` tag, and shed some light on the available customizations:

1. The `A` and `F` expressions are evaluated: the rendering engine looks in the [context stack](#the-context-stack) for boxes that return a non-empty box for the keys "A" and "F". The key-extraction service is provided by a customizable `KeyedSubscriptFunction`.
    
    This is how NSObject exposes its properties, and Dictionary, its keys.

2. The customizable `FilterFunction` of the F box is evaluated with the A box as an argument.
    
    The Result box may well depend on the customizable value of the A box, but all other facets of the A box may be involved. This is why there are various types of [filters](#filters).

3. The rendering engine then looks in the context stack for all boxes that have a customized `WillRenderFunction`. Those functions have an opportunity to process the Result box, and eventually return another one.
    
    This is how, for example, a boxed [NSDateFormatter](Docs/Guides/goodies.md#nsformatter) can format all dates in a section: its `WillRenderFunction` formats dates into strings.

4. The resulting box is ready to be rendered. For regular and inverted section tags, the rendering engine queries the customizable boolean value of the box, so that `{{# F(A) }}...{{/}}` and `{{^ F(A) }}...{{/}}` can't be both rendered.
    
    The Bool type obviously have a boolean value, but so does String, so that empty strings are considered [falsey](#boolean-values).

5. The resulting box gets eventually rendered: its customizable `RenderFunction` is executed. Its `Rendering` result is eventually HTML-escaped, depending on its content type, and appended to the final template rendering.
    
    [Lambdas](#lambdas) use such a `RenderFunction`, so do [pre-rendering filters](#pre-rendering-filters) and [custom rendering filters](#custom-rendering-filters).

6. Finally the rendering engine looks in the context stack for all boxes that have a customized `DidRenderFunction`.
    
    This one is used by [Localizer](Docs/Guides/goodies.md#localizer) and [Logger](Docs/Guides/goodies.md#logger) goodies.

All those customizable properties are gathered in the most low-level function that returns a box:

```swift
public func Box(
    value value: Any? = nil,
    boolValue: Bool? = nil,
    keyedSubscript: KeyedSubscriptFunction? = nil,
    filter: FilterFunction? = nil,
    render: RenderFunction? = nil,
    willRender: WillRenderFunction? = nil,
    didRender: DidRenderFunction? = nil) -> MustacheBox
```

We'll below describe each of them individually, even though you can provide several ones at the same time:

- `boolValue`
    
    The optional *boolValue* parameter tells whether the Box should trigger or prevent the rendering of regular `{{#section}}...{{/}}` and inverted `{{^section}}...{{/}}` tags. The default value is true, unless the function is called without argument to build the empty box: `Box()`.
    
    ```swift
    // Render "true", "false"
    let template = try! Template(string:"{{#.}}true{{/.}}{{^.}}false{{/.}}")
    try! template.render(Box(boolValue: true))
    try! template.render(Box(boolValue: false))
    ```

- `value`
    
    The optional *value* parameter gives the boxed value. The value is used when the box is rendered (unless you provide a custom RenderFunction).
    
    ```swift
    let aBox = Box(value: 1)
    
    // Renders "1"
    let template = try! Template(string: "{{a}}")
    try! template.render(Box(["a": aBox]))
    ```

- `keyedSubscript`
    
    The optional *keyedSubscript* parameter is a `KeyedSubscriptFunction` that lets the Mustache engine extract keys out of the box. For example, the `{{a}}` tag would call the subscript function with `"a"` as an argument, and render the returned box.
    
    The default value is nil, which means that no key can be extracted.
    
    Check the `KeyedSubscriptFunction` type in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) for more information ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).
    
    ```swift
    let box = Box(keyedSubscript: { (key: String) in
        return Box("key:\(key)")
    })
    
    // Renders "key:a"
    let template = try! Template(string:"{{a}}")
    try! template.render(box)
    ```

- `filter`
    
    The optional *filter* parameter is a `FilterFunction` that lets the Mustache engine evaluate filtered expression that involve the box. The default value is nil, which means that the box can not be used as a filter.
    
    Check the `FilterFunction` type in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) for more information ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).
    
    ```swift
    let box = Box(filter: Filter { (x: Int?) in
        return Box(x! * x!)
    })
    
    // Renders "100"
    let template = try! Template(string:"{{square(x)}}")
    try! template.render(Box(["square": box, "x": Box(10)]))
    ```

- `render`
    
    The optional *render* parameter is a `RenderFunction` that is evaluated when the Box is rendered.
    
    The default value is nil, which makes the box perform default Mustache rendering:
    
    - `{{box}}` renders the built-in Swift String Interpolation of the value, HTML-escaped.
    - `{{{box}}}` renders the built-in Swift String Interpolation of the value, not HTML-escaped.
    - `{{#box}}...{{/box}}` does not render if `boolValue` is false. Otherwise, it pushes the box on the top of the context stack, and renders the section once.
    - `{{^box}}...{{/box}}` renders once if `boolValue` is false. Otherwise, it does not render.
    
    Check the `RenderFunction` type in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) for more information ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).
    
    ```swift
    let box = Box(render: { (info: RenderingInfo) in
        return Rendering("foo")
    })
    
    // Renders "foo"
    let template = try! Template(string:"{{.}}")
    try! template.render(box)
    ```

- `willRender` & `didRender`
    
    The optional *willRender* and *didRender* parameters are a `WillRenderFunction` and `DidRenderFunction` that are evaluated for all tags as long as the box is in the context stack.
    
    Check the `WillRenderFunction` and `DidRenderFunction` type in [CoreFunctions.swift](Mustache/Rendering/CoreFunctions.swift) for more information ([read on cocoadocs.org](http://cocoadocs.org/docsets/GRMustache.swift/0.9.3/Typealiases.html)).
    
    ```swift
    let box = Box(willRender: { (tag: Tag, box: MustacheBox) in
        return Box("baz")
    })
    
    // Renders "baz baz"
    let template = try! Template(string:"{{#.}}{{foo}} {{bar}}{{/.}}")
    try! template.render(box)
    ```


Built-in goodies
----------------

The library ships with built-in [goodies](Docs/Guides/goodies.md) that will help you render your templates: format values, render array indexes, localize templates, etc.
