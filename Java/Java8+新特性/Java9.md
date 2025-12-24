#Java9

## 1、模块化
模块化是Java9引入的一个新特性，模块化允许开发者将代码组织成模块，模块之间可以相互依赖，模块之间可以相互访问。
### 模块化收益
1、强封装性
Java9之前：public修饰符意味着所有包可见（即便在不同JAR中）。
Java9：模块必须声明导出的包，未导出包即使是public类也不可被访问
好处：隐藏内部实现，减少耦合，避免反射滥用（通过opens控制反射访问）
```
// 模块A
module module.a {
    // 没有 exports com.example.api
    // 包内的 public 类对其他模块不可见
}
package com.example.api;
public class MyClass { } // ❌ 其他模块无法访问

// 模块B
module module.b {
    requires module.a; // ✅ 依赖声明了，但...
}
package test;
import com.example.api.MyClass; // ❌ 编译错误：包未导出

内部API访问：JDK内部API（如sun.misc.*）默认受限
# 需要添加JVM参数
--add-opens java.base/sun.misc=ALL-UNNAMED

```
2、可靠的依赖管理
a.避免缺失依赖导致的运行时错误（如 ClassNotFoundException）。
b.防止重复依赖和版本冲突（通过模块路径而非类路径）
3、运行时镜像体积减小
a.使用 jlink 工具打包应用所需的模块，生成轻量级运行时镜像
### 模块声明位置
模块根目录：与包同级（module-info.java）
```
module com.example.myapp {
    // 1. 依赖声明
    requires java.base;                    // 隐式依赖（自动包含）
    requires java.sql;
    requires transitive com.common.lib;    // 传递依赖
    
    // 2. 导出包（API暴露）
    exports com.example.api;
    exports com.example.model to com.client; // 限定导出
    
    // 3. 开放包（反射访问）
    opens com.example.internal;
    opens com.example.entities to hibernate.core;
    
    // 4. 服务消费/提供
    uses com.example.spi.Service;
    provides com.example.spi.Service 
        with com.example.impl.MyService;
    
    // 5. 允许反射访问（运行时）
    open module com.example.myapp {        // 替代opens每个包
        // 声明...
    }
}
```
### JDK8升级至JDK9的兼容性
1. 类路径模式（兼容模式）- 推荐升级路径
java --class-path app.jar:lib/* com.example.Main
a.所有代码在"未命名模块"中运行，相互可见
b.自动访问所有JDK模块（java.se模块聚合）

2. 模块路径模式（非模块化JAR）
java --module-path app.jar:lib/* --class-path ...
a.非模块化JAR成为"自动模块"
b.可读取所有其他模块，导出所有包

3. 模块化模式
java --module-path mods/ -m com.example/app
a.需要完整的模块声明

### Java9升级前后类路径查找策略对比
Java8及之前
Classpath: lib/a.jar:lib/b.jar:app.jar
查找策略：
1. 按顺序扫描所有JAR和目录
2. 找到第一个匹配的类就加载
3. 容易出现：
    - 重复类（不同JAR）
    - 版本冲突
    - NoClassDefFoundError（运行时才发现）
Java9及之后
双路径机制：
      --module-path mods/     # 模块路径（优先）
      --class-path lib/*      # 类路径（兼容）
查找优先级：
1. 模块路径（named modules → automatic modules）
         a. 模块声明依赖优先
         b. 明确的服务加载（uses/provides）

2. 类路径（unnamed module）
    - 所有类在一个未命名模块中
    - 可读取所有其他模块
    - 导出所有包（兼容性）

## 2、G1成为默认垃圾回收器
## 3、快速创建不可变集合
增加了List.of()、Set.of()、Map.of() 和 Map.ofEntries()等工厂方法来创建不可变集合

```
List.of("Java", "C++");
Set.of("Java", "C++");
Map.of("Java", 1, "C++", 2);
```
## String 存储结构优化
String类在JDK9中，String类存储结构进行了优化，String类不再使用char[]数组存储字符串，而是使用byte[]数组存储字符串。
