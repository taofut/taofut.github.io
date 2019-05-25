---
title: "jackson常用注解【servyou总结】"
date: 2019-05-22 15:18:29
tags: servyou总结
categories: 开发中常用注解
---
### 开发问题一（@JsonInclude的使用）
在写后台接口回传数据到前端时，遇到一些细节问题，下面直接看代码。我要回传的对象类如下，从数据库查出来的任何类型数据都放data里参与序列化返回给前端。
<!--more-->

```java
@JsonInclude(JsonInclude.Include.NON_EMPTY)
public class NewFrameResponseResult<T> {

    private boolean success;

    private T data;

    private String error;

}
```
**情况** 
假设数据库查出来为null，那么data就是null，执行sql是没问题的，success返回true。由于类上面定义了*@JsonInclude(JsonInclude.Include.NON_EMPTY)*，所以data为null是不会在序列化结果中显示的。前端获取的返回结果：{"success":true}，后面的js代码直接拿data报错，因为data根本不存在。
**改进**
这时候就面临一个问题，我数据库是没查到数据，但是data这个属性还是要返回的，因为这是前端js获取数据的唯一字段，可以在没数据的情况下返回一个空的data（数组长度为零，但不为null）。

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class NewFrameResponseResult<T> {

    private boolean success;

    private T data;

    private String error;

    public static NewFrameResponseResult success(Object data) {
        return new NewFrameResponseResult(data == null ? Collections.EMPTY_LIST : data);
    }

}
```
这里允许返回空对象，所以注解就要变为*@JsonInclude(JsonInclude.Include.NON_NULL)*，然后在返回的方法里判断为null，就将data转换为空集合。前端获取的返回结果：{"success": true,"data": []}
**说明**
*JsonInclude.Include.NON_EMPTY*：属性为空或者null都不参与序列化。
*JsonInclude.Include.NON_NULL*：属性为null不参与序列化。

### 开发问题二（@JsonIgnore和@JsonIgnoreProperties的使用）
这次是将数据库查出来的菜单目录封装成树结构，有一个树节点实体TreeNode。由于查出来的数据在封装树的时候需要定义pid参与子节点遍历，但是前端返回结果中是不需要pid这个字段的，那么就在pid字段上加上注解@JsonIgnore，这样pid既参与了树结构的封装，同时也可以不让前端接收到。

```java
@JsonInclude(JsonInclude.Include.NON_EMPTY)
public class TreeNode {
    //根节点类型
    public static final String NODE_TYPE_ROOT = "ROOT";
    //父节点类型（下面还存在子节点）
    public static final String NODE_TYPE_GROUP = "GROUP";
    //子节点类型（只有子节点）
    public static final String NODE_TYPE_LEAF = "LEAF";
    //节点名称
    private String name;
    //节点id
    private String nodeId;
    //节点类型
    private String nodeType;
    //图标
    private String className = "iconfont";
    //节点url
    private String url;
    //父节点id
    @JsonIgnore
    private String pid;
    //当前节点下的子节点
    private List<TreeNode> children;
    //菜单状态Hot（标记点击量高的菜单）
    private String otherInfo;

}
```
**说明**
*@JsonIgnore（作用在属性或方法）*：用来告诉Jackson在处理时忽略该注解标注的 java pojo 属性。
*@JsonIgnoreProperties（作用在类）*：@JsonIgnoreProperties和@JsonIgnore的作用相同，都是告诉Jackson该忽略哪些属性，不同之处是@JsonIgnoreProperties是类级别的，并且可以同时指定多个属性。

```java
@JsonIgnoreProperties(value = {"pid","nodeType"})
public class TreeNode {
    //根节点类型
    public static final String NODE_TYPE_ROOT = "ROOT";
    //父节点类型（下面还存在子节点）
    public static final String NODE_TYPE_GROUP = "GROUP";
    //子节点类型（只有子节点）
    public static final String NODE_TYPE_LEAF = "LEAF";
    //节点名称
    private String name;
    //节点id
    private String nodeId;
    //节点类型
    private String nodeType;
    //图标
    private String className = "iconfont";
    //节点url
    private String url;
    //父节点id
    private String pid;
    //当前节点下的子节点
    private List<TreeNode> children;
    //菜单状态Hot（标记点击量高的菜单）
    private String otherInfo;

}
```
### @JsonIgnoreType的使用
@JsonIgnoreType标注在类上，当其他类引用该类时，该属性将被忽略。

```java
@JsonIgnoreType
public class User {
    private Long id;
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
}

public class Role {
    private String name;
    private String desc;
    private User user;
}
```