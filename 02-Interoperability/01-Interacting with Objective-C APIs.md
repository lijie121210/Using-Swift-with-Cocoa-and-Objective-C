# 与 Objective-C API 交互

本页包含内容：

- [初始化](#initialization)
    - [类工厂方法和便利构造器](#Class_Factory_Methods_and_Convenience_Initializers)
    - [可失败初始化](#Failable_Initialization)
- [访问属性](#accessing_properties)
- [方法](#working_with_methods)
- [id 兼容性](#id_compatibility)
    - [无法识别的选择器和可选链语法](#Unrecognized_Selectors_and_Optional_Chaining)
    - [将 AnyObject 向下转换](#Downcasting_AnyObject)
- [为空性和可选类型](#Nullability_and_Optionals)
- [轻量泛型](#Lightweight_Generics)
- [扩展](#extensions)
- [闭包](#closures)
    - [避免强引用循环](#Avoiding_Strong_Reference_Cycles_When_Capturing_self)
- [对象比较](#object_comparison)
    - [哈希](#Hashing)
- [Swift 类型兼容性](#swift_type_compatibility)
    - [配置 Swift 在 Objective-C 中的接口](#Configuring_Swift_Interfaces_in_Objective-C)
    - [强制动态派发](#Requiring_Dynamic_Dispatch) 
- [Objective-C 选择器](#objective_c_selectors)
    - [使用 performSelector 发送消息](#Sending_Messages_with_performSelector) 

*互用性* 是能让 Swift 和 Objective-C 相接合的特性，这使你能够在一种语言编写的文件中使用另一种语言。当你准备开始把 Swift 融入到你的开发流程中时，学会如何利用互用性来重新定义，改善并增强你编写 Cocoa 应用的方式真是极好的。

互用性很重要的一点就是允许你在编写 Swift 代码时使用 Objective-C API。导入一个 Objective-C 框架后，可以使用原生的 Swift 语法实例化这些类并与之交互。

<a name="initialization"></a>
## 初始化

要在 Swift 中实例化一个 Objective-C 类，你应该使用 Swift 语法调用它的一个构造器。

Objective-C 构造器以`init`开头，如果它接收参数，则会以`initWith`开头。Objective-C 构造器导入到 Swift 后，`init`前缀变为`init`关键字，以此表明该方法是 Swift 构造方法。如果构造器接收参数，`With`单词会被移除，方法名的其余部分则会被分配为相应的命名参数。

例如，思考如下 Objective-C 构造器声明：

```objective-c
- (instancetype)init;
- (instancetype)initWithFrame:(CGRect)frame
                        style:(UITableViewStyle)style;
```

Swift 中等价的构造器声明如下所示：

```swift
init() { /* ... */ }
init(frame: CGRect, style: UITableViewStyle) { /* ... */ }
```

Objective-C 和 Swift 构造器语法的区别在实例化对象时将更为明显：

在 Objective-C 这样写：

```objective-c
UITableView *myTableView = [[UITableView alloc] initWithFrame:CGRectZero 
                                                        style:UITableViewStyleGrouped];
```

在 Swift 则应该这样写：

```swift
let myTableView: UITableView = UITableView(frame: CGRectZero, style: .Grouped)
```

注意，你不需要调用`alloc`，Swift 会为你处理。另外，调用任何 Swift 构造方法时都不会出现“init”单词。

你可以在赋值时显式地声明类型，也可以省略类型，Swift 能从构造器自动推断出类型。

```swift
let myTextField = UITextField(frame: CGRect(x: 0.0, y: 0.0, width: 200.0, height: 40.0))
```

此处的`UITableView`和`UITextField`对象和你在 Objective-C 中实例化的一样。你可以用同样的方式使用它们，例如访问属性或者调用各自类中的方法。

<a name="Class_Factory_Methods_and_Convenience_Initializers"></a>
### 类工厂方法和便利构造器

为了统一和简洁，Objective-C 中的类工厂方法被导入为 Swift 中的便利构造器，从而能使用同样简洁的构造器语法。

例如，在 Objective-C 像这样调用一个工厂方法：

```objective-c
UIColor *color = [UIColor colorWithRed:0.5 green:0.0 blue:0.5 alpha:1.0];
```

在 Swift 应该这样写：

```swift
let color = UIColor(red: 0.5, green: 0.0, blue: 0.5, alpha: 1.0)
```

<a name="Failable_Initialization"></a>
### 可失败初始化

在 Objective-C，构造器会直接返回它们初始化的对象。初始化失败时，为了告知调用者，Objective-C 构造器会返回`nil`。在 Swift，这种模式被内置到语言特性中，被称为*可失败初始化*。

许多系统框架中的 Objective-C 构造器在导入到 Swift 时会被检查初始化是否会失败。你可以在你的 Objective-C 类中，使用 *为空性注释* 来指明初始化是否会失败，正如 [为空性和可选类型](#Nullability_and_Optionals) 小节所描述的。如果初始化不会失败，这些 Objective-C 构造器会作为`init(...)`导入，而如果初始化会失败，则会作为`init?(...)`导入。在没有任何为空性注释的情况下，Objective-C 构造器会作为`init!(...)`导入。

例如，当指定路径的图片文件不存在时，`UIImage(contentsOfFile:)`构造器初始化`UIImage`对象便会失败。你可以用可选绑定语法对这种构造器的可选类型的返回值进行解包。

```swift
if let image = UIImage(contentsOfFile: "MyImage.png") {
    // 加载图片成功。
} else {
    // 无法加载图片。
}
```

<a name="accessing_properties"></a>
## 访问属性

在 Objective-C 中使用`@property`语法声明的属性，导入到 Swift 时，会遵循以下规则：

- 拥有为空性属性特性的属性（`nonnull`，`nullable`，`null_resettable`），作为可选类型或者非可选类型的属性导入。请参阅 [为空性和可选类型](#Nullability_and_Optionals) 小节。
- 拥有`readonly`属性特性的属性，作为只读计算型属性（`{ get }`）导入。
- 拥有`weak`属性特性的属性，作为标记`weak`关键字（`weak var`）的属性导入。
- 拥有`assign`，`copy`，`strong`，`unsafe_unretained`属性特性的属性，导入后会拥有相应的内存管理策略。
- 原子属性特性（`atomic`，`nonatomic`）在 Swift 属性声明中并无相应体现，不过 Objective-C 中具有原子性的属性在 Swift 中将保持其原子性。
- 存取器属性特性（`getter=`，`setter=`）会被 Swift 忽略。

在 Swift 中使用点语法对属性进行存取，直接使用属性名即可，不需要附加圆括号。

例如，如下代码设置了`UITextField`对象的`textColor`和`text`属性：

```swift
myTextField.textColor = UIColor.darkGrayColor()
myTextField.text = "Hello world"
```

> 注意  
> `darkGrayColor()`后跟一对圆括号，因为它是一个类方法，而不是一个属性。

在 Objective-C，一个有返回值的无参数方法可以像属性那样使用点语法调用。但它们会被导入为 Swift 中的实例方法，只有在 Objective-C 中使用`@property`声明的属性才会导入为 Swift 中的属性。方法的导入和调用请参阅 [方法](#working_with_methods) 小节。

<a name="working_with_methods"></a>
## 方法

在 Swift 中使用点语法调用方法。

当 Objective-C 方法导入到 Swift 时，Objective-C 选择器的第一部分将会成为基本方法名并出现在圆括号的前面。第一个参数将直接在圆括号中出现，并且没有参数名。选择器的其余部分作为相应的参数名，出现在方法的圆括号内。选择器的所有部分在调用时都必须写上。

例如，在 Objective-C 这样写：

```objective-c
[myTableView insertSubview:mySubview atIndex:2];
```

在 Swift 则应该这样写：

```swift
myTableView.insertSubview(mySubview, atIndex: 2)
```

如果调用一个无参数的方法，依旧必须在方法名后面跟上一对圆括号：

```swift
myTableView.layoutIfNeeded()
```

<a name="id_compatibility"></a>
## id 兼容性

Swift 引入了`AnyObject`类型，用以表示任意类型的对象，类似 Objective-C 的`id`类型。Swift 将`id`导入为`AnyObject`，不仅保证了类型安全，还维持了无类型对象的灵活性。

例如，如同`id`类型，可以为`AnyObject`类型的常量或变量分配任意类的对象。如果是变量，还可以为其重新分配不同类的对象。

```swift
var myObject: AnyObject = UITableViewCell()
myObject = NSDate()
```

也可以使用`AnyObject`类型的值调用任意 Objective-C 方法或者访问任意属性，而不用将其转换为具体的类型。这包括标记了`@objc`特性的兼容于 Objective-C 的方法和属性。

```swift
let futureDate = myObject.dateByAddingTimeInterval(10)
let timeSinceNow = myObject.timeIntervalSinceNow
```

<a name="Unrecognized_Selectors_and_Optional_Chaining"></a>
### 无法识别的选择器和可选链语法

因为`AnyObject`类型的值在运行时才能确定其真实类型，所以有可能在不经意间写出不安全代码。在 Swift，和在 Objective-C 一样，试图调用不存在的方法将引发未识别的选择器错误。

例如，如下代码没有任何编译警告，但是会引发运行时错误：

```swift
myObject.characterAtIndex(5)
// 崩溃，myObject 无法响应那个方法。
```

Swift 使用可选来防止这种不安全的行为。当用`AnyObject`类型的值调用一个 Objective-C 方法时，方法调用在行为上类似于隐式解包可选类型。可以像调用协议中的可选方法那样，使用可选链语法在`AnyObject`上调用方法。

> 注意  
> 通过`AnyObject`访问属性将总是返回一个可选类型的值,而不会引发运行时错误。若属性原本就返回可选类型的值，则会返回一个双重包装的可选类型，例如`AnyObject!?`。

例如，在下面的代码中，第一和第二行代码不会被执行，因为`count`属性和`characterAtIndex:`方法不存在于`NSDate`对象中。`myCount`常量会被推断成可选类型的`Int`，并被赋值为`nil`。你也可以使用`if-let`语句来有条件地解包这种可能无法响应的方法的返回值，就像第三行所做的一样。

```swift
// myObject 为 AnyObject 类型，其真实类型为 NSDate。
// myCount 类型为 Int?，其值为 nil。
let myCount = myObject.count
// myChar 类型为 unichar?，其值为 nil。
let myChar = myObject.characterAtIndex?(5)
// 条件分支不会被执行。
if let fifthCharacter = myObject.characterAtIndex?(5) {
    print("Found \(fifthCharacter) at index 5")
}
```

> 注意  
> 尽管在使用`AnyObject`类型的值调用方法时不需要强制解包，但强制解包可以防止一些意料之外的行为。

<a name="Downcasting_AnyObject"></a>
### 将 AnyObject 向下转换

如果能确定`AnyObject`对象的具体类型，将其向下转换到具体类型是很有用的。然而，由于`AnyObject`类型可以表示任意类型的对象，因此向下转换到具体类型无法确保一定成功。

可以使用条件转换操作符（as?），这将返回试图转换到的具体类型的可选类型的值：

```swift
let userDefaults = NSUserDefaults.standardUserDefaults()
let lastRefreshDate: AnyObject? = userDefaults.objectForKey("LastRefreshDate")
if let date = lastRefreshDate as? NSDate {
    print("\(date.timeIntervalSinceReferenceDate)")
}
```

如果确信对象的类型，可以使用强制下转操作符（`as!`）：

```swift
let myDate = lastRefreshDate as! NSDate
let timeInterval = myDate.timeIntervalSinceReferenceDate
```

然而，一旦强制下转失败，将引发一个运行时错误：

```swift
let myDate = lastRefreshDate as! NSString // 错误
```

<a name="Nullability_and_Optionals"></a>
## 为空性和可选类型

在 Objective-C，用于操作对象的原始指针的值可能会是`NULL`（在 Objective-C 中称为`nil`）。而在 Swift，所有的值，包括结构体与对象的引用，都能确保是非空值。作为替代，可以将可能会值缺失的值包装为该类型的 *可选类型* 。当你需要表示值缺失的情况时，你可以将其赋值为`nil`。请参阅 [*The Swift Programming Language 中文版*](http://wiki.jikexueyuan.com/project/swift/) 中的 [可选类型](http://wiki.jikexueyuan.com/project/swift/chapter2/01_The_Basics.html#optionals) 小节获取更多关于可选类型的信息。

在 Objective-C 可以使用 *为空性注释* 来指明一个参数类型，属性类型或者返回值类型是否可以为`NULL`或者`nil`值。单个类型声明可以使用`_Nullable`和`_Nonnull`注释，单个属性声明可以使用`nullable`，`nonnull`，`null_resettable`属性特性，范围注释可以使用`NS_ASSUME_NONNULL_BEGIN`和`NS_ASSUME_NONNULL_END`宏。如果一个类型没有任何为空性注释，Swift 就无法辨别它是可选类型还是非可选类型，因此将其作为隐式解包可选类型导入。

- 以`_Nonnull`或者范围宏注释的类型，会作为 *非可选类型* 导入到 Swift。
- 以`_Nullable`注释的类型，会作为 *可选类型* 导入到 Swift。
- 没有为空性注释的类型，会作为 *隐式解包可选类型* 导入到 Swift。

举个例子，思考如下 Objective-C 声明：

```objective-c
@property (nullable) id nullableProperty;
@property (nonnull) id nonNullProperty;
@property id unannotatedProperty;
 
NS_ASSUME_NONNULL_BEGIN
- (id)returnsNonNullValue;
- (void)takesNonNullParameter:(id)value;
NS_ASSUME_NONNULL_END
 
- (nullable id)returnsNullableValue;
- (void)takesNullableParameter:(nullable id)value;
 
- (id)returnsUnannotatedValue;
- (void)takesUnannotatedParameter:(id)value;
```

导入到 Swift 后将如下所示：

```Swift
var nullableProperty: AnyObject?
var nonNullProperty: AnyObject
var unannotatedProperty: AnyObject!
 
func returnsNonNullValue() -> AnyObject
func takesNonNullParameter(value: AnyObject)
 
func returnsNullableValue() -> AnyObject?
func takesNullableParameter(value: AnyObject?)
 
func returnsUnannotatedValue() -> AnyObject!
func takesUnannotatedParameter(value: AnyObject!)
```

大多数 Objective-C 系统框架，包括 Foundation，都已经提供了为空性注释，这使你能以更加类型安全的方式去操作各种值。

<a name="Lightweight_Generics"></a>
## 轻量泛型

Objective-C 的`NSArray`，`NSSet`以及`NSDictionary`类型的声明在使用轻量泛型参数化后，被导入到 Swift 时会附带容器中元素的类型信息。

举个例子，思考如下 Objective-C 属性声明：

```objective-c
@property NSArray<NSDate *>* dates;
@property NSSet<NSString *>* words;
@property NSDictionary<NSURL *, NSData *>* cachedData;
```

导入到 Swift 后将如下所示：

```swift
var dates: [NSDate]
var words: Set<String>
var cachedData: [NSURL: NSData]
```

> 注意  
> 除了 Foundation 中的集合类型，其他 Objective-C 类型使用轻量泛型会被 Swift 忽略。

<a name="extensions"></a>
## 扩展

Swift 扩展和 Objective-C 分类相似。扩展可以为既有类，结构和枚举扩充功能，即使它们是在 Objective-C 中定义的。通过导入相应的模块，还可以为系统框架中的类型或者自定义类型定义扩展。

例如，可以扩展`UIBezierPath`类，基于三角形的边长与起点来创建一个简单的三角形路径。

```swift
extension UIBezierPath {
    convenience init(triangleSideLength: CGFloat, origin: CGPoint) {
        self.init()
        let squareRoot = CGFloat(sqrt(3.0))
        let altitude = (squareRoot * triangleSideLength) / 2
        moveToPoint(origin)
        addLineToPoint(CGPoint(x: origin.x + triangleSideLength, y: origin.y))
        addLineToPoint(CGPoint(x: origin.x + triangleSideLength / 2, y: origin.y + altitude))
        closePath()
    }
}
```

还可以利用扩展来增加属性（包括类型属性和静态属性）。然而，这些属性必须是计算型属性，因为扩展不能为类，结构体，枚举添加存储型属性。

下面这个例子为`CGRect`结构增加了一个叫`area`的计算型属性：

```swift
extension CGRect {
    var area: CGFloat {
        return width * height
    }
}
let rect = CGRect(x: 0.0, y: 0.0, width: 10.0, height: 50.0)
let area = rect.area
```

甚至可以利用扩展为类添加协议符合性而无需子类化。如果协议是在 Swift 中定义的，还可以为结构或是枚举添加协议符合性，即使它们是在 Objective-C 中定义的。

注意，无法通过扩展覆盖类型中既有的方法与属性。

<a name="closures"></a>
## 闭包

Objective-C 的 block 会作为符合其调用约定的 Swift 闭包导入，通过`@convention(block)`特性表示。例如，下面是一个 Objective-C 的 block 变量：

```objective-c
void (^completionBlock)(NSData *, NSError *) = ^(NSData *data, NSError *error) {
    // ...
}
```

它在 Swift 中如下所示：

```swift
let completionBlock: (NSData, NSError) -> Void = { (data, error) in 
    // ...
}
```

Swift 闭包与 Objective-C block 兼容，因此可以把一个 Swift 闭包传递给一个接受 block 作为参数的 Objective-C 方法。而且因为 Swift 闭包与函数类型相同，甚至可以直接传递一个 Swift 函数的函数名过去。

闭包与 block 有相似的捕获语义，但有个关键的不同：被捕获的变量可以直接修改，而 block 在默认情况下捕获的只是变量的值拷贝。换言之，Swift 闭包在默认情况下捕获的变量等效于 Objective-C 的`__block`变量。

<a name="Avoiding_Strong_Reference_Cycles_When_Capturing_self"></a>
### 避免强引用循环

在 Objective-C，如果 block 捕获了`self`，一定要慎重考虑内存管理问题。

block 会维持对被捕获对象的强引用，包括`self`。一旦`self`也持有对 block 的强引用，例如一个 copy 语义的 block 属性，就将导致强引用循环。可以让 block 捕获`self`的弱引用来避免此问题：

```objective-c
__weak typeof(self) weakSelf = self;
self.block = ^{
    __strong typeof(self) strongSelf = weakSelf;
    [strongSelf doSomething];
};
```

和 Objective-C block 类似，Swift 闭包也会维持对被捕获对象的强引用，包括`self`。为了避免强引用循环，可以在闭包的捕获列表中指定`self`为`unowned`：

```swift
self.closure = { [unowned self] in
    self.doSomething()
}
```

请参阅 [*The Swift Programming Language 中文版*](http://wiki.jikexueyuan.com/project/swift/) 中的 [解决闭包引起的循环强引用](http://wiki.jikexueyuan.com/project/swift/chapter2/16_Automatic_Reference_Counting.html#resolving_strong_reference_cycles_for_closures) 小节获取更多信息。

<a name="object_comparison"></a>
## 对象比较

在 Swift，比较两个对象可以使用两种方式。第一种，使用 `==` 运算符，比较两个对象内容是否相同。第二种，使用 `===` 运算符，判断常量或者变量是否引用同一对象。

Swift 为源自 `NSObject` 的类实现了 `Equatable` 协议，并提供了 `==` 运算符和 `===` 运算符的默认实现。`==` 运算符的默认实现会调用 `isEqual:` 方法，`===` 运算符的默认实现则会检查指针是否相同。你不应该为 Objective-C 类型重写这两个运算符。

`NSObject` 类为 `isEqual:` 方法提供的基本实现仅仅是比较两个指针是否相同。可以根据需要在子类中重新实现 `isEqual:` 方法，基于对象内容进行比较，而不是比较指针。关于如何实现对象比较逻辑的更多信息，请参阅 [Object comparison](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectComparison.html#//apple_ref/doc/uid/TP40008195-CH37)。

> 注意  
> Swift 为 `!=` 和 `!==` 运算符提供了实现，无需再进行重写。

<a name="Hashing"></a>
### 哈希

Swift 为源自 `NSObject` 的类实现了 `Hashable` 协议，并提供了 `hashValue` 属性的默认实现，即调用 `NSObject` 的 `hash` 属性。

`NSObject` 子类在重写 `isEquals:` 方法时，还应该重写 `hash` 属性。

<a name="swift_type_compatibility"></a>
## Swift 类型兼容性

在 Swift 中子类化 Objective-C 类时，该 Swift 子类及其成员，即属性、方法、下标和构造器，在 Objective-C 中均可直接使用。但这不包括一些 Swift 独有特性，如下列所示：

- 泛型
- 元组
- 原始值类型不是 `Int` 类型的枚举
- 结构体
- 顶级函数
- 全局变量
- 类型别名
- 可变参数
- 嵌套类型
- 柯里化函数

Swift API 转换到 Objective-C 时：

- Swift 可选类型会被标注 `__nullable`。
- Swift 非可选类型会被标注 `__nonnull`。
- Swift 常量存储属性和计算属性会成为 Objective-C 只读属性。
- Swift 变量存储属性会成为 Objective-C 读写属性。
- Swift 类型方法会成为 Objective-C 类方法。
- Swift 构造器和实例方法会成为 Objective-C 实例方法。
- Swift 中抛出错误的方法会成为接受 `NSError **` 参数的 Objective-C 方法。如果这种 Swift 方法没有指定返回类型，那么相应的 Objective-C 方法会拥有 `BOOL` 类型的返回值。

例如，思考如下 Swift 声明：

```swift
class Jukebox: NSObject {

    var library: Set<String>

    var nowPlaying: String?

    var isCurrentlyPlaying: Bool {
        return nowPlaying != nil
    }

    init(songs: String...) {
        self.library = Set<String>(songs)
    }

    func playSong(named name: String) throws {
        // 播放歌曲，若歌曲不可用则抛出错误
    }
}
```

上述代码导入到 Objective-C 后如下所示：

```Objective-C
@interface Jukebox : NSObject
@property (nonatomic, copy) NSSet<NSString *> * __nonnull library;
@property (nonatomic, copy) NSString * __nullable nowPlaying;
@property (nonatomic, readonly) BOOL isCurrentlyPlaying;
- (nonnull instancetype)initWithSongs:(NSArray<NSString *> * __nonnull)songs OBJC_DESIGNATED_INITIALIZER;
- (BOOL)playSong:(NSString * __nonnull)name error:(NSError * __nullable * __null_unspecified)error;
@end
```

> 注意  
> 无法在 Objective-C 中继承一个 Swift 类。

<a name="Configuring_Swift_Interfaces_in_Objective-C"></a>
### 配置 Swift 在 Objective-C 中的接口

在某些情况下，需要更精确地控制如何将 Swift API 暴露给 Objective-C。你可以使用 `@objc(name)` 特性来改变类、属性、方法、枚举类型，以及枚举用例暴露给 Objective-C 代码的声明。

例如，如果 Swift 类的类名包含 Objective-C 中不支持的字符，就可以为其提供一个在 Objective-C 中的别名。如果要为 Swift 函数提供 Objective-C 别名，使用 Objective-C 选择器语法，并记得在选择器参数片段后添加冒号（`:`）。

```swift
@objc(Color)
enum Цвет: Int {

    @objc(Red)
    case Красный

    @objc(Black)
    case Черный
}

@objc(Squirrel)
class Белка: NSObject {

    @objc(color)
    var цвет: Цвет = .Красный

    @objc(initWithName:)
    init (имя: String) {
        // ...
    }

    @objc(hideNuts:inTree:)
    func прячьОрехи(количество: Int, вДереве дерево: Дерево) {
        // ...
    }
}
```

对 Swift 类使用 `@objc(name)` 特性时，该类将在 Objective-C 中可用并且没有任何命名空间。因此，这个特性在迁徙被归档的 Objecive-C 类到 Swift 时会非常有用。由于归档文件中存储了被归档对象的类名，因此应该使用 `@objc(name)` 特性来指定被归档的 Objective-C 对象的类名，这样归档文件才能通过新的 Swift 类解档。

> 注意  
> 相反，Swift 还提供了 `@nonobjc` 特性，可以让一个 Swift 声明在 Objective-C 中不可用。可以利用它来解决桥接方法循环，以及允许重载 Objective-C 类中的方法。另外，如果一个 Objective-C 方法在 Swift 中被重写后，无法再以 Objective-C 的语言特性呈现，例如将参数变为了可变参数，那么这个方法必须标记为 `@nonobjc`。

<a name="Requiring_Dynamic_Dispatch"></a>
### 强制动态派发

即使将 Swift API 暴露给 Objective-C 运行时，也无法保证调用属性、方法、下标或构造器时的动态派发。Swift 编译器依旧可能绕过 Objective-C 运行时，通过消虚拟化或内联成员访问来优化代码的性能。

可以使用 `dynamic` 修饰符来强制通过运行时系统动态派发。一般很少需要强制动态派发，但是，如果要使用键值观察技术或者 `method_exchangeImplementations` 这种在运行时替换方法实现的函数，就必须使用动态派发。如果 Swift 编译器内联了方法的实现或者消虚拟化对它的访问，新的方法实现就不会被调用了。

> 注意  
> 标记 `dynamic` 修饰符的声明无法再标记 `@nonobjc` 特性。

<a name="objective_c_selectors"></a>
## Objective-C 选择器

Objective-C 选择器是一种用于引用 Objective-C 方法名的类型。在 Swift，Objective-C 选择器用 `Selector` 结构体表示。可以使用 `#selector` 表达式创建一个选择器，例如 `let mySelector = #selector(MyViewController.tappedButton)`，这里直接使用了一个 Objective-C 方法作为子表达式。被引用的 Objective-C 方法还可以加上圆括号，并使用 `as` 运算符来消除重载方法之间的歧义，例如 `let anotherSelector = #selector(((UIView.insertSubview(_:at:)) as (UIView) -> (UIView, Int) -> Void))`。

```swift
import UIKit
class MyViewController: UIViewController {

    let myButton = UIButton(frame: CGRect(x: 0, y: 0, width: 100, height: 50))
    
    override init?(nibName nibNameOrNil: String?, bundle nibBundleOrNil: NSBundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        let action = #selector(MyViewController.tappedButton)
        myButton.addTarget(self, action: action, forControlEvents: .TouchUpInside)
    }
    
    func tappedButton(sender: UIButton!) {
        print("tapped button")
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```

<a name="Sending_Messages_with_performSelector"></a>
### 使用 performSelector 发送消息

可以使用 `performSelector(_:)` 方法以及它的变体向兼容 Objective-C 的对象发送消息。

`performSelector` 系列 API 可以向指定线程发送消息，或者延迟发送没有返回值的消息。该系列 API 同步执行，并返回隐式解包可选类型的非托管对象（`Unmanaged<AnyObject>!`），这是因为返回值的类型和所有权无法在编译期决定。请参阅 [非托管对象](03-Working%20with%20Cocoa%20Data%20Types.md#%E9%9D%9E%E6%89%98%E7%AE%A1%E5%AF%B9%E8%B1%A1) 小节获取更多信息。

```swift
let string: NSString = "Hello, Cocoa!"
let selector = #selector(NSString.lowercaseStringWithLocale(_:))
let locale = NSLocale.currentLocale()
if let result = string.performSelector(selector, withObject: locale) {
    print(result.takeUnretainedValue())
}
// 打印输出 "hello, cocoa!"
```

向一个对象发送无法识别的选择器将造成接收者调用 `doesNotRecognizeSelector(_:)` 方法，其默认实现是引发 `NSInvalidArgumentException` 异常。

```swift
let array: NSArray = ["delta", "alpha", "zulu"]
// 下面这句代码不会导致编译错误，因为该选择器存在于 NSDictionary 中
let selector = #selector(NSDictionary.allKeysForObject)
// 下面这句代码将导致异常，因为 NSArray 无法响应该选择器
array.performSelector(selector)
```

在 Objective-C 运行时中直接向对象发送消息并非内在安全的，因为编译器无法保证消息发送的结果，也无法保证消息是否能在第一时间被处理。因此，并不鼓励使用 `performSelector` 系列 API，除非代码确实依赖于 Objective-C 运行时提供的动态方法决议。否则，正如 [id 兼容性](#id_compatibility) 小节所描述的，将对象转换为 `AnyObject` 类型，再使用可选链语法调用方法会更为安全和方便。
