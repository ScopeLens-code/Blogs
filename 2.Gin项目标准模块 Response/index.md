### Response的意义
在这种前后端分离的WEB工程之中，前后端交互的方式就是通过网络通信。  
其中前端能获取到的信息就包括HTTP请求中提供的HTTP状态码，以及其中携带的返回数据。  

在开发中我们一般会将其分为两个层，网络层(HTTP)和业务层(数据)。

HTTTP状态码是表示网络请求的状态，例如404 not found、200 正常通信、500 后端错误，在常用的网络请求库*axios*中遇到非200的状态码是会直接抛出错误，这些状态码往往展现的是网络请求的异常，只有一些特定的状态码用来表示业务状态，比如401无权限，但是这些都算是约定俗成的东西，具体都看你们前后端如何协作。  

而网络层的状态往往并不能详细表示业务层面的状态，例如虽然网络请求是200成功访问了，但是后端并没有返回数据，其中可能存在很复杂的原因，例如没有权限，查询条件错误，查询结果为空等等，我们并不能直接通过使网络请求返回500来笼统回复前端，这会使得前端难以给予用户正确的提示，降低用户体验。  

### Response的封装
通过前面的叙述，我们可以知道，在业务层来展示操作结果，并且包括明细是有必要的。

下面我就直接来展示我的封装结构，这并不是一个复杂的结构，仅仅提供一个思考方向。

#### 1.状态码与提示信息：
在业务层使用与网络层类似的架构理解起来更简单，表达也更明晰。

```
//标准结构
type CodeMsg struct {
	Code int    //状态码
	Msg  string //提示信息
}

// 常用业务错误码
var (
	Success      = CodeMsg{0, "success"}
	ServerError  = CodeMsg{500, "服务器内部错误"}
	InvalidParam = CodeMsg{400, "参数错误"}
	Unauthorized = CodeMsg{401, "未授权"}
	Forbidden    = CodeMsg{403, "无权限"}
	NotFound     = CodeMsg{404, "资源不存在"}
)

// 可选：支持自定义错误
func NewCodeMsg(code int, msg string) CodeMsg {
	return CodeMsg{Code: code, Msg: msg}
}
```


#### 2.返回体：
一个功能完备且标准的返回体会让前后端交互变得轻松，用来传递对于的数据。

```
type Response struct {
	Code    int         `json:"code"`            // 状态码
	Message string      `json:"message"`         // 提示信息
	Data    interface{} `json:"data,omitempty"`  // 数据
	Total   int64       `json:"total,omitempty"` // 分页总数，可选
}

几乎所有的api返回体都可以通过这个结构进行处理，例如登录状态检测，分页数据返回
e.g.
{
    "code": 0,
    "message": "success", //表示后端正常接收到请求
    "data": {
        "isLogin": true //判断登录状态
    }
}
```

#### 3.构造器：
一个标准的返回体自然需要一个构造器去直观的创建，直接使用构建者模式(Builder)
```
通过链式调用来插入数据，构造返回体
type RespBuilder struct {
	resp Response
}

func New() *RespBuilder {
	return &RespBuilder{resp: Response{}}
}

func (r *RespBuilder) WithCodeMsg(cm CodeMsg) *RespBuilder {
	r.resp.Code = cm.Code
	r.resp.Message = cm.Msg
	return r
}

func (r *RespBuilder) WithMessage(msg string) *RespBuilder {
	r.resp.Message = msg
	return r
}

func (r *RespBuilder) WithData(data interface{}) *RespBuilder {
	r.resp.Data = data
	return r
}

func (r *RespBuilder) WithTotal(total int64) *RespBuilder {
	r.resp.Total = total
	return r
}

func (r *RespBuilder) JSON(c *gin.Context) {
	c.JSON(http.StatusOK, r.resp)
}

```

#### 4.常用封装：
虽然做到前三部中就已经可以正常使用了，但是在实际生产的情况下，我们会对使用率高的链式调用进行二次封装
```
//请求成功，传入上下文依赖和返回的数据，code设置为0
func RespSuccess(c *gin.Context, data interface{}) {
	New().WithCodeMsg(Success).WithData(data).JSON(c)
}

//请求失败，传入上下文依赖和标准错误信息
func RespFail(c *gin.Context, cm CodeMsg) {
	New().WithCodeMsg(cm).JSON(c)
}

//请求失败，传入上下文依赖和自定义错误信息
func RespFailMsg(c *gin.Context, cm CodeMsg, msg string) {
	New().WithCodeMsg(cm).WithMessage(msg).JSON(c)
}

//请求成功，传入上下文依赖和分页数据
func RespPage(c *gin.Context, data interface{}, total int64) {
	New().WithCodeMsg(Success).WithData(data).WithTotal(total).JSON(c)
}

//通用函数：自定义 HTTP 状态码 + CodeMsg + 可选 message
func RespWithStatus(c *gin.Context, status int, cm CodeMsg, msg ...string) {
	message := cm.Msg
	if len(msg) > 0 && msg[0] != "" {
		message = msg[0]
	}
	c.JSON(status, Response{
		Code:    cm.Code,
		Message: message,
	})
}

e.g.
//这是一个代码片段，用来创建blog数据
if err := ctx.ShouldBindJSON(&blogCreateDetail); err != nil { //获取前端传递参数
		response.RespFail(ctx, response.InvalidParam) //获取失败
		return
	}

blogUpdateDetail, err := service.CreateBlog(&blogCreateDetail)//创建数据库数据
if err != nil {
    response.RespFail(ctx, response.ServerError) //服务端创建数据失败
    return
}

······其他操作

response.RespSuccess(ctx, response.Success) //构建成功
```

### 结束
一个好的返回结构会让前后端开发不容易吵架，开发更轻松，不过具体的情况还是看自己的公司，自己的团队如何制定这个标准，符合自己开发习惯的才是最好的。