---
title: "@RequestParam和@RequestBody的区别【servyou总结】"
date: 2019-05-21 15:17:46
tags: servyou总结
categories: 开发中常用注解
---
最近在公司写后台接口供前端调用，代码都是按照接口文档的要求实现的，并且自己用postman测试了一遍，参数取值都OK。接口文档的参数传递方式为：{xtdm:'001'}，请求方式都是POST，然后我就自然而然的采用了application/json方式测试。<!--more-->
采用application/json方式测试接口之后，发现后台springmvc接收参数使用@RequestParam注解是取不到值的，于是我换成@RequestBody注解就取到值了，当时问题解决。
后面整合前端联调接口时，发现前端请求到不了我的controller（http:400），但是postman始终是可以的。看了下前端的ajax请求，发现并不是按照文档的格式传递的参数。虽然是POST请求，但是地址栏也会跟参数过来，于是我尝试把@RequestBody又改为@RequestParam，结果又可以了，然后postman又不行了。

### 分析和结论
我开始采用application/json方式测试，@RequestBody是支持的，后来发现前端的请求方式其实是application/x-www-form-urlencoded，所以@RequestBody就不支持了。只有@RequestParam才支持，并且不管是POST还是GET请求都是OK的。后来我将postman请求方式改为application/x-www-form-urlencoded，验证结果跟前端的一致。所以不要着急写后台接口，一定要明确前端的请求方式是哪一种，这样就避免了返工的情况。
### @RequestParam和@RequestBody的使用场景
**@RequestParam**
① 支持POST和GET请求。
② 只支持Content-Type：为application/x-www-form-urlencoded编码的内容。Http协议中，如果不指定Content-Type，则默认传递的参数就是application/x-www-form-urlencoded类型）

**@RequestBody**
① 不支持GET请求。
② 必须要在请求头中申明content-Type（如application/json）springMvc通过HandlerAdapter配置的HttpMessageConverters解析httpEntity的数据，并绑定到相应的bean上。
### @RequestParam案例
使用@RequestParm用于绑定controller上的参数，可以是多个参数，也可以是一个Map集合，GET，POST均可。

```java
@PostMapping(value = "requestParam")
@ResponseBody
public Boolean requestParam(@RequestParam(name = "code",required = false) String code){
    System.out.println("code:"+code);
    return true;
}
```
@RequestParm中name属性是指定参数名，required属性默认为ture，表示必传。若为false则为非必传。属性有defaultValue默认值选项，若该参数为null时，会将默认值填充到参数上。
在项目中还可以使用map来接收多个参数：

```Java
@RequestMapping(value = "/menu.spring",method = RequestMethod.POST)
public NewFrameResponseResult queryTreeList(@RequestParam Map<String, Object> param){
    return NewFrameResponseResult.success(fyhsMenuTreeService.queryTreeList(param));
}
```

### @RequestBody案例
@RequestBody绑定一个实体。

```Java
@PostMapping(value = "requestBody")
@ResponseBody
public User requestBody(@RequestBody  User user){
    System.out.println("user:"+user.getName());
    return user;
}
```

@RequestBody也能采用map来接收多个参数。

```Java
@RequestMapping(value = "/menu.spring",method = RequestMethod.POST)
public NewFrameResponseResult queryTreeList(@RequestBody Map<String, Object> param){
    return NewFrameResponseResult.success(fyhsMenuTreeService.queryTreeList(param));
}
```
