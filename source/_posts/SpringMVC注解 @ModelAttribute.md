---
title: SpringMVC注解 @ModelAttribute
date: 2018-05-11 13:44:53
tags:
   - java
   - spring
categories: Java
---

在 SpringMVC 的 Controller 中使用 `@ModelAttribute` 时，应用位置包括下面几种：

- 应用在方法上。
- 应用在方法的参数上。
- 应用在方法上，并且方法也使用了`@RequestMapping`

<!-- more -->

### 1.应用在方法上

首先说明一下，被 `@ModelAttribute` 注解的方法会在Controller每个方法执行之前都执行，因此对于一个Controller中包含多个URL的时候，要谨慎使用。

1) 使用 `@ModelAttribute` 注解无返回值的方法

```java
@Controller
@RequestMapping("/modelattributeTest")
public class ModelAttributeTestController1 {

    @ModelAttribute
    public void myModel(@RequestParam(required = false) String abc, Model model) {
        model.addAttribute("attributeName", abc);
    }

    @RequestMapping(value = "/test1")
    public String test1() {
        return "modelattributetest/test1";
    }
}
```

这个例子，在请求`/modelattributeTest/test1?abc=aaa`后，会先执行 `myModel`方法，然后接着执行`test1`方法，参数`abc`的值被放入`Model`中后，接着被带到 `test`方法中。

当返回视图 `modelattributetest/test1`时，`Model` 会被带到页面上，当然你在使用 `@RequestParam`的时候可以使用`required`来指定参数是否是必须的。

如果把 `myModel`和 `test1` 合二为一后的方法可以为，这也是我们最常用的方法：

```java
@RequestMapping(value = "/test2")
public String test1(@RequestParam(required = false) String abc, Model model) {
    model.addAttribute("attributeName", abc);
    return "modelattributetest/test1";
}
```

2）使用 `@ModelAttribute` 注解带有返回值的方法

```java
@ModelAttribute
public String myModel(@RequestParam(required = false) String abc) {
    return abc;
}

或者

@ModelAttribute
public Student myModel(@RequestParam(required = false) String abc) {
    Student stu = new Student(abc);
     return stu;
}

或者

@ModelAttribute
public int myModel(@RequestParam(required = false) int number) {
     return number;
}
```

对于这种情况，返回值对象会被默认放到隐含的 `Model`中，在 `Model`中的 `key`为 “返回值首字母小写”，`value` 为返回的值。

**上面3种情况等同于：**

```java
model.addAttribute("string", abc);
model.addAttribute("int", number);
model.addAttribute("student", stu);

```

如果只能这样，未免太局限了，我们很难接受`key` 为 `string`、`int`、`float` 等等这样的。

想自定义其实很简单，只需要给`@ModelAttribute`添加`value`属性即可，如下：

```java
@ModelAttribute(value = "num")
 public int myModel(@RequestParam(required = false) int number) {
    return number;
}
```

这样就相当于

```java
model.addAttribute(“num”, number);
```

### 2.使用@ModelAttribute注解方法的参数

```java
@Controller
@RequestMapping("/modelattributeTest3")
public class ModelAttributeTestController3 {

    @ModelAttribute(value = "attributeName")
    public String myModel(@RequestParam(required = false) String abc) {
        return abc;
    }

    @ModelAttribute
    public void myModel3(Model model) {
        model.addAttribute("name", "SHANHY");
        model.addAttribute("age", "28");
    }

    @RequestMapping(value = "/test1")
    public String test1(@ModelAttribute("attributeName") String str, 
            @ModelAttribute("name") String str2,
            @ModelAttribute("age") String str3) {
        return "modelattributetest/test1";
    }

}
```

从代码中可以看出，使用 `@ModelAttribute`注解的参数，意思是从前面的 `Model`中提取对应名称的属性。

这里提及一下 `@SessionAttributes` 的使用：

1. 如果在类上面使用了 `@SessionAttributes(“attributeName”)` 注解，而本类中恰巧存在`attributeName`

   ，则会将 `attributeName` 放入到 session 作用域。

2. 如果在类上面使用了`@SessionAttributes(“attributeName”)`注解，SpringMVC 会在执行方法之前，自动从 session 中读取 key 为 `attributeName` 的值，并注入到 `Model`中。
   所以我们在方法的参数中使用`@ModelAttribute(“attributeName”)`就会正常的从 `Model`读取这个值，也就相当于获取了 `session`中的值。

3. 使用了 `@SessionAttributes`之后，Spring 无法知道什么时候要清掉 `@SessionAttributes` 存进去的数据，如果要明确告知，也就是在方法中传入 `SessionStatus`对象参数，并调用 `status.setComplete()`就可以了。

这两点大家好好尝试下，或者找一下关于 `@SessionAttributes`更详细的介绍。

### 3.应用在方法上，并且方法也使用了@RequestMapping

**如下代码：**

```java
@Controller
@RequestMapping("/modelattributeTest4")
public class ModelAttributeTestController4 {

    @RequestMapping(value = "/test1")
    @ModelAttribute("name")
    public String test1(@RequestParam(required = false) String name) {
        return name;
    }

}
```

这种情况下，返回值 `String （或者其他对象）`，就不再是视图了。还是我们上面将到的放入 `Model` 中的值，此时对应的页面就是 `@RequestMapping`的值 test1。