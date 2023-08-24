---
title: SpringBoot 参数校验组件
tags:
  - 转载
author: 其他人
categories: 转载
keywords: 转载
lang: cn
abbrlink: 3028893647
date: 2023-08-24 00:00:00
updated: 2023-08-24 00:00:00
---
# 一坨一坨的 if/else 参数校验，终于被 SpringBoot 参数校验组件整干净了！



数据的校验的重要性就不用说了，即使在前端对数据进行校验的情况下，我们还是要对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些 HTTP 工具直接向后端请求一些违法数据。

最普通的做法就像下面这样。我们通过 `if/else` 语句对请求的每一个参数一一校验。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416078.png)

这样的代码，小伙伴们在日常开发中一定不少见，很多开源项目都是这样对请求入参做校验的。

但是，不太建议这样来写，这样的代码明显违背了 **单一职责原则**。大量的非业务代码混杂在业务代码中，非常难以维护，还会导致业务层代码冗杂！

实际上，我们是可以通过一些简单的手段对上面的代码进行改进的！这也是本文主要要介绍的内容！

废话不多说！下面我会结合自己在项目中的实际使用经验，通过实例程序演示如何在 SpringBoot 程序中优雅地的进行参数验证(普通的 Java 程序同样适用)。

不了解的朋友一定要好好看一下，学完马上就可以实践到项目上去。

并且，本文示例项目使用的是目前最新的 Spring Boot 版本 2.4.5!（截止到 2021-04-21）

示例项目源代码地址：https://github.com/CodingDocs/springboot-guide/tree/master/source-code/bean-validation-demo 。

## 添加相关依赖

如果开发普通 Java 程序的的话，你需要可能需要像下面这样依赖：

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416103.png)

不过，相信大家都是使用的 Spring Boot 框架来做开发。

基于 Spring Boot 的话，就比较简单了，只需要给项目添加上 `spring-boot-starter-web` 依赖就够了，它的子依赖包含了我们所需要的东西。另外，我们的示例项目中还使用到了 Lombok。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416150.png)

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416141.png)

但是！！！Spring Boot 2.3 1 之后，`spring-boot-starter-validation`已经不包括在了 `spring-boot-starter-web` 中，需要我们手动加上！

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416148.png)

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416122.png)

##  验证 Controller 的输入

### 验证请求体

验证请求体即使验证被 `@RequestBody` 注解标记的方法参数。

**`PersonController`**

我们在需要验证的参数上加上了`@Valid`注解，如果验证失败，它将抛出`MethodArgumentNotValidException`。默认情况下，Spring 会将此异常转换为 HTTP Status 400（错误请求）。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416061.png)

**`PersonRequest`**

我们使用校验注解对请求的参数进行校验！

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416101.png)

正则表达式说明：

- `^string` : 匹配以 string 开头的字符串
- `string$` ：匹配以 string 结尾的字符串
- `^string$` ：精确匹配 string 字符串
- `(^Man$|^Woman$|^UGM$)` : 值只能在 Man,Woman,UGM 这三个值中选择

**`GlobalExceptionHandler`**

自定义异常处理器可以帮助我们捕获异常，并进行一些简单的处理。如果对于下面的处理异常的代码不太理解的话，可以查看这篇文章 [《SpringBoot 处理异常的几种常见姿势》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485568&idx=2&sn=c5ba880fd0c5d82e39531fa42cb036ac&chksm=cea2474bf9d5ce5dcbc6a5f6580198fdce4bc92ef577579183a729cb5d1430e4994720d59b34&token=1924773784&lang=zh_CN&scene=21#wechat_redirect)。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416125.png)

**通过测试验证**

下面我通过 `MockMvc` 模拟请求 `Controller` 的方式来验证是否生效。当然了，你也可以通过 `Postman` 这种工具来验证。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416136.png)

**使用 `Postman` 验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230825000458295.png)

### 验证请求参数

验证请求参数（Path Variables 和 Request Parameters）即是验证被 `@PathVariable` 以及 `@RequestParam` 标记的方法参数。

**`PersonController`**

**一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数。**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416158.png)

**`ExceptionHandler`**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416151.png)

**通过测试验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416180.png)

**使用 `Postman` 验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416164.png)

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416169.png)

## 验证 Service 中的方法

我们还可以验证任何 Spring Bean 的输入，而不仅仅是 `Controller` 级别的输入。通过使用`@Validated`和`@Valid`注释的组合即可实现这一需求！

一般情况下，我们在项目中也更倾向于使用这种方案。

**一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数。**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416165.png)

**通过测试验证：**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416192.png)

输出结果如下：

```
name 不能为空
sex 值不在可选范围
```

## Validator 编程方式手动进行参数验证

某些场景下可能会需要我们手动校验并获得校验结果。

我们通过 `Validator` 工厂类获得的 `Validator` 示例。另外，如果是在 Spring Bean 中的话，还可以通过 `@Autowired` 直接注入的方式。

```
@Autowired
Validator validate
```

具体使用情况如下：

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230825000458311.png)

输出结果如下：

```
sex 值不在可选范围
name 不能为空
```

## 自定以 Validator(实用)

如果自带的校验注解无法满足你的需求的话，你还可以自定义实现注解。

### 案例一:校验特定字段的值是否在可选范围

比如我们现在多了这样一个需求：`PersonRequest` 类多了一个 `Region` 字段，`Region` 字段只能是`China`、`China-Taiwan`、`China-HongKong`这三个中的一个。

**第一步，你需要创建一个注解 `Region`。**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416180-2891856.png)

**第二步，你需要实现 `ConstraintValidator`接口，并重写`isValid` 方法。**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416185.png)

现在你就可以使用这个注解：

```
@Region
private String region;
```

**通过测试验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416183.png)

**使用 `Postman` 验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416223.png)

### 案例二:校验电话号码

校验我们的电话号码是否合法，这个可以通过正则表达式来做，相关的正则表达式都可以在网上搜到，你甚至可以搜索到针对特定运营商电话号码段的正则表达式。

```
PhoneNumber.java
```

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416200.png)

```
PhoneNumberValidator.java
```

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416210.png)

搞定，我们现在就可以使用这个注解了。

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230825000539686.png)

**通过测试验证**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416200-2891856.png)

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416239.png)

## 使用验证组

验证组我们基本是不会用到的，也不太建议在项目中使用，理解起来比较麻烦，写起来也比较麻烦。简单了解即可！

当我们对对象操作的不同方法有不同的验证规则的时候才会用到验证组。

我写一个简单的例子，你们就能看明白了！

**1.先创建两个接口，代表不同的验证组**

```
public interface AddPersonGroup {
}
public interface DeletePersonGroup {
}
```

**2.使用验证组**

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416237.png)

通过测试验证：

![Image](%20SpringBoot%20%E5%8F%82%E6%95%B0%E6%A0%A1%E9%AA%8C%E7%BB%84%E4%BB%B6/640-20230824234416215.png)

验证组使用下来的体验就是有点反模式的感觉，让代码的可维护性变差了！尽量不要使用！

## 常用校验注解总结

`JSR303` 定义了 `Bean Validation`（校验）的标准 `validation-api`，并没有提供实现。`Hibernate Validation`是对这个规范/规范的实现 `hibernate-validator`，并且增加了 `@Email`、`@Length`、`@Range` 等注解。`Spring Validation` 底层依赖的就是`Hibernate Validation`。

**JSR 提供的校验注解**:

- `@Null` 被注释的元素必须为 null
- `@NotNull` 被注释的元素必须不为 null
- `@AssertTrue` 被注释的元素必须为 true
- `@AssertFalse` 被注释的元素必须为 false
- `@Min(value)` 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)` 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@Size(max=, min=)` 被注释的元素的大小必须在指定的范围内
- `@Digits (integer, fraction)` 被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past` 被注释的元素必须是一个过去的日期
- `@Future` 被注释的元素必须是一个将来的日期
- `@Pattern(regex=,flag=)` 被注释的元素必须符合指定的正则表达式

**Hibernate Validator 提供的校验注解**：

- `@NotBlank(message =)` 验证字符串非 null，且长度必须大于 0
- `@Email` 被注释的元素必须是电子邮箱地址
- `@Length(min=,max=)` 被注释的字符串的大小必须在指定的范围内
- `@NotEmpty` 被注释的字符串的必须非空
- `@Range(min=,max=,message=)` 被注释的元素必须在合适的范围内

## 拓展

经常有小伙伴问到：“`@NotNull` 和 `@Column(nullable = false)` 两者有什么区别？”

我这里简单回答一下：

- `@NotNull`是 JSR 303 Bean 验证批注,它与数据库约束本身无关。
- `@Column(nullable = false)` : 是 JPA 声明列为非空的方法。

总结来说就是即前者用于验证，而后者则用于指示数据库创建表的时候对表的约束。