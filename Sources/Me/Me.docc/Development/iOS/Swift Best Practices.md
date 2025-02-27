# Swift Best Practices

Here you can find coding guidelines which are battle tested over years. 

## Overview

Partially these coding guidelines are borrowed from popular ones like:

- [Lickability](https://github.com/Lickability/swift-best-practices)
- [LinkedIn](https://github.com/linkedin/swift-style-guide)
- etc.

## Xcode Settings

#### Editor

Be sure to have the values below set in `Xcode` > `Settings` > `TextEditing`

| Display                                                                                                                 | Editing                                                                                                                 | Indentation                                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| ![XcodeTextEditingDisplay](https://github.com/Blackjacx/Development/assets/794372/e2505341-ccf5-4010-87cf-7c5bc4144a7b) | ![XcodeTextEditingEditing](https://github.com/Blackjacx/Development/assets/794372/7aaf932d-b77b-4f6b-a097-aade72f1dfea) | ![XcodeTextEditingIndentation](https://github.com/Blackjacx/Development/assets/794372/65f1acc3-dd1c-4c3b-8349-ba2db1748e67) |

#### Spell Checking

Enable spell checking under `Edit` > `Format` > `Spelling and Grammar` > `Check Spelling While Typing`. We use en-UK
as default language.

## Misc

#### Avoid Code Duplications

This way you avoid limitless growing codebase plus, more important, you avoid fixing issues in multiple places. This is one of the most important rules.

#### Don't use the default case if it's possible that the enum is extended in the future

Switch is an extremely useful construct in Swift since the compiler warns about missing cases. If you use the default case you disable this behavior and loose one of the most powerful features of Swift. It makes sense however to use the default case if you don't want case-specific behavior for every case, but only for one or some of them. In this case it doesn't break anything if new cases get added. Also if you switch over plain value types, a default case is harmless.

#### Never omit specifying the value of string-typed enum cases

Imagine you create a string-typed enum and omit the values for every case since Swift infers the value from the case name. Now when you refactor one enum case from somewhere in your app, the inferred value also changes, which is not desired i.e. in JSON keys. You would introduce hard to find errors.

#### Don't use .none in as enum case

Your own implementation could easily be confused with `Optional.none` when your enum typed property is optional. To avoid this think about an alternative name for this case.

#### Avoid subclasses where possible and finalize your classes

Because super- and subclass logic becomes hard to maintain, try to split up logic into separate structs or classes instead. By marking your classes final, some optimizations are enabled which among other things speeds up compilation. Remember that since structs are not inheritable, they are final by design.

#### Set thickness of lines device dependent

When you set the thickness of lines to `1.0 / UIScreen.main.scale` then the will appear equally on all devices.

#### Localized-strings used in a framework F must be defined there

If don't follow this rule, **F** depends on the target/module where the string is defined and thus is not modular/independent anymore. If e.g. another target/module uses a class of **F** that contains a localized string which is outside of **F** the translation could not be found.

#### Name selectors with the pattern on<action_name>

Having a unified pattern for naming selectors helps to make code understandable faster. The on\* pattern is also used in other programming languages like JavaScript.

#### Avoid multiple statements in one line / long lines

They are harder to read. They are harder to understand. They are even not visible when you work in multiple editor windows side by side - or even worse break to the next line. They could be misunderstood by new developers as an indicator to use it everywhere which will greatly degrade the codebase.

#### Use typealiases instead of primitive types when appropriate

For example when you have to declare a property in a specific unit like `timeInSeconds` you usually call it `time` and use the type `TimeInterval` which is nothing more than a typalias. Use them to create an implicit context and keep property names swiftily short. Remember: They also appear in code completion and give you a hint about the property that you cannot infer from a short name like in `getWeight() -> Kilograms`. The latter example is a much better declaration than `getWeight() -> Double`. Read more about that in [Sundells Blog Article](https://www.swiftbysundell.com/posts/the-power-of-type-aliases-in-swift).

#### Use Camel Case for acronyms like URL or HTML

For names like `isHtmlValid` or `isUrlComplete` [Camel Case](https://medium.com/better-programming/string-case-styles-camel-pascal-snake-and-kebab-case-981407998841) improves readability of function and variable names (though it is different than Apple does it). Also see [this](https://www.hackingwithswift.com/swift/3.0/lower-camel-case) article.

## Colors

#### Use Semantic Colors

Use semantic colors instead of `red`, `blue`, etc. Semantic colors describe in which context they are used instead of which color they actually are. Good examples are `backgroundColor`, `buttonColor`, `buttonHighlightColor`, `labelColor`, etc. Using semantic colors enables the color to automatically adapt to changing OS settings like dark mode or high contrast mode.

## Framework Extensions

#### Never set translatesAutoresizingMaskIntoConstraints manually

We have the functions `<subview>.addTo(<parent>)` and `<subview>.addMaximizedTo(<parent>)` which do that automatically.

#### UI Alerts

Use UIAlertController extension to display alerts and action sheets.

#### Fonts

Never define a font in code, but rather use or extend the presets in the Fonts enum.

#### Colors

Never define a color in code, but rather use or extend the Palette struct and the related UIColor extension.

#### Date Formatter

You don't have to configure your own `DateFormatter` objects, use `SHDateFormatter` instead. If it doesn't contain a date format you need, extend it in `SHDateFormatter+Extensions`.

#### Hiding UIStackView Subviews

Use the `showArrangedSubview` method in the `UIView+Extensions` to show or hide arranged subviews.

#### Native Type Extensions

Before implementing extensions on any build-in types, have a look in the `Extensions` framework. We already implemented a lot and most likely it is already there. The rule is when it has no app dependencies it goes to the `Extensions` framework otherwise it should be part of the app.

## UIView • UIViewController Architecture

These guidelines are intended for code-based view/view controller creation without Interface Builder and teach you how to work Auto Layout programmatically.

#### Use special function to setup and add EACH subview

Setup each subview of a view or view controller in its own function! The Name of the function has to be `private func setup<property_name>`. This function should set all parameters necessary for the first setup. This includes `contentHuggingPriority` and `contentCompressionResistancyPriority`. It is called only once in init (UIView) or viewDidLoad (UIViewController). Right before the function returns add the view to its superview. This pattern turned out to be one of the cleanest structuring methods possible for the swift language. The advantage is that you have a clean interface of properties on top of the class which helps to recognize all subviews at one glance - in contrast of setting up views in a property closure. Furthermore you can use `self` since these functions are called after initialization. You have a clear overview of the adding order of subviews. Additionally you prevent one long function that sets up all properties of all subviews - typically init or viewDidLoad. Last but not least if you stick to this pattern it is much easier to understand the structure of the whole app since you'll find this pattern all over again.

#### Use special function to setup layout constraints

Always use `private func setupAutoLayout()` to create constraints between all subviews! Call it right after the last `private func setup*` function in viewDidLoad (UIViewController) or init (UIView). This activates all layout constraints.

#### Configure view in viewDidLoad (or loadView)

Never access the view in the initializer of a UIViewController subclass. Accessing the view property causes forces it to be loaded. You should never force the view to be loaded in the initializer since this is done automatically when the view is added to the view hierarchy. By default the view should be created in `loadView()`. Additional configurations are done in `viewDidLoad()`. After the view has been loaded you can start making modifications to it like adding subviews, etc.

Sometimes we initialize VCs just to pass it around. Setting up the whole view hierarchy just blocks resources which we do not need at that moment.

## Access Qualifier

#### Declare symbols internal if you want to test them

To be able to use symbols in tests declare them as `internal` (no prefix) instead of `private` even if they are not used outside of the containing object. If you don't need to test them and they are not required outside of the containing object declare it as `private`.

## Code Format

#### Use the self syntax exclusively in closures and appropriate initializers

So it will become easier to track down future retain cycles that are caused by missing `[weak self]` in the closure definition. Just leave out `self` if it is not necessary. Also use the `self.` syntax to initially assign constructor parameters to their respective properties if they have the same name.

#### Name closures according to what triggers them 

I saw that closures have been named according to what they do, e.g.
- `openMenuClosure`
- `displayAccountScreenClosure`

But closures are no functions. Instead they are more like delegates and we delegate because the delegating object does/should not know how to or it simply cannot perform a specific task. It only knows **how** to trigger it. Therefore better these closures should be named according to their trigger:
- `onButtonTap` if we have a generic view which really just has a button
- `onBottomButtonTap` if we have multiple buttons and the one in question is named `bottomButton`

The reason behind all of this is because the action may be part of a reusable piece of code and that code should not know what happens when the action is triggered. This is a general rule of thumb when it comes to modularization/abstraction/reuse.

#### Line Breaks

Do not leave a blank line underneath a every function declaration. This results in more compact code blocks which can be perceived as **one** block easier.

#### Comments

Comment functions and properties using `///` only if it enhances their comprehensibility. Generally comment code snippets if you think it is reasonable. Sometimes design choices or complex structures are not immediately obvious to understand, in such cases comments are useful for others and your future self.

#### No file header

Remove comment header for newly created files since it will be always outdated (copyright, spelling errors in file names, etc.).

#### Symbol Order

Place the nested types at the bottom of the class/struct definition with a `MARK: - Sub-Types` above. Place protocol conformances above these nested types and mark them with their respective protocol name, e.g. `MARK: - StatePresentable`.

All other symbols (properties, function, etc.) should be ordered by ascending privateness, so `public`, `internal`, `private`.

`static` symbols are always above `non-static` ones.

#### Code Organization

Do not use extensions for organizing/modularizing your code! Instead use `// MARK: - <#title#>` statements to define section in your code.

The advantages of this and specifically moving all protocols in the main object definition are:

- the main class declaration shows all conformances of the class/struct
- clean method list (`⌃+6`)
- a unified file structure results in less searching for symbols, very similar to the `func setup<view>()` convention

See also the customizable SwiftLint rule [type_contents_order](https://realm.github.io/SwiftLint/type_contents_order.html).

#### Name bools with the pattern isState

This is easy to read and matches the swifty naming conventions in modern iOS frameworks.

#### Line Length

Try to achieve a maximum line length of 80-120 characters. This has several reasons:

- Some developers work with 2 parallel editors (2 editors, 1 editor / 1 canvas, 1 editor / 1 assistant / etc.) and maybe also the simulator side by side.
- Horizontal scrolling is quite annoying
- Not every developer has a 4K monitor. Be inclusive!
- During presentations the font size has to be increased which reduces the available space in the editor
- Many command line applications work with 120 or 80 columns

In all of the cases above the editor's code should be fully readable **without** horizontal scrolling and there should be no need to constantly adjust the Xcode panels as this is pretty stressful.

Code that looks ugly with a line-length limit of 80-120 is a great indicator that refactoring is needed.

Please check out the SwiftLint rule [line_length](https://realm.github.io/SwiftLint/line_length.html).

#### Avoid typealias for Nested structs

Typealiases have a bunch of advantages but for nested structs, this are the cons:
1. When `⌘`-clicking the typealias, Xcode will take you to the alias. What you want instead is navigate to the original object.
2. Quick-Open (`⌘ + ⇧ + o`) leads you to the typealias. You actually want the original object as suggestion. It also can happen that you end up with multiple suggestions of the same type in the quick-open dialog (one for the alias and one for the real object). 
3. When you add the same nested object under another parent struct, specifying the same aliases would be confusing. It could even lead to name-space conflicts, depending on where you declare them. In your codebase it hides the information which struct is actually meant.
4. When you add a typealias for each nested struct, you would end up with tons of additional code on top of many files. 

For these reasons I advise against using typealiases for nested objects. If you use them because of line length issues, meanwhile there are nice formatting options in Xcode that streamline indentation (`⌃ + m` in combination with SwiftFormat works perfectly).

## Xcode

#### No White-Spaces In Group/Folder Names

Scripts do not like whitespaces. Since we have a lot of automation by scripts, please use `_` as replacement for them.

## Accessibility

 [This article](https://developer.apple.com/news/?id=v56qu1b3) by Apple is a good introduction and reference for how to use Voice Over.

#### Use the accessibilityId of a view to make it identifiable in Unit/UI Tests or Voice Over announcements

The syntax for those identifiers should be `aid.<view_controller>.<ui_element>` or if further scoping is needed `aid.<view_controller>.<parent_view>.<ui_element>`.

#### Set the appropriate accessibility traits if not set automatically

While buttons usually have it implicitly set, labels functioning as e.g. headers need need their traits set to `.header`. The same applies for search fields. In cases were a UIView serves as a button, it's trait should be set to `.button`.

#### Use post notifications to inform about screen/layout changes

Use it always when a new screen/popup/alert is displayed and you want to draw the users attention to it. Use it sparsely in other situations since it interrupts messages spoken by Voice Over on user interaction. An alternative would be to manually refocus an element and re-read it's label or hint, containing the new information.

#### Disable Voice Over for images

Blind people cannot see the image. If you enable Voice over here it should be a meaningful and short description of what the image shows.

#### Group elements to declutter the UI for Voice Over users

Focusable elements should be kept to the necessary minimum. For example, labels and controls that belong together should be grouped to one accessibility element. The element should have its label's text for its a11y label and the control as its action. Compare the title + switch cells in the iOS Settings App.

#### Any colors should have a sufficient contrast

Use at least 4.5:1 for fonts <18pt and 3:1 for fonts >=18pt. You can also read in the official [WCAG20 docs](https://www.w3.org/TR/WCAG20/#visual-audio-contrast-contrast) about this topic.

#### Any view that contains text should support Dynamic Type

These views should adapt to different font sizes so the UI looks good when the user scales up/down the font in the iOS settings. Support of the highest Dynamic Type setting is enough. `Larger Accessibility Sizes` are not required for now.

## Testing

#### Extract testing code to helper functions

Extract testing code to helper functions if you need to write the same testing code over and over again to test similar things. The problem here is that Xcode will report the failure in your helper function. This is easily fixable by writing your function like this: `func verify(file: StaticString = #file, line: UInt = #line) { XCTAssertEqual(1, 3, file: file, line: line) }`! The error message will jump to the call site of the extracted function.

#### Prefer XCTUnwrap over guard let

XCTUnwrap requires only 1 line instead of at least 3 for the guard. It makes tests much more readable. The error message is even better and most important **standardized**: `XCTUnwrap failed: expected non-nil value of type "UIDatePicker"`. No worries the `it` closures of Quick will handle thrown errors correctly.

## Git

#### Conflicts on a branch have to be resolved by the author

This is super important since only the author truly knows how to integrate the changes of a branch into the development branch. The author is responsible for the PR and after making the changes ready for review there should not be changes by anybody else because they can lead to issues which fall back to the author in the end. Instead ask the author to do the changes. If conflict resolution is especially difficult, work together with the developer who created the commit that triggers the conflict.

This basically happens during the review.
