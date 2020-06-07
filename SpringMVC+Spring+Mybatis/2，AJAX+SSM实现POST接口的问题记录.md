# 问题的记录和解决
## 关于RequestBody和RequestParam引发的问题
#### 联系与区别
```
    @CrossOrigin
    @RequestMapping(value = "/login", method = RequestMethod.POST, produces="application/json;charset=utf-8")
    @ResponseBody
    public Object getUserbyAccount(@RequestBody String msg) {
        //对msg做JSON解析
        //业务逻辑
    }
```
```
    @CrossOrigin
    @RequestMapping(value = "/login", method = RequestMethod.POST, produces="application/x-www-form-urlencoded")
    @ResponseBody
    public Object getUserbyAccount(@RequestParam("account") String account, @RequestParam("password") String password) {
        //业务逻辑
    }
```
  二者都可以用作接收POST参数的方式（GET后期尝试之后再添加），区别在于RequestBody可以接收Body中的字符串或者json信息 application/json;charset=utf-8，而RequestParam只能接收固定的格式，包括application/x-www-form-urlencoded， multipart/form-data这些有本身格式与form相关的格式。其用法如上面实例中所示。
  注意RequestParam访问不到Body中的数据
#### 对应的三种数据格式分别是什么？
* application/json;charset=utf-8：字面意思可以看出，就是JSON格式的信息，一般是储存再Request Body部分中。并不被原生form表单支持，需要单独取出每个input中的数据。
* application/x-www-form-urlencoded： 这其实是最常见的form表单提交方式，原生form表单如果不做特殊处理，就是直接用这个数据方式提交的。提交的数据按照 key1=val1&key2=val2 的方式进行编码，key和val都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。
* multipart/form-data：这个数据格式常用于文件上传，也是原生form表单支持的。本数据格式会产生一个--${bound}，用来分隔文件的几个部分。
## form表单中的提交数据如何用ajax取出来？要注意的问题有什么？
> 首先我们必须明确，ajax提交数据与<input type="submit">不能共存。我们需要将submit换成button，然后对form表单中的数据采取以下两种方式取出。  
* 方式一：
```
        var account = $.trim($("#account").val())
        var password = $.trim($("#password").val())
        var data = 
                    {
                        user_account:account,
                        user_password:password
                    }
        var dataSend = JSON.stringify(data)
```
这个方式是将数据按照id值一个一个取出，再拼接成JSON。
* 方式二：
```
        var data = $("#loginForm").serialize()
```
这个方式是将form中的数据直接以JSON的方式取出，data的格式是application/x-www-form。在后端采用RequestParam取出。
## CORS
在使用AJAX做前后端分离的数据传输时，会产生跨域问题。可以总结为，在使用XHR（XML HTTP Request）进行前端向后端的交互时，就会产生这个问题。需要在后端采用配置或注解或者filter的方式解决。  
具体的跨域问题在另一篇笔记中记录。