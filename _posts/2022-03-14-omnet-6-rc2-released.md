---
layout: post
title: OMNeT++ 6.0 Release Candidate 2 Available
category: Software
---
Please welcome the [OMNeT++ 6.0 Release Candidate 2](/download/preview). OMNeT++ 6.0 is the result of more than three years of work, and includes many essential new features that we would already have a hard time without. The present changelog summarizes all changes made during the 15+ pre-releases.

<!--more-->

We briefly summarize the changes below in each part of OMNeT++ before going into the details.

The most prominent new feature is the new Python-based Analysis Tool in the IDE. The use of Python under the hood allows for arbitrarily complex computations to be performed on the data, visualizing the result in the most appropriate form chosen from a multitude of plot types, and producing publication quality output, all while using an intuitive user interface that makes straightforward tasks easy and convenient. Custom computations and custom plots are also easily accessible. The tool is able to handle large quantities of data. The Python APIs are also available outside the IDE (e.g. for standalone scripts), and a command-line tool for viewing and exporting charts created in the IDE also exists (`opp_charttool`).

The NED language now supports parameters that carry C++ objects as values (type `object`), which can be used to parameterize modules with structured data (e.g. nontrivial configuration), packet prototypes, function objects, etc. Structured data may come from NED functions like `readCSV()` or `readJSON()` which parse data files, or may be specified directly in NED or ini files using JSON syntax. The syntax of ini files has even been adjusted to make it more convenient to write multi-line JSON values in it. Further new functionality includes the string match operator `=~`, the "spaceship" operator `<=>`, and support for Universal Function Call Syntax (UFCS). Incompatible changes include the change in the interpretation of parameter names that are not qualified with the `this` or `parent` keywords, and the necessity to mark module parameters with `@mutable` that are allowed to be set at runtime. Embedding NED files into simulation binaries for easier dissemination has also become possible.

Message descriptions (msg files) have undergone even bigger changes. An import system has been added to make the content of a msg file available in others. The generated code and class descriptors can now be widely customized via properties. Targeted C++ blocks have been introduced for injecting C++ code into various places in the generated source files. Altogether, these (and further, smaller) features facilitate writing significantly cleaner msg files, especially in large projects like INET.

The format of ini files have been made more flexible: the `Config` word in section headers is now optional, and long lines can be broken up to multiple lines without using trailing backslashes (just indent the continuation lines).

In the simulation kernel, the most important change is the introduction of the Transmission Updates API, which allows in-progress packet (frame) transmissions to be modified, i.e. aborted, shortened, or extended. This API is necessary for implementing L2 features like frame preemption or on-the-fly frame aggregation. Other changes include the addition of utility APIs like `scheduleAfter()`, `rescheduleAt()` and `rescheduleAfter()`, refinements around module deletion and the introduction of the `preDelete()` virtual member function, refinements in the signals and listeners APIs, improved expression evaluation support, and the addition of string-handling utility functions, just to name a few.

Regarding statistics recording, perhaps the most significant addition is the `demux` filter, which allows splitting a single stream (of emitted values) to multiple streams. The filter makes it possible to record statistics subdivided by some criteria, e.g. to record statistics per TCP connection, per remote IP address, per traffic class, etc., in INET. Further improvements include the addition of the `warmup` filter and the `autoWarmupFilter` statistic attribute that allow computed statistics to be calculated correctly also in the presence of a nonzero warm-up period. Result files now hold more metadata: parameter values and configuration entries are now also (optionally) recorded. This change, together with other, smaller adjustments, cause new result files not be understood by previous versions of OMNeT++.

A significant amount of work has been put into improving the looks and functionality of Qtenv: material-style icons, HIDPI support, dark theme support, and countless small features that improve usability, especially around the Object Inspector and the Log View. For example, it is now possible to set the Messages view of the Log to display all timestamps as a delta to a user-specified reference timestamp; or, the Object Inspector now allows selecting a different display mode for a tree node. One important change is that method call animations are now disabled by default.

The Sequence Chart tool in the IDE has been significantly improved, both visually and functionally, with the goal of allowing the user to configure the chart in a way that facilitates understanding of the sequence of events in the simulation. The tools provided for that purpose are the possibility to hide unnecessary elements (unrelated arrows, events, etc), support for user-defined coloring, interactively collapsible compound module axes, horizontally expanded events so that the arrows of nested method calls do not overlap, and more. The eventlog file format has also changed in a non-backward-compatible way, due to the addition of extra elements that allow the Sequence Chart tool to be faster and more scalable.

Generating documentation from NED and MSG files has been made possible from the command line as well, using the new `opp_neddoc` tool. Functionality has been extended with the possibility of incorporating external information into the generated pages.

The C++ debugging experience has been made more pleasant in several ways. For example, the "Debug on Error" and "Debug Next Event" functionality in Qtenv may now invoke the integrated debugger of the Simulation IDE, which is especially useful when the simulation was launched from the IDE. The User Guide also contains a hint on how to configure simulations to invoke VS Code as debugger.

Due to improvements in the toolchain and the build process, Windows users may see the linking time of large models like INET Framework to drop dramatically (1-2 orders magnitude, e.g. from several minutes to several seconds). On macOS, Apple Silicon, is currently supported with x86-64 emulation.

Now, the details:

NED:

  - Semantics change: Within a submodule's or connection's curly brace block, the interpretation of parameter, gate, and submodule references that don't use an explicit `parent` or `this` qualifier has changed. They no longer refer to the enclosing compound module's item, but to the item of (or within) the local submodule or channel object. The interpretation of item references outside subcomponent blocks has remained unchanged. An example:

        network Network {
          parameters:
              int foo;
          submodules:
            node1: { ... }
            node2: Node {
              foo = foo; // ERROR: self-reference! Change to 'foo=parent.foo'.
              bar = this.foo;  // OK
              baz = node1.foo;  // ERROR: refers to yet-uncreated submodule node2.node1!
              bax = parent.node1.foo;  // OK
            }
        }

    The set of forms accepted by `exists()` and `sizeof()` has been expanded. Notably, `exists()` can now be used to check the existence of an item in a submodule vector (as submodule vectors may now contain holes, due to `@omittedTypename`). Also, the `index()` syntax has been removed, `index` is only accepted without parens.

    Note that referencing a submodule's submodule from NED (i.e. the `a.b` or `this.a.b` syntax within a submodule block) is generally wrong, because the network is built top-down, and submodules are only created after parameters have already been set up. (An exception is using it in the values of `volatile` parameters, because they are evaluated later.)

    Error messages have been revised and expanded to be more helpful and facilitate porting NED code from 5.x.

    To write NED which is compatible with both OMNeT++ version 5.x and 6.0, qualify all references within subcomponent blocks with explicit `parent` or `this`, and require OMNeT++ 5.7 which is the first and only 5.x version that understands the `parent` keyword.

  - NED grammar: Added `object` as parameter type. Parameters of this type may hold objects subclassed from `cOwnedObject` that they can take ownership of.

  - NED parameters of type `object` may have a `@class()` property, with the syntax `@class(classname)` or `@class(classname?)`. The property specifies that the parameter should only accept objects of the given type and its subclasses. The referenced class must be registered via `Register_Class()` or `Register_Abstract_Class()`. Parameters declared without the question mark in `@class()` don't accept `nullptr` as value, while the ones with question mark do.

  - The NED expression syntax has been extended to accept JSON-style constructs, i.e. arrays and dictionaries (termed "object" in JSON), and the `nullptr` keyword. The array syntax is a list of values enclosed in square brackets, for example `[1, 2, 3]`, and the array is accessible as a `cValueArray` object in C++. The dictionary syntax uses curly braces: `{"foo" : 1, "bar" : 2 }`, and dictionaries are presented as `cValueMap` objects. If a dictionary is prefixed by a name, then the name is interpreted as a class name, and values are interpreted as fields of the object: `cMessage { name: "hello", kind: 1}`.

  - Object parameters allow simple values (int, double, bool, string) as well. C++-wise, they are stored in the parameter in `cValueHolder`-wrapped `cValue` objects.

  - New NED functions have been introduced: `get()` (return an element of an array or dictionary), `size()` (returns the size of an array or dictionary), `eval()` (evaluates a NED expression given as string), `dup()` (clones an object). `dup()` simply calls an object's C++ `dup()` method; it is useful because module parameters of type `object` only accept objects they fully own, which can be worked around using `dup()`. An example:

      object a = [1,2,3];
      object b = dup(a);  // without dup() it is an error: value is owned by parameter 'a'

    As `dup()` invoked on containers only duplicates the objects owned by the container, you may need extra `dup()` call when referring to objects owned by other parameters. Example:

        object a = {};
        object b = dup([a,a]);  // ERROR (a's inside the array are not cloned)
        object c = [dup(a), dup(a)]  // OK

  - NED grammar: Made operator precedence more similar to C/C++. Relational operators (`==, !=, <, <=, >, >=`) used to be on the same precedence level; now `==` and `!=` have lower precedence than the rest.

  - NED grammar: Added the `=~` (match) and `<=>` (comparison, a.k.a. "spaceship") operators, the `undefined` keyword, and `bool()` as conversion function.

  - String constants are now accepted with apostrophes too. This makes it possible to write quotation marks in strings without the need to escape them.

  - For quantities (numbers with measurement units), the rules have changed slightly. When specifying quantities with multiple terms, a minus sign is only accepted at the front, to reduce the chance of confusion in arithmetic expressions. I.e. `-5s100ms` is valid as a quantity string (and means -5.1s), but `5s-100ms` is no longer.

    Also, the plain "0" constant is no longer accepted at places where a quantity with a measurement unit is expected. Reason: it was confusing when the expected unit was dB or dBm. E.g. in `omnetpp.ini`, should `**.power = 0` be interpreted as 0W or 0dBm?

  - Added support for Universal Function Call Syntax (UFCS) to NED, which means that now any function call may also be written as if the function was a method of its first argument (provided it has one). That is, `f(x,..)` can now be also written in the alternative form `x.f(...)`. This results in improved readability in certain cases, and allows chaining function calls.

  - `xmldoc()` (and other file-reading functions) now interpret the file name as relative to the directory the expression comes from. When `xmldoc()` occurs in an included ini file, the file name is now interpreted as relative to the directory containing the included ini file (as opposed to being relative to the main ini file or the working directory.) If `xmldoc()` occurs in a NED file, the file name is relative to the directory where that NED file was loaded from.

  - NED functions for reading (from file) and parsing (from string) of CSV, JSON and XML files: `parseCSV()`, `readCSV()`, `parseExtendedCSV()`, `readExtendedCSV()`, `parseJSON()`, `readJSON()`, `parseExtendedJSON()`, `readExtendedJSON()`, `parseXML()`, `readXML()`. The "extended" variants support expressions in the file, which will be evaluated during parsing. The XML functions are aliases to the existing `xml()`/`xmldoc()` functions. Bonus file-related functions: `readFile()`, `workingDir()`, `baseDir()`, `resolveFile()`, `absFilePath()`.

  - The body of a parametric submodule now allows assigning apparently nonexistent parameters. For example, in the example below, the `sleepTime` assignment does not cause an error if `FooApp` has a `sleepTime` parameter but `IApp` does not:

        app: <default("FooApp")> like IApp {
            parameters:
                address = parent.address;
                sleepTime = 1s;  // ignored if app has no 'sleepTime' parameter
        }

  - Implemented `@omittedTypename` property. `@omittedTypename` allows one to specify a NED type to use when `typename=""` is specified for a parametric submodule or channel. `@omittedTypename` can be specified on a module interface or channel interface, or on a submodule or connection. It should contain a single (optional) value. The value names the type to use. If it is absent, the submodule or channel will not be created. (The connection will be created without a channel object.)

  - The `@mutable` property was introduced for parameters. If the module (or channel) is prepared for a parameter to change its value at runtime (i.e. the new value takes effect), then it should be marked with `@mutable`. If `@mutable` is missing, then trying to change the parameter will now result in a runtime error ("Cannot change non-mutable parameter".)

    The motivation is that in a complex model framework, there is usually a large number of (module or channel) parameters, and so far it has not been obvious for users which parameters can be meaningfully changed at runtime. For example, if a simple module did not implement `handleParameterChange()` or the `handleParameterChange()` method did not handle a particular parameter, then the user could technically change that NED parameter at runtime, but the change did not take effect. This has often caused confusion.

    Parameters of `ned.DelayChannel` and `ned.DatarateChannel` are now marked `@mutable`.

    To allow running existing simulation models that have not yet been updated with `@mutable` annotations, we have added the `parameter-mutability-check` configuration option. Setting `parameter-mutability-check=false` will give back the old behavior (not raising an error).

  - Added the `expr()` operator to NED, with the purpose of allowing models to accept formulas or expressions which it could use e.g. to determine the processing time of a packet based on packet length or other properties, decide whether a packet should be allowed to pass or should be filtered out based on its contents (filter condition), or to derive x and y coordinates of a mobile node in the function of time.

    The argument to `expr()` is a NED expression which will typically contain free variables, (like x and y in `expr(x+y)`). The `expr()` operator creates an object that encapsulates the expression, so it can be assigned to parameters of the type `object`. On the C++ side, the module implementation should query the parameter to get at the expression object (of type `cOwnedDynamicExpression`), and then it may bind the free variables and evaluate the expression as often as it wishes.

  - In `xml()` and `xmldoc()` element selectors, `$MODULE_INDEX`, `$PARENTMODULE_NAME` and similar variables now evaluate to the empty string if they are not applicable, instead of raising an error that was often problematic.

  - Improved error reporting during network building: when an error occurs during assigning a parameter from a NED file, the error message now includes the location of the assignment (file:line in NED file).


MSG:

  - Made the hitherto experimental operation mode and feature set of the message compiler official. This feature set was originally added around OMNeT++ 5.3, and could be enabled by passing the `--msg6` option to `opp_msgtool`. There was also a no-op `--msg4` option that selected the old (OMNeT++ 4.x compatible) operation. Most of the OMNeT++ 6.0 pre-releases shipped with the new operation mode being the default (i.e. `--msg6` became a no-op), with features continually being added and refined. Then, finally, the old code and the `--msg4` option were removed altogether.

    The new operation mode represents a complete overhaul of the message compiler, with significant non-backward-compatible changes to the MSG language. These features were largely motivated by the needs of INET 4.

  - The message compiler is now aware of the OMNeT++ library classes, as if they were imported by default. Declarations come from `sim_std.msg`.

  - Added import support. A message file can now reference definitions in other message files using the `import` keyword. Type announcements are no longer needed (in fact, they are ignored with a warning), and there is now much less need for `cplusplus` blocks as well.

  - Classes without an `extends` clause no longer have `cObject` as their default base class. If `cObject` should be the base class, `extends cObject` needs to be added explicitly.

  - Field getters now return `const` reference. Separate `get..ForUpdate()` getters that return non-`const` are generated to cover uses cases when the contained value (typically an object) needs to be modified in-place.

  - Added targeted `cplusplus` blocks, with the syntax of `cplusplus(<target>) {{..}}`. The target can be `h` (the generated header file -- the default), `cc` (the generated C++ file), `<classname>` (content is inserted into the declaration of the type, just before the closing curly bracket), or `<classname>::<methodname>` (content is inserted into the body of the specified method). For the last one, supported methods include the constructor, copy constructor (use `Foo&` as name), destructor, `operator=`, `copy()`, `parsimPack()`, `parsimUnpack()`, etc., and the per-field generated methods (setter, getter, etc.).

  - Enum names can now be used as field type name, i.e. `int foo @enum(Foo)` can now be also written as `Foo foo`.

  - Support for `const` fields (no setter generated/expected).

  - Support for pointer and owned pointer fields. Owned pointers are denoted with the `@owned` field property. Non-owned pointers are simply stored and returned; owned pointers imply delete-in-destructor and clone-on-dup behavior. Owned pointer fields have a remover method generated for them (`removeFoo()` for a `foo` field, or `removeFoo(index)` if the `foo` field is an array). The remover method removes and returns the object in the field or array element, and replaces it with a `nullptr`. Additionally, if the object is a `cOwnedObject`, `take()` and `drop()` calls are also generated into the bodies of the appropriate methods. There is also an `@allowReplace` property that controls whether the setter method of an owned pointer field is allowed to delete the previously set object; the default is `@allowReplace(false)`.

  - More convenient dynamic arrays: inserter, appender and eraser methods are now generated into the class. For example `insertFoo(index,value)`, `appendFoo(value)`, and `eraseFoo(index)` are generated for a `foo[]` field.

  - Support for pass-by-value for fields. Annotate the field with `@byValue` for that. `@byValue` and many other properties can also be specified on the class, and they are inherited by fields that instantiate that type.

  - Additional C++ base classes may be specified with the `@implements` property.

  - The `@str` property causes an `str()` method to be generated; the expression to be returned from the method should be given in the value of the property.

  - The `@clone` property specifies code to duplicate (one array element of) the
    field value.

  - The `@beforeChange` class property specifies the name of a method to be called from all generated methods that mutate the object, e.g. from setters. It allows implementing objects that become immutable ("frozen") after an initial setup phase.

  - The `@custom` field property causes the field to only appear in descriptors, but no code is generated for it at all. One can inject the code that implements the field (data member, getter, setter, etc.) via targeted `cplusplus` blocks.

  - The `@customImpl` field property, suppresses generating implementations for the field's accessor methods, allowing custom implementations to be supplied by the user.

  - Added the `@abstract` field and class property. For a field, it is equivalent to the `abstract` keyword; for classes, it marks the whole class as abstract.

  - Abstract fields no longer require the class to be marked with `@customize`.

  - Message files now allow more than one `namespace <namespace>;` directive. The `namespace;` syntax should be used to return to the toplevel C++ namespace.

  - The names of generated method can be overridden with the following field properties: `@setter`, `@getter`, `@getterForUpdate`, `@sizeSetter`, `@sizeGetter`, `@inserter`, `@eraser`, `@appender`, `@remover`, etc.

  - Data types can be overridden with the following properties: `@cppType`, `@datamemberType`, `@argType`, `@returnType`, `@sizeType`.

  - Changed C++ type for array sizes and indices, i.e. the default of `@sizeType`, from `int` to `size_t`.

  - Added support for setting pointer members and array sizes via class descriptors. (`cClassDescriptor` had no facility for that.) This involves adding two methods to `cClassDescriptor` (`setFieldArraySize()` and `setFieldStructValuePointer()`), and support for the `@resizable()` and `@replaceable` field attributes that tell the message compiler to generate the respective code in the class.

  - The `@toString` and `@fromString` properties specify a method name or code fragment to convert the field's value to/from string form in class descriptors (`getFieldValueAsString()` and `setFieldValueFromString()` methods of `cClassDescriptor`). In the absence of `@toString`, previous versions converted the value to string by writing it to a stream using `operator<<`; now the `str()` method of the object is used if it has one. If neither `@toString` nor `str()` exist, an empty string is returned.

  - Likewise, the `@toValue` and `@fromValue` properties specify a method name or code fragment to convert the field's value to/from `cValue` form in class descriptors ((`getFieldValue()` and `setFieldValue()` methods of `cClassDescriptor`)).

  - Better code for generated classes, e.g. inline field initializers, and use of the `=delete` syntax of C++11 in the generated code.

  - Better code generated for descriptors, e.g. symbolic constants for field indices.

  - The list of reserved words (words that cannot be used as identifiers in MSG files; it is the union of the words reserved by C++ and by the MSG language) has been updated.

  - A complete list of supported properties (not all of them are explicitly listed above) can be found in an Appendix of the Simulation Manual.

Ini files:

  - It is now possible to break long lines without using a trailing backslash. Continuation lines are marked as such by indenting them, i.e. an indented line is now interpreted as a continuation of the previous line. (It is not possible to break a line inside a string constant that way.) Breaking lines using a trailing backslash way still works (and it can also be used to break string constants, too). Indentation-based line continuation has the advantage over backslashes that it allows placing comments on intermediate lines (whereas with backslashes, the first `#` makes the rest of the lines also part of the comment).

  - The `Config` prefix in section headers is now optional, that is, the heading `[Config PureAloha]` may now be also written as `[PureAloha]`, with the two being equivalent.

Simulation kernel / Modules, channels, programming model:

  - Added the `scheduleAfter()`, `rescheduleAt()`, `rescheduleAfter()` methods to `cSimpleModule`. They are mainly for convenience, but using `rescheduleAt()` instead of `cancelEvent()` + `scheduleAt()` will eventually allow for a more efficient implementation.

  - Change in the parameter change notification mechanism: Calls to the `handleParameterChange()` method during initialization are no longer suppressed. Because now every change results in a notification, the umbrella `handleParameter(nullptr)` call at the end of the initialization is no longer needed and has been removed. The consequence is that `handleParameterChange()` methods need to be implemented more carefully, because they may be called at a time when the module may not have completed all initialization stages. Also, if an existing model relied on `handleParameter(nullptr)` being called, it needs to be updated.

  - Improvements in the multi-stage initialization protocol with regard to dynamic module creation. Modules now keep track of the last init stage they completed (`lastCompletedInitStage`). During an init stage, initialization of modules is restarted if creation/deletion is detected during iteration; modules already initialized in the previous round are recognized and skipped with the help of `lastCompletedInitStage`.

  - `cModule` now keeps track of submodule vectors as entities, and not just as a collection of submodules with vector indices, meaning that we can now distinguish between nonexistent and zero-size submodule vectors. Random access of vector elements has also became more efficient (constant-time operation). Several new methods have been added as part of this change: `hasSubmoduleVector()`, `getSubmoduleVectorSize()`, `addSubmoduleVector()`, `deleteSubmoduleVector()`, `setSubmoduleVectorSize()`, `getSubmoduleVectorNames()`.

  - As part of the above change, several `cModule` and `cModuleType` methods have been added or updated. A partial list:

    - In `cModule`, the `hasSubmodule()`, `getSubmoduleNames()`, `hasGateVector()`, `hasGates()` methods have been added for consistency between submodule and gate APIs.
    - The return type of the `getGateNames()` method has been changed from `std::string<const char*>` to `std::vector<std::string>`, for consistency with `getSubmoduleNames()`.
    - `cModule`: added `setIndex()` and `setNameAndIndex()`.
    - `cModule`, `cGate`: `getIndex()`, `getVectorSize()`, `gateSize()` and similar methods now throw exception for non-vector submodules/gates.
    - `cModule`: add separate `addGateVector()` method instead misusing `addGate()` for creating gate vectors, also for consistency with `addSubmoduleVector()`.
    - In `cModuleType::create()`, the `vectorSize` parameter of `create()` has been removed.
    - In `cModuleType::createScheduleInit()`, an index argument was added to allow creating submodule vector elements.
    - `cModule`: added `addSubmodule()` method which flows more naturally than `cModuleType::create()`.

  - There have been changes in submodule and channel iterators. `SubmoduleIterator` has been rewritten due to the change in how submodule vectors are represented, which may affect the iteration order in some cases. Iterators now throw an exception if a change occurs in the list of submodules/channels during iteration. Their APIs have also changed a little: `operator--` was removed, and `init(m)` was renamed to `reset()`.

  - Optimized `cModule::ChannelIterator` by letting `cModule` maintain a linked list of channels (`cChannel`), so that `ChannelIterator` doesn't have to search through the whole compound module to find them.

  - Module name and full name (i.e. "name[index]") are now stringpooled, which reduces memory usage in exchange for a small build-time extra cost. In addition, string pools now use `std::unordered_map` instead of `std::map`, which results in improved performance.

  - `cComponent`: added `getNedTypeAndFullName()` and `getNedTypeAndFullPath()`. They are especially useful in constructing error messages in NED functions.

Simulation kernel / Signals and notifications:

  - `intpar_t` was renamed to `intval_t`, and `uintval_t` was added. Both are guaranteed to be at least 64 bits wide.

  - Signal listeners changed to use `intval_t` and `uintval_t` instead of `long` and `unsigned long`. This change was necessary because `long` is only 32 bits wide on Windows. This affects methods of `cListener` and subclasses like `cResultFilter` and `cResultRecorder`.

  - Emitting `nullptr` as a string (`const char*` overload of `emit()`) is disallowed. The reason is that `nullptr` cannot be represented in `std::string`, which causes problems e.g. in result filters and recorders.

  - Added two new model change notifications: `cPostModuleBuildNotification`, `cPostComponentInitializeNotification`.

  - Minor changes in some model change notification classes. In `cPreModuleAddNotification`, the `vectorSize` field was removed (as it was redundant), and in `cPostModuleDeleteNotification`, the interpretation of the index field has changed a little: if the deleted module was not part of a module vector, index is now set to -1 instead of 0.

Simulation kernel / Utilities:

  - Two utility functions were added to `cObject`: `getClassAndFullPath()` and `getClassAndFullName()`. They are mostly useful in logging and error messages.

  - `cMatchExpression`: The `field =~ pattern` syntax replaces `field(pattern)`. Also removed the implicit OR syntax. The old syntaxes looked confusing, and made it difficult to tell apart the concise notation from expression-style notation.

  - Added `opp_component_ptr<T>`. It implements a smart pointer that points to a `cComponent` (i.e. a module or channel), and automatically becomes `nullptr` when the referenced object is deleted. It is a non-owning ("weak") pointer, i.e. the pointer going out of scope has no effect on the referenced object. `opp_component_ptr<T>` can be useful in implementing modules that hold pointers to other modules and want to be prepared for those modules getting deleted. It can also be useful for simplifying safe destruction of compound modules containing such modules.

  - New classes: `opp_pooledstring`, `opp_staticpooledstring`. They provide pool- backed string storage (reference-counted and non-reference-counted, respectively). In turn, the `cStringPool` class was removed; use either of the pooled string classes or or `opp_staticpooledstring::get(const char *)` instead.

  - Several string functions have been made available for models. A representative partial list: `opp_isempty()`, `opp_isblank()`, `opp_nulltoempty()`, `opp_trim()`, `opp_split()`, `opp_splitandtrim()`, `opp_join()`, `opp_stringendswith()`, `opp_substringbefore()`, etc.

  - The `cStringTokenizer` class has been rewritten. It now supports features like optional skipping of empty tokens, optional trimming of tokens, optional honoring of quotes, optional honoring of parens/braces/brackets (i.e. the input string is not broken into tokens in the middle of a parenthesized expression).

Simulation kernel / Visualization support:

  - Added display name support to modules. Display name is a string that optionally appears in Qtenv next to (or instead of) the normal module name, in order to help the user distinguish between similarly-named submodules. For example, application-layer modules `app[0]`, `app[1]`, etc. in INET may be given descriptive names like "file transfer", "video", "voice", or "CBR", and have them displayed in Qtenv. Display names may be set using the `display-name` per-module configuration option, or programmatically by calling `setDisplayName()`.

  - Added the `g=<group>` (layout group) display string tag, which makes it possible to apply predefined arrangements like row, column, matrix or ring to a group of unrelated submodules. This layouting feature was previously only available for submodule vectors. When "g" tags are used, submodules in the same group are now regarded for layouting purposes as if they were part of the same submodule vector.

  - Made it possible to specify display strings in the configuration. The value given in the `display-string` per-component configuration option is merged into the component's display string in the same way inheritance or submodule display strings work: it may add, overwrite or remove items from it.

  - Added text alignment support to text and label figures: `cFigure::Alignment` enum, `getAlignment()`/`setAlignment()` in `cAbstractTextFigure`.

  - Added the `toAlpha()` method and a constructor taking `Color` to `cFigure::RGBA`.

Simulation kernel / Transmission updates:

  - The initial `send()` is interpreted as: "packet transmission begins now, packet content and duration are, as things are now, going to be this".

    Following that, an "update" (or any number of updates) can be sent. An update is a packet with the updated ("actual") content, and with a "remaining transmission duration" attached. Updates may only be sent while transmission is still ongoing.

    As an example, aborting a transmission is done by sending a packet with a truncated content and a remaining duration of zero.

    Transmission updates are paired with the original packet they modify using a transmissionId. The transmissionId is normally chosen to be the packet ID of the original transmission. Channels should understand updates and handle them accordingly.

    Receivers that receive the packet at the end of the reception, which is the default operating mode, will only receive the final update. The original packet and intermediate updates are absorbed by the simulation kernel.

    Receivers that receive the packet at the start of the reception (see `cGate::setDeliverImmediately()`, previously called `setDeliverOnReceptionStart()`) should be prepared to receive all of the original packet and the updates, and handle them appropriately. Tx updates can be recognized from `cPacket::isUpdate()` returning `true`. `cPacket::getRemainingDuration()` returns the remaining transmission duration, and `cPacket::getDuration()` the total transmission duration.

    As a safeguard against unprepared modules accidentally processing tx updates as normal full-blown packets, the module is only given tx updates if it explicitly declares that it is able to handle them properly. The latter is done by the module calling `setTxUpdateSupport(true)` before receiving packets, e.g. in `initialize()`.

    Non-transmission channels treat tx updates in the same way as they treat any other messages and packets (they ignore the `cPacket::isUpdate()` flag).

    Details and related changes follow.

  - `send()` and `sendDirect()` now accept a `SendOptions` struct where optional parameters such as delay can be passed in. `sendDelayed()` and other `send()`/`sendDirect()` variants now convert their extra args to a `SendOptions`, and delegate to the "standard" `send()`/`sendDirect()` versions. `SendOptions` was introduced as a means to handle combinatorial explosion of `send()` variants.

  - For methods that participate in the send protocol (`cGate::deliver()`, `cModule::arrived()`, `cChannel::processMessage()`), `SendOptions` was added.

  - `cDatarateChannel` now allows the sender to explicitly specify the packet duration in `SendOptions`, overriding the duration that the channel would compute from the packet length and the channel datarate.

  - `cDatarateChannel`'s datarate is now optional: set it to 0 or `nan` to leave it unspecified. This change was necessary to support transmitting frames with per-frame data rate selection. If the datarate is unspecified, the packet duration must be supplied in the send call, otherwise a runtime error will be raised.

  - `cDatarateChannel`: non-packet messages now pass through without interfering with packets.

  - `cDatarateChannel`: disabled channels now let transmission updates through, so that it is possible for the transmitter module to abort the ongoing packet transmission.

  - Tx updates (without duration/remainingDuration) are allowed on paths without transmission channels.

  - In `cChannel::processMessage()`, `result_t` was renamed `cChannel::Result`, and it is now a proper return value (not an output parameter).

  - `remainingDuration` was added to `cChannel::Result`.

  - `cDatarateChannel` now optionally allows multiple concurrent transmissions, with or without any bookkeeping and associated checks. This is useful for modeling a channel with multiple subchannels or carriers. The operating mode has to be selected programmatically, with the channel's `setMode()` method. Possible modes are `SINGLE`, `MULTI` and `UNCHECKED`.

  - The `forceTransmissionFinishTime()` method of channels has been deprecated. It was always meant as a temporary device to allow implementing aborting frame transmissions, and now with the arrival of the new transmission update API there is no reason to use it any more. Simulations using it should be migrated to the new API.

  - Renamed `setDeliverOnReceptionStart()` to `setDeliverImmediately()`.

  - Added `cSimpleModule::supportsTxUpdates()` flag.

  - `cPacket` now carries a `remainingDuration` field.

  - `cPacket`: eliminated `FL_ISRECEPTIONSTART`; `isReceptionStart()` now uses `remainingDuration` as input; added a similar `isReceptionEnd()` method.

  - `cPacket::str()` overhaul to reflect new fields and uses.

  - In the APIs, send delay and propagation delay, which were sort of combined into a single value, are now distinct values, handled separately.

Simulation kernel / Module deletion:

  - Added the `preDelete()` method to `cComponent`. This is an initially empty virtual function that the user can override to add tasks to be done before the module (or channel) is deleted. When `deleteModule()` is called on a compound module, it first invokes `preDelete()` for each module in the submodule tree, and only starts deleting modules after that. `preDelete()` can help simplify network or module deletion in a complex simulation that involves model change listeners.

  - `cIListener`'s destructor now unsubscribes from all places it was subscribed to. This change was necessitated by the following `deleteModule()` change.

  - `deleteModule()`: Module destruction sequence was changed so that when deleting a compound module, the compound module's local listeners are notified about the deletion of the submodules.

  - `deleteModule()` internals refactored. The motivation was to avoid doing things like firing pre-model-change notifications from a halfway-deleted module. Now we do every potentially risky thing (such as deleting submodules and disconnecting gates) from `doDeleteModule()`, and only delete the module object when it is already barebones (no submodules, gates, listeners, etc). With this change, the deletion sequence is now pretty much the reverse of the setup sequence.

  - Now it is allowed for modules to be deleted (including self-deletion) and created at will during initialization.

Simulation kernel / Expressions, object parameters, JSON values:

  - Under the hood, all expression parsing and evaluation tasks now use the same new generic extensible expression evaluator framework. It is used for NED expressions, object matching, scenario constraint, statistics recording, result selection in the Simulation IDE and in `opp_scavetool`, in `cDynamicExpression` and `cMatchExpression`, etc. In the new framework, expression parsing and the translation of the AST to an evaluator tree are done in separate steps, which makes the library very versatile. Evaluators include support for shortcutting logical and conditional operators, constant folding, and an `undefined` value (anything involving `undefined` will evaluate to `undefined`).

  - `cNedValue` was renamed to `cValue` (compatibility typedef added), as it is now a generic value container used throughout the simulation kernel (`cPar`, `cExpression`, `cClassDescriptor`, etc.), i.e. it is no longer specific to NED.

  - `cValue`'s `doubleValue()` and `intValue()` methods now throw an exception when called on a value that has a measurement unit, in order to reduce usage mistakes. If the value has a unit, call either `doubleValueInUnit()` / `intValueInUnit()`, or `doubleValueRaw()`/`intValueRaw()` plus `getUnit()`.

  - `cValue` changed to hold `any_ptr` (see later) instead of `cObject*`. This change involves several changes, e.g. type `OBJECT` renamed to `POINTER`, and `pointerValue()` added.

  - `cPar`: Added support for object parameters. New type constant: `OBJECT`. New methods: `setObjectValue()`, `objectValue()`, `operator=(cObject*)`, `operator cObject*`.

  - `cPar`: Added `cValue`-based generic access: `getValue()`, `setValue()`.

  - Store origin (file:line) info in `cPar` parameters (more precisely, in `cDynamicExpression`), so we can report it on evaluation errors. Most visible change: `cPar::parse()` gained an extra `FileLine` argument. Also, `cDynamicExpression` now has `get/setSourceLocation()`.

  - Added `cValueArray` and `cValueMap` classes for representing JSON data in NED expressions. A third class is `cValueHolder`, a wrapper around `cValue`, which is only used when a non-object value (double, string, etc) is assigned to a NED parameter of the type `object`. All three classes subclass from `cValueContainer`. Note that behavior of the `dup()` methods of `cValueArray` and `cValueMap` is consistent with that of `cArray` and `cQueue`, i.e. only those objects that are owned by the cloned container are duplicated.

  - NED functions that take or return values of type `object` are now allowed.

  - NED functions can now be defined with the alternative signature:

      cValue f(cExpression::Context *context, cValue argv[], int argc)

    in addition to the existing signature

      cValue f(cComponent *contextComponent, cValue argv[], int argc)

    The `cExpression::Context` argument allows one to access the context component, and also the directory where the ini file entry or the NED file containing the expression occurred (the "base directory"). `Define_NED_Function()` accepts both signatures. The base directory is useful for functions like `xmldoc()` that want to access files relative to the location of the NED expression.

  - `cNedFunction` now allows to search for NED functions by name AND accepted number of args.

  - `cDynamicExpression` has been reimplemented using the new internal `Expression` class, and support for user-defined variables, members, methods, and functions was added. As a consequence, the public interface of the class has significantly changed as well.

  - Added the `cOwnedDynamicExpression` class which holds the result of a NED `expr()` operator. `cOwnedDynamicExpression` is both `cOwnedObject` and `cDynamicExpression` (multiple inheritance). To make this possible, the `cObject` base class was removed from `cExpression`.

  - `cClassDescriptor`: Use exception instead of returning `false` for indicating error. The return types of the following methods changed from `bool` to `void`: `setFieldValueAsString()`, `setFieldArraySize()`, `setFieldStructValuePointer()`.

  - `cClassDescriptor`: Added support for setting pointer members and array sizes via class descriptors. New methods: `setFieldArraySize()`, `setFieldStructValuePointer()`.

  - `cClassDescriptor`: Added `getFieldValue()`/`setFieldValue()` methods to allow accessing fields in a typed way, using `cValue`. Previously existing methods `getFieldValueAsString()`/`setFieldValueAsString()` only allowed string-based access. In MSG files, the `@toValue()` and `@fromValue()` properties can be used to provide code to convert objects or fields to `cValue`.

  - `cClassDescriptor`: Methods changed to use `any_ptr` instead of `void*` for passing the object. (`any_ptr` is a smart pointer class that provides type safety for `void*` pointers.) Pointers need to be put into and extracted from `any_ptr` using the new `toAnyPtr()` / `fromAnyPtr()` functions. They have specialized versions for each type (via templates and overloading). For new types, the message compiler generates `toAnyPtr()`/`fromAnyPtr()` in the header file. For the simulation library classes, these methods come from `sim_std_m.h` (generated from `sim_std.msg`); `sim_std_m.h` is now part of `<omnetpp.h>`.

Simulation kernel / Fingerprints:

  - Due a bugfix in `cHasher::add(const char *)`, fingerprints that involve hashing strings changed their values.

  - The implementation of `cHasher::add(const std::string&)` was changed to be consistent with the `add(const char *)` overload. This may cause fingerprint changes in models that use it.

  - Changed the way fingerprints are computed from figures. Most importantly, fingerprints are now affected by all visual properties, not just geometry information. This change only affects fingerprints that contain the 'f' (=figures) ingredient.

  - The introduction of the new expression evaluation framework also somewhat affects fingerprints. The fact that logical operators and inline-if are now shortcutting may change the fingerprint of some simulations, due to consuming fewer random numbers during expression evaluation.

Simulation kernel / Miscellaneous:

  - The `getModuleByPath()` method was changed to never return `nullptr`, even if an empty string is given as path. Instead, it will throw an exception if the module was not found. This change makes this method consistent with other getter methods in the simulation library, and allows `nullptr` checks to be removed from model code that uses it. A new method, `findModuleByPath()` was added for cases when an optionally existing module needs to be found. These methods, initially defined on `cModule`, have been moved to `cComponent` so that they can be called on channels too.

  - Signature change of the `cVisitor::visit(cObject *obj)` virtual method: it can now request end of iteration via returning `false` (hence, return type changed from `void` to `bool`) instead of throwing `EndTraversalException`. Existing `cVisitor` subclasses in model code will need to be adjusted.

  - `cPar`: Implemented `isMutable()` and the mechanism behind the new `@mutable` property.

  - `cProperty`: Added `getNumKeys()` method; `updateWith()` made public.

  - `cConfiguration`: Removed `getParameterKeyValuePairs()`. Instead, `getKeyValuePairs()` made smarter with an extra `flags` parameter to be able to handle the various use cases.

  - `cMessage`: Allowed `isMessage()` to be called from subclasses.

  - `cEnvir`: New result recording related methods: `recordComponentType()`,
    `recordParameter()`.

  - `cEnvir`: Added `getConnectionLine()`, which returns the coordinates of a connection arrow. This is for certain custom animations.

  - `cEnvir`: Added `pausePoint()`, an animation-related experimental API.

  - Result filters: Two new methods in the `cResultListener` interface: `emitInitialValue()` and `callEmitInitialValue()`.

  - `cResultFilter`, `cResultRecorder`: Grouped `init()` args into a `Context` struct, The old `init()` methods have been preserved as deprecated (and invoked from the new `init()`) in case an existing filter/recorder overrides them. Note that potential external calls to the old `init()` method won't work any more (they will have no effect), and need to be changed to the new version.

  - Added `MergeFilter`, a result filter that allows multiple inputs, and multiplexes them onto its singe output. It is available (as the `merge()` function) in the `source=` part of `@statistic`.

  - Fixed histogram loading issue in the output scalar file (.sca). Bin edges that are very close could become equal in the file if insufficient printing precision was set, rendering the file unreadable. The issue is now handled both during result file writing (if such condition is detected, bin edges are written with full [16-digit] precision) and reading (zero-width bins are merged into adjacent nonzero-width bin).

Simulation kernel / Cleanup:

  - Removed obsolete/deprecated classes and methods. A partial list: `cVarHistogram`, `cLegacyHistogram`, `cLongHistogram`, `cDoubleHistogram`, `cWeightedStdDev`, `cDensityEstBase`; `detailedInfo()` method; `timeval_*()` functions; `cHistogram` methods `setRangeAuto()`, `setRangeAutoLower()`, `setRangeAutoUpper()`, `setNumCells()`, `setCellSize()`; `operator()` of iterator classes in `cArray`/`cModule`/`cQueue`; `cFigure`/`cCanvas` deprecated methods `addFigureAbove()` and `addFigureBelow()`; many `cAbstractHistogram` (ex-`cDensityEstBase`) methods, etc. Instead of the removed `timeval_*()` methods, use `opp_get_monotonic_clock_usecs()` or `opp_get_monotonic_clock_nsecs()`, and perform the arithmetic in `int64_t`.

  - Refactoring: Some classes, methods and variables related to ownership management were renamed: `cDefaultOwner` -> `cSoftOwner`; `defaultOwner` -> `owningContext`, etc.

  - The `setPerformFinalGC()` method was removed. It was meant for internal use, and pretty much unused by model code.

  - Removed the support for OMNeT++ 4.x fingerprints (`USE_OMNETPP4x_FINGERPRINTS`).

  - `WITH_OMNETPP4x_LISTENER_SUPPORT` was removed.

  - Source code modernization: use in-class member initializers wherever possible. The source code now requires a C++14 compiler.

  - Internal classes, global variables, etc moved into the `omnetpp::internal` namespace.

Runtime:

  - Accept expressions as value for (most) config options. For options that accept values both with and without quotes (types STRING, FILENAME, FILENAMES, PATH), a heuristic decides whether a string is to be taken literally or to be evaluated as an expression. Expressions may also use NED operators, module parameters and other NED expression features. For example, it is possible to use the module vector index in the value of the `display-name` option: `**.app[0..3].display-name = "cbr-" + string(index)`

  - Allow parameter values to be specified on the command line. For example, `--**.mss=512` is equivalent to inserting the `**.mss=512` line near the top of the configuration in `omnetpp.ini`.

  - Do not complain about missing `omnetpp.ini` if a `--network` command-line option is present.

  - There were several improvements related to the NED path. The `NEDPATH` environment variable has been renamed to `OMNETPP_NED_PATH`, but the old one is still recognized for backward compatibility. Multiple `-n` options are now accepted. Also, the `NEDPATH` environment variable used to be ignored when a `-n` option was present, no longer so. Excluding packages from NED file loading has also been implemented. NED exclusions can be specified in multiple ways: the `-x <packagelist>` command-line option, the `OMNETPP_NED_PACKAGE_EXCLUSIONS` environment variable, and the `ned-package-exclusions` configuration option (they accept semicolon- separated lists).

  - Change in NED loading: Now, if a package has files in several distinct source trees, only one of them may contain a `package.ned` file.

  - There were also several improvements related the image path. The `-i <imgpath>` option command-line option has been added. It may occur multiple times. The image path is now defined jointly by the `OMNETPP_IMAGE_PATH` environment variable, `-i` command-line options, and the `image-path` configuration option. `./bitmaps` was removed from default image path, as it was virtually unused. The default value `./images` now comes from the default value of the image-path configuration option.

  - Added support for embedding NED files into a binary (an executable or library). The `cpp` subcommand of `opp_nedtool` generates C++ code that contains the content of NED files as string constants, which can then be compiled into the simulation binary. When the simulation program is started, these embedded NED files will be loaded automatically, and the original NED files will no longer be needed for running simulations. Optional garbling, which prevents NED source code from being directly readable inside the compiled binaries, is also available. A `makefrag` file can be used to integrate the NED-to-C++ translation into the build process. To see an example `makefrag` file, view the `makefrag` help topic in `opp_nedtool`.

  - Several global config options were changed to be per-run: `scheduler-class`, `debug-on-errors`, `print-undisposed`, `fingerprintcalculator-class`.

  - Added the `parsim-num-partitions` config option, which makes it possible to explicitly declare the number of partitions to be used with parallel simulation. (Before, it was explicitly taken by OMNeT++ from MPI or the respective communication layer.)

  - Added the `config-recording` configuration option, which controls the amount of configuration options to save into result files.

  - Allow recording actual module/channel parameter values into the output scalar file, via the new `param-recording` per-object boolean configuration option. Note that parameters will be recorded as "parameter" result items, not as scalars. For volatile parameters, the expression itself will be recorded (e.g. `exponential(0.5)`), not any particular value drawn from it.

  - Added support for result directory subdivision: the `resultdir-subdivision` boolean configuration option and the `${iterationvarsd}` variable. The motivation is that the performance of various file managers (including Eclipse's Project Explorer) tends to degrade severely if there are more than a few thousand files in a single directory. In OMNeT++, this problem occurs when a parameter study produces a large number of result files. The common workaround is to "hash" the files into a subdirectory tree. (For example, git also uses this technique to store the contents of `.git/objects` dir).

    In OMNeT++, such feature can be turned on by setting `resultdir-subdivision=` `true` in the configuration. Since it is natural to use the iteration variables to define the directory hierarchy, directory subdivision makes use of the new `${iterationvarsd}` configuration variable. This variable is similar to `${iterationvarsf}` but it contains slashes instead of commas, which makes it suitable for creating a directory tree. Enabling directory subdivision will cause `/${configname}/${iterationvarsd}` to be appended to the name of the results directory (see `result-dir` config option), causing result files to be created in a subdirectory tree under the results directory.

    If you want to set up the directory hierarchy in a different way, you can do so by setting the `result-dir` config option and appending various variables to the value. E.g.: `result-dir = "results/${repetition}"`

  - Added the `${datetimef}` inifile variable, which contains the current timestamp in a form usable as part of a file name.

  - Eventlog recording: Implemented snapshot and incremental index support to increase performance and scalability. This introduces significant (breaking) changes in the `elog` file format, and adds a set of associated configuration options; see the Sequence Chart section below for details.

  - Added `%<` (trim preceding whitespace) to the list of accepted directives for log prefixes.

  - The obsolete command-line options `-x`, `-X`, `-g`, and `-G` were removed.

  - Made the `-q` option more permissive: if `-c` is missing, assume `General`

  - Added the `-e` option, which prints the value of the given configuration option.

  - `opp_run -v` output now includes the system architecture OMNeT++ was built for (e.g. `ARCH_X86_64` or `ARCH_AARCH64`).

  - `-h resultfilters` and `-h resultrecorders` now include the descriptions in the list of result filters and recorders.

  - Added two new `-h` topics: `latexconfigvars`, `sqliteschema`. They are mainly used for producing info for the Appendices in the manual.

  - In order to reduce OMNeT++'s external dependencies, we now use an embedded copy of Yxml as the default XML parser. Yxml (https://dev.yorhel.nl/yxml) is a small and fast SAX-style XML parser with a liberal license.

    Support for using LibXML2 remained in place, but very few users will actually need it. Expat support has been removed. Note that no significant functionality is lost when Yxml is used instead of LibXML2. LibXML2 is only needed for Schema or DTD-based validation (and possibly, default value completion), which virtually no simulation models require. Also note that Yxml-based parsing scales much better than LibXML2, both performance-wise and regarding memory usage.

  - The bundled SQLite sources were upgraded to version 3.30.1.

Statistics recording:

  - Added the `warmup` filter. This filter discards values during the warm-up period, and is automatically inserted at the front of the result filter chains when the `warmup-period` config option is present.

  - Added the `autoWarmupFilter` statistic attribute that allows one to disable auto-adding the `warmup` filter to a statistic. Example:

      `@statistic[foo](record=vector;autoWarmupFilter=false);`

    This will cause all values from the `foo` signal to be recorded, even values emitted during the warm-up period (if one is set).

    However, the real motivation behind this feature is to allow the user to add the warm-up filter at a non-default location in the filter chain, because the default location is not always correct.

    For example, `@statistic[foo](source=min;record=vector)` is equivalent to `@statistic[foo](source=min(warmup(foo));autoWarmupFilter=false;record=vector)`, and records (as vector) the minimum of the values which were emitted after the warmup period is over. In contrast, if we replace `min(warmup(foo))` with `warmup(min(foo))`, it will compute the minimum of ALL values, but only starts recording the result after the warmup period has expired.

    This is a crucial difference sometimes. For example, a statistic might record queue length computed as the difference of the number of messages that entered the queue and those that left it:

      `@statistic[queueLen](source=count(pkIn)-count(pkOut);record=vector); //INCORRECT`

    In this case, if a warmup period is set, the result may even go negative because the `pkIn` and `pkOut` signals that arrive during the warmup period are ignored, and if `pkOut` arrives after that, we are at -1. The correct solution is to add the `warmup` filter after the difference between arrivals and departures have been computed:

      `@statistic[queueLen](source=warmup(count(pkIn)-count(pkOut));autoWarmupFilter=false;record=vector); //OK`

    The `autoWarmupFilter` option exists because either location for the warmup filter (beginning or end of the chain, or even mid-chain) may make sense for certain statistics. The model author needs to decide per-statistic which one is correct.

  - Added the `demux` filter, which allows splitting the stream of values arriving from a signal to multiple streams. For example, if values from a `foo` signal are tagged with the labels `first`, `second` and `third`, then the following statistics:

      `@statistic[foo](record=count(demux(foo)));`

    will produce three scalars: `first:foo:count(demux)`, `second:foo:count(demux)`, and `third:foo:count(demux)`. The labels are taken from the (full) name of the details object specified in the `emit()` call.

    This filter is especially useful if you intend to save multiple instances of the same statistics from the same module (e.g. per-connection TCP statistics).

  - In result files, the `,vector` suffix is now suppressed in the titles of vector results (similarly also `,histogram` and `,stats`), as they simply echo the result item's type. (They were there in the first place because recording mode is automatically appended to result titles `@statistic(title=...)` after a comma; it is now suppressed for the mentioned recording modes.)

  - Added the `merge` filter which multiplexes several inputs onto a single output. It is available in the `source=` part of `@statistic`.

  - Result filters: The `count` and `sum` filters now record the initial zero value as well.

  - Result files now include two new result attributes for each item: `recordingmode`, which is the item in the `@statistic(record=...)` list that produced the given result item, and `moduledisplaypath`, the module/channel path that contains display names instead of the normal names where available.

  - `sumPerDuration` filter: fix computation when invoked during warmup period.

Cmdenv:

  - Added the `cmdenv-fake-gui` boolean configuration option, which enables "fake GUI" functionality during simulation. "Fake GUI" means that `refreshDisplay()` is periodically invoked during simulation, in order to mimic the behavior of a graphical user interface like Qtenv. It is useful for batch testing of simulation models that contain visualization. Several further configuration options exist for controlling the details: `cmdenv-fake-gui-after-event-probability`, `cmdenv-fake-gui-before-event-probability`, `cmdenv-fake-gui-on-hold-numsteps`, `cmdenv-fake-gui-on-hold-probability`, `cmdenv-fake-gui-on-simtime-numsteps`, `cmdenv-fake-gui-on-simtime-probability`, `cmdenv-fake-gui-seed`.

Qtenv:

  - Modernized look: Material-style SVG-based icons, HIDPI support, dark theme support.

  - New actions on the main toolbar: "Debug next event", "Debug on errors", "Show animation parameters". ("Load NED file" was removed from the toolbar as it was rarely needed, but it's still available from the menu.)

  - The default digit separator used in the main simulation time and event number displays was changed to space.

  - A lot of effort was made to refine packet animation, also with regard to the new "transmission updates" API. For example, the animation filter now affects deliveries as well, and transmissions on ideal channels are now shown as a full-length line. Transmission updates are drawn as "notches" on the message line.

  - Turned off animating method calls by default. The rationale is that method call animations usually expose low-level (C++ implementation level) details on the GUI, which are rarely of interest to a casual user.

  - Added the possibility to disable method call visualization locally (for that module type) via the context menu, even when the global switch in the Preferences dialog is turned on.

  - Added support for showing a submodule's display name under the icon instead of, or in addition to, the normal name. The format can be selected in the context menu.

  - The view mode (grouped/flat/inheritance/children) in the Object Inspector used to be tied to the type (class) of the object displayed in the inspector. Since that resulted in too much mode switching while the user navigated the object tree, and the switching logic was not easily discoverable by the user, we removed the feature of per-type remembering of view modes. The view mode now only changes when the user explicitly switches in on the UI.

  - In the Object Inspector, added the possibility to select display mode (Children/Grouped/Flat/Inheritance) per-node. If display mode override is specified for a node, it will affect the whole subtree. The display mode for the selected tree can be changed via hotkey (Ctrl+B) or via the context menu.

  - The "Set Sending Time as Reference" option was added to the messages view context menu. This option makes it possible to set a reference time, and display all other times as a delta compared to it.

  - In the messages view, there was a slight change in the notation used for the source and destination modules of the message, in order to make it unambiguous. Use explicit "." and "^" to indicate the location of the module. Also, it now uses arrows of uniform lengths everywhere.

  - The "Allow Backward Arrows for Hops" option was added to the messages view context menu. When this option is enabled, it allows the use of backward arrows to ensure a consistent relative ordering of modules in the log. For example, if two modules A and B exchange messages, this option will cause the window to display them as "A-->B" and "A<--B", as opposed to the default "A-->B" and "B-->A". This sometimes makes the log easier to follow.

  - Added support for new columns in the packet log view: TxUpdate, Durations, Length, Info. This was implemented in the `cDefaultMessagePrinter` class which is part of the simulation library. This change makes it possible to see if a packet is a transmission update (except if the simulation installs its own packet printer like INET does).

  - In the messages view, simulation time is now formatted with digit grouping on.

  - In the log, the banners of component init stages that do not print anything are now suppressed.

  - Added the "Fira Code" font as embedded resource, and set it to be used by default for the log window. The reason is that it provides nice "-->" arrow ligatures in the Messages view.

  - Prevent manually enabled "Debug on Errors" setting from being turned off by (lack of) configuration option.

  - Fix osgEarth viewpoints ignoring SRS (PR #851).

  - Countless small improvements and bug fixes.

  - Raised the minimum required Qt version to 5.9.

Tkenv:

  - Tkenv has been removed. Use Qtenv for all simulations.

Tools:

  - The names of all command-line tools now begin with `opp_`.

  - Added `opp_ide`, an alternative to the `omnetpp` command that launches the Simulation IDE.

  - Added `opp_neddoc`, a tool that makes it possible to generate HTML documentation from NED files from the command line. `opp_neddoc` works by launching the IDE in headless mode.

  - `opp_nedtool` was rewritten for convenience and features. The main points are the following:
      - Removed msgc-related functionality, now it is really just about NED.
      - Command-line interface redesigned for better usability (subcommands, better help, etc.)
      - Added support for generating C++ source (`cpp` subcommand), for embedding NED files into an executable or library.
      - The tool now accepts directories as command-line arguments, too, and processes all NED files in them.
      - More convenient output (no `*_n.ned`, `*_n.xml`).
      - Removed obsolete `@listfile` and `@@listfile` support.
      - Fixed bugs in splitting NED files to one NED type per file.

  - `opp_msgtool` was rewritten in the style of `opp_nedtool`. Details:
      - Made `--msg6` the default mode, `--msg4` was removed.
      - The command-line interface now uses subcommands; if no subcommand is specified, the default action is C++ code generation. The rarely useful `-T` (type of next file) and `-h` (here) options have been removed.
      - Better help. A `builtindefs` help page has also been added, which prints the built-in declarations.

  - `opp_scavetool` had several improvements. A brief summary:
      - The tool now accepts directories as command-line arguments, too, and recursively loads scalar and vector files from them.
      - Command-line arguments may now contain double asterisk wildcards (`**`) in addition to normal wildcards (`*`,`?`). Double asterisks can match any number of directory levels. Example: `results/**.vec` matches all `.vec` files anywhere under the `results/` directory. (When specifying wildcard arguments on the command line, be sure to quote them so that the shell does not expand them before `opp_scavetool` gets invoked.)
      - Added options to limit vector data by simulation time: `--start-time <time>`, `--end-time <time>`.
      - Accepts exporter options in the `--<key>=<value>` notation as well, not only with `-x`.
      - In filter expressions, run attributes now have to be referred to with the `runattr:` prefix. The `attr:` prefix now only matches result attributes.
      - In exporters, the way of representing histograms in the output has slightly changed: underflow/overflow values are now saved separately instead of as underflow/overflow "bins". Thus, there are now one more bin edges than bin values.
      - The possibility to apply vector operations to vector results was removed from `opp_scavetool` and exporters. The recommended way of processing output vectors is with Python and NumPy.
      - The `help` subcommand was refined.

  - Added `opp_charttool`, a tool for "running" ANF files on the command line, e.g. for image or data export. Filtering options (`-i`, `-n`) allow multiple charts to be selected for exporting. The `templates` subcommand lists the available chart templates.

  - `opp_featuretool`: Extensive refactoring as well as refinement of its operation and command-line options. In general, the tool has become more forgiving: a missing `.featurestates` file or extra/missing entries in it no longer causes an error or warning. Missing file and file entries are initialized with the default enablement state; extra entries are preserved (in the hope they'll be useful, e.g. after switching to a different topic branch). The content of the `.nedexclusions` file is now also automatically adjusted without stopping with an error. User-visible changes are the following:
      - Removed the `validate` and `repair` subcommands (all commands implicitly validate and repair now).
      - Better validation of the `.oppfeatures` file: Detect and report duplicate feature IDs and unresolved feature dependencies.
      - Check for the existence of the C++ source root and NED folders.
      - Preserve unknown features in the `.featurestate` file instead of fail/warn about them. Reason: they often occur when switching between branches.
      - `isenabled -v` prints list of disabled features to stdout, not to stderr
      - Complain less about fixable problems.
      - Non-verbose operation by default.
      - Improved error/warning messages.

  - `opp_makemake`: The `makefrag` file is now included instead of copied into the generated makefile. Also, a `help` target has been added to the generated makefiles: typing `make help` now prints the list of accepted targets (`all`, `clean`, etc.) and user-settable makefile variables (`MODE`, `V`, `CFLAGS`, etc.), complete with helpful descriptions and usage examples.

  - `opp_test`: The set of possible test outcomes has been refined. Possible outcomes are now `PASS`, `FAIL`, `ERROR`, `SKIP` and `EXPECTEDFAIL`. `SKIP` means that the test was not performed, because e.g. it would test an optional feature which is currently disabled; `ERROR` means that the test was not performed because of an unrecoverable error (i.e. crash or exception). The diagnostic output for failed tests is now a colorized diff instead of a simple printout of the expected and the actual contents. It is possible to designate a test case as an "expected failure", by specifying `%expected-failure: <reason>` in the test file. This is useful if we know that a certain test case is correct, but it is not possible to fix it for some reason.

IDE / Analysis Tool:

  - The Analysis Tool in the IDE has been rewritten, using a new approach. The number one goals were to allow ARBITRARY computations to be performed on the data, VISUALIZE the result in the most appropriate form chosen from a multitude of chart/plot types, and produce PUBLICATION QUALITY output. Additional goals were an intuitive user interface that makes straightforward tasks easy and convenient, scalability that allows the processing of a large amount of simulation results inside the IDE, and facilities to make (most of) the UI's functionality available on the command line as well.

    The key technologies the new Analysis Tool employs are Python, NumPy, Pandas, and Matplotlib. The use of Python and Pandas gives access to advanced data manipulation and analysis capabilities, while Matplotlib offers dozens of plot types out of the box, endless customizations, and proven high-quality output. A large effort was made to properly integrate these technologies into the IDE, so that using the Analysis Tool provides a superior user experience, and brings the power of Python close to users without necessarily requiring them to write code in Python.

    Common charting tasks are accessible with a few mouse clicks, via predefined Chart Templates that can be configured using dialogs. The heart of all Chart Templates is Python code that performs all of the computation and plotting. The Python code behind a chart can be viewed and freely edited using an integrated language-aware editor. Matplotlib plots appear inside the Analysis Tool's UI, and they are live (i.e. not just images): zooming, panning, and interacting with possible embedded controls in the plot all work seamlessly. IDE-native plots can also be chosen as an alternative to Matplotlib, as they provide increased scalability at the cost of being less flexible and featureful.

    Special attention was paid to scalability. The UI is now easily capable of
    dealing with at least several tens of thousands of result files, and several
    million result items.

    The OMNeT++ Python API for querying, transforming and plotting simulation results is available in the form of Python packages for use in Python scripts outside the IDE as well. There is also a command-line tool (`opp_charttool`) that can "run" Analysis (anf) files e.g. for image export. `opp_charttool` can be integrated e.g. into the build process of a paper written in LaTeX.

    The documentation of the Analysis Tool's user interface can be found in the User Guide. A reference of the Python API has a dedicated Appendix in the Simulation Manual, while the "Result Recording and Analysis" chapter in the same Manual provides some practical guidance on how to use the API.

IDE / Sequence Chart Tool:

  - The visualization of events have been horizontally expanded on the chart, now it uses long rounded rectangles instead of small circles. This change allows the visualization of method calls without their arrows overlapping. If method calls are hidden then the chart falls back to the old behavior where events are small circles.

  - The set of displayed axes can be explicitly configured independently from the set of displayed events. The chart now also supports displaying events on a compound module axis if the corresponding submodule axis is hidden. Collapsing and expanding module axes can be done using the mouse.

  - The chart now also displays a set of axis headers on the left side that shows the module hierarchy in a compact and interactive way.

  - All parts of the sequence chart can be independently shown/hidden.

  - The selection has been extended with support for axes and method call arrows.

  - A new time measurement feature has been added that allows measuring the time difference between multiple selected events and other interesting points in time.

  - Several automatic configuration presets have been added (e.g. network level, full detail, default) to make the configuration for the most common use cases easier.

  - A new pattern-matching-based user-defined colorization (e.g. message sends, method calls, axes, events) feature has been added. This feature allows creating more easily understandable sequence charts by encoding certain model properties as colors.

  - The chart can display diagnostics information such as eventlog file statistics and operation statistics.

  - Added support for SVG export.

IDE / NED Documentation Generator:

  - The documentation generator now allows you to inject your own documentation fragments (possibly generated by 3rd-party tools) at various points in the documentation.

  - Modern look for the generated pages (Material design).

  - It is now possible to specify excluded directories during documentation generation, in order to be able to exclude samples, showcases etc. that are not integral part of the documentation.

  - Now generates `nedtags.xml` and `msgtags.xml` files which are compatible with the `doxytags.xml` file that Doxygen generates. This allows crosslinking to the NED documentation from a Sphinx-based documentation, using the doxy-link plugin.

  - Generate usage and inheritance diagrams only if they are meaningful.

  - Use SVG files as the usage/inheritance diagram image. Removed map files as SVG already contains proper links.

  - Generate "Implementors" table for interfaces. The Implementors table includes indirect implementors as well.

  - Fix: Inheritance diagrams were missing in module interface pages, even if they had implementors. This bug made INET documentation much less useful.

  - The functionality of the documentation generator is now also available from the command line (`opp_neddoc`).

IDE / Inifile Editor:

  - Added support for ignorable options. User can mark custom config options as ones to be ignored by the editor.

  - Config option values that contain expressions are no longer falsely flagged as errors.

  - The confusing "Unused entry (does not match any parameters)" warning that often appeared falsely for INET simulations due to deficiencies in analyzing the network structure has been re-worded to include the possibility that it is false, and has also been demoted from being "warning" to "info".

  - Eliminated the asynchronous/delayed form page reread in Form mode, which often caused editing glitches.

IDE Miscellaneous:

  - Install Simulation Models dialog: Added support for changing the project name and location on download.

  - Turned on showing the line numbers ruler in source editors by default.

  - Added an XML editor, Terminal (a view that contains a shell prompt), PyDev (Python editor), and the Eclipse Marketplace client to the IDE.

  - The simulation launcher now passes NED exclusions to the simulation program as `-x` options. NED exclusions typically come from disabled project features, e.g. in INET.

  - Added SVG export capability to the NED Editor.

  - Updated to Eclipse 4.22, CDT 10.5 and PyDev 9.2.

  - The JRE is now provided by the Eclipse JustJ project, so the IDE no longer requires Java to be installed on the host OS.

  - Added support for ARM-based Linux distros.


C++ Debugging:

  - Better debugging experience by fine-tuning the compiler options, especially with the clang compiler.

  - Added LLDB pretty printers for various OMNeT++ types. They can be useful if you use LLDB-based external debuggers like XCode or VS Code. You should manually import them from the LLDB debugger console with the following command:

      command script import <OMNETPP_ROOT>/python/omnetpp/lldb/formatters/omnetpp.py

  - Allow Qtenv's "Debug on Error" and "Debug Next Event" functionality to use the integrated debugger of the Simulation IDE. This required multiple changes. First, the IDE was extended to accept an URL which, when opened in the IDE, causes the integrated debugger to start a debug session and attach to a process given with its PID. This URL is:

    `omnetpp://cdt/debugger/attach?pid=<pid>`

    The URL can also be opened from the command line, by running the `omnetpp` command with the URL as argument. The command opens the URL in the existing IDE instance if it is already running, or starts a new one if it does not.

    The second change was to update the default value of the OMNeT++ `debugger-attach-command` configuration option to use the above command. (Previously it has used various other debuggers which were likely to be found on the host OS: GDB, Nemiver, VS Code, etc.)

    Hint: You can force Qtenv to use Visual Studio Code as your external debugger by setting the following environment variable:

      export OMNETPP_DEBUGGER_COMMAND="code --open-url \"vscode://vadimcn.vscode-lldb/launch/config?{request:'attach', pid:'%u'}\""

    Alternatively, you can specify the same value (`code --open-url ...`) to the "debugger-attach-command" configuration option in `omnetpp.ini`.

Result file format changes:

  - There were a number of changes in the format of result files (.sca and .vec files), so the file format version has been bumped from 2 to 3. (The file format is recorded at the top of each result file in a `version` line.) The set of changes is relatively small; see details below.

  - Iteration variables are no longer saved as run attributes but in separate `itervar` lines, in order to be able to tell them apart with certainty.

  - `param` lines (which recorded parameter assignment entries in the configuration) have been replaced with the more general `config` lines. `config` lines record all configuration entries, not just parameter assignments.

  - In version 2, concrete parameter values (if requested) were recorded as scalars, whereas in version 3 they are recorded as a separate result item type, in `par` lines. This allows the recording of volatile parameters (as expressions) and non-numeric values as well.

  - Component NED type names are now recorded, as `typename` pseudo parameters.

  - The `sum` and `sqrsum` fields of weighted statistics are no longer recorded, as they are irrelevant.

SQLite result file format changes:

  - SQLite result files underwent a similar change in the database schema. Files in the old format are no longer understood. A brief summary of changes follows.

  - The `runParam` table was renamed to `runConfig`, and its columns were also similarly renamed: `paramKey`, `paramValue` and `paramOrder` became `configKey`, `configValue` and `configOrder`, respectively.

  - Iteration variables are in a new table called `runItervar`, with columns `runId`, `itervarName` and `itervarValue`.

  - Two new tables were added for representing the new result item types: `parameter` (with the columns `paramId`, `runId`, `moduleName`, `paramName` and `paramValue`), and `paramAttr` (with the columns `paramId`, `attrName` and `attrValue`). Note that `paramValue` contains values in the syntax it would have to be written in a NED file or in `omnetpp.ini`, i.e. string constants include the quotation marks.

Eventlog file format and configuration:

  - The eventlog file format has been changed substantially. You won't be able to open older eventlog files with this version of the IDE, nor new files in previous IDE versions.

  - The most prominent change is the introduction of snapshot and index chunks in addition to the already recorded event chunks. The former contains a complete snapshot of the relevant simulation state without referring to any other line in the file. The latter provides the set of changes in the relevant simulation state since the last index or snapshot chunk. The contents of the index chunk is expressed as references to other lines of the eventlog file. This change in the eventlog format provides the foundation for a better user experience in the IDE especially for large eventlog files.

  - Several new eventlog-recording-related configuration options have been added:

    - The `eventlog-snapshot-frequency` configuration option specifies how often snapshots are recorded in the eventlog file. Snapshots help various tools to handle large eventlog files more efficiently. Specifying greater value means less help, while smaller value means bigger eventlog files.

    - The `eventlog-index-frequency` configuration option specifies how often indices are recorder in the eventlog file. An index is much smaller than a full snapshot, but it only contains the differences since the last index. Specifying greater value means less help, while smaller value means bigger eventlog files.

    - The `eventlog-max-size` configuration option specifies the maximum size of the eventlog file in bytes. The eventlog file is automatically truncated when this limit is reached.

    - The `eventlog-min-truncated-size` configuration options specifies the minimum size of the eventlog file in bytes after the file is truncated. Truncation means older events are discarded while newer ones are kept.

    - The `eventlog-options` configuration option allows for recording only certain categories (e.g. text, message, module, method call, display string), to reduce the eventlog file size.

Build environment:

  - If a file called `setenv_local` is present in the installation root, it is automatically sourced from the `setenv` script. `setenv_local` can be used to set up user specific environment variables.

  - The content of the `$PLATFORM` makefile variable has changed. Until now, it contained a `<platform>.<architecture>` pair (e.g. `win32.x86_64`) which was misleading. From now on, `Makefile.inc` provides `$PLATFORM` (e.g. `win32`) and `$ARCH` (e.g. `x86_64`) as separate variables. `makefrag` files must be updated if `$PLATFORM` variable was used in them.

  - To build OMNeT++ with Akaroa support, `WITH_AKAROA=yes` must be specified in the `configure.user` file. The configuration script no longer tries to autodetect Akaroa without the user explicitly requesting it.

Windows Support:

  - On Windows, the toolchain (compiler, libraries, etc.) can now be found in `tools/win32.x86_64`.

  - Setting of environment variables has been moved into the `setenv` script. It is automatically sourced when `mingwenv.cmd` or `vcenv.cmd` is started.

  - On Windows, libraries and other dependencies used by simulations, such as Qt, OSG, osgEarth, etc., are now separated from the rest of the toolchain binaries (compiler, etc.). You can find them in the `opt` subdirectory of the `tools/win32.x86_64` folder.

  - On Windows, when creating a shared library, the build system now generates a proper import library and also a module definition file (`<targetname>.dll.a`, `<targetname>.def`). External projects must link against that library instead of linking directly with the `<targetname>.dll` file. (Note that the linker automatically uses the `.dll.a` files if they are present in the library path and falls back to use the pure `.dll` file only if the import library is not present. It means that you don't have to do anything extra to use these files apart from making them available along your `.dll` files.)

  - The Windows toolchain now contains the (much faster) LLD linker. LLD is also used on other platforms instead of the GNU linker, if available. For large projects, LLD is dramatically faster than the standard GNU linker. In one instance, the use of LLD, together with other changes that were partly done in INET, reduced INET linking time on Windows from 380s to 2s in DEBUG mode, and from 90s to 5s in RELEASE mode on one of our dev boxes.


macOS support:

  - On macOS, the toolchain (compiler, libraries, etc.) can now be found in `tools/macos.x86_64`.

  - Rudimentary support for Apple silicon: `source setenv` now automatically launches a (nested) x86_64 emulation shell on ARM-based Macs.

Sample simulations:

  - Added "openstreetmap", an example simulation that displays a city map, with some animated car traffic on it. Map data come from an OSM file exported from openstreetmap.org.

  - Added "petrinets", an example simulation demonstrating how to implement stochastic Petri nets, a well-known formalism for the description of distributed systems.

  - Added the "wiredphy" sample simulation to demonstrate transmission updates.

  - The "fifo" example simulation was extended with a TandemQueues network and related configurations in omnetpp.ini.

  - The "resultfiles" folder project now contains scalar and vector result files from the Fifo1, Fifo2, and the new TandemQueues simulations.

Documentation:

  - Simulation Manual has been updated. The largest changes were in the "Message Descriptions" and "Result Analysis" sections (which were practically rewritten). There are also two new appendices: "Python API for Chart Scripts", and "Message Class/Field Properties".

  - User Guide: The chapter about the Analysis Tool ("Result Analysis") has been rewritten.

  - Install Guide has been updated. New sections in the macOS chapter: "Enabling Development Mode in the Terminal", "Running OMNeT++ on Apple Silicon". We also added instructions on installing OMNeT++ on WSL (Windows Subsystem for Linux) on Windows 10, which is important because we found that running OMNeT++ on WSL is considerably faster than using it with the MinGW toolchain.

  - Converted all of our structured documents (User Guide, Install Guide, etc.) from their original source format (DocBook or AsciiDoc) to reStructuredText, except for the Simulation Manual which remains in LaTeX.

[Download it here](/download/preview).