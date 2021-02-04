# HttpRunner

## 1、介绍

HttpRunner 是一款面向 HTTP(S) 协议的通用测试框架，只需编写维护一份 Python/YAML/JSON 脚本，即可实现自动化测试、性能测试、线上监控、持续集成等多种测试需求。



### 1.1 设计理念

拥抱开源，不重复造轮子，而是将优秀的开源的项目组装成战车



### 1.2 主要特征

- 开源

- 包含Requests的全部特性，轻松实现HTTP(S)的各种测试需求
- 结合Pytest测试框架，以Python/YAML/JSON格式定义测试用例，用例管理简洁优雅
- 支持[HAR](https://httparchive.org/) 记录并转换为测试用例
- 支持[allure](https://docs.qameta.io/allure/) 报告，测试报告清晰明了
- 通过重复使用[locust](https://locust.io/) ，您可以进行性能测试



## 2、安装说明

### 2.1 下载安装

使用pip命令进行安装

```
pip install httprunner
```

安装后校验是否安装成功，可以使用如下命令进行校验

```
hrun -V
3.1.4
```

若版本号正常显示，则说明安装正常

安装HttpRunner后，以下5个命令会写入系统环境变量配置。

* httprunner：主命令，用于所有功能。 
* hrun`：指令`httprunner run的别名，用于运行YAML/JSON/Pytest 测试用例。
* hmake`: 指令`httprunner make的别名，将YAML/JSON用例转换成pytest用例。 
* har2case`：指令`httprunner har2case的别名，将HAR文件转换成 Python/YAML/JSON 用例。 
* locust: 利用[locust](https://locust.io/) 运行性能测试。



## 3、基础概念

### 3.1 测试步骤

测试用例是测试步骤的 有序 集合，而对于接口测试来说，每一个测试步骤应该就对应一个 API 的请求描述。



### 3.2 测试用例

HttpRunner `v3.x` 支持3种用例格式：`python`、`YAML`和`JSON`。强烈建议以`python格式而不是以前的`YAML /JSON`格式编写和维护测试用例。一条测试用例（testcase）应该是为了测试某个特定的功能逻辑而精心设计的，并且至少包含如下几点：

- 明确的测试目的
- 明确的输入
- 明确的测试步骤
- 明确的预期结果



### 3.3 测试套件

`测试用例集` 是 `测试用例` 的 `无序` 集合



## 4、创建项目

### 4.1 脚手架

httprunner可以快速构建项目的必要目录，不必自己一个一个的配置与搭建，如果你想创建一个测试项目，可以使用脚手架快速创建

创建项目：

```
httprunner startproject 项目名称
```



### 4.2 项目文件结构

![image-20210202170632662](.\images\image-20210202170632662.png)

- har : 可以存放录制导出的.har文件
- reports  : 存储测试报告
- testcases :  存放测试用例
- .env : 放置在项目根目录下，存放一些环境变量
- .gitignore:  设置上传到git时需要忽略哪些文件信息
- debugtalk.py : 放置在项目根目录下（借鉴了pytest的conftest文件的设计）
  - 该文件存在时，将作为项目根目录定位标记，其所在目录即被视为项目工程根目录
  - 该文件不存在时，运行测试的所在路径（`CWD`）将被视为项目工程根目录
  - 测试用例文件中的相对路径（例如`.csv`）均需基于项目工程根目录
  - 运行测试后，测试报告文件夹（`reports`）会生成在项目工程根目录



### 4.3 运行脚手架项目

官方提供给我们两个小例子，因此我们无需任何编辑即可运行测试。

例子放在testcases下，是两个yml文件：

![image-20210202171244735](.\images\image-20210202171244735.png)



在项目demo的上级目录执行：

```
hrun demo
```



## 5、执行方式

### **5.1  httprunner的两种执行方式**

####  5.1.1.  pytest interfaceDemo

​		使用pytest执行

​		注意，前提是已经将yml或json文件转换成对应的**pytest文件**，否则不生效



#### **5.1.2.  hrun interfaceDemo**

​		命令等价于**httprunner run interfaceDemo**，其中先进行**httprunner make json/yml**，会将**json/yml文件先转换为python文件**，之后再执行**hrun**(httprunner run)。

​		如果python文件是已经存在的（你直接编写的python文件，而不是yml或者json），httprunner会直接运行你的python脚本，不需要进行转换，**官方推荐：直接使用python编写**。

​		运行后在tacecases目录下会生成了三个py文件，生成的py文件会加上_test后缀，如果yml或者json文件有修改，需要再次执行yml文件才会将py文件同步，或者直接修改py文件。

![snipaste_20201211_151427](.\images\snipaste_20201211_151427.png)

同时会自动生成logs日志文件，每一个yml都会对应生成一个日志文件，每一个testcase脚本都会有唯一的id，对应了日志文件的文件名：

![snipaste_20201211_162233](.\images\snipaste_20201211_162233.png)







## 6、快速上手

​		为了简化测试用例的编写工作，HttpRunner 实现了测试用例生成的功能，简单来说，就是当前主流的抓包工具和浏览器都支持将抓取得到的数据包导出为标准通用的 HAR 格式（HTTP Archive），然后HttpRunner 实现了将 HAR 格式的数据包转换为`Python/YAML/JSON`格式的测试用例文件的功能。

### 6.1 获取 HAR 数据包

在转换生成测试用例之前，需要先将抓取得到的数据包导出为 HAR 格式的文件。

在Fiddler中的操作方式为，选中需要转换的接口（可多选或全选），点击File，在悬浮的菜单目录中点击【Export Session】，格式选择`HTTP Archive(.har)`后保存即可。

![image-20210114144537892](.\images\image-20210114144537892.png)



### 6.2 har2case生成用例

HttpRunner 3.x版本开始，har2case 将HAR文件默认转换成 python 文件，强烈建议以python格式而不是以前的YAML / JSON格式编写和维护测试用例。当然，你也可以生成 YAML/JSON 测试用例。 只需要在 har2case 命令后 添加-2y或-2j。

```
--生成py文件   	 har2case 文件路径
--生成yaml文件   har2case 文件路径 -2y  
--生成json文件   har2case 文件路径 -2j
```

> har2case生成的用例代码过度冗余，推荐大家自己编写用例
>
> 那我们结合一个百度小案例去了解如何编写一条用例吧



## 7、 编写测试用例

### 7.1 测试用例结构

在httprunner中，测试用例组织主要基于两个概念： 

- 测试模块：一个python文件对应一个模块

* 测试用例(testcase): 对应一个类，包含单个或多个测试步骤。 
* 测试步骤(teststep): 对应Python中 teststeps下的一个节点，描述单次接口测试的全部内容，包括发起接口请求、解析响应结果、检验结果等。

也就是说一个testcase里（就是一个Python文件）可以有一个或者多个测试步骤，就是teststeps[]列表里的Step。

![image-20210204112026189](.\images\image-20210204112026189.png)



### 7.2 参数详解

#### 7.2.1 config（全局配置）

​		每个测试用例都应该有`config`部分，用以配置用例级别的参数

##### name（必要）

​		指定用例名，将在日志和报告中展示。

![image-20210202182117898](.\images\image-20210202182117898.png)



##### base_url(可选)

​		指定被测产品的通用路径，如果`base_url`中设定了通用路径，测试步骤中的url只能写相对路径。当你要在不同环境下测试时，这个配置非常有用。

![image-20210202182338651](.\images\image-20210202182338651.png)



##### variables(可选)

​		指定用例的通用参数。每个测试步骤都可以引用在config中variables设置的参数。但是step内部的 variables 优先级高于config中的 variables

![image-20210202182649721](.\images\image-20210202182649721.png)

##### verify（可选）

​		通常设置为False，当请求https请求时，就会跳过验证。如果你运行时候发现抛错SSLError，可以检查一下是不是verify没传，或者设置了True。



#### 6.3.2.2 teststeps（测试步骤）

​		每个测试用例都有1个或多个测试步骤（List[step]），每个测试步骤对应一个API请求或其他用例的引用。



##### RunRequest(name)

​		测试步骤中的`RunRequest`用于发送API请求和校验response。 RunRequest的name用来定义测试步骤的名称，将出现在log和测试报告中。

![image-20210203100033626](.\images\image-20210203100033626.png)





##### with_variables

​		设置测试步骤的变量。每个测试步骤的变量都是独立的，如果想在多个测试步骤中共享变量，需要在			config variables中定义。测试步骤中的变量，会覆盖config variables中的同名变量。

![image-20210203101158939](.\images\image-20210203101158939.png)



##### 		.with_headers

​		设置请求的headers，相当于[requests.request](https://requests.readthedocs.io/en/master/api/#requests.request) 中的headers。

##### 		.with_cookies

​		设置Http请求cookies，相当于[requests.request](https://requests.readthedocs.io/en/master/api/#requests.request) 中的cookies。



##### 请求方式及提交参数格式

​	设置http方法和Url,如果config中设置了通用路径，那step内部的url只能写相对路径。

- with_params：设置url的参数，相当于[requests.request](https://requests.readthedocs.io/en/master/api/#requests.request) 中的 params
- with_data：设置url的参数，相当于[requests.request](https://requests.readthedocs.io/en/master/api/#requests.request) 中的 data
- with_json：设置url的参数，相当于[requests.request](https://requests.readthedocs.io/en/master/api/#requests.request) 中的 json



![image-20210203102144300](.\images\image-20210203102144300.png)





##### 		validate（断言）

在httprunner框架中，可以使用assert来进行断言。

原理：用[jmespath](https://jmespath.org/) 提取Json response的内容，并进行断言校验。

格式：

```
 断言方式: (jmespath表达式, expected_value, message)
```

- jmespath表达式：指定断言内容
- expected_value: 指定期望值
- message: 用于描述断言失败原因

支持的断言方式：

```
equal: 等于
contains: 预期结果是否被包含在实际结果中
```

![image-20210203151934482](.\images\image-20210203151934482.png)



## 8、 扩展

### 8.1 参数化

**httprunner3.x支持三种格式的参数化数据**

如需对某测试用例(testcase)实现参数化数据驱动，需要使用parameters关键字，定义参数名称并指定数据源取值方式。



#### 8.1.1 列表形式实现参数化

单个参数实现参数化

```
import pytest
from httprunner import HttpRunner, Config, Step, RunRequest,Parameters


class TestBaiduSearch(HttpRunner):

    @pytest.mark.parametrize("params", Parameters({"wwd": ["12306", "10086"]}))
    def test_start(self, params):
        super().test_start(params)

    config = Config("访问百度搜索12306用例")\
        .base_url("http://www.baidu.com")\
        .variables(wd="12306")

    teststeps = [

        Step(
            RunRequest("1")
            .with_variables(wd="10086")
            .get("/s")

            .with_params(**{"wd":"$wwd"})
            .validate()
            .assert_equal("status_code", 200)
        )

    ]
```

多个 参数如何实现参数化：

```
import pytest
from httprunner import HttpRunner, Config, Step, RunRequest,Parameters


class TestAPIServer(HttpRunner):

	#此处采用parameters定义数据
    @pytest.mark.parametrize("params", Parameters({"username-password": [["admin", "123456"],["karsa", "123456"]]}))
    def test_start(self, params):
        super().test_start(params)

    config = Config("登录用例")\
        .base_url("http://127.0.0.1:8888/api/private/v1/")\
        .variables()\
        .verify(False)

    teststeps = [
        Step(
            RunRequest("登录接口")
            .with_variables()
            .post("/login")
            .with_data({"username":"$username", "password": "$password"})
        )
    ]
```

如有多个参数，则需要在键处以-分割，后续一个小列表代表一条数据

![image-20210204102527016](.\images\image-20210204102527016.png)



#### 8.1.2 **debugtalk的回调函数**

**在debugtalk.py中定义一个函数，返回列表：**

```
def get_data():
    return [
               {"username":"admin", "password":"123456"},
               {"username":"karsa", "password":"123456"}
    ]
```

**在用例文件进行调用：**

```
import pytest
from httprunner import HttpRunner, Config, Step, RunRequest,Parameters


class TestAPIServer(HttpRunner):
    @pytest.mark.parametrize("params", Parameters({"username-password": "${get_data()}"}))
    def test_start(self, params):
        super().test_start(params)

    config = Config("登录用例")\
        .base_url("http://127.0.0.1:8888/api/private/v1/")\
        .variables()\
        .verify(False)   

    teststeps = [
        Step(
            RunRequest("登录接口")
            .with_variables()
            .post("/login")
            .with_data({"username":"$username", "password": "$password"})
        )
    ]
```

![image-20210204102901329](.\images\image-20210204102901329.png)

#### 8.1.3 csv数据文件驱动

**使用csv文件作为参数化输入时注意三点：**

**1，csv文件中的title要为变量名**

![image-20210204103057183](.\images\image-20210204103057183.png)

**2，csv映射的时候，参数名要以“-”分割**

**3，csv的路径要使用相对路径，不支持绝对路径不支持\\符号的路径**

csv文件代码如下

```
username,password
admin,123456
karsa,123456
```

测试用例如下

```
import pytest
from httprunner import HttpRunner, Config, Step, RunRequest,Parameters


class TestAPIServer(HttpRunner):
    @pytest.mark.parametrize("params", Parameters({"username-password": "${parameterize(data/demo_test.csv)}"}))
    def test_start(self, params):
        super().test_start(params)

    config = Config("登录用例")\
        .base_url("http://127.0.0.1:8888/api/private/v1/")\
        .variables()\
        .verify(False)

    teststeps = [
        Step(
            RunRequest("登录接口")
            .with_variables()
            .post("/login")
            .with_data({"username":"$username", "password": "$password"})
        )
    ]
```

![image-20210204103346261](.\images\image-20210204103346261.png)



### 8.2 关联

​		众所周知，在接口测试工作中，接口之间参数的关联是再常见不过的应用，Httprunner3.x在这一块的支持上用法也是相当丰富，下面结合实例简单记录一下使用技巧

#### 8.2.1 extract 参数提取

在用例步骤中提取变量，不需要导出就可以向下面的步骤中传递变量，看例子

![image-20210203160200749](.\images\image-20210203160200749.png)

如上例子：

1. 在`Step1`中通过`extract()`提取了token变量,此时是不需要声明方法`export()`导出的。

​	2.在`Step2`中直接通过`$token`引用导出的变量，即可完成参数的传递和关联



#### 8.2.2 export 参数导出

httprunner中的debugtalk提供了非常方便的辅助功能

有时候我们希望能跨文件使用公用的参数。
比如登录生成一个token，后面的用例都可以去引用这个token值，或者有些复杂的逻辑，需要写个函数去实现，比如操作数据库等
httprunner中可以使用 `debugtalk.py` 写辅助函数，实现复杂的功能。

在debugtalk中定义一个返回token的函数

```
def get_token():
    username="admin"
    password="123456"
    data = {"username": username, "password": password}
    res = requests.post("http://127.0.0.1:8888/api/private/v1/login", data=data)
    token = res.json()["data"]["token"]
    return token
```

用例文件调用如下

![image-20210203162354813](.\images\image-20210203162354813.png)

也可不必在confog中导出直接调用辅助函数



### 8.3 实现文件上传

对于上传文件类型的测试场景，HttpRunner 集成 [requests_toolbelt](https://toolbelt.readthedocs.io/en/latest/uploading-data.html) 实现了上传功能。

使用内置 upload 关键字，可轻松实现上传功能

![image-20210204103557746](.\images\image-20210204103557746.png)



## 9、 测试报告

### 9.1 pytest-html

受益于pytest的集成，HttpRunner v3.x可以利用pytest所有插件，包括`pytest-html`和`allure-pytest`

**pytest-html报告**

pytest-html 插件随HttpRunner一起安装。当你运行测试用例想生成html报告时，可以在命令行中添加 --html
也可在pytest.ini配置文件当中追加参数使用

**命令格式**：

```
pytest 用例路径 --html=reports/report.html
```

**配置文件：**

```
[pytest]
addopts = -s --html=reports/report.html
testpaths = 
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```



### 9.2 allure报告

我们测试的结果一定是通过一个报告来进行体现。Allure 是一个独立的报告插件，可以生成美观易读的报告

`allure-pytest`是HttpRunner的可选依赖项，如果想生成allure报告时，需要单独安装：

```
pip install allure-pytest
```

或者在安装httprunner时选择安装：

```
pip install "httprunner[allure]"
```

一旦`allure-pytest`安装好，可以用`hrun/pytest`使用如下功能： * `--alluredir = DIR`：在指定目录中生成“魅力”报告



**Allure报告在运行以后会生成存储结果的数据文件，程序运行结束后，我们会在项目中得到一个 json 文件**

```
hrun 用例路径 --alluredir=reports
```

- --alluredir 后面的 reports 为数据文件 输出的目录名
- 如果希望目录名叫 result 那么可以将命令行参数改为 --alluredir=result



**要在测试完成后查看实际报告，您需要使用一个allure工具。

首先要安装allure辅助程序

   1.https://bintray.com/qameta/generic/allure2 下载allure- 2.7.0

2. 解压缩到一个目录（不经常动的目录）

3. 将压缩包内的 bin 目录配置到 path 系统环境变量

4. 在命令行中敲 allure 命令，如果提示有这个命令，即为成功



**通过命令生成html格式的allure报告**

```
allure generate reports/allure -o reports/allure/html
```



## 10、项目实战

**使用HttpRunner完成api_server的接口测试**

### 10.1 新建项目

使用脚手架工具快速创建项目



### 10.2 用例组织

创建python文件编写用例



### 10.3 执行测试



### 10.4 生成报告









