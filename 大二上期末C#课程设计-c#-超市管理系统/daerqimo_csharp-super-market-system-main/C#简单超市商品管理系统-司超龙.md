## 简单超市商品管理系统



### 一、简单介绍

参考B站视频：https://www.bilibili.com/video/BV1VJ411W72j?p=4,

说明：视频时长`4：23：42`,视频年代较早，主要使用的是Form,这里我使用的WPF替换Form来实现相关显示操作



开发环境

- VisualStudio 2019、SQLServer 2019、SSMS 18

基于C#和SQLServer 实现 

1. 超市用户管理员账户的登录、密码修改
2. 超市商品的增删改查，使用WPF显示

主要涉及到的知识点

1. WPF窗体：Label、TextBox、Button、DataDrid、TreeView、GroupBox、ComoBox等控件的基本使用

2. 数据库操作：DBHealperl工具类、SqlConnection、SqlCommond、SqlDataAndpter适配器等

3. WPF窗口设置个性化图标、自定义背景颜色、

4. 数据库信息显示：修改前数据的回显、Filter过滤显示指定字段值的记录（显示特价商品）、ComoBox的数据绑定显示

   



### 二、实现流程

#### 2.1 登录页面实现

![2020-12-20_091922](C:\Users\司超龙\Documents\Scrshot\2020-12-20_091922.png)

#### 2.2 数据库环境的搭建

涉及表

- User登陆信息
- Commodity商品表
- CommodityList商品详细信息表
- CommoditySort商品分类对照表
- Unit商品单位对照表



注意：各表的主键需要自增，建表的时候需要设置，否则修改还需要重新建表。

#### 2.3连接数据库的类DBHelper

实现

- 获取数据库的连接
- 关闭数据库的连接
- 数据库连接处于中断状态重新连接



#### 2.4登录逻辑

1. 获取数据连接对象
2. 从表单获取登录信息，对于空信息提示，书写sql语句，查询数据库，返回登录结果
3. 点击取消按钮，关闭当前窗体

遇到的一些问题

- 密码框的处理需要`PasswordBox`控件
- 点击取消按钮关闭当前窗体,`this.close()`



#### 2.5登录成功主窗体功能、修改密码功能

登录成功之后，主窗体``SuperMarketMain.xaml``

- 菜单选项：可以修改用户密码、退出系统，查看帮助
- 有商品管理、商品类别管理的入口

具体逻辑

1. 登录成功，关闭登录窗口，显示主窗体

2. 退出实现：点击退出按钮，根据`Form`弹窗`DialogResult`判断是否退出程序

3. 修改密码实现：修改密码要求输入旧密码，因此需要记录当前的旧密码，验证之后才能修改密码

   - 创建UserBean，封装数据，在登录成功之后，将数据存入`user`对象

   - 将user保存数据，进行窗体间传值，传给`superMarketMain`窗口

   - 登录成功的时候，需要将数据库中的`user`对应的`UserID`信息，封装到`UserBean`中



#### 2.6 登录成功主窗体功能、关于

点击关于按钮后，弹出一个窗口`About`.xaml，显示此系统的相关信息

注意：

- 需要设置 窗口的 不能调整大小、不能最大、最小化

- 同时设置初始位置在屏幕的中央即`ResizeMode="NoResize"`
- 此窗口不关闭，无法操作其他窗口即`ShowDialog()`



#### 2.7 商品管理功能管理



将数据库的信息写到DataGrid中，这个地方需要注意

##### 需要注意Form的显示和WPF的显示还是有区别的

- `DataSource`是WinForm里的操作，控件是`DataGridView`，

- 在WPF里是DataGrid，只能是`ItemSource`

注意：前台里的Loaded事件一定要加上，否则不显示

```xaml
<DataGrid x:Name="dataGrid" ItemsSource="{Binding}" HorizontalAlignment="Left" Height="auto" Margin="210,104,-0.4,0" VerticalAlignment="Top" Width="auto" Background="{x:Null}" Loaded="dataGrid_Loaded">
            <DataGrid.Columns >
                
            </DataGrid.Columns>
  </DataGrid>
```



```C#
private void productsMange()
        {
            dt = new DataTable();
            DBHeaper dBHeaper = new DBHeaper();

            try
            {
                if(dBHeaper.Connection.State == ConnectionState.Closed)
                {
                    dBHeaper.Connection.Open();
                }
                //这里的显示需要注意，花了很长时间才搞定
                string sql = string.Format(@"select * from Commodity");
                DataSet ds = new DataSet();
                SqlDataAdapter adapter = new SqlDataAdapter(sql, dBHeaper.Connection);
                adapter.Fill(ds, "Commodity");
                dt = ds.Tables["Commodity"];
                DataView dv = new DataView(dt);


                dataGrid.ItemsSource = dv;
                
            }
            catch(Exception e)
            {
                MessageBox.Show("数据库连接错误！", "系统提示", MessageBoxButton.OK, MessageBoxImage.Error);

            }
            finally
            {
                if (dBHeaper.Connection.State == ConnectionState.Open)
                {
                    dBHeaper.Connection.Close();
                }

            }
        }
```



一般数据库信息显示到DataGrid中有两种方式：

- Sql代码
- 直接添加数据库EF模型

#### 2.8筛选显示特价商品

使用过滤器Filter完成，判断点击的TreeView空间，过滤指定字段值的行



- DataTable有一个属性RowRilter可以对DataTable的数据进行过虑，利用这个特性可以对.net的控件绑定实现数据筛选。DataTable可以认为是.Net一个数据内存容器，我们一般都是从数据库中从读取放到一个DataTable中，最后绑定到.net的控件上。

- DataTable有一个属性DefaultView可以返回对应DataTable的一个DataView实例，它是类似一个视图，可以对里面的数据行进行筛选，排序。

- 筛选某个字段满足指定条件的记录

```DataView dv = myDs.Tables[0].DefaultView;
dv.RowFilter = "Year=1427"; 
gv.DataSource = dv;
//Year这个是myDs.Tables[0]的一个字段。这样就是只有Year的值等于1427的记录绑定gv控件。
```



#### 2.9商品增

1. 首先是WPF窗口的设置，增和修改公用一个窗口

2. 注意不要忘记ComboBox的绑定和Loaded事件，**在loaded事件根据id来判断是添加还是修改**

   ```c#
   			string sql = "select * from CommoditySort order by sortid";
                   SqlDataAdapter adapter = new SqlDataAdapter(sql,dBHeaper.Connection);
                   adapter.Fill(ds, "sort");
   
                   //绑定数据
                   DataView dv = new DataView(ds.Tables["sort"]);
   			//显示指定字段列
                   combSort.ItemsSource = dv;
                   combSort.DisplayMemberPath = "sortname";
   ```

   

3. 输入数据判空之后，添加绑定数据到数据库

遇到的问题：

- 对于ComoBox选项存入Commoduty表的是sortId,只能是```SelectedValued```不能是```SelectedIndex```，因为随着商品类别的增、删，**商品id 和 选择的索引 对应关系会改变**
- 对于添加的商品需要数据库的自增id，因此需要设置表的id自增
- 对于数据的每一项都要进行空值判断，防止空指针、格式转换异常



#### 2.10商品改

增、改公用一个add窗口，根据id变量的赋值情况来区分方法调用

- 增：点击增加商品图标，将id初始化为0，调用增加商品的方法
- 改：点击修改商品图标，前提是这个商品已经被选中行，需要将选中行的id取出来，赋值给变量id,然后进行后续的修改方法的调用

 关于DataGrid选中行获取指定列的值，需要



```c#
		add add = new add();
            //获取选中行的id
            DataRowView rowDv = (DataRowView)dataGrid.SelectedItem;
            add.id = Convert.ToInt32(rowDv["编号"]);

            //添加窗口
            add.ShowDialog();
            //重新绑定DataGrid

            this.productsMange();
```



对于改

- 回显数据：回显数据需要在ComboBox的loaded事件中调用回显方法

  注意ComoBox的回显

  ```c#
  this.combSort.SelectedValue = Convert.ToInt32( reader["sortid"]);
  ```

  

- 修改数据：调用修改的方法



#### 2.11商品删除

和前面的增、改类似，需要注意的问题

- 需要注意增加确认删除的提示，提高用户体验感
- ComoBox的SelectedIndex 和 数据库中的 sortid 之间不是数值相等的关系，差值不定，需要使用SelectedValue



#### 2.12 商品类别的增、删、改



和前面商品的增删改查类似

只不过需要增加 商品名的重复判断，如果存在就不能添加,不存在才能添加。



### 三、实现过程出现的bug



1. 数据库信息显示在DataGrid中，WPF和Form是有区别的,而且需要加上`loaded`时间才能显示

2. 对于ComoBox选项存入Commoduty表的是sortId,只能是```SelectedValued```不能是```SelectedIndex```，因为随着商品类别的增、删，**商品id 和 选择的索引 对应关系会改变**

   - 而且前面需要设置：`SelectValuePath="sortname"`

3. 获取选中行的字段值，根据字段值进行回显

4. 商品类别存在重复可添加的bug

   ```c#
    			SqlDataReader reader = cmd.ExecuteReader();
   			//开始使用reader != null 来判断发现 恒成立
   			//需要改为reader.Read()判断是否有数据可读
                   if (reader.Read())
                   {
                       flag = true;
                       MessageBox.Show("商品类别已经存在~", "系统提示", MessageBoxButton.OK, MessageBoxImage.Information);
                       
   
                   }
   ```

   

