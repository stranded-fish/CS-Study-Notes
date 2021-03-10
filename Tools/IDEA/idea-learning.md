# IDEA 学习

该教程主要针对：**IDEA 设置**、**功能** 与 **快捷键** 等学习。

目录：

- [IDEA 学习](#idea-学习)
  - [IDEA 设置](#idea-设置)
    - [主题设置](#主题设置)
    - [代码模板设置](#代码模板设置)
    - [部署设置](#部署设置)
  - [快捷键](#快捷键)
  - [参考链接](#参考链接)

## IDEA 设置

### 主题设置

TODO 背景颜色、字体、同步主题等设置

### 代码模板设置

**eg 1. 生成文件模板**

新建文件时，自动在该文件前添加相关内容。如：新建类时自动添加注释信息。

设置步骤：

1. 依次打开：File -> Settings -> Editor -> File and Code Templates；
2. 选择模板应用范围 Scheme: Default（全局配置） / Project（当前工程）；
3. 左侧 Files 选择 Class；
4. 在右侧编辑框中添加相应注释。

![设置类注释模板](https://i.loli.net/2021/01/03/amJZTQjBOE3nH8X.png)

类注释模板示例：

```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
 * ${DESCRIPTION}
 *
 * @author Tom
 * @date ${YEAR}/${MONTH}/${DAY}
 */
public class ${NAME} {
}
```

注：

* 注释模板添加到 includes 当中也能实现相应效果。
* `${DESCRIPTION}` 将在创建类时弹出窗口，填写描述，`${YEAR},${TIME}` 等可以自动获取系统时间并填写。
* Interface、Enum 等相关设置同上。

**eg 2. 生成代码模板**

输入相应缩写 + <kbd>Tab</kbd> 键 或 <kbd>Enter</kbd>，生成相应代码模板。如：sout、iter

设置步骤：

1. 依次打开：File -> Settings -> Editor -> Live Templates；
2. 选择右侧相应的模板组，如：Java 相关模板则选择 Java，若没有合适的组，则点击最右侧加号，选择新建 Template Group 并命名；
3. 点击最右侧加号，在选定组中新建 Live Template；
4. 设置代码缩写 Abbreviation，模板描述 Description，模板内容 Template text，编辑变量 Edit variables，模板作用范围以及触发方式（Tab）。

![设置代码模板](https://i.loli.net/2021/01/03/lHhZa6Nq7vPF91V.png)
![设置模板变量](https://i.loli.net/2021/01/03/95yRadsDTeG7VCt.png)

方法注释模板示例：

```java
**
 * $description$
 *
 * @param 
 * @return $returns$
 * @author Tom
 * @date $date$
 */
```

注：省略了注释开头的 `/` ，触发方法为在相关方法前一行通过 <kbd>/</kbd> + Abbreviation + <kbd>Tab</kbd> 生成模板。若不省略 `/` ，则无法在函数外获得该函数的参数以及返回值。

### 部署设置

设置步骤：

1\. 打开设置 Settings；

2\. 设置 Connection；
![Connection](https://i.loli.net/2021/03/10/BxUjFvhE9cePIwS.png)

3\. 设置 Mappings；
![Mappings](https://i.loli.net/2021/03/10/D3FXf5LhrBAYgxb.png)

4\. 部署。

选择要部署的内容 -> 右键 -> Deployment -> Upload to serverName。

![部署到远程服务器](https://i.loli.net/2021/03/10/A9BNjkMV7rf28GK.png)

## 快捷键

## 参考链接

* [Idea代码模板和自定义代码模板的使用](https://www.jianshu.com/p/fb34214f68ba)
* [idea生成类注释和方法注释的正确方法](https://blog.csdn.net/qq_34581118/article/details/78409782)
* [IntelliJ IDEA直接上传文件到服务器](https://blog.csdn.net/u010588262/article/details/72235842)

<!-- 1. 修改编辑区背景颜色
   1. settings -> Editor -> Color Scheme -> General -> Text -> Default text -> background R199 G237 B204
1. 设置快捷键
