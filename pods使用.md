
###使用：

pod install :  为每一个它安装的pod库在Podfile.lock文件中写入其版本号，pod install会只下载Podfile.lock文件中指定的版本，而不会去检查这个库是否有更新的版本。对于不在Podfile.lock文件中的pod库，pod install会搜索这个pod库在Podfile文件中指定的版本

pod update :  先联网更新本地索引文件。忽略podfile.lock文件，直接以podfile文件描述安装所有框架，没有指定版本号的安装最新版本，安装完成后重新生成podfile.lock

pod update --no-repo-update : 不联网更新，使用本地索引文件更新，然后同上。

本地repo位置 cd ~/.cocoapods/repos/    : 存放的都是.spec索引文件，不是实际源码.

---

##原理：

####概念：
* 远程索引库(.spec文件)： 存放着所有发布的第三方框架的描述信息，包括：1.框架名称。2.版本。3.源码地址。
* pod setup : 把远程索引库克隆下来，生成本地索引库。
详细见图

---

###spec
需要填的信息：	

```
	
  	s.name         = "MGLiveChannel"      // 框架名称
  	
  	s.version      = "0.0.1"					// 版本 
  	
  	s.summary      = "MGLiveChannel beta" // 简要描述
  	
  	s.description  = "MGLiveChannel beta : beta for develop & test" // 详细描述信息--要比summary长!
  	
 	s.homepage     = "https://gitee.com/eliswee/MGVideoRepo" // 项目首页
	
	s.license      = "MIT" // 许可协议
	
	s.author       = { "周丰泽" => "zhoufengze@migu.cn" }  // 作者信息
	
	s.platform    = :ios, "8.0" // 最低版本
	
	s.source      = { :git => "https://gitee.com/eliswee/MGVideoRepo.git", :tag => "#{s.version}" } // 源码地址---以(s.version)为tag到源码仓库找源码！！！
	
	s.source_files  = "Classes", "Classes/**/*.{h,m}"  // 找到源码地址以后要下载安装的文件路径, **表示通配符--通配所有目录。*.{h,m} ：表示所有的.h 和 .m 文件
	
	s.exclude_files = "Classes/Exclude" // 这是啥？
	
	# s.resource  = "icon.png"  // 资源？
  	# s.resources = "Resources/*.png" // 资源？
  	
  	s.requires_arc = true // 需要ARC环境
  	s.resources    = ['MGLiveChannel/MGLiveChannel.xcassets'] // 依赖资源
  	s.dependency 	'Masonry' // 依赖库
  	s.dependency 	'MGPlayerDetail' // 依赖库
	
```
 	
 ---      

####通过trunk上传自己的.spec到cocoapods官方的索引库：
注册trunk:		
 	
```
pod trunk register 343185087@qq.com 'zhoufengze'  --verbose
// --verbose 表示打印log日志
// 收到邮件后打开连接验证邮箱

pod trunk push xxx.spec  
// 推送.spec文件到 https://github.com/CocoaPods/Specs (需要审核)
// 也可以到仓库地址先fork, 然后 pull request  , 等待作者同意
// push过程会验证.spec文件内容

pod repo remove NAME
// 删除库NAME Deletes the remote named `NAME` from the local spec-repos directory at
      `~/.cocoapods/repos/.`
      
 pod repo add master http://git.oschina.net/akuandev/Specs.git
 // 添加索引库
 
 pod setup
 // 初始化(下载服务器中所有第三方框架信息, 缓存到电脑本地)
 
  ~/Library/Caches/CocoaPods/
  //索引缓存路径：如果发现框架信息本地已经缓存, 但是无法搜索框架, 可以删除这个索引文件, 重新生成
  
 
```

---

使用本地库： 

podfile 的ruby语法:

	.spec 里面的 s.source为空

	pod 'libName' , :path => '../lib/'   // 使用本地的.spec路径, .spec应该和库同目录--库已经提交到本地git--已有.spec中对应tag的版本, pod install 导入的框架在Pods工程的Development Pods目录下

---

pod远程私有库：

```
-1. 
先创建远程索引库.


pod repo add 框架名称 地址.git

1.
pod lib create NAME  // 创建 框架模板

2.
把框架源码放入 LIBNAME/Classes/

3.移动到 Example ， pod install

3. 打开Example内的工程，编译，修改 .podspec 

4. 回到 .podspec 的目录 , pod lib lint 本地验证   如果框架依赖私有库，则要自定义源：pod spec lint --sources='http://183.192.162.101:8089/clients/mgSpecs.git,https://github.com/CocoaPods/Specs' --verbose  --allow-warnings --use-libraries
本地验证不会验证tag、如果依赖.a静态库要加命令：--use-libraries

5. pod spec lint  远程验证
远程仓库必须有对应代码 且 有对应 tag, 就是把当前创建的 模板 上传到框架库.git

6. pod repo push  索引库库名  库名.podspec
索引库要是已经添加过的 ， pod repo里面有的，  这一步实际上是往本地的库路径放入.podspec ,然后会自动上传到远端
pod repo push 162-clients-mgspecs MGLogin.podspec --verbose  --allow-warnings --use-libraries

7. podfile 添加私有库源，用起来。
source 'source 'http://183.192.162.101:8089/clients/mgSpecs.git''


8.更新框架代码 :
要先修改 .podspec  的s.version 
把框架源码tag更新，然后推送框架源码到git
根据索引库名称提交 .podspec 到本地索引库
-如果在提交.podspec时发现本地索引库有变更(比如系统生成的.DS_Store), 保存索引库地址，然后pod remove索引库，再pod add索引库    
如果给本地框架添加文件，要执行一次pod install
```	

---
##资源问题：

1. 库的bundle 应该用 bundleforClass 方法加载bundle
2. 图片资源的加载:	

```
// [UIImage imgageNamed: ]方法会在car(you), mainBundle 中找图片
    NSBundle *currentBundle = [NSBundle bundleForClass:[self class]];
    // 名称要写全xxx@2x.png
    NSString *imagePath = [currentBundle pathForResource:@"tabbar_bg@2x.png" ofType:nil inDirectory:@"XMGFMMain.bundle"];
    UIImage *image = [UIImage imageWithContentsOfFile:imagePath];
```
适配问题： 可以用runtime拦截加载图片，[UIScreen mainScreen].scale 方法判断当前屏幕，然后拼接@2x 或者@3x
---

拆分子库：在.podspec中写：

```
s.subspec 'Base'  do |z| // 子库的别名
z.source_files = '路径' // 子库的配置
z.dependency 'AFNetworking' // 子库的依赖
end

// 使用: podfile中：

pod 'XXXX/Base'
pod 'XXXX/Category'
或者：
pod 'XXXX', :subspecs => ['Base', 'Category']

```



