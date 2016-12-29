# caas

caas还在原型设计状态，不可用。

下面几个维度可能决定最终配置文件的差异。

- 客户端版本
- 服务器版本
- 城市
- 设备
- token

## Objective-C

```Objective-C
/**
 * 第二个参数是默认值
 */
(void *)get(NSString *key, ...);
(int)getInt(NSString *key, ...);
(float)getFloat(NSString *key, ...);
(NSString)getString(NSString *key, ...);

get("system.site.name")
// will print http://example.com

get("ui.homepage.font_size", 12);
// 如果不存在，将返回12

get("ui.homepage.banner[0].image_url")
// [index] 

get("ui.homepage.banner[last]") 

get("ui.homepage.banner[-2]")

get("current.sitename", "shmg");

get("site.${current.sitename}.articles)

get("sites[@site.name=SHMG]", "shmg");
// current.sitename 作为变量访问指定的site的属性名

```
## Objective-C中初步设计
```Objective-C

CSConfigAPI.h
      |----CSConfigAPIDelegate：包含请求的所有回调
CSConfigObject.h -------------- config的基类
CSBaseConfigRequest.h --------- config的请求基类
CSBaseConfigResponse.h -------- config的响应基类

```
```Objective-C
用法：
@interface DemoViewController () <CSConfigAPIDelegate>

@property (nonatomic, strong) CSConfigAPI *configAPI;

@property (nonatomic, strong) CSBaseConfigRequest *baseConfigRequest;

@end

@implementation DemoViewController

#pragma mark - Life Cycle
- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.title = @"测试";
    
    // 调用获取配置信息
    [self.configAPI loadBaseConfig:self.baseConfigRequest];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
}

#pragma mark - CSConfigAPIDelegate
- (void)onBaseConfigDone:(CSBaseConfigRequest *)request reponse:(CSBaseConfigResponse *)response {
    // 获取配置信息成功
}

- (void)configRequest:(id)request didFailWithError:(NSError *)error {
    // 获取配置信息失败
}

#pragma mark - Init
- (CSConfigAPI *)configAPI {
    if (!_configAPI) {
        _configAPI = [[CSConfigAPI alloc] init];
        _configAPI.delegate = self;
    }
    return _configAPI;
}

- (CSBaseConfigRequest *)baseConfigRequest {
    if (!_baseConfigRequest) {
        _baseConfigRequest = [[CSBaseConfigRequest alloc] init];
        _baseConfigRequest.configPath = @"https//www.baidu.com/";// 配置的路径
        _baseConfigRequest.params = @[@"banners"];// 配置的参数
    }
    return _baseConfigRequest;
}

@end
```


## 本地配置文件作为数据源

```
{
  current: {
  	sitename: "shmg"
  },
  sites: [
    {
      "name": "SHMG",
      "articles": [
      ]
    },
    {
      "name": "SHMG Incubator"
    }
  ]
}
```

## HTTP API

caas提供一组api接口用于读取和保存配置信息。客户端使用http api接口读取配置，接口返回json格式的数据。

## 查询配置文件
```
GET /?module=system HTTP 1.1
HOST: host

HTTP/1.1 200 OK
Content-Type: application/json

{
  "name": "SHMG"
}
```

## 请求参数

请求参数是 QueryString

- module string 如system可以得到system这个模块的所有数据

## 修改配置文件

配置键值

```
POST /?module=system HTTP 1.1
HOST: host

HTTP/1.1 200 OK
Content-Type: text/html

keypath=system.site.name&value=SHMG
```

追加到数组的尾部

```
keypath=system.site.name[]&value=SHMG
```

设置Object

```
keypath=system.site.image&value={
  "url": "",
  "context-type": "image/jpeg"
}
```


