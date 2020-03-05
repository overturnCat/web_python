
# blog的修改

## 参考
为标题编号 [yanwei](https://yanwei.github.io/misc/markdown-auto-number-title.html)

makrdown2 extras信息 https://github.com/trentm/python-markdown2/wiki/Extras
pygments-css https://github.com/richleland/pygments-css

## 安装
要求 python3.6以上
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:jonathonf/python-3.6
sudo apt-get update
sudo apt-get install python3.6
```

安装依赖
```sh
# 安装ssh
sudo apt-get install openssh-server
# 安装mysql, nginx, supervisor
sudo apt-get install nginx supervisor python3 mysql-server python3-pip
# 安装需要的python库
pip install jinja2 aiomysql aiohttp  pygments markdown2
# 创建网站目录
mkdir /srv/
mkdir /srv/awesome
```

**配置Supervisor**  
编写一个Supervisor的配置文件awesome.conf，存放到/etc/supervisor/conf.d/目录下：
```
[program:awesome]

command     = /srv/awesome/www/app.py
directory   = /srv/awesome/www
user        = www-data
startsecs   = 3

redirect_stderr         = true
stdout_logfile_maxbytes = 50MB
stdout_logfile_backups  = 10
stdout_logfile          = /srv/awesome/log/app.log
```
配置文件通过[program:awesome]指定服务名为awesome，command指定启动app.py。
然后重启Supervisor后，就可以随时启动和停止Supervisor管理的服务了：

```
sudo supervisorctl reload
sudo supervisorctl start awesome
sudo supervisorctl status
```

**配置Nginx**  
Supervisor只负责运行app.py，我们还需要配置Nginx。把配置文件awesome放到/etc/nginx/sites-available/目录下：

```
server {
    listen      80; # 监听80端口

    root       /srv/awesome/www;
    access_log /srv/awesome/log/access_log;
    error_log  /srv/awesome/log/error_log;

    # server_name awesome.liaoxuefeng.com; # 配置域名

    # 处理静态文件/favicon.ico:
    location /favicon.ico {
        root /srv/awesome/www;
    }

    # 处理静态资源:
    location ~ ^\/static\/.*$ {
        root /srv/awesome/www;
    }

    # 动态请求转发到9000端口:
    location / {
        proxy_pass       http://127.0.0.1:9000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

然后在/etc/nginx/sites-enabled/目录下创建软链接：
```
$ pwd
/etc/nginx/sites-enabled
$ sudo ln -s /etc/nginx/sites-available/awesome .
```

让Nginx重新加载配置文件，不出意外，我们的awesome-python3-webapp应该正常运行：
```
$ sudo /etc/init.d/nginx reload
```

## 为markdown添加额外功能
1. 增加markdown拓展， 在handers.py中修改原本 `blog.html_content = markdown2.markdown(blog.content`为
```python
blog.html_content = markdown2.markdown(blog.content, extras=["header-ids", "toc", "code-friendly", "fenced-code-blocks"])
```
> header-ids : 给head自动编号，只能识别ascii码，空格会转为'-'  
> toc : 暂时未实现  
> code-friendly : 取消\_\_的强调功能, 为mathjax语法做准备  
> fenced-code-blocks : code用pre包围  
> [其他](#implemented-extras) 

2. 给代码区添加颜色
* 首先`pip install pygments`, markdown2的code highlighting需要pygments的支持
* 在static/css/目录下`git clone https://github.com/richleland/pygments-css `
* 在pygments-css目录下的default.css文件中的`.highlight`改为`.codehilite`
* 在templates/\_\_base\_\_.html文件中加入如下一行`<link rel="stylesheet" href="/static/css/pygments-css/default.css">` 

3. 添加mathjax数学公式支持
在\_\_base\_\_.html中加入
```html
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML' async></script>
```

4. 标题自动编号
在css文件夹创建文件auto-number-title.css

```txt
.uk-article h1 { counter-reset: h2counter; }
.uk-article h2 { counter-reset: h3counter; }
.uk-article h3 { counter-reset: h4counter; }
.uk-article h4 { counter-reset: h5counter; }
.uk-article h5 { counter-reset: h6counter; }
.uk-article h6 { }
.uk-article h2:before {
  counter-increment: h2counter;
  content: counter(h2counter) ".\0000a0\0000a0";
}
.uk-article h3:before {
  counter-increment: h3counter;
  content: counter(h2counter) "."
            counter(h3counter) ".\0000a0\0000a0";
}
.uk-article h4:before {
  counter-increment: h4counter;
  content: counter(h2counter) "."
            counter(h3counter) "."
            counter(h4counter) ".\0000a0\0000a0";
}
.uk-article h5:before {
  counter-increment: h5counter;
  content: counter(h2counter) "."
            counter(h3counter) "."
            counter(h4counter) "."
            counter(h5counter) ".\0000a0\0000a0";
}
.uk-article h6:before {
  counter-increment: h6counter;
  content: counter(h2counter) "."
            counter(h3counter) "."
            counter(h4counter) "."
            counter(h5counter) "."
            counter(h6counter) ".\0000a0\0000a0";
}
```
在templates/blog.html的block中添加
```css
<link rel="stylesheet" type="text/css" href="static/css/auto-number-title.css" />
```


## 添加分类功能
1. 修改数据库添加catagory列
```sql
use awesome;
alter table `blogs` add column `catagory` varchar(50) NOT NULL DEFAULT '未分类';
GRANT ALL PRIVILEGES ON `awesome`.* TO 'ayuan'@'localhost';
FLUSH   PRIVILEGES; 
```
2. 博客编写页面加入分类栏
    1. 在`templates/blog.html`中添加分类的输入
    2. 在`Model.py` 修改Blog的模型
    3. 在`handers.py` 中修改`api_create_blog`和`api_update_blog`
3. 主页面加入分类栏, 并可以按种类进行显示查询
    1. 在`orm.py` 中添加可以查询分类的select API
    ```python
    @classmethod
    @asyncio.coroutine
    def findDictinct(cls, selectField):
        ' find number by select and where. '
        sql = ['select %s, count(*) num from `%s` group by %s' % (selectField, cls.__table__, selectField)]
        args = []
        rs = yield from select(' '.join(sql), args)
        # 返回所有符合条件的dict对象, 如r=User.findAll(where='admin=1')
        return [cls(**r) for r in rs]
    ```
    2. 在主页显示分类, 修改`handlers.py中的index函数`, 使其传入catagory参数
    3. 在handlers.py中添加分类函数
    ```
    @get('/blog/catagory/{cata}')
    def index_catagory(cata, *, page='1'):
        logging.info("/blog/catagory/{}".format(cata))
        page_index = get_page_index(page)
        num = yield from Blog.findNumber('count(id)', where="catagory='%s'" % cata)
        logging.info("blog/catagory num:{}".format(num))
        page = Page(num)
        catagory = yield from Blog.findDictinct('catagory')
        if num == 0:
            blogs = []
        else:
            blogs = yield from Blog.findAll(orderBy='created_at desc', where="catagory='%s'" % cata, limit=(page.offset, page.limit))
        return {
            '__template__': 'blogs.html',
            'page': page,
            'blogs': blogs,
            'catagory': catagory
        }
    ```
