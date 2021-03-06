# 与 Cocoa 数据类型协作

本节内容包括：

- [字符串](#strings)
    - [本地化](#Localization)
- [数值](#numbers)
- [集合类](#collection_classes)
    - [数组](#Arrays) 
    - [集合](#Sets)
    - [字典](#Dictionaries)
- [错误](#errors)
- [Foundation 数据类型](#foundation_data_types)
- [Foundation 函数](#foundation_functions)
- [Core Foundation](#core_foundation)
    - [重映射类型](#Remapped_Types)
    - [内存受管理的对象](#Memory_Managed_Objects)
    - [非托管对象](#Unmanaged_Objects)

Swift 提供了高效便捷的方式来处理 Cocoa 数据类型，从而增强与 Objective-C 之间的互用性。

Swift 会自动将一些 Objective-C 类型转换为 Swift 类型，也会将一些 Swift 类型转换为 Objective-C 类型。在 Objective-C 和 Swift 中还有一些数据类型可以互换使用。那些可转换或者可互换的数据类型被称为 *桥接* 数据类型。例如，在 Swift，可以将一个`Array`类型的值传递给一个接收`NSArray`对象的方法。也可以在一个桥接类型和它的对应类型间相互转换。可以使用`as`操作符在桥接类型间转换，或者显式提供常量或变量的类型，由 Swift 负责桥接这些数据类型。

Swift 还提供了一些简单便捷的方法与 Foundation 数据类型配合，在操作 Foundation 数据类型时，语法上更加自然和统一。

<a name = "strings"></a>
## 字符串

Swift 会在`String`类型和`NSString`类型间自动桥接。这意味着可以将一个 Swift 的`String`类型的值用在任何使用`NSString`对象的地方，并且可以得益于它们各自的特性，例如`String`类型的字符串插值，以及一些基于 Swift 设计的 API，乃至`NSString`类的广泛功能。因此，几乎不必再直接使用`NSString`类。事实上，当 Objective-C API 导入到 Swift 时，所有`NSString`类型会被替换为`String`类型。当 Swift API 导入到 Objective-C 时，所有`String`类型又会被替换为`NSString`类型。

要对字符串进行桥接，只需导入 Foundation 框架。例如，可以用 Swift 字符串访问`capitalizedString`属性，这是`NSString`类中的一个属性，Swift 会自动将`String`类型的值桥接为`NSString`对象来访问该属性。这个属性甚至会返回一个`String`类型的值，因为相应的类型在被导入的时候就被转换了。

```swift
import Foundation
let greeting = "hello, world!"
let capitalizedGreeting = greeting.capitalizedString
// capitalizedGreeting: String = Hello, World!
```

如果确实需要使用`NSString`对象，可以用`String`类型的值进行转换。另外，一个`NSString`对象总是可以转换为一个`String`类型的值，不必使用可选类型版本的类型转换操作符（`as?`）。也可以显式指明常量或变量的类型，利用 Swift 的字符串字面量创建一个`NSString`对象。

```swift
import Foundation
let myString: NSString = "123"
if let integerValue = Int(myString as String) {
    print("\(myString) is the integer \(integerValue)")
}
// 打印 "123 is the integer 123"
```

> 注意  
> 一个`String`实例无法用`AnyObject`表示，因为`AnyObject`只能表示 class 类型的实例，而`String`是个结构类型。然而，经过桥接后，`String`就可以赋值给`AnyObject`类型的常量或者变量，因为它已经被桥接为`NSString`。

<a name="Localization"></a>
### 本地化

在 Objective-C，通常用`NSLocalizedString`系列的宏来本地化字符串。这套宏包括`NSLocalizedString`，`NSLocalizedStringFromTable`，`NSLocalizedStringFromTableInBundle`，以及`NSLocalizedStringWithDefaultValue`。而在 Swift，只用一个函数就可以实现跟整套`NSLocalizedString`宏一样的功能，即`NSLocalizedString(key:tableName:bundle:value:comment:)`函数。该函数为`tableName`，`bundle`和`value`参数提供了默认值。可以用该函数取代以前的宏。

<a name = "numbers"></a>
## 数值

Swift 会自动将某些原生数值类型桥接为`NSNumber`，例如`Int`和`Float`。通过这样的桥接，可以基于其中某种类型创建一个`NSNumber`：

```swift
let n = 42
let m: NSNumber = n
```

还可以传递一个`Int`类型的值给一个接收`NSNumber`类型的参数。需要注意的是，`NSNumber`可以包含多种不同的类型，因此反向传递并不成立，例如不能把`NSNumber`类型传递给一个接收`Int`类型的参数。

下面所列出的类型都会自动桥接为`NSNumber`：

- `Int`
- `UInt`
- `Float`
- `Double`
- `Bool`

> 注意  
> Swift 中的这些数值类型都是结构类型，因此无法直接用`AnyObject`表示，必须先桥接为`NSNumber`。

<a name = "collection_classes"></a>
## 集合类

Swift 会分别将 `NSArray`、`NSSet` 和 `NSDictionary` 桥接为 `Array`、`Set` 和 `Dictionary`。这意味着在处理集合时，不但可以得益于 Swift 强大的算法和自然的语法，还可将 Foundation 和 Swift 集合类型互换使用。

<a name="Arrays"></a>
### 数组

Swift 会在 `Array` 类型和 `NSArray` 类之间自动桥接。使用轻量泛型的 `NSArray` 对象会被桥接为 `[ObjectType]` 类型的 Swift 数组。如果 `NSArray` 对象没有使用轻量泛型，那么它会被桥接为 `[AnyObject]` 类型的 Swift 数组。

例如，考虑以下 Objective-C 声明：

```objective-c
@property NSArray<NSDate *>* dates;
- (NSArray<NSDate *> *)datesBeforeDate:(NSDate *)date;
- (void)addDatesParsedFromTimestamps:(NSArray<NSString *> *)timestamps;
```

导入到 Swift 后将如下所示：

```swift
var dates: [NSDate]
func datesBeforeDate(date: NSDate) -> [NSDate]
func addDatesParsedFromTimestamps(timestamps: [String])
```

如果某个实例是 Objective-C 或 Swift 类的实例，或者能够进行桥接，那么这个实例就兼容 `AnyObject`。可以将任意 `NSArray` 对象桥接成 Swift 数组，因为所有 Objective-C 对象都兼容 `AnyObject`。因此，Swift 编译器会在导入 Objective-C API 时将 `NSArray` 替换成 `[AnyObject]`。

将 `NSArray` 对象桥接为 Swift 数组后，还可以将数组向下转换成特定的类型。与从 `NSArray` 桥接到 `[AnyObject]` 不同，从 `AnyObject` 类型向下转换成具体类型无法确保一定成功。由于直到运行时编译器才能确定数组中的元素能否向下转换为所指定的类型，因此，可以使用可选类型版本的类型转换操作符 `as?`，将 `[AnyObject]` 转换为 `[SomeType]`。如果确信转换一定会成功，则可以使用强制类型转换操作符 `as!`。例如，如果能确定 Swift 数组只包含 `NSView` 类的实例（或者 `NSView` 子类的实例），就可以将 `AnyObject` 类型的数组向下转换为 `NSView` 类型的数组。如果数组中的某个元素实际上不是 `NSView` 对象，那么转换结果会为 `nil`。

```swift
let swiftyArray = foundationArray as [AnyObject]
if let downcastedSwiftArray = swiftArray as? [NSView] {
    // downcastedSwiftArray 只包含 NSView 对象
}
```

也可以在 for 循环中将 `NSArray` 对象直接向下转换为特定类型的 Swift 数组：

```swift
for view in foundationArray as! [NSView] {
    // view 的类型为 NSView
}
```

从 Swift 数组桥接为 `NSArray` 对象时，Swift 数组里的元素必须兼容 `AnyObject`。例如，`[Int]` 类型的 Swift 数组包含 `Int` 类型的元素。`Int` 类型并不是类类型，但由于 `Int` 类型可桥接成 `NSNumber` 类，因此 `Int` 类型兼容 `AnyObject`。综上所述，可以将 `[Int]` 类型的 Swift 数组桥接为 `NSArray` 对象。如果 Swift 数组里的某个元素不兼容 `AnyObject`，那么桥接到 `NSArray` 对象时就会发生运行时错误。

> 注意  
> 为了优化性能，将集合强制向下转换为特定类型的集合时，例如 `NSArray as! [NSView]`，针对集合中每个元素的类型检查可能会被推迟到它们被单独访问时。因此，将数组强制转换为不兼容的类型总会成功，直到元素被访问时才可能因为类型转换失败而引发运行时错误。  
> 使用可选类型版本的向下转换操作符进行转换时，例如 `NSArray as? [NSView]`，将立即针对每个元素进行类型检查，如果任意元素类型转换失败就会直接返回 `nil`。

还可以直接利用 Swift 数组字面量创建 `NSArray` 对象，这同样遵循上面提到的桥接规则。将常量或变量声明为 `NSArray` 类型并为其赋值数组字面量时，Swift 将会创建 `NSArray` 对象，而不是 Swift 数组。

```swift
let schoolSupplies: NSArray = ["Pencil", "Eraser", "Notebkko"]
// schoolSupplies 是一个包含 NSString 对象的 NSArray 对象
```

在上面的例子中，Swift 数组字面量包含三个 `String` 字面量。因为 `String` 会桥接为 `NSString`，因此数组字面量被桥接成 `NSArray` 对象，并成功赋值给 `schoolSupplies` 常量。

在 Objective-C 代码中使用 Swift 类或者协议时，导入器会将被导入的 API 中所有 Swift 数组替换为 `NSArray` 对象。如果传递 `NSArray` 对象给接收数组的 Swift API，但是元素类型不兼容，就会发生运行时错误。如果 Swift API 返回 Swift 数组，但却不能被桥接为 `NSArray` 对象，也会发生运行时错误。

<a name="Sets"></a>
### 集合

Swift 也会在 `Set` 类型和 `NSSet` 类之间自动桥接。使用轻量泛型的 `NSSet` 对象会被桥接为 `Set<ObjectType>` 类型的 Swift 集合。如果 `NSSet` 对象没有使用轻量泛型，那么它会被桥接为 `Set<NSObject>` 类型的 Swift 集合。

例如，考虑以下 Objective-C 声明：

```objective-c
@property NSSet<NSString *>* words;
- (NSSet<NSString *> *)wordsMatchingPredicate:(NSPredicate *)predicate;
- (void)removeWords:(NSSet<NSString *> *)words;
```

导入到 Swift 后将如下所示：

```swift
var words: Set<String>
func wordsMatchingPredicate(predicate: NSPredicate) -> Set<String>
func removeWords(words: Set<String>)
```

> 注意  
> 和 `NSArray` 不同，未使用轻量泛型的 `NSSet` 会被桥接为 `Set<NSObject>` 而不是 `Set<AnyObject>`，因为 Swift 中的 `Set` 类型要求其元素必须符合 `Hashable` 协议。

所有 `NSSet` 对象都可以桥接为 Swift 集合，因为所有 Objective-C 对象都可以桥接为 `AnyObject`。因此，Swift 编译器会在导入 Objective-C API 时，将所有 `NSSet` 类替换为 `Set<NSObject>`。同理，在 Objective-C 中使用 Swift 类或者协议的时候，导入器会将兼容 Objective-C 的 Swift 集合重新映射为 `NSSet` 对象。

将 `NSSet` 对象桥接为 Swift 集合后，还可以将集合向下转换为特定类型。就如同 Swift 数组的向下转换一样，Swift 集合的向下转换也无法确保一定成功。对 `Set<NSObject>` 使用 `as?` 运算符向下转换为特定类型的结果将是可选值。

```swift
let swiftSet = foundationSet as Set<NSObject>
if let downcastedSwiftSet = swiftSet as? Set<UITouch> {
    // downcastedSwiftSet 只包含 UITouch 对象
}
```

还可以直接利用 Swift 数组字面量创建 `NSSet` 对象，这同样遵循上面提到的桥接规则。显式地将常量或者变量声明为 `NSSet` 类型，并为其赋值数组字面量时，Swift 将会创建 `NSSet` 对象，而不是 Swift 集合。

```swift
let amenities: NSSet = ["Sauna", "Steam Room", "Jacuzzi"]
// amenities 是一个包含 NSString 对象的 NSSet 对象
```

<a name="Dictionaries"></a>
### 字典

Swift 同样会在 `Dictionary` 类型和 `NSDictionary` 类之间自动桥接。使用轻量泛型的 `NSDictionary` 对象会被桥接为 `[ObjectType]` 类型的 Swift 字典。如果 `NSDictionary` 对象没有使用轻量泛型，那么它会被桥接为 `[NSObject : AnyObject]` 类型的 Swift 字典。

例如，考虑以下 Objective-C 声明：

```objective-c
@property NSDictionary<NSURL *, NSData *>* cachedData;
- (NSDictionary<NSURL *, NSNumber *> *)fileSizesForURLsWithSuffix:(NSString *)suffix;
- (void)setCacheExpirations:(NSDictionary<NSURL *, NSDate *> *)expirations;
```

导入到 Swift 后将如下所示：

```swift
var cachedData: [NSURL : NSData]
func fileSizesForURLsWithSuffix(suffix: String) -> [NSURL : NSNumber]
func setCacheExpirations(expirations: [NSURL : NSDate])
```

> 注意  
> 和 `NSSet` 类似，未使用轻量泛型的 `NSDictionary` 桥接为 Swift 字典后，键类型为 `NSObject`，而不是 `AnyObject`，因为 Swift 中的 `Dictionary` 类型要求其元素必须符合 `Hashable` 协议。

所有 `NSDictionary` 对象均可桥接为 Swift 字典，因为所有 Objective-C 对象都兼容 `AnyObject`。如果某个实例是 Objective-C 或 Swift 类的实例，或者能够进行桥接，该实例就兼容 `AnyObject`。因此，Swift 编译器会在导入 Objective-C API 时，将所有 `NSDictionary` 类替换成 `[NSObject : AnyObject]`。同理，在 Objective-C 中使用 Swift 类或者协议的时候，导入器会将兼容 Objective-C 的 Swift 字典重新映射为 `NSDictionary` 对象。

将 `NSDictionary` 对象转换为 Swift 字典后，还可以将字典向下转换为特定类型。就如同 Swift 数组的向下转换一样，Swift 字典的向下转换也无法确保一定成功。对 `[NSObject : AnyObject]` 使用 `as?` 操作符向下转换为特定类型的结果将是可选值。

在进行反向转换，也就是将 Swift 字典转换为 `NSDictionary` 对象的过程中，其键值都必须是某个类的实例，或者能够桥接为某个类的实例。

还可以直接利用 Swift 字典字面量创建 `NSDictionary` 对象，这同样遵循上面提到的桥接规则。显式地将常量或者变量声明为 `NSDictionary` 类型，并为其赋值字典字面量时，Swift 将会创建 `NSDictionary` 对象，而不是 Swift 字典。

```swift
let medalRankings: NSDictionary = ["Gold": "1st Place", "Silver": "2nd Place", "Bronze": "3rd Place"]
// medalRankings 是一个包含 NSString 对象的 NSDictionary 对象
```

<a name = "errors"></a>
## 错误

Swift 会在`ErrorType`类型和`NSError`类之间自动桥接。会产生错误的 Objective-C 方法会被导入为 Swift 的`throws`方法。反过来，Swift 的`throws`方法会依照 Objecitive-C 中的错误约定，被导入为会产生错误的 Objective-C 方法。

符合`ErrorType`协议，并使用`@objc`特性声明的 Swift 枚举，会在自动生成的头文件中生成一个`NS_ENUM`声明和一个表示相应错误域名的`NSString`常量。例如以下 Swift 枚举声明：

```swift
@objc public enum CustomError: Int, ErrorType {
    case A, B, C
}
```

在自动生成的头文件中相应的 Objective-C 声明如下：

```objective-c
// Project-Swift.h
typedef SWIFT_ENUM(NSInteger, CustomError) {
    CustomErrorA = 0,
    CustomErrorB = 1,
    CustomErrorC = 2,
};
static NSString * const CustomErrorDomain = @"Project.CustomError";
```

请参阅 [错误处理](04-Adopting%20Cocoa%20Design%20Patterns.md#%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86) 小节获取更多有关 Swift 和 Objective-C API 中错误处理的信息。

<a name = "foundation_data_types"></a>
## Foundation 数据类型

Swift 提供了一些简单便捷的方法与 Foundation 数据类型配合。使用这些方法操作`CGSize`和`CGPoint`之类的类型时，在语法上会显得更加自然和统一。例如，可以使用如下语法创建一个`CGSize`结构：

```swift
let size = CGSize(width: 20, height: 40)
```

这些方法还能以更自然的方式调用函数操作 Foundation 中的结构。

```swift
let rect  = CGRect(x: 50, y: 50, width: 100, height: 100)
let width = rect.width // 等价于 CGRectGetWidth(rect)
let maxX  = rect.maxY  // 等价于 CGRectGetMaxY(rect)
```

Swift 将`NSUInteger`和`NSInteger`桥接为`Int`，Foundation API 中的这些类型都会桥接为`Int`。在 Swift，出于一致性考虑，尽可能地使用`Int`，但如果必须使用一个无符号整数类型，也可以使用`UInt`。

<a name = "foundation_functions"></a>
## Foundation 函数

在 Swift，`NSLog`函数仍然可以在系统控制台打印信息，其格式化语法和在 Objective-C 中一样。

```swift
NSLog("%.7f", pi)
// 输出 "3.1415927" 到控制台
```

Swift 也提供了`print(_:separator:terminator:)`这样的打印函数。得益于字符插值机制，这些函数简洁易用、功能全面，足以胜任一般的打印需求。

`NSAssert`函数在 Swift 中不复存在，取而代之的是`assert`函数。

<a name = "core_foundation"></a>
## Core Foundation

Core Foundation 类型会被自动导入为完备的 Swift 类。无论是否提供了内存管理注释，Swift 都会自动管理 Core Foundation 对象的内存，包括自己实例化的 Core Foundation 对象。在 Swift，可以将每一对可免费桥接的 Fundation 和 Core Foundation 类型互换使用。先将 Core Foundation 类型桥接为 Foundation 类型，就可以进一步桥接为 Swift 标准库类型。

<a name="Remapped_Types"></a>
### 重映射类型

Swift 导入 Core Foundation 类型时，会将这些类型的名字重映射，从类型名字的末端移除 *Ref*，所有的 Swift 类都是引用类型，因此该后缀是多余的。

Core Foundation 的`CFTypeRef`类型会重映射为`Anyobject`类型。以前使用`CFTypeRef`的地方，现在该换成`AnyObject`了。

<a name="Memory_Managed_Objects"></a>
### 内存受管理的对象

对于从带内存管理注释的 API 返回的 Core Foundation 对象，Swift 会自动对其进行内存管理，不再需要调用`CFRetain`，`CFRelease`，或者`CFAutorelease`函数。

如果自定义的 C 函数或 Objective-C 方法返回 Core Foundation 对象，需要用`CF_RETURNS_RETAINED`或者`CF_RETURNS_NOT_RETAINED`宏注释这些函数或方法，从而自动插入内存管理代码。还可以使用`CF_IMPLICIT_BRIDGING_ENABLED`和`CF_IMPLICIT_BRIDGING_DISABLED`宏包围那些遵循 Core Foundation 内存管理规定、命名规定的 C 函数声明，从而能够根据命名推导出内存管理策略。

如果只调用那些不会间接返回 Core Foundation 对象的带内存管理注释的 API，那么可以略过本节的剩余部分了。否则，需要进一步学习有关非托管 Core Foundation 对象的知识。

<a name="Unmanaged_Objects"></a>
### 非托管对象

Swift 导入那些没有内存管理注释的 API 时，将不会自动对返回的 Core Foundation 对象进行内存管理。Swift 将这些返回的 Core Foundation 对象包装在一个`Unmanaged<Instance>`结构中。所有被间接返回的 Core Foundation 对象也都是非托管对象。例如，这有个不带内存管理注释的 C 函数：

```objective-c
CFStringRef StringByAddingTwoStrings(CFStringRef string1, CFStringRef string2)
```

Swift 会将其导入为如下形式：

```swift
func StringByAddingTwoStrings(_: CFString!, _: CFString!) -> Unmanaged<CFString>! {
    // ...
}
```

从没有内存管理注释的 API 接收到非托管对象后，在使用它之前，必须将它转换为能够接受内存管理的对象。这样，Swift 就可以对其进行内存管理。`Unmanaged<T>`结构提供了两个方法，用于将一个非托管对象转换为一个可接受内存管理的对象，即`takeUnretainedValue()`方法和`takeRetainedValue()`方法。这两个方法均会返回解包后的对象，可以根据 API 返回的是 unretained 对象还是 retained 对象来选择合适的方法。

例如，假设上面的 C 函数不会 retain `CFString`对象。在使用这个对象前，应该使用`takeUnretainedValue()`函数。

```swift
let memoryManagedResult = StringByAddingTwoStrings(str1, str2).takeUnretainedValue()
// memoryManagedResult 是一个接受内存管理的 CFString
```

当然，也可以对一个非托管对象使用`retain()`，`release()`和`autorelease()`方法，但是这种做法并不推荐。

要了解更多信息，请参阅 [*Memory Management Programming Guide for Core Foundation*](https://developer.apple.com/library/prerelease/ios/documentation/CoreFoundation/Conceptual/CFMemoryMgmt/CFMemoryMgmt.html#//apple_ref/doc/uid/10000127i) 中的 [*Core Foundation Object Lifecycle Management*](https://developer.apple.com/library/prerelease/ios/documentation/CoreFoundation/Conceptual/CFMemoryMgmt/Articles/lifecycle.html#//apple_ref/doc/uid/TP40002439-SW1) 小节。
