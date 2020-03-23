###swig是JS模板引擎，它有如下特点：

    1.根据路劲渲染页面
    2.面向对象的模板继承，页面复用动态页面快速上手功能强大。

##**Swig 使用指南**
###**如何使用**
    API
    swig.init({
      allowErrors: false,   
      autoescape: true,
      cache: true,
      encoding: 'utf8',
      filters: {},
      root: '/',
      tags: {},
      extensions: {},
      tzOffset: 0
    });
###**options:**

**allowErrors:** 

    默认值为 false。将所有模板解析和编译错误直接输出到模板。如果为 true，则将引发错误，抛出到 Node.js 进程中，可能会使您的应用程序崩溃。
    
**autoescape:** 

    默认true，强烈建议保持。字符转换表请参阅转义过滤器。
    true: HTML安全转义 
    false: 不转义，除非使用转义过滤器或者转义标签
    
**'js':** 

    js安全转义
    
**cache:** 

    更改为 false 将重新编译每个请求的模板的文件。正式环境建议保持true。
    
**encoding:** 

    模板文件编码
    
**root:** 

    需要搜索模板的目录。如果模板传递给 swig.compileFile 绝对路径(以/开头)，Swig不会在模板root中搜索。如果传递一个数组，使用第一个匹配成功的数组项。
    
**tzOffset:** 

    设置默认时区偏移量。此设置会使转换日期过滤器会自动的修正相应时区偏移量。
    
**filters:** 
    
    自定义过滤器或者重写默认过滤器，参见自定义过滤器指南。
    
**tags** 

    自定义标签或者重写默认标签，参见自定义标签指南。

**extensions**

    添加第三方库，可以在编译模板时使用，参见参见自定义标签指南。
    
##**nodejs**
    var tpl = swig.compileFile("path/to/template/file.html");
    var renderedHtml = tpl.render({ vars: 'to be inserted in template' });
    or

    var tpl = swig.compile("Template string here");
    var renderedHtml = tpl({ vars: 'to be inserted in template' });
    
##**结合Express**
    npm install express
    npm install consolidate
然后

    app.engine('.html', cons.swig);
    app.set('view engine', 'html');
    
##**浏览器**
Swig浏览器版本的api基本与nodejs版相同，不同点如下：

    不能使用swig.compileFile，浏览器没有文件系统
    你必须提前使用swig.compile编译好模板
    按顺序使用extends, import, and include，同时在swig.compile里使用参数templateKey来查找模板

    var template = swig.compile('<p>{% block content %}{% endblock %}</p>', { filename: 'main' });
    var mypage = swig.compile('{% extends "main" %}{% block content %}Oh hey there!{% endblock %}', { filename: 'mypage' });
    
    
##**基础**
###**swig的变量：**
    `{{ foo.bar }}`或`{{ foo['bar'] }}`
    //如果变量未定义，输出空字符。

###**swig的标签：**
####**extends**：使当前模板继承父模板，必须在文件最前

    参数： file父模板相对模板root的相对路径，将在后面介绍如何实现模板继承。

####**block：**定义一个块，使之可以被继承的模板重写，或者重写父模板的同名块，在继承`block`块时可以使用父模板中已定义的部分
    
    参数： name块的名字，必须以字母数字下划线开头

####**parent：**将父模板中同名块注入当前块中

    {% block head %}
        {% parent %}
        <link rel="stylesheet" href="custom.css">
    {% endblock %} 

####**include：**包含一个模板到当前位置，这个模板将使用当前上下文 
    
    参数： file包含模板相对模板 root 的相对路径 

        {% include "a.html" %}
        {% include "template.js" %}
        //将引入的文件内容放到被引用的地方
####**raw：**停止解析标记中任何内容，所有内容都将输出
    
    参数： file父模板相对模板 root 的相对路径

####**for：**遍历对象和数组

    参数：
    x：当前循环迭代名；
    in:语法标记；
    y:可迭代对象。
    
    {% for x in y %}
        {{ x }}
    {% endfor %}
        
特殊循环变量 

    loop.index：当前循环的索引（1开始） 
    loop.index0：当前循环的索引（0开始） 
    loop.revindex：当前循环从结尾开始的索引（1开始） 
    loop.revindex0：当前循环从结尾开始的索引（0开始） 
    loop.key：如果迭代是对象，是当前循环的键，否则同 loop.index 
    loop.first：如果是第一个值返回 true 
    loop.last：如果是最后一个值返回 true 
    loop.cycle：一个帮助函数，以指定的参数作为周期

example：
    
    eg1------------------------------
    
    {% for x in y %}
        {% if loop.first %}<ul>{% endif %}
        <li>{{ loop.index }} - {{ loop.key }}: {{ x }}</li>
        {% if loop.last %}</ul>{% endif %}
    {% endfor %}
   
    eg2 ------------------------------
    
    {% for item in items %}
        <li class="{{ loop.cycle('odd', 'even') }}">{{ item }}</li>
    {% endfor %}
    
    eg3-----------------------------
    
    在 for 标签里使用 else
    {% for person in people %}
        {{ person }}
    {% else %}
        There are no people yet!
    {% endfor %}
    ------------------------------------
####**if**：条件语句
    参数：接受任何有效的JavaScript条件语句

    {% if x %}{% endif %}
    {% if !x %}{% endif %}
    {% if not x %}{% endif %}
    
    {% if x and y %}{% endif %}
    {% if x && y %}{% endif %}
    {% if x or y %}{% endif %}
    {% if x || y %}{% endif %}
    {% if x || (y && z) %}{% endif %}
example：

    `if else` 和 `else if`
    
    {% if foo %}
        //Some content.
    {% elseif "foo" in bar %}
        //Content if the array `bar` has "foo" in it.
    {% else %}
        Fallback content.
    {% endif %}

    {% if x [operator] y %}
        //Operators: ==, !=, <, <=, >, >=, ===, !==
    {% endif %}
    
    {% if x == 'five' %}
        //The operands can be also be string or number literals
    {% endif %}
    
    {% if x|length === 3 %}
        //You can use filters on any operand in the statement.
    {% endif %}
    
    {% if x in y %}
        //If x is a value that is present in y, this will return true.
    {% endif %}
####**autoescape：**改变当前变量的自动转义行为 
    参数： 
    on：当前内容是否转义
    type: 转义类型，js 或者 html，默认 html
    
    example:
        input = '<p>Hello "you" & /'them/'</p>';
        {% autoescape false %}
            {{ input }}
        {% endautoescape %}
        // <p>Hello "you" & 'them'</p>
        
        {% autoescape true %}
            {{ input }}
        {% endautoescape %}
        //<p>Hello "you" & 'them' </p>
        
        {% autoescape true "js" %}
            {{ input }}
        {% endautoescape %}
        // /u003Cp/u003EHello /u0022you/u0022 & /u0027them/u0027/u003C/u005Cp/u003E 
####**set:**设置一个变量，在当前上下文中复用

    {% set foo = [0, 1, 2, 3, 4, 5] %} 
    
    {% for num in foo %}
        <li>{{ num }}</li> 
    {% endfor %} 
    
####**filter:**对整个块应用过滤器
    参数： 
    filter_name: 过滤器名字 
    若干传给过滤器的参数 父模板相对模板 root 的相对路径
    
example:

    {% filter uppercase %}oh hi, {{ name }}{% endfilter %}
    {% filter replace "." "!" "g" %}Hi. My name is Paul.{% endfilter %}

输出

    OH HI, PAUL
    Hi! My name is Paul!
    
####**spaceless:**尝试移除html标签间的空格

    {% spaceless %}
    {% for num in foo %}
        <li>{{ loop.index }}</li>
    {% endfor %}
    {% endspaceless %}
输出
    <li>1</li><li>2</li><li>3</li>
    
####**import:**允许引入另一个模板的宏进入当前上下文
    参数： 
    file: 引入模板相对模板 root 的相对路径 
    as: 语法标记 var: 分配给宏的可访问上下文对象
    
expample:

    {% import 'formmacros.html' as form %}

    {# this will run the input macro #}
    {{ form.input("text", "name") }}
    
    {# this, however, will NOT output anything because the macro is scoped to the "form" object: #}
    {{ input("text", "name") }}
    
####**macro:** 创建自定义可服用的代码段
    参数：...: 
    用户定义
    
example:

    {% macro input type name id label value error %}
        <label for="{{ name }}">{{ label }}</label>
        <input type="{{ type }}" name="{{ name }}" id="{{ id }}" value="{{ value }}"{% if error %} class="error"{% endif %}>
    {% endmacro %}

然后像下面使用

    <div>{{ input("text", "fname", "fname", "First Name", fname.value, fname.errors) }}</div>
    <div>{{ input("text", "lname", "lname", "Last Name", lname.value, lname.errors) }}</div>

输出如下

    <div>
        <label for="fname">First Name</label>
        <input type="text" name="fname" id="fname" value="Paul">
    </div>
    <div>
        <label for="lname">Last Name</label>
        <input type="text" name="lname" id="lname" value="" class="error">
    </div>
          
##**变量过滤器**
    用于修改变量。变量名称后用 | 字符分隔添加过滤器。您可以添加多个过滤器。

例子

    {{ name|title }} was born on {{ birthday|date('F jS, Y') }}
    
    and has {{ bikes|length|default("zero") }} bikes.
也可以使用 filter 标签来为块内容添加过滤器

    {% filter upper %}oh hi, paul{% endfilter %}

###内置过滤器

**add(value)**

    使变量与value相加，可以转换为数值字符串会自动转换为数值。
    
**addslashes** 

    用 \ 转义字符串
    
**capitalize** 
    大写首字母
    
**date(format[, tzOffset])** 

    转换日期为指定格式
    format： 格式 
    tzOffset： 时区
    
**default(value)** 

    默认值（如果变量为undefined，null，false）
    
**e**
    
    同`escape`

**escape([type])** 
    转义字符
    默认： &, <, >, ", ' 
    js: &, <, >, ", ', =, -, ;

**first** 

    返回数组第一个值

**join(glue)** 

    同`[].join`
    
**json_encode([indent])** 

    类似JSON.stringify, indent为缩进空格数
    
**last** 

    返回数组最后一个值
    
**length** 

    返回变量的length，如果是object，返回key的数量

**lower** 

    同''.toLowerCase()

**raw** 

    指定输入不会被转义

**replace(search, replace[, flags])** 
   
    同**''.replace**
    
**reverse** 

    翻转数组

**striptags** 

    去除html/xml标签

**title** 

    大写首字母

**uniq** 

    数组去重

**upper** 

    同`''.toUpperCase`
    
**url_encode** 

    同`encodeURIComponent`
    
**url_decode** 

    同decodeURIComponemt

###自定义过滤器
创建一个 myfilter.js 然后引入到 Swig的初始化函数中

    swig.init({ filters: require('myfilters') });
在 myfilter.js 里，每一个`filter`方法都是一个简单的 js 方法，下例是一个翻转字符串的 `filter`：

    exports.myfilter = function (input) {
        return input.toString().split('').reverse().join('');
    };
你的 `filter` 一旦被引入，你就可以向下面一样使用：

    {{ name|myfilter }}

    {% filter myfilter %}I shall be filtered{% endfilter %}
你也可以像下面一样给 `filter` 传参数：

    exports.prefix = function(input, prefix) {
        return prefix.toString() + input.toString();
    };

    {{ name|prefix('my prefix') }}

    {% filter prefix 'my prefix' %}I will be prefixed with "my prefix".{% endfilter %}
    
    {% filter prefix foo %}I will be prefixed with the value stored to `foo`.{% endfilter %}
###模板继承:
####Swig 使用 extends 和 block 来实现模板继承

    example:
    
    //layout.html
    
    <!doctype html>
    <html>
        <head>
            <meta charset="utf-8">
            <title>
                {% block title %}
                    My Site
                {% endblock %}
            </title>
            {% block head %}
            {% endblock %}
        </head>
        <body>
            {% block content %}
            {% endblock %}
        </body>
    </html> 
    
    //index.html
    
    {% extends './layout.html' %}
    
    {% block title %}
        My Page
    {% endblock %}
    
    {% block head %}
        {% parent %}
    {% endblock %}
    
    {% block content %}
        <p>This is just an awesome page.</p>
        <h1>hello,lego.</h1>
        <script>
            //require('pages/index/main');
        </script>
    {% endblock %}
    
####swig模板经过编译后：
    <!doctype html>
    <html>
        <head>
            <meta charset="utf-8">
            <title>My Page</title>
        </head>
        <body>
            <p>This is just an awesome page.</p>
            <h1>hello,lego.</h1>
            <p>test</p>
            <script>
                //require('pages/index/main');
            </script>
        </body>
    </html>
###swig模板在fis3中的应用
    swig的模板继承可以更好的帮我们组织代码结构，更好的复用代码。类似jello扩展的velocity标签。
    
        如上个例子，可以将公用代码写在一个文件里，作为母版页，需要的页面就继承这个页面，而且页面中的block又可以方便我们自定义需要的内容。在fis3构建中使用时，调用swig编译插件，将swig标签解析成正常的html文件即可。
