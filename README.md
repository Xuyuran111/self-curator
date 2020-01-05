# 关于交互可视化python-flaskweb的技术文档
-----------------------------------------------------------------------------------------------------------------------------------

###### 我的姓名：许钰然
###### 我的学号：181013059
###### 我的班级：2018级网络与新媒体1班
###### 我的python合作伙伴：李珊珊
###### 数据及表格地图提供者：彭海晴师姐
###### 我的文档介绍：这是一个关于Python期末项目的文档介绍，下面将进行具体内容阐述

## 项目代码
- Github URL:[https://github.com/Xuyuran111/python-qmkstry](https://github.com/Xuyuran111/python-qmkstry)

## pythonanywhere部署

- Pythonanywhere URL：[http://xuyuran.pythonanywhere.com](http://xuyuran.pythonanywhere.com)

## 项目总结介绍

- 本次的项目主题是受教育程度与总生育率、青春期生育率的关系；
- 整个项目主要由[数据故事](http://xuyuran.pythonanywhere.com/jiaohu/)；[功能查询页一](http://xuyuran.pythonanywhere.com/jiaohu_1)；[功能查询页二](http://xuyuran.pythonanywhere.com/jiaohu_2)这三个页面组合而成。

## 我主要负责的交互页面是功能查询页一：[别的国家生的多吗？](http://xuyuran.pythonanywhere.com/jiaohu_1)；
- 该页面的主要功能是通过在下拉框中选择国家进行查看所选国家的历年生育率，再通过点击查询按钮，页面中便会出现相对应的柱状图及该国家的表格数据，由此实现交互功能。

- 交互功能页面一的表格数据来源于“API_SP.DYN.TFRT.IN_DS2_en_csv_v2_612877.csv”这一个csv文件；功能一页面主要由template/jiaohu.html渲染而成，采用jiaohu.html与word_2.html中的图进行嵌套渲染

###### 对于整个flask框架的制作，由我和合作伙伴李珊珊共同合作完成，以下是我们对于该项目的具体阐述！

## HTML档描述
- 利用[Phantom](https://html5up.net/phantom) 的主题模板进行重构
- default.html模板由navigation.html、scripts.html、footer.html进行嵌套渲染，采用jinja2标准进行嵌套，并设定变量content进行主要任务传参
- 主页由default.html与index.html构成
- 功能页为jiaohu.html、jiaohu_1.html、jiaohu_2.html渲染生成
- 数据故事为jiaohu.html与word_1.html~word_6.html六个图表html进行嵌套渲染。

## Python档描述
- app.py为主要路由函数设置
- new_pyecharts.py为pyecharts函数的设置，将pyecharts模块加上本身数据进行封装，供功能页面多次调用

## Web App动作描述
- web App主要实现列表循环实现下拉框表单的选取与数据的传递
- 设定主页，可选择单一功能页面进行数据图表的查看，也可对于数据故事进行查看后，在数据故事页面进行数据的选择与图表的交互
- 实现在同一页面中，两个from表单共存提交：通过将request模块的值进行字典化，通过选择判断提交的表单为哪个功能表单，进行if判断后返回所需的功能页面

## 数据交互
- 是否含有复杂数据结构的循环（列表循环、字典循环、集合循环）（20%）
利用jinja2进行选择下拉表单的列表循环

- 是否含有合适的推导式
 new_pyecharts.py中overlap_line_scatter3()函数中将列表数据转换成int格式
 ```python
 def overlap_line_scatter3(city) -> Bar:
    city_index = list(df5["地区"]).index("{}".format(city))
    fxjy = list(df5.iloc[city_index].to_dict().values())[1:][::-1]
    fxjy = [int(fxjy[i]) for i in range(len(fxjy))]
    bar = (
        Bar(opts.InitOpts(width='1150px', height='550px',
                          theme=ThemeType.LIGHT))  # opts.InitOpts(width = '810px',height = '500px')
            .add_xaxis(list(df5.columns)[1:][::-1])
            .add_yaxis("教育经费支出", fxjy)
            .set_series_opts(label_opts=opts.LabelOpts(is_show=False))
            # .add_yaxis)("商家B", Faker.values()
            .set_global_opts(title_opts=opts.TitleOpts(title="{}省教育经费情况".format(city), subtitle="",
                                                       subtitle_textstyle_opts=opts.TextStyleOpts(color="red",
                                                                                                  font_size=14,
                                                                                                  font_style="italic")),
                             legend_opts=opts.LegendOpts(pos_right="15%")
                             )

    )
    line = (
        Line()
            .add_xaxis(list(df5.columns)[1:][::-1])
            .add_yaxis("教育经费支出", fxjy)
            .set_series_opts(label_opts=opts.LabelOpts(is_show=False))

    )
    bar.overlap(line)
    return bar.render('templates/word_5.html')
 ```
- 是否含有适当的条件判断（20%）
 在app.py中利用if判断将request模块的值进行字典化，通过选择判断提交的表单为哪个功能表单，进行if判断后返回所需的功能页面
 ```python
 @app.route('/jiaohu/1',methods=['POST'])
def jiaohu1():
    the_select_region = list(df["Country Name"])
    the_select_region_1 = list(df5["地区"])
    page_title = '交互式可视化数据故事'
    page_description = '这是python与交互式可视化自我策展成果'
    print(list(request.values.to_dict().keys())[0])
    if list(request.values.to_dict().keys())[0] == "the_region_selected":
        the_region = request.form["the_region_selected"]
        overlap_line_scatter(the_region)
        table_1 = pd.DataFrame(df.iloc[list(df["Country Name"]).index("{}".format(the_region))]).T.to_html()
        with open("templates/word_2.html", encoding="utf8", mode="r") as f:
            plot_all = "".join(f.readlines())
        return render_template('default.html',
                               title=page_title,
                               description=page_description,
                               content=render_template('jiaohu_1.html',
                                                       word_2=plot_all,
                                                       table_1=table_1,
                                                       the_select_region=the_select_region,
                                                       ))
    else:
        the_region_1 = request.form["the_region_selected_1"]
        overlap_line_scatter3(the_region_1)
        table_2 = pd.DataFrame(df5.iloc[list(df5["地区"]).index("{}".format(the_region_1))]).T.to_html()
        with open("templates/word_5.html", encoding="utf8", mode="r") as f1:
            plot_all_1 = "".join(f1.readlines())
        return render_template('default.html',
                               title=page_title,
                               description=page_description,
                               content=render_template('jiaohu_2.html',
                                                        word_5=plot_all_1,
                                                       table_2 = table_2,
                                                       the_select_region_1=the_select_region_1))
 ```

## 自定义函数与模块功能
- 利用pyecharts进行图表的函数定义

<br />

## 本地运行

1.克隆我的项目
  ```
  $ git clone 
  $ cd app_web
  ```

2. 用virtualenv进行虚拟化并激活:
  ```
  $ virtualenv --no-site-packages env
  $ source env/bin/activate
  ```

3. 下载所需要的文件:
  ```
  $ pip install -r requirements.txt
  ```

5. 运行项目:
  ```
  $ python app.py
  ```

6. 访问本地连接 [http://localhost:5000](http://localhost:5000)

### 不想本地访问？
* 可以直接通过我的Pythonanywhere网址进行直接访问：[http://xuyuran.pythonanywhere.com](http://xuyuran.pythonanywhere.com)

---------------------------------------------------------------

### 项目开展的相关阶段
#### 项目开展前期阶段---自我策展阶段

---------------------------------------------------------------
##### 自我介绍
* 姓名：许钰然
* 班级：2018级网络与新媒体1班
* 学号：181013059
##### Python能力
1. 掌握pyhon的一些基础语法的使用，
   如：
   - if... elif...else语句, for语句，while语句
   - 还学会了元组，列表，字典，数据库模块等基础知识点
2. 会使用pycharm，jupyter等进行Python的编写
3. Web应用的制作
   如：
   - [颜色选择器](http://xuyuran.pythonanywhere.com/)
   - 正在尝试做数据库MySQL（将数据存入数据库）
4. 在老师的指导下完成了相关的课本案例、云端、日志

##### 学习成果展示
- [网页设计](http://neria.gitee.io/xuyuran/)
- [个人gitee](https://gitee.com/neria)
- [个人GitHub](https://github.com/Xuyuran111)
- [个人简历](http://neria.gitee.io/resume/)

##### 个人介绍

1.我是一个平易近人的人，也喜欢去尝试新事物；
2.喜欢和他人合作，会虚心学习；
3.在学习上，有努力的学习专业知识，也有存在不足的地方，但我会努力去弥补不足，虚心去向他人请教学习，也希望搭档我们之间可以互补，合作

##### 联系方式

- QQ：1445433140
- 微信：XYR13424047613

###### 望大家能选择我，与我进行下一步的合作！
-------------------------------------------------------------------------------------------------------------------------------------
