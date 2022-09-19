# iOS 15 - iOS 16 开发 适配 总结帖

### iOS16 Xcode14适配内容：
```
新增控件UICalendarView，显示日期支持单选与多选
新增控件UIEditMenuInteraction，取代 UIMenuController、UIMenuItem

新增控件UIFindInteraction 文本内容查找与替换
新增控件UIPasteControl 读取剪贴板中的内容，跨 App 读取需要授权弹框

UIImage 新增的构造函数，支持 SF Symbols 新增的类别 Variable
LARightStore 存储、获取 keychain 数据

iOS 16 真机调试开启，设置-隐私与安全-开发者模式

UIScreen.main 将会废弃，建议使用 (UIApplication.shared.connectedScenes.first as? UIWindowScene)?.screen
支持 setValue() 方法设置设备的方向，替换为 UIWindowScene 的 requestGeometryUpdate() 方法。
UISheetPresentationController 支持自定义显示的 UIViewController 的大小。
UINavigationItem 改动

新增属性 style 描述 UINavigationItem 在 UINavigationBar 上的布局
新增属性 backAction 用于自定义 UIViewController 返回button事件
新增属性 titleMenuProvider 用于给当前导航栏的标题添加操作菜单

UIPageControl 支持垂直显示、设置指示器、设置当前页图片。
UITableView、UICollectionView 使用 Cell Content Configuration 时支持使用 UIHostingConfiguration 包装 SwiftUI 代码定义 Cell 的内容。
UITableView、UICollectionView 新增 selfSizingInvalidation 参数，使Cell可以自动调整大小
UIMenu 支持尺寸 small 、 medium 、 large
UIDevice 获取设备信息时，只能获取设备的名称，隐私权限增强
WidgetFamily 新增分类 accessory ，支持 iOS 锁屏显示和 watchOS 表盘显示

URLSession 建议通过连接迁移来优化网络切换场景下的 TCP 连接重建，降低网络的延迟。

class ViewController: UIViewController {
    lazy var session: URLSession = {
        let configuration = URLSessionConfiguration.default
        // MultipathServiceType是一个枚举类型，App可以采用不同的策略来利用这些网络通道
        configuration.multipathServiceType = .handover
        let session = URLSession(configuration: configuration)
        return session
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
    }
}


打开系统通知设置界面的 URL Scheme 从
UIApplicationOpenNotificationSettingsURLString替换为openNotificationSettingsURLString。
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        // 新的通知设置URL Scheme
        let urlString = UIApplication.openNotificationSettingsURLString
        if let url = URL(string: urlString), UIApplication.shared.canOpenURL(url) {
            UIApplication.shared.open(url, options: [:], completionHandler: nil)
        }
    }
}


UIScreen.main即将被废弃，建议使用(UIApplication.shared.connectedScenes.first as? UIWindowScene)?.screen。
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        // 新的获取UIScreen尺寸的方法
        if let screen = (UIApplication.shared.connectedScenes.first as? UIWindowScene)?.screen {
            print(screen)
        }
    }
}

```

## 对于iOS15适配汇总以及遇到的问题

注意：以下适配内容，必须适配的会以"必须"标出
```
UITableView Section的header增加默认间距
if (@available(iOS 15.0, *)) {
    self.tableView.sectionHeaderTopPadding = 0;
}
```

导航栏异样
iOS 15中，导航栏的问题比较明显，调试之后发现是UINavigationBar部分属性的设置在iOS 15上是无效的，查看导航栏特性API，苹果对导航栏的性能做了优化，默认情况下，如果导航栏与视图没有折叠，导航栏的背景透明，如果系统检测到有重叠的话，会变成毛玻璃的效果。

注意⚠️：UINavigationBarAppearance是iOS 13更新的API，iOS 15 navigationBar的相关属性设置要通过实例UINavigationBarAppearance来实现。

在iOS 13 UINavigationBar新增了scrollEdgeAppearance属性，但在iOS 14及更早的版本中此属性只应用在大标题导航栏上。在iOS 15中此属性适用于所有导航栏。

对于scrollEdgeAppearance属性的说明：
```
When a navigation controller contains a navigation bar and a scroll view, part of the scroll view’s content appears underneath the navigation bar. If the edge of the scrolled content reaches that bar, UIKit applies the appearance settings in this property.
If the value of this property is nil, UIKit uses the settings found in the standard Appearance property, modified to use a transparent background. If no navigation controller manages your navigation bar, UIKit ignores this property and uses the standard appearance of the navigation bar.

//旧代码
- (void)customizeInterface {
    //设置Nav的背景色和title色
    UINavigationBar *navigationBarAppearance = [UINavigationBar appearance];
    [navigationBarAppearance setBackgroundImage:[UIImage imageWithColor:[UIColor whiteColor]] forBarMetrics:UIBarMetricsDefault];
    [navigationBarAppearance setShadowImage:[UIImage new]];
    [navigationBarAppearance setTintColor:[UIColor colorWithHexString:@"333333"]];//返回按钮的箭头颜色
    NSDictionary *textAttributes = @{
        NSFontAttributeName:[UIFont fontWithName:@"SourceHanSerifSC-Bold" size:18.0],
        NSForegroundColorAttributeName: [UIColor colorWithHexString:@"333333"],
    };
    [navigationBarAppearance setTitleTextAttributes:textAttributes];
    navigationBarAppearance.backIndicatorImage = [UIImage imageNamed:@"image_common_navBackBlack"];
    navigationBarAppearance.backIndicatorTransitionMaskImage = [UIImage imageNamed:@"image_common_navBackBlack"];
    [[UITextField appearance] setTintColor:kColorLightBlue];//设置UITextField的光标颜色
    [[UITextView appearance] setTintColor:kColorLightBlue];//设置UITextView的光标颜色
    [[UISearchBar appearance] setBackgroundImage:[UIImage imageWithColor:[UIColor whiteColor]] forBarPosition:0 barMetrics:UIBarMetricsDefault];
}
//新代码
- (void)customizeInterface {
    //设置Nav的背景色和title色
    if (@available(iOS 13.0, *)) {
        UINavigationBarAppearance *appearance = [[UINavigationBarAppearance alloc] init];
        [appearance setBackgroundColor:[UIColor whiteColor]];
        [appearance setBackgroundImage:[UIImage imageWithColor:[UIColor whiteColor]]];
        [appearance setShadowImage:[UIImage imageWithColor:[UIColor whiteColor]]];
        appearance.titleTextAttributes = @{NSFontAttributeName:[UIFont systemFontOfSize:18.0f weight:UIFontWeightSemibold],NSForegroundColorAttributeName: [UIColor colorWithHexString:@"333333"]};
        [appearance setBackIndicatorImage:[UIImage imageNamed:@"image_common_navBackBlack"] transitionMaskImage:[UIImage imageNamed:@"image_common_navBackBlack"]];
        [[UINavigationBar appearance] setScrollEdgeAppearance: appearance];
        [[UINavigationBar appearance] setStandardAppearance:appearance];

    }
    
    UINavigationBar *navigationBarAppearance = [UINavigationBar appearance];
    [navigationBarAppearance setBackgroundImage:[UIImage imageWithColor:[UIColor whiteColor]] forBarMetrics:UIBarMetricsDefault];
    [navigationBarAppearance setShadowImage:[UIImage new]];
    [navigationBarAppearance setTintColor:[UIColor colorWithHexString:@"333333"]];
    [navigationBarAppearance setTitleTextAttributes:@{NSFontAttributeName:[UIFont systemFontOfSize:18.0f weight:UIFontWeightSemibold],NSForegroundColorAttributeName: [UIColor colorWithHexString:@"333333"]}];
    navigationBarAppearance.backIndicatorImage = [UIImage imageNamed:@"image_common_navBackBlack"];
    navigationBarAppearance.backIndicatorTransitionMaskImage = [UIImage imageNamed:@"image_common_navBackBlack"];

    [[UITextField appearance] setTintColor:[UIColor colorWithHexString:System_color]];
    [[UITextView appearance] setTintColor:[UIColor colorWithHexString:System_color]];
    [[UISearchBar appearance] setBackgroundImage:[UIImage imageWithColor:[UIColor whiteColor]] forBarPosition:0 barMetrics:UIBarMetricsDefault];
}
```

## UITabbar
tabbar和navigationBar的问题属于同一类，tabbar背景颜色设置失效，字体设置失效，阴影设置失效问题。

```
//旧代码
self.tabBar.backgroundImage = UIColor.white.image
self.tabBar.shadowImage = UIColor.init(0xEEEEEE).image
item.setTitleTextAttributes(norTitleAttr, for: .normal)
item.setTitleTextAttributes(selTitleAttr, for: .selected)
```


注意⚠️：首先是背景色设置失效，需要用UITabBarAppearance来设置

```
//新代码
if #available(iOS 15, *) {
    let bar = UITabBarAppearance.init()
    bar.backgroundColor = UIColor.white
    bar.shadowImage = UIColor.init(0xEEEEEE).image
    let selTitleAttr = [
        NSAttributedString.Key.font: itemFont,
        NSAttributedString.Key.foregroundColor: UIColor.theme
    ]
    bar.stackedLayoutAppearance.selected.titleTextAttributes = selTitleAttr // 设置选中attributes
    self.tabBar.scrollEdgeAppearance = bar
    self.tabBar.standardAppearance = bar
}
```


## 注意⚠️：iOS 15.4及以上，在12小时制下，通过字符串获取NSDate取到的值为nil

注意⚠️：iOS 15.4及以上，在12小时制下，通过NSDate获取时间字符串HH情况下依旧取得12小时制结果。

注意⚠️：苹果官方对NSDateFormatter的解释中可以看出，要想在任何时候输出固定格式的日期，需要设置.local。

```
##OC
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init];
dateFormatter.locale = [[NSLocale alloc] initWithLocaleIdentifier:@"zh_Hans_CN"];
// 或者
//dateFormatter.locale = [NSLocale systemLocale]; 
dateFormatter.calendar = [[NSCalendar alloc]initWithCalendarIdentifier:NSCalendarIdentifierISO8601];


##Swift
let formatter = DateFormatter()
formatter.locale = Locale.init(identifier: "zh_Hans_CN")
formatter.calendar = Calendar.init(identifier: .iso8601)

注意⚠️：如果我们需要使用日历相关，我们还需要设置dateFormatter的日历格式。
dateFormatter.calendar = [[NSCalendar alloc]initWithCalendarIdentifier:NSCalendarIdentifierISO8601];

+ (NSDateFormatter *)DateFormatter{
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init];
    dateFormatter.locale = [NSLocale systemLocale];
    dateFormatter.calendar = [[NSCalendar alloc]initWithCalendarIdentifier:NSCalendarIdentifierISO8601];
    return dateFormatter;
}

```





