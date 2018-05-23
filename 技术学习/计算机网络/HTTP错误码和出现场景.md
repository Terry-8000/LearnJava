# HTTP错误码和出现场景

### 405

```
"error": "Method Not Allowed",
"message": "Request method 'GET' not supported",
```

**场景：**

当发布的服务不支持当前请求形式的时候就会报405错误。比如上面这个错误是由于我通过Spring Boot发布了的服务是post的形式，而我请求的方式是get。

### 400

```
"error":"Bad Request",
"message":"Required String parameter 'name' is not present",
```

**场景：**

由于语法格式有误，服务器无法理解此请求。比如上面的错误就是我使用springboot发布了一个服务：

```
@RequestMapping(value = "/adduser", method = RequestMethod.POST)
@ResponseBody
public User addUser(@RequestParam("name") String name, @RequestParam("age") Integer age) {
  User user = new User();
  user.setName(name);
  user.setAge(age);
  return userRep.save(user);
}
```

在前台通过json格式发送数据导致的。因为@RequestParam是接收表单数据的，无法识别前台发来的json格式。

```
Content-Type: application/json

{
    "name": "name",
    "age" : 12
}
```

换成表单格式就可以请求成功了。

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryBRi81vNtMyBL97Rb

------WebKitFormBoundaryBRi81vNtMyBL97Rb
Content-Disposition: form-data; name="name"

name1
------WebKitFormBoundaryBRi81vNtMyBL97Rb
Content-Disposition: form-data; name="age"

12
------WebKitFormBoundaryBRi81vNtMyBL97Rb--
```

### 415

```
"error":"Unsupported Media Type",
"message":"Content type 'multipart/form-data;boundary=----WebKitFormBoundaryOTm4oeTbegDmbXAx;charset=UTF-8' not supported"
```

**场景：**

不支持的媒体类型。上面的错误是用springboot发布的服务，不支持表单形式的数据。

```java
@RequestMapping(value = "/adduser", method = RequestMethod.POST)
@ResponseBody
public User addUser(@RequestBody User user) {
  return userRep.save(user);
}
```



