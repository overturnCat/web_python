# blog的修改
## 为markdown添加额外功能
1. 增加markdown拓展， 在handers.py中修改原本 `blog.html_content = markdown2.markdown(blog.content`为
```python
blog.html_content = markdown2.markdown(blog.content, extras=["header-ids", "toc", "code-friendly"])
```
> header-ids:给head自动编号，只能识别ascii码，空格会转为'-'
> 
> toc:暂时未实现
>
> code-friendly:取消\_\_的强调功能, 为mathjax语法做准备
2. 添加mathjax数学公式支持
在\_\_base\_\_.html中加入
```html
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML' async></script>
```

## 添加分类功能
1. 修改数据库添加catagory列

```sql

```
