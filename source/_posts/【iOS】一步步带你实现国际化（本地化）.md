---
title: 【iOS】一步步带你实现国际化（本地化）
date: 2019-03-20 13:48:23
tags:
---





# 一、基本设置

### 1. 先在 Project 的 Info 里添加项目所要支持的语言

![添加需要支持的语言](https://upload-images.jianshu.io/upload_images/2910208-93ea57da91ea9116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 设置 .strings 文件（多语言文件）



这里先说明一下关于 NSLocalizedString 的坑点：

- 当你使用第一种方法实现多语言本地化，意味着你的 .strings 文件名字必须是 `Localizable `,否则你的多语言实现，到最后显示的却只能是key，所以这种方法适合于让应用跟随系统的语言版本进行切换，而用户却不能手动进行切换。
- 如果我们要实现的是既可以跟随系统，又可以用户自己切换的话，那我们就需要用第三种方法

```
1. NSLocalizedString(<#key#>, <#comment#>)
2. NSLocalizedStringFromTable(<#key#>, <#tbl#>, <#comment#>)
3. NSLocalizedStringFromTableInBundle(<#key#>, <#tbl#>, <#bundle#>, <#comment#>)
4. NSLocalizedStringWithDefaultValue(<#key#>, <#tbl#>, <#bundle#>, <#val#>, <#comment#>)
```


⌘ + N 开始创建 .strings 文件，我这里是要实现可以跟随系统，又可以用户自己切换语言，所以我这里命名为 LocalizationTest

![创建 .strings 文件](https://upload-images.jianshu.io/upload_images/2910208-06d89e893c976041.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



选中你创建的.string文件，点击 Xcode 右边栏中的 Localization 中的按钮，添加需要的语言包

![添加需要的语言包](https://upload-images.jianshu.io/upload_images/2910208-9aa08ede46b5d3f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加所要支持的语言，将其勾选上](https://upload-images.jianshu.io/upload_images/2910208-820659f0c6c2e10a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



添加完毕后系统会自动帮我们生成对应的语言包文件

![语言包文件](https://upload-images.jianshu.io/upload_images/2910208-01aad9f9af264270.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在对应的语言包文件中添加需要转换语言的字符串即可，格式为：`"Key" = "对应的Value";`

![](https://upload-images.jianshu.io/upload_images/2910208-e3e0486593930c6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2910208-4055f7c3f0db2bbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 二、Code部分

创建一个工具类来切换语言
```
#import <Foundation/Foundation.h>

typedef enum : NSUInteger {
    kChinese = 0,
    kEnglish,
} Language;


@interface LocalizedTool : NSObject

+ (NSBundle *)bundle;//获取当前资源文件

+ (void)initLanguage;//初始化语言文件

+ (Language)language;//获取应用当前语言

+ (void)setLanguage:(Language)language;//设置当前语言

@end
```
```
#import "LocalizedTool.h"

@implementation LocalizedTool

static NSBundle *bundle = nil;
+ (NSBundle *)bundle {
    return bundle;
}

//首次加载时检测语言是否存在
+ (void)initLanguage {
    NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
    NSString *currLanguage = [def valueForKey:@"LocalLanguageKey"];
    if (!currLanguage) {
        NSArray *preferredLanguages = [NSLocale preferredLanguages];
        currLanguage = preferredLanguages[0];
        if ([currLanguage hasPrefix:@"en"]) {
            currLanguage = @"en";
        } else {
            currLanguage = @"zh-Hans";
        }
        [def setValue:currLanguage forKey:@"LocalLanguageKey"];
        [def synchronize];
    }
    //获取文件路径
    NSString *path = [[NSBundle mainBundle] pathForResource:currLanguage ofType:@"lproj"];
    bundle = [NSBundle bundleWithPath:path];//生成bundle
}

//获取当前语言
+ (Language)language {
    NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
    NSString *language = [def valueForKey:@"LocalLanguageKey"];
    if ([language isEqualToString:@"en"]) {
        return kEnglish;
    }
    return kChinese;
}

//设置语言
+ (void)setLanguage:(Language)language {
    NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
    NSString *currLanguage = [userDefaults valueForKey:@"LocalLanguageKey"];
    
    NSString *languageStr = language == kChinese ? @"zh-Hans" : @"en";
    if ([currLanguage isEqualToString:languageStr]) {
        return;
    }
    [userDefaults setValue:languageStr forKey:@"LocalLanguageKey"];
    [userDefaults synchronize];
    
    NSString *path = [[NSBundle mainBundle] pathForResource:languageStr ofType:@"lproj"];
    bundle = [NSBundle bundleWithPath:path];
}

@end
```
在AppDelegate中初始化应用语言，每次启动app都会从本地提取用户上次选定的语种
```
//初始化应用语言
- (void)setLanguage {
    [LocalizedTool initLanguage];
    [LocalizedTool setLanguage:[LocalizedTool language]];
}
```
项目中所有需要转换语言的字符串都要用系统提供的宏来写，为了方便调用，我们可以再写一个宏
```
/**
 系统提供的宏
 @param key     字符串对应的key
 @param tbl     你创建的.string文件的名称
 @param bundle  语言包所在的bundle
 @param comment 对key的描述，一般使用时候直接就是nil就行（在工具类中生成了bundle文件，所以这里固定为[LocalizedTool bundle]）
 */
NSLocalizedStringFromTableInBundle(<#key#>, <#tbl#>, <#bundle#>, <#comment#>)

// 自写宏定义
#define LocalizedStringWithKey(key) NSLocalizedStringFromTableInBundle(key, @"LocalizationTest", [LocalizedTool bundle], nil)

```
写一个切换语言的按钮，切换后刷新UI和完成其他操作
```
// 切换语言
- (IBAction)onSwitch:(UISwitch *)sender {
    if ([LocalizedTool language] == kChinese) {
        [LocalizedTool setLanguage:kEnglish];
    } else {
        [LocalizedTool setLanguage:kChinese];
    }

    self.languageLab.text = [LocalizedTool language] == kChinese ? @"中文" : @"英文";
    self.titleLab.text = LocalizedStringWithKey(@"Title");
}
```



# 三、App名称的国际化

关于App名称的修改，方法是一样的，只是我们需要定义一个名称为 `InfoPlist` 的 .string 文件，必须是这个，我们知道 CFBundleDisplayName 是程序的名称，所以我们把它当成key，后边写入对应语言的App名称即可。最后切换手机的语言来验证是否成功

![](https://upload-images.jianshu.io/upload_images/2910208-43a9ef8fc74f0dba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 参考文章

- [https://www.cnblogs.com/whongs/p/6727610.html](https://www.cnblogs.com/whongs/p/6727610.html)
- [https://www.jianshu.com/p/b5959505f18e](https://www.jianshu.com/p/b5959505f18e)
- [https://www.jianshu.com/p/3c211fd81be7](https://www.jianshu.com/p/3c211fd81be7)



# Demo

- [https://github.com/Mingor/LocalizationDemo](https://github.com/Mingor/LocalizationDemo)