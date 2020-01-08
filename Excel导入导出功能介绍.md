# Excel导入导出功能介绍
介绍 Excel 导入导出功能的使用方式、设计思路，方便调用和后续维护。

## 功能介绍
- 支持普通二维表格的生成模版、导入数据、导出数据
- 根据 `Java` 类 + 注解的方式来做模版，无需每个数据结构都要打开 excel 自己做弄模版
- 支持模版的必填项提示、字典翻译、导入导出不同字段、固定表头、字段注释

## 调用方法
1. 在 Service 层声明一个静态常量，创建模版定义结构（以车队为例）：
    ```
    private static final ExcelTemplateCore.DataStructureDefinition<TspMotorcade> TEMPLATE = parseType(TspMotorcade.class)
        .orElseThrow(() -> new IllegalArgumentException("类型 TspMotorcade 没有标记 @ExcelTemplateEntity"));
    ```
    一定要把 `parseType` 的结果储存起来，因为该方法很重，每次都调用的话会影响性能。
    
2. 导出数据的方法：
    ```java
    // 第一个参数是模版的定义结构，第二个参数是数据列表，从数据库里查出来的那些，返回值是二进制 excel 文件
    final byte[] bytes = ExcelTemplateCore.exportData2003(TEMPLATE, entityList);
    ```

3. 生成模版的方法：
    ```java
    // 参数是模版的定义结构，返回值是二进制 excel 文件
    final byte[] bytes = ExcelTemplateCore.generateTemplate2003(TEMPLATE);
    ```

4. 导入数据的方法：
    ```java
    // 参数是模版的定义结构，返回值是解析结果，包含 Entity 列表（泛型）和每行失败原因。
    // 第二个参数是一个 InputStream ，可以从文件获取，也可以从 HttpServletRequest 中进行获取
    final ExcelTemplateResult<TspMotorcade> xes = ExcelTemplateCore.resolveData2003(TEMPLATE, 
            new FileInputStream("D:/xx3.xls"));
    // 解析成功的数据
    final List<TspMotorcade> items = xes.getItems();
    // 解析失败的数据行号及原因
    final List<ExcelTemplateResult.FailReason> failReasons = xes.getFailReasons();
    ```
    注意不能把 `ExcelTemplateResult` 直接作为序列化类，因为它包含了异常堆栈信息。
    
## 设计简介
Excel模版功能分为两块组件：核心组件和注解组件。 

核心组件的功能就是，只要正确地实现了 `ExcelTemplateCore.DataStructureDefinition`，就可以进行导入导出和生成模版了；

注解组件就是通过解析 `ExcelTemplateEntity` 和 `ExcelTemplateField` 注解， 
来生成 `ExcelTemplateCore.DataStructureDefinition` ，以此实现了注解方式的 Excel 模版定义，加速开发。  

后续维护时，必须明确这两层组件的关系，经过分析后，再决定新功能需要加在哪一层。
