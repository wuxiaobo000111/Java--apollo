
# 概述

这里介绍一下swagger的常用注解@Api、@ApiOperation、@ApiImplicitParams、@ApiImplicitParam、@ApiParam、@ApiModel、@ApiModelProperty、ApiResponses、@ApiResponse这几个常用的。


# @Api

用在请求的类上，表示对类的说明

|  属性 | 描述  |
|--|--|
| tags |  说明该类的作用，非空时将覆盖value的值|
| value |  描述类的作用|
| description |  对api资源的描述，在1.5版本后不再支持|
| basePath |  基本路径可以不配置，在1.5版本后不再支持|
| position|  如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持|
| produces |  设置MIME类型列表（output），例："application/json, application/xml"，默认为空|
| consumes |设置MIME类型列表（input），例："application/json, application/xml"，默认为空|
| protocols |   设置特定协议，例：http， https， ws， wss。|
| authorizations |   获取授权列表（安全声明），如果未设置，则返回一个空的授权值。|
| hidden |   默认为false， 配置为true 将在文档中隐藏|


## 示例代码

```java
        @Api(tags="登录请求")
        @Controller
        @RequestMapping(value="/highPregnant")
        public class LoginController {}
```

# ApiOperation

用在请求的方法上，说明方法的用途、作用


|  属性 | 描述  |
|--|--|
| value |  说明方法的用途、作用|
| notes |  方法的备注说明|
| tags |   操作标签，非空时将覆盖value的值|
| response |   响应类型（即返回对象）|
| responseContainer |   声明包装的响应容器（返回对象类型）。有效值为 "List", "Set" or "Map"。|
| responseReference |  指定对响应类型的引用。将覆盖任何指定的response（）类|
| httpMethod |  指定HTTP方法，"GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH"|
| position |   如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持|
| nickname |  第三方工具唯一标识，默认为空|
| produces |  设置MIME类型列表（output），例："application/json, application/xml"，默认为空|
| consumes |  设置MIME类型列表（input），例："application/json, application/xml"，默认为空|
| protocols |  设置特定协议，例：http， https， ws， wss。|
| authorizations |   获取授权列表（安全声明），如果未设置，则返回一个空的授权值。|
| hidden |   默认为false， 配置为true 将在文档中隐藏|
| responseHeaders |   响应头列表|
| code |    响应的HTTP状态代码。默认 200|
| extensions | 扩展属性列表数组|

## 示例代码

```java
        @ResponseBody
        @PostMapping(value="/login")
        @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
        public UserModel login(@RequestParam(value = "name", required = false) String account,
        @RequestParam(value = "pass", required = false) String password){}
```


# ApiImplicitParams

用在请求的方法上，表示一组参数说明,@ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面

|  属性 | 描述  |
|--|--|
| name |  参数名，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致|
| value |  参数的汉字说明、解释|
| required |  参数是否必须传，默认为false [路径参数必填]|
| paramType | 1. header --> 请求参数的获取：@RequestHeader 2. query --> 请求参数的获取：@RequestParam,3. path（用于restful接口）--> 请求参数的获取：@PathVariable 4.body（不常用）,5.form（不常用）|
| dataType |  参数类型，默认String，其它值dataType="Integer"|
| defaultValue | 参数的默认值|
| allowableValues | 限制参数的可接受值。1.以逗号分隔的列表   2、范围值  3、设置最小值/最大值|
| access | 允许从API文档中过滤参数。|
| allowMultiple |   指定参数是否可以通过具有多个事件接受多个值，默认为false|
| example   | 单个示例|
| examples| 参数示例。仅适用于BodyParameters|


## 示例代码

```java
        @ResponseBody
        @PostMapping(value="/login")
        @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
        @ApiImplicitParams({
                    @ApiImplicitParam(name = "name", value = "用户名", required = false, paramType = "query", dataType = "String"),
                    @ApiImplicitParam(name = "pass", value = "密码", required = false, paramType = "query", dataType = "String")
            })
        public UserModel login(@RequestParam(value = "name", required = false) String account,
                @RequestParam(value = "pass", required = false) String password){}

```


# @ApiModel

用于响应类上，表示一个返回响应数据的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）@ApiModelProperty：用在属性上，描述响应类的属性


@ApiModelProperty的属性值



|  属性 | 描述  |
|--|--|
| value |  此属性的简要说明。|
| name |  允许覆盖属性名称|
| name |  参数名，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致|
| allowableValues| 限制参数的可接受值。1.以逗号分隔的列表   2、范围值  3、设置最小值/最大值|
| access | 允许从API文档中过滤属性。|
| notes   |  目前尚未使用。|
|dataType            |参数的数据类型。可以是类名或者参数名，会覆盖类的属性名称。|
|required           | 参数是否必传，默认为false|
|position           | 允许在类中对属性进行排序。默认为0|
| hidden            | 允许在Swagger模型定义中隐藏该属性。|
| example             |   属性的示例。|
|readOnly          |  将属性设定为只读。|
| reference       |    指定对相应类型定义的引用，覆盖指定的任何参数值|

## 示例代码

```java
 @ApiModel(value="用户登录信息", description="用于判断用户是否存在")
        public class UserModel implements Serializable{

           private static final long serialVersionUID = 1L;

           /**
            * 用户名
            */
           @ApiModelProperty(value="用户名")
           private String account;

           /**
             * 密码
             */
            @ApiModelProperty(value="密码")
           private String password;


        }
```

# @ApiResponses

用在请求的方法上，表示一组响应@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息


|  属性 | 描述  |
|--|--|
|code|数字，例如400|
|message|信息，例如"请求参数没填好"|
|response|抛出异常的类|

## 示例代码

```java
        @ResponseBody
        @PostMapping(value="/update/{id}")
       @ApiOperation(value = "修改用户信息",notes = "打开页面并修改指定用户信息")
        @ApiResponses({
            @ApiResponse(code=400,message="请求参数没填好"),
            @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
        })
        public JsonResult update(@PathVariable String id, UserModel model){}

```

# ApiParam

用在请求方法中，描述参数信息

|  属性 | 描述  |
|--|--|
|  name|  参数名称，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致|  
|  value|  参数的简要说明。|  
|  defaultValue|  参数默认值|  
|  required |  属性是否必填，默认为false [路径参数必须填]|  


## 示例代码

```kava
         @ResponseBody
         @PostMapping(value="/login")
         @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
         public UserModel login(@ApiParam(name = "name", value = "用户名", required = false) @RequestParam(value = "name", required = false) String account,
                @ApiParam(name = "pass", value = "密码", required = false) @RequestParam(value = "pass", required = false) String password){}
```