

+++

title = "YuanyaTools"
description = "YuanyaTools"
tags = ["it", "go","project"]

+++



# YuanyaTools



## Excel

https://github.com/360EntSecGroup-Skylar/excelize

安装

```shell
github.com/360EntSecGroup-Skylar/excelize
```

如果使用Go Modules管理软件包，安装v2

```shell
github.com/360EntSecGroup-Skylar/excelize/v2
```

### 读

```go
import (
	"github.com/360EntSecGroup-Skylar/excelize"
)
func readExcel(excelPath string) [][]string {
	xlsx, err := excelize.OpenFile(excelPath)
	if err != nil {
		fmt.Println("open excel error,", err.Error())
		os.Exit(1)
	}
	sheetIndex := xlsx.GetActiveSheetIndex()   //获取激活状态下sheetIndex
	sheetName := xlsx.GetSheetName(sheetIndex) //根据sheetIndex获取sheetName
	rows, err := xlsx.GetRows(sheetName)       //根据sheetName获取cells的内容
	return rows
}
```



## 短信

### 阿里云

https://github.com/aliyun/alibaba-cloud-sdk-go

1. 阿里云控制台短信服务：https://dysms.console.aliyun.com/dysms.htm

2. 国内消息 - 签名管理 - 添加签名

   - 验证码：随便填写合法合理内容，就可以通过审核
   - 通用：需要签名来源证明，否则无法通过审核

3. 国内消息 - 模板管理 - 添加模板。得到模版CODE

   - 验证码：随便填写合法合理内容
   - 短信通知
   - 推广短息

4. 概览 - AccessKey - 创建AccessKey。得到AccessKeySecret

5. 程序 https://api.aliyun.com/new#/?product=Dysmsapi

   ```go
   func SendMsmAuthCode() {
   	client, err := dysmsapi.NewClientWithAccessKey("REGION_ID"/*区域ID，如cn-chengdu*/,
   		"ACCESS_KEY_ID"/*概览-AccessKey-AccessKeyId*/, "ACCESS_KEY_SECRET"/*概览-AccessKey-AccessKeyId*/)
   	if err != nil {
   		panic(err)
   	}
   
   	request := dysmsapi.CreateSendSmsRequest()
   	request.Scheme = "https"
   
   	request.PhoneNumbers = "PHONE"         //接收短信的手机号
   	request.SignName = "SIGN_NAME"         //国内消息-签名管理-签名名称
   	request.TemplateCode = "TEMPLATE_CODE" //国内消息-模板管理-模板CODE
   
   	authCode := fmt.Sprintf("%06v", rand.New(rand.NewSource(time.Now().UnixNano())).Int31n(1000000))
   	par, err := json.Marshal(map[string]interface{}{
   		"code": authCode, //验证码
   	})
   	request.TemplateParam = string(par)
   
   	response, err := client.SendSms(request)
   	if err != nil {
   		fmt.Print(err.Error())
   	}
   	fmt.Printf("response is %#v\n", response)
   }
   ```

   



## 图形验证码

在服务器端负责生成图形化验证码，并以数据流的形式供前端访问获取，同时将生成的验证码存储到全局的缓存中，在本案例中，我们使用redis作为全局缓存，并设置缓存失效时间。当用户使用用户名和密码进行登录时，进行验证码验证。验证通过即可继续进行登录

一个开源的验证码工具库：https://github.com/mojocn/base64Captcha

### Redis

```go
//redis_config.json配置文件
"redis_config": {
    "addr": "127.0.0.1",
    "port": "6379",
    "password": "",
    "db": 0
}
```

```go
type RedisConfig struct {
    Addr string json:"addr"
    Port string json:"port"
    Db   int    json:"db"
}

```

进行了redis配置以后，需要对redis进行初始化。可以封装redis初始化操作函数

```go
type RedisStore struct {
    redisClient *redis.Client
}

var Redis *redis.Client

func InitRediStore() *RedisStore {
    config := GetConfig().RedistConfig

    Redis = redis.NewClient(&redis.Options{
        Addr:     config.Addr + ":" + config.Port,
        Password: "",
        DB:       config.Db,
    })

    customeStore := &RedisStore{Redis}
    base64Captcha.SetCustomStore(customeStore)

    return customeStore
}
```

为customeStore提供Set和Get两个方法

```go
func (cs *RedisStore) Set(id string, value string) {
    err := cs.redisClient.Set(id, value, time.Minute*10).Err()
    if err != nil {
        log.Println(err.Error())
    }
}

func (cs *RedisStore) Get(id string, clear bool) string {
    val, err := cs.redisClient.Get(id).Result()
    if err != nil {
        toolbox.Error(err.Error())
        return ""
    }
    if clear {
        err := cs.redisClient.Del(id).Err()
        if err != nil {
            toolbox.Error(err.Error())
            return ""
        }
    }
    return val
}
```

对Redis进行初始化和定义完成以后，需要在main中调用一下初始化操作InitRediStore

```go
func main(){
    ...
    //Redis配置初始化
     toolbox.InitRediStore()
    ...
}
```



### 验证码生成和验证

本项目中采用的验证码的生成库支持三种验证码，分别是：audio，character和digit。我们选择character类型。定义Captcha.go文件，实现验证码的生成和验证码函数的定义。在进行验证码生成时，默认提供验证码的配置，并生成验证码后返回给客户端浏览器。如下是生成验证码的函数定义

```go
//生成验证码
func GenerateCaptchaHandler(w http.ResponseWriter, r *http.Request) {

    // parse request parameters
    // 接收客户端发送来的请求参数

    postParameters := ConfigJsonBody{
        CaptchaType: "character",
        ConfigCharacter: base64Captcha.ConfigCharacter{
            Height:             60,
            Width:              240,
            Mode:               3,
            ComplexOfNoiseText: 0,
            ComplexOfNoiseDot:  0,
            IsUseSimpleFont:    true,
            IsShowHollowLine:   false,
            IsShowNoiseDot:     false,
            IsShowNoiseText:    false,
            IsShowSlimeLine:    false,
            IsShowSineLine:     false,
            CaptchaLen:         6,
            BgColor: &color.RGBA{
                R: 3,
                G: 102,
                B: 214,
                A: 254,
            },
        },
    }
    // 创建base64图像验证码

    var config interface{}
    switch postParameters.CaptchaType {
    case "audio":
        config = postParameters.ConfigAudio
    case "character":
        config = postParameters.ConfigCharacter
    default:
        config = postParameters.ConfigDigit
    }
    captchaId, captcaInterfaceInstance := base64Captcha.GenerateCaptcha(postParameters.Id, config)
    base64blob := base64Captcha.CaptchaWriteToBase64Encoding(captcaInterfaceInstance)

    // 设置json响应
    w.Header().Set("Content-Type", "application/json; charset=utf-8")
    body := map[string]interface{}{"code": 1, "data": base64blob, "captchaId": captchaId, "msg": "success"}
    json.NewEncoder(w).Encode(body)
}
```

验证码接口解析：图形化验证码是用户名和密码登录功能的数据，属于Member模块。因此在MemberController中增加获取验证码的接口解析，如下

```go
func (mc *MemberController) Router(engine *gin.Engine){
    //获取验证码
    engine.GET("/api/captcha", mc.captcha)
}
```

验证码验证：

```go
func CaptchaVerify(r *http.Request) bool {
    //parse request parameters
    //接收客户端发送来的请求参数
    decoder := json.NewDecoder(r.Body)
    var postParameters ConfigJsonBody
    err := decoder.Decode(&postParameters)
    if err != nil {
        log.Println(err)
    }

    defer r.Body.Close()

    //比较图像验证码
    verifyResult := base64Captcha.VerifyCaptcha(postParameters.Id, postParameters.VerifyValue)
    return verifyResult
}
```



