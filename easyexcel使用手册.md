## easyexcel使用手册
### 简介easyexcel
* Java领域解析、生成Excel比较有名的框架有Apache poi、jxl等。但他们都存在一个严重的问题就是非常的耗内存。如果你的系统并发量不大的话可能还行，但是一旦并发上来后一定会OOM或者JVM频繁的full gc。
* EasyExcel是阿里巴巴开源的一个excel处理框架，以使用简单、节省内存著称。EasyExcel能大大减少占用内存的主要原因是在解析Excel时没有将文件数据一次性全部加载到内存中，而是从磁盘上一行行读取数据，逐个解析。
* EasyExcel采用一行一行的解析模式，并将一行的解析结果以观察者的模式通知处理（AnalysisEventListener）。
### 开始使用

1. 引入依赖(版本尽量高一点点,我使用jdk11,就和2.1的不适配,jdk8可使用2.1)
```xml
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>3.2.0</version>
    </dependency>
```
2. 写入数据到文件中去
```java
    //指定表格文件路径
    String filename = "D:\\360TEST\\test.xlsx";
    //实体信息列表
    ArrayList<DemoData> demoData = new ArrayList<>();
    Collections.addAll(demoData,new DemoData(1,"ledger","ledgerbbbb"),new DemoData(2,"alex","ledgerbbbb"));
    //写入
    EasyExcel.write(filename,DemoData.class).sheet("学生列表").doWrite(demoData);
```
* 结果展示
![](http://ledger.byethost33.com/097dea460126416454b50ae55b45b1d.png)

* 类注解
注解名称 | 参数 | 作用
--- | --- | --- 
@HeadRowHeight | short value() | easyExcel注解设置**表头**的**行高**
@ContentRowHeight | short value() | easyExcel注解设置**内容**的**行高**
@ColumnWidth | short value() | easyExcel注解设置**列宽**,设置给所有

* 属性注解
注解名称 | 参数 | 作用
--- | --- | --- 
@ExcelProperty | (value = "学生编号",index = 0,order = 100) | 设置Excel表头名称,在表头是学生编号的这一列下面,排序是先后顺序,默认是integer的最大值,即默认插在最后面,index列所在的位置为0
@ExcelProperty | (value = "学生姓名",index = 1,order = 100) | 设置Excel表头名称,在表头是学生姓名的这一列下面,排序是先后顺序,默认是integer的最大值,即默认插在最后面,index列所在的位置为1
@ExcelProperty | (value = "学生老师",index = 2,order = 100) | 设置Excel表头名称,在表头是学生老师的这一列下面,排序是先后顺序,默认是integer的最大值,即默认插在最后面,index列所在的位置为2
@ColumnWidth | short value() | easyExcel注解设置**列宽**,单独设置给这个列








