# iOS16
iOS 15 - iOS 16 开发 适配 总结帖

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
UITabbar
tabbar和navigationBar的问题属于同一类，tabbar背景颜色设置失效，字体设置失效，阴影设置失效问题。

//旧代码
self.tabBar.backgroundImage = UIColor.white.image
self.tabBar.shadowImage = UIColor.init(0xEEEEEE).image
item.setTitleTextAttributes(norTitleAttr, for: .normal)
item.setTitleTextAttributes(selTitleAttr, for: .selected)
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



