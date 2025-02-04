# XCGLogger
##### By: Dave Wood
- Cerebral Gardens http://www.cerebralgardens.com/
- Twitter: [@DaveWoodX](https://twitter.com/DaveWoodX)

### tl;dr
A debug log module for use in Swift projects. Allows you to log details to the console (and optionally a file), just like you would have with NSLog or println, but with additional information, such as the date, function name, filename and line number.

Go from this:

```Simple message```

to this:

```2014-06-09 06:44:43.600 [Debug] [AppDelegate.swift:40] application(_:didFinishLaunchingWithOptions:): Simple message```

### Compatibility

XCGLogger works in both iOS and OS X projects. It is a Swift library intended for use in Swift projects.

Swift does away with the C preprocessor, which kills the ability to use ```#define``` macros. This means our traditional way of generating nice debug logs is dead. Resorting to just plain old ```println``` calls means you lose a lot of helpful information, or requires you to type a lot more code.

Use XCGLogger version 3.x for Swift 2.0, XCGLogger version 2.x for Swift 1.2, and XCGLogger version 1.x for Swift 1.1 and below.

### Communication _(Hat Tip AlamoFire)_

* If you need help, use Stack Overflow. (Tag '[xcglogger](http://stackoverflow.com/questions/tagged/xcglogger)')
* If you'd like to ask a general question, use [Stack Overflow](http://stackoverflow.com/questions/tagged/xcglogger).
* If you found a bug, open an issue.
* If you have a feature request, open an issue.
* If you want to contribute, submit a pull request.

### How to Use

Add the XCGLogger project as a subproject to your project, and add either the iOS or OS X library as a dependancy of your target(s).
Under the General tab of your target, add the XCGLogger.framework to the Embedded Binaries.

Then, in each source file:

```Swift
import XCGLogger
```

In your AppDelegate, declare a global constant to the default XCGLogger instance.

```Swift
let log = XCGLogger.defaultInstance()
```

**Note**: previously this was ```XCGLogger.sharedInstance()```, but it was changed to better reflect that you can create multiple instances.

In the
```Swift
application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) // iOS
```

or

```Swift
applicationDidFinishLaunching(aNotification: NSNotification) // OS X
```

function, configure the options you need:

```Swift
log.setup(.Debug, showThreadName: true, showLogLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: "path/to/file", fileLogLevel: .Debug)
```

The value for ```writeToFile:``` can be a ```String``` or ```NSURL```. If the file already exists, it will be cleared before we use it. Omit a value or set it to ```nil``` to log to the console only. You can optionally set a different log level for the file output using the ```fileLogLevel``` parameter. Set it to ```nil``` or omit it to use the same log level as the console.

Then, whenever you'd like to log something, use one of the convenience methods:

```Swift
log.verbose("A verbose message, usually useful when working on a specific problem")
log.debug("A debug message")
log.info("An info message, probably useful to power users looking in console.app")
log.warning("A warning message, may indicate a possible error")
log.error("An error occurred, but it's recoverable, just info about what happened")
log.severe("A severe error occurred, we are likely about to crash now")
```

The different methods set the log level of the message. XCGLogger will only print messages with a log level that is >= its current log level setting.

### Advanced Use

It's possible to create multiple instances of XCGLogger with different options. For example, you only want to log a specific section of your app to a file, perhaps to diagnose a specific issue a user is seeing. In that case, create alternate instances like this:

```Swift
let fileLog = XCGLogger()
fileLog.setup(.None, showThreadName: true, showLogLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: "path/to/file", fileLogLevel: .Debug)
fileLog.info("Have a second instance for special use")
```

You can create alternate log destinations (besides the two built in ones for the console  and a file). Your custom log destination must implement the ```XCGLogDestinationProtocol``` protocol. Instantiate your object, configure it, and then add it to the ```XCGLogger``` object with ```addLogDestination```. Take a look at ```XCGConsoleLogDestination``` and ```XCGFileLogDestination``` for examples.

Each log destination can have its own log level. Setting the log level on the log object itself will pass that level to each destination. Then set the destinations that need to be different.

##### Selectively Executing Code

As of version 1.9, all log methods operate on closures. Using the same syntactic sugar as Swift's ```assert()``` function, this approach ensures we don't waste resources building log messages that won't be printed anyway, while at the same time preserving a clean call site.

For example, the following log statement won't waste resources if the debug log level is suppressed:

```Swift
log.debug("The description of \(thisObject) is really expensive to create")
```

Similarly, let's say you have to iterate through a loop in order to do some calculation before logging the result. In Objective-C, you could put that code block between ```#if``` ```#endif```, and prevent the code from running. But in Swift, previously you would need to still process that loop, wasting resources. With ```XCGLogger``` it's as simple as:

```Swift
log.debug {
    var total = 0.0
    for receipt in receipts {
        total += receipt.total
    }

    return "Total of all receipts: \(total)"
}
```

In cases where you wish to selectively execute code without generating a log line, you can use the methods: ```verboseExec```, ```debugExec```, ```infoExec```, ```warningExec```, ```errorExec```, and ```severeExec```.

##### Custom Date Formats

As of version 2.0, you can create your own NSDateFormatter object and assign it to the logger.

```Swift
let dateFormatter = NSDateFormatter()
dateFormatter.dateFormat = "MM/dd/yyyy hh:mma"
dateFormatter.locale = NSLocale.currentLocale()
log.dateFormatter = dateFormatter
```

##### Enhancing log messages with colour

XCGLogger now supports the XcodeColors plug-in (https://github.com/robbiehanson/XcodeColors). Once installed, each log level will have its own colour. These colours can be customized as desired. See the sample projects for examples. If using multiple logger's, you could alternatively set each logger to its own colour.

##### Initialization using Closure

Alternatively you can use a closure to initialize your global variable, so that all initialization is done in one place
```Swift
let log: XCGLogger = {
    let log = XCGLogger.defaultInstance()
    log.setup(.Debug, showThreadName: true, showLogLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: nil, fileLogLevel: .Debug)
    
    let dateFormatter = NSDateFormatter()
    dateFormatter.dateFormat = "MM/dd/yyyy hh:mma"
    dateFormatter.locale = NSLocale.currentLocale()
    log.dateFormatter = dateFormatter
    
    return log
}()
```

##### Alternate Configurations

By using Swift build flags, different log levels can be used in debugging versus staging/production.
Go to Build settings -> Swift Compiler - Custom Flags -> Other Swift Flags and add ```-DDEBUG``` to the Debug entry.

```Swift
# if DEBUG
    log.setup(.Debug, showThreadName: true, showLogLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: nil)
# else
    log.setup(.Severe, showThreadName: true, showLogLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: nil)
# endif
```

You can set any number of options up in a similar fashion. See the updated iOSDemo app for an example of using different log destinations based on options, search for ```USE_NSLOG```.


### To Do

- Add more examples of some advanced use cases
- Add additional log destination types
- Add Objective-C support
- Add log file rotation options

### More

If you find this library helpful, you'll definitely find these other tools helpful:

Watchdog: http://watchdogforxcode.com/  
Slender: http://martiancraft.com/products/slender  
Briefs: http://giveabrief.com/  

Also, please check out my book **Swift for the Really Impatient** http://swiftforthereallyimpatient.com/

### Change Log

* **Version 3.0**: *(2015/09/09)* - Bug fix, and WatchOS 2 suppport (thanks @ymyzk)
* **Version 2.4**: *(2015/09/09)* - Minor bug fix, likely the last release for Swift 1.x
* **Version 3.0b3**: *(2015/08/24)* - Added option to include the log identifier in log messages #79
* **Version 2.3**: *(2015/08/24)* - Added option to include the log identifier in log messages #79
* **Version 3.0b2**: *(2015/08/11)* - Updated for Swift 2.0 (Xcode 7 Beta 5)
* **Version 2.2**: *(2015/08/11)* - Internal restructuring, easier to create new log destination subclasses. Can disable function names, and/or dates. Added optional new log destination that uses NSLog instead of println().
* **Version 3.0b1**: *(2015/06/18)* - Swift 2.0 support/required. Consider this unstable for now, as Swift 2.0 will likely see changes before final release, and this library may undergo some architecture changes (time permitting).
* **Version 2.1.1**: *(2015/06/18)* - Fixed two minor bugs wrt XcodeColors.
* **Version 2.1**: *(2015/06/17)* - Added support for XcodeColors (https://github.com/robbiehanson/XcodeColors). Undeprecated the \*Exec() methods.
* **Version 2.0**: *(2015/04/14)* - Requires Swift 1.2. Removed some workarounds/hacks for older versions of Xcode. Removed thread based caching of NSDateFormatter objects since they're now thread safe. You can now use the default date formatter, or create and assign your own and it'll be used. Added Thread name option (Thanks to Nick Strecker https://github.com/tekknick ). Add experimental support for CocoaPods. 
* **Version 1.9**: *(2015/04/14)* - Deprecated the \*Exec() methods in favour of just using a trailing closure on the logging methods (Thanks to Nick Strecker https://github.com/tekknick ). This will be the last version for Swift 1.1.
* **Version 1.8.1**: *(2014/12/31)* - Added a workaround to the Swift compiler's optimization bug, restored optimization level back to Fastest
* **Version 1.8**: *(2014/11/16)* - Added warning log level (Issue #16)
* **Version 1.7**: *(2014/09/27)* - Reorganized to be used as a subproject instead of a framework, fixed threading
* **Version 1.6**: *(2014/09/09)* - Updated for Xcode 6.1 Beta 1
* **Version 1.5**: *(2014/08/23)* - Updated for Xcode 6 Beta 6
* **Version 1.4**: *(2014/08/04)* - Updated for Xcode 6 Beta 5, removed `__FUNCTION__` workaround
* **Version 1.3**: *(2014/07/27)* - Updated to use public/internal/private access modifiers
* **Version 1.2**: *(2014/07/01)* - Added exec methods to selectively execute code
* **Version 1.1**: *(2014/06/22)* - Changed the internal architecture to allow for more flexibility
* **Version 1.0**: *(2014/06/09)* - Initial Release

