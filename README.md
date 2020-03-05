# blog的修改
## 为markdown添加额外功能

* 增加markdown拓展， 在handers.py中修改原本 `blog.html_content = markdown2.markdown(blog.content`为
```python
blog.html_content = markdown2.markdown(blog.content, extras=["header-ids", "toc", "code-friendly"])
```
> header-ids:给head自动编号，只能识别ascii码，空格会转为'-'
> 
> toc:暂时未实现
>
> code-friendly:取消\_\_的强调功能, 为mathjax语法做准备

* 添加mathjax数学公式支持
在\_\_base\_\_.html中加入
```html
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML' async></script>
```

## 添加分类功能
1. 修改数据库添加catagory列
```sql
alter table `blogs` add column `catagory` varchar(50) NOT NULL DEFAULT '未分类';
```

2. 博客编写页面加入分类栏
    1. 在`templates/blog.html`中添加分类的输入
    2. 在`Model.py` 修改Blog的模型
3. 主页面加入分类栏, 并可以按种类进行显示查询
