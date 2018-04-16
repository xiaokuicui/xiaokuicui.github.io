---
layout: post
title: JSR 303-Bean Validation介绍及学习笔记
categories: JSR
description: JSR 303-Bean Validation介绍及学习笔记
keywords: JSR BeanValidation
---

> JCP（Java Community Process）成立于1998年，是使有兴趣的各方参与定义Java的特征和未来版本的正式过程。JCP使用JSR（Java规范请求，Java Specification Requests）作为正式规范文档，描述被提议加入到Java体系中的的规范和技术。 JSR变为final状态前需要正式的公开审查，并由JCP Executive Committee投票决定。最终的JSR会提供一个参考实现，它是免费而且公开源代码的；还有一个验证是否符合API规范的Technology Compatibility Kit。

# 关于 Bean Validation

在任何时候，当你要处理一个应用程序的业务逻辑，数据校验是你必须要考虑和面对的事情。应用程序必须通过某种手段来确保输入进来的数据从语义上来讲是正确的。在通常的情况下，应用程序是分层的，不同的层由不同的开发人员来完成。很多时候同样的数据验证逻辑会出现在不同的层，这样就会导致代码冗余和一些管理的问题，比如说语义的一致性等。为了避免这样的情况发生，最好是将验证逻辑与相应的域模型进行绑定。

Bean Validation 为 JavaBean 验证定义了相应的元数据模型和 API。缺省的元数据是 Java Annotations，通过使用 XML 可以对原有的元数据信息进行覆盖和扩展。在应用程序中，通过使用 Bean Validation 或是你自己定义的 constraint，例如 @NotNull, @Max, @ZipCode， 就可以确保数据模型（JavaBean）的正确性。constraint 可以附加到字段，getter 方法，类或者接口上面。对于一些特定的需求，用户可以很容易的开发定制化的 constraint。Bean Validation 是一个运行时的数据验证框架，在验证之后验证的错误信息会被马上返回。

下载 JSR 303 – Bean Validation 规范 <http://beanvalidation.org/specification/>

Hibernate Validator 是 Bean Validation 的参考实现 . Hibernate Validator 提供了 JSR 303 规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。如果想了解更多有关 Hibernate Validator 的信息，请查看 <http://www.hibernate.org/subprojects/validator.html>

# Bean Validation 中的 constraint

Bean Validation 中内置的 constraint

Constraint                  | 详细信息
--------------------------- | ----------------------------
@Null                       | 被注释的元素必须为null
@NotNull                    | 被注释的元素必须部位null
@AssertTrue                 | 被注释的元素必须为true
@AssertFalse                | 被注释的元素必须为false
@Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Size(max, min)             | 被注释的元素的大小必须在指定的范围内
@Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内
@Past                       | 被注释的元素必须是一个过去的日期
@Future                     | 被注释的元素必须是一个将来的日期
@Pattern(value)             | 被注释的元素必须符合指定的正则表达式

Hibernate validator附加的 constraint

Constraint | 详细信息
---------- | -------------------
@Email     | 被注释的元素必须是电子邮箱地址
@Length    | 被注释的字符串的大小必须在指定的范围内
@NotEmpty  | 被注释的字符串的必须非空
@Range     | 被注释的元素必须在合适的范围内

一个 constraint 通常由 annotation 和相应的 constraint validator 组成，它们是一对多的关系。也就是说可以有多个 constraint validator 对应一个 annotation。在运行时，Bean Validation 框架本身会根据被注释元素的类型来选择合适的 constraint validator 对数据进行验证。

有些时候，在用户的应用中需要一些更复杂的 constraint。Bean Validation 提供扩展 constraint 的机制。可以通过两种方法去实现，一种是组合现有的 constraint 来生成一个更复杂的 constraint，另外一种是开发一个全新的 constraint。

# 创建一个包含验证逻辑的简单应用(基于JSP)

## 声明了contraint的JavaBean

清单 1 Order.java

```java
 public class Order{
   // 必须不为 null, 大小是 10
 @NotNull
 @Size(min = 10, max = 10)
 private String orderId;
 // 必须不为空
 @NotEmpty
 private String customer;
 // 必须是一个电子信箱地址
 @Email
 private String email;
 // 必须不为空
 @NotEmpty
 private String address;
 // 必须不为 null, 必须是下面四个字符串'created', 'paid', 'shipped', 'closed'其中之一
 // @Status 是一个定制化的 contraint
 @NotNull
 @Status
 private String status;
 // 必须不为 null
 @NotNull
 private Date createDate;
 // 嵌套验证
 @Valid
 private Product product;

…
 getter 和 setter
 }
```

清单2.Product.java

```java
public class Product {
 // 必须非空
 @NotEmpty
 private String productName;
 // 必须在 8000 至 10000 的范围内
 // @Price 是一个定制化的 constraint
 @Price
 private float price;
…
 Getter 和 setter
 }
```

清单3.OrderQuery.java

```java
// 'to'所表示的日期必须在'from'所表示的日期之后
 // @QueryConstraint 是一个定制化的 constraint
 @QueryConstraint
 public class OrderQuery {
 private Date from;
 private Date to;
… omitted …
 Getter and setter
 }
```

## 定制化的constraint

@Price是一个定制化的 constraint，由两个内置的 constraint 组合而成。

清单4.@Price的annotation部分

```java
// @Max 和 @Min 都是内置的 constraint
@Max(10000)
@Min(8000)
@Constraint(validatedBy = {})
@Documented
@Target( { ElementType.ANNOTATION_TYPE, ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface Price {
String message() default "错误的价格";
Class<?>[] groups() default {};
Class<? extends Payload>[] payload() default {};
}
```

@Status是一个新开发的constraint

清单5.@Status的annotation部分

```java
@Constraint(validatedBy = {StatusValidator.class})
@Documented
@Target( { ElementType.ANNOTATION_TYPE, ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface Status {
String message() default "不正确的状态 , 应该是 'created', 'paid', shipped', closed'其中之一";
Class<?>[] groups() default {};
Class<? extends Payload>[] payload() default {};
}
```

清单6.@Status的constraint validator部分

```java
public class StatusValidator implements ConstraintValidator<Status, String>{
private final String[] ALL_STATUS = {"created", "paid", "shipped", "closed"};
public void initialize(Status status) {
}
public boolean isValid(String value, ConstraintValidatorContext context) {
if(Arrays.asList(ALL_STATUS).contains(value))
return true;
return false;
}
}
```

# Bean Validation API使用示例

## 创建订单

用户在创建一条订单记录时，需要填写以下信息：订单编号，客户，电子信箱，地址，状态，产品名称，产品价格 对于这些信息的校验,使用Bean Validation API

清单7.代码片段

```java
protected void doPost(HttpServletRequest req, HttpServletResponse resp)
throws ServletException, IOException {
HttpSession session = req.getSession();
// 从 request 中获取输入信息
String orderId = (String) req.getParameter("orderId");
String customer = (String) req.getParameter("customer");
String email = (String) req.getParameter("email");
String address = (String) req.getParameter("address");
String status = (String) req.getParameter("status");
String productName = (String) req.getParameter("productName");
String productPrice = (String) req.getParameter("productPrice");
// 将 Bean 放入 session 中
Order order = new Order();
order.setOrderId(orderId);
order.setCustomer(customer);
order.setEmail(email);
order.setAddress(address);
order.setStatus(status);
order.setCreateDate(new Date());
Product product = new Product();
product.setName(productName);
if(productPrice != null && productPrice.length() > 0)
product.setPrice(Float.valueOf(productPrice));
order.setProduct(product);
session.setAttribute("order", order);
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
Set<ConstraintViolation<Order>> violations = validator.validate(order);
if(violations.size() == 0) {
session.setAttribute("order", null);
session.setAttribute("errorMsg", null);
resp.sendRedirect("creatSuccessful.jsp");
} else {
StringBuffer buf = new StringBuffer();
ResourceBundle bundle = ResourceBundle.getBundle("messages");
for(ConstraintViolation<Order> violation: violations) {
buf.append("-" + bundle.getString(violation.getPropertyPath().toString()));
buf.append(violation.getMessage() + "<BR>\n");
}
session.setAttribute("errorMsg", buf.toString());
resp.sendRedirect("createOrder.jsp");
}
}
```

如果用户不填写任何信息提交订单，相应的错误信息将会显示在页面上 其实在整个程序的任何地方都可以调用 JSR 303 API 去对数据进行校验，然后将校验后的结果返回。

清单8.调用JSR 303 API进行校验

```java
Order order = new Order();
…
 ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
 Validator validator = factory.getValidator();
 Set<ConstraintViolation<Order>> violations = validator.validate(order);
```
