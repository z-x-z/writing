#wpf学习

##项目模版

* console
* WPF
  * 三个层次
    1. Solution层
    2. Project层 -> 程序集(assmbly)
       1. console
       2. wpf
       3. class library
    3. Source code
       1. Properties
       2. References
       3. App.xaml
          * App.xaml.cs
       4. 

##xaml语言

* xaml语言 + 后台cs代码(partial) -> 合并后的cs代码 -> 最终的程序

* 声明性语言

* Attributes

  * 对象基本属性: Title="MainWindow"

  * 名称空间(xmlns): 

    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"(默认名称空间)

    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"(名称为x)

## wpf与winform比较

* 平面结构
* 树形结构

## wpf基础控件

1. ###Grid控件

   描述:

   

   属性:

   ​	1.``Grid.RowDefinitions``与``Grid.ColumnDefinitions``

   ​		可以用来定义行与列可分别在其中定义行与列，并设置其高度与宽度

   ```xaml
   <!--Grid.RowDefinitions定义行-->
           <Grid.RowDefinitions>
               <RowDefinition></RowDefinition><!--一行-->
               <RowDefinition Height="Auto"></RowDefinition><!--一行，自适应高度-->
           </Grid.RowDefinitions>
   
           <!--Grid.ColumnDefinitions定义列-->
           <Grid.ColumnDefinitions>
       				<!--将Grid均匀分成1:2:3宽度比的三列-->      
               <ColumnDefinition Width="*"></ColumnDefinition><!--一列-->
               <ColumnDefinition Width="2*"></ColumnDefinition><!--一列-->
               <ColumnDefinition Width="3*"></ColumnDefinition><!--一列-->
           </Grid.ColumnDefinitions>
    <!--在第二行第一列上加一个按钮-->
   <Button Grid.Column="0" Grid.Row="0" Grid.RowSpan="2" Grid.ColumnSpan="2">你好</Button>
   ```

   2. 1

1. ### window

   描述:

   ​	主窗体界面

   属性:

   1. SizeToContent: 设置窗体大小的自适应情况，设置为"WidthAndHeight"时将根据窗体的长宽自适应调整窗体中的内容

2. ###TextBlock控件

   描述:

   ​	只读的文本框，不允许编辑。

   属性:

   ​	1. Text: 文本框内容

3. ### TextBox \ PassWord

   描述:

   ​	可写的文本框

   属性:

   ​	

4. ### DataGrid

   描述:

   ​	显示数据表格

5. ### ListBox

   

6. ### Combox

   描述:

   ​	下拉列表单

   属性:

   1. comboxItem
   2. selectedItem

   ​	

7. 















