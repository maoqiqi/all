# 表单

## 目录

* [处理表单的输入](#处理表单的输入)
* [验证表单的输入](#验证表单的输入)
  * [必填字段](#必填字段)
  * [数字](#数字)
  * [中文](#中文)
  * [英文](#英文)
  * [电子邮件地址](#电子邮件地址)
  * [手机号码](#手机号码)
  * [下拉菜单](#下拉菜单)
  * [单选按钮](#单选按钮)
  * [复选框](#复选框)
  * [日期和时间](#日期和时间)
  * [身份证号码](#身份证号码)
* [预防跨站脚本](#预防跨站脚本)
* [防止多次递交表单](#防止多次递交表单)
* [处理文件上传](#处理文件上传)
  * [客户端上传文件](#客户端上传文件)

## 处理表单的输入

先来看一个表单递交的例子,我们有如下的表单内容,命名成文件login.gtpl(放入当前新建项目的目录里面)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<form action="/login" method="post">
    <label>用户名:<input type="text" name="username"></label>
    <label>密码:<input type="password" name="password"></label>
    <input type="submit" value="登录">
</form>

</body>
</html>
```

上面递交表单到服务器的`/login`,当用户输入信息点击登录之后,会跳转到服务器的路由`login`里面,我们首先要判断这个是什么方式传递过来,POST还是GET呢？

http包里面有一个很简单的方式就可以获取,我们在前面web的例子的基础上来看看怎么处理login页面的form数据。

```
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	_ = r.ParseForm() // 解析url传递的参数,对于POST则解析响应包的主体（request body）
	// 注意:如果没有调用ParseForm方法,下面无法获取表单的数据
	fmt.Println(r.Form) // 这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	}
	_, _ = fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) // 获取请求的方法
	if r.Method == "GET" {
		t, _ := template.ParseFiles("login.gtpl")
		log.Println(t.Execute(w, nil))
	} else {
		// 请求的是登录数据,那么执行登录的逻辑判断
		fmt.Println("username:", r.Form["username"])
		fmt.Println("password:", r.Form["password"])
	}
}

func main() {
	http.HandleFunc("/", sayhelloName)       // 设置访问的路由
	http.HandleFunc("/login", login)         // 设置访问的路由
	err := http.ListenAndServe(":9090", nil) // 设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

通过上面的代码我们可以看出获取请求方法是通过`r.Method`来完成的,这是个字符串类型的变量,返回GET, POST, PUT等method信息。

login函数中我们根据`r.Method`来判断是显示登录界面还是处理登录逻辑。当GET方式请求时显示登录界面,其他方式请求时则处理登录逻辑,如查询数据库、验证登录信息等。

当我们在浏览器里面打开`http://127.0.0.1:9090/login`的时候,出现如下界面:

![用户登录界面](images/form_login.png?raw=true)

如果你看到一个空页面,可能是你写的 login.gtpl 文件中有错误,请根据控制台中的日志进行修复。 

我们输入用户名和密码之后发现在服务器端是不会打印出来任何输出的,为什么呢？默认情况下,Handler里面是不会自动解析form的,必须显式的调用`r.ParseForm()`后,你才能对这个表单数据进行操作。
我们修改一下代码,在`fmt.Println("username:", r.Form["username"])`之前加一行`r.ParseForm()`,重新编译,再次测试输入递交,现在是不是在服务器端有输出你的输入的用户名和密码了。

`r.Form`里面包含了所有请求的参数,比如URL中query-string、POST的数据、PUT的数据,所以当你在URL中的query-string字段和POST冲突时,会保存成一个slice,里面存储了多个值,
Go官方文档中说在接下来的版本里面将会把POST、GET这些数据分离开来。

现在我们修改一下login.gtpl里面form的action值`http://127.0.0.1:9090/login`修改为`http://127.0.0.1:9090/login?username=astaxie`,再次测试,服务器的输出username是不是一个slice。服务器端的输出如下：

![服务器端打印接收到的信息](images/form_login_slice.png?raw=true)

`request.Form`是一个url.Values类型,里面存储的是对应的类似`key=value`的信息,下面展示了可以对form数据进行的一些操作:

```
v := url.Values{}
v.Set("name", "Ava")
v.Add("friend", "Jess")
v.Add("friend", "Sarah")
v.Add("friend", "Zoe")
// v.Encode() == "name=Ava&friend=Jess&friend=Sarah&friend=Zoe"
fmt.Println(v.Get("name"))
fmt.Println(v.Get("friend"))
fmt.Println(v["friend"])
```
 
> Request本身也提供了FormValue()函数来获取用户提交的参数。如r.Form["username"]也可写成r.FormValue("username")。
调用r.FormValue时会自动调用r.ParseForm,所以不必提前调用。r.FormValue只会返回同名参数中的第一个,若参数不存在则返回空字符串。


## 验证表单的输入

开发Web的一个原则就是,不能信任用户输入的任何信息,所以验证和过滤用户的输入信息就变得非常重要,我们经常会在微博、新闻中听到某某网站被入侵了,存在什么漏洞,
这些大多是因为网站对于用户输入的信息没有做严格的验证引起的,所以为了编写出安全可靠的Web程序,验证表单输入的意义重大。

我们平常编写Web应用主要有两方面的数据验证,一个是在页面端的js验证(目前在这方面有很多的插件库,比如ValidationJS插件),一个是在服务器端的验证,我们这小节讲解的是如何在服务器端验证。

### 必填字段

你想要确保从一个表单元素中得到一个值,例如前面小节里面的用户名,我们如何处理呢？Go有一个内置函数`len`可以获取字符串的长度,这样我们就可以通过len来获取数据的长度,例如：

```
if len(r.Form["username"][0])==0{
	// 为空的处理
}
```

`r.Form`对不同类型的表单元素的留空有不同的处理,对于空文本框、空文本区域以及文件上传,元素的值为空值,而如果是未选中的复选框和单选按钮,则根本不会在r.Form中产生相应条目,
如果我们用上面例子中的方式去获取数据时程序就会报错。所以我们需要通过`r.Form.Get()`来获取值,因为如果字段不存在,通过该方式获取的是空值。
但是通过`r.Form.Get()`只能获取单个的值,如果是map的值,必须通过上面的方式来获取。

### 数字

你想要确保一个表单输入框中获取的只能是数字,例如,你想通过表单获取某个人的具体年龄是50岁还是10岁,而不是像"一把年纪了"或"年轻着呢"这种描述。

如果我们是判断正整数,那么我们先转化成int类型,然后进行处理

```
getint, err := strconv.Atoi(r.Form.Get("age"))
if err != nil {
    // 数字转化出错了,那么可能就不是数字
}
// 接下来就可以判断这个数字的大小范围了
if getint > 100 {
    // 太大了
}
```

还有一种方式就是正则匹配的方式。

```
if m, _ := regexp.MatchString("^[0-9]+$", r.Form.Get("age")); !m {
	return false
}
```

对于性能要求很高的用户来说,这是一个老生常谈的问题了,他们认为应该尽量避免使用正则表达式,因为使用正则表达式的速度会比较慢。
但是在目前机器性能那么强劲的情况下,对于这种简单的正则表达式效率和类型转换函数是没有什么差别的。
如果你对正则表达式很熟悉,而且你在其它语言中也在使用它,那么在Go里面使用正则表达式将是一个便利的方式。

> Go实现的正则是[RE2](http://code.google.com/p/re2/wiki/Syntax),所有的字符都是UTF-8编码的。

### 中文

有时候我们想通过表单元素获取一个用户的中文名字,但是又为了保证获取的是正确的中文,我们需要进行验证,而不是用户随便的一些输入。
对于中文我们目前有两种方式来验证,可以使用 `unicode` 包提供的 `func Is(rangeTab *RangeTable, r rune) bool` 来验证,
也可以使用正则方式来验证,这里使用最简单的正则方式,如下代码所示:

```
if m, _ := regexp.MatchString("^\\p{Han}+$", r.Form.Get("realname")); !m {
	return false
}
```

### 英文

我们期望通过表单元素获取一个英文值,例如我们想知道一个用户的英文名,应该是astaxie,而不是asta谢。

我们可以很简单的通过正则验证数据：

```
if m, _ := regexp.MatchString("^[a-zA-Z]+$", r.Form.Get("engname")); !m {
	return false
}
```

### 电子邮件地址

你想知道用户输入的一个Email地址是否正确,通过如下这个方式可以验证：

```
if m, _ := regexp.MatchString(`^([\w\.\_]{2,10})@(\w{1,})\.([a-z]{2,4})$`, r.Form.Get("email")); !m {
	fmt.Println("no")
}else{
	fmt.Println("yes")
}
```

### 手机号码

你想要判断用户输入的手机号码是否正确,通过正则也可以验证：

```
if m, _ := regexp.MatchString(`^(1[3|4|5|8][0-9]\d{4,8})$`, r.Form.Get("mobile")); !m {
	return false
}
```

### 下拉菜单

如果我们想要判断表单里面`<select>`元素生成的下拉菜单中是否有被选中的项目。有些时候黑客可能会伪造这个下拉菜单不存在的值发送给你,那么如何判断这个值是否是我们预设的值呢？

我们的select可能是这样的一些元素。

```
<select name="fruit">
<option value="apple">apple</option>
<option value="pear">pear</option>
<option value="banana">banana</option>
</select>
```

那么我们可以这样来验证

```
slice:=[]string{"apple","pear","banana"}

v := r.Form.Get("fruit")
for _, item := range slice {
	if item == v {
		return true
	}
}
return false
```

### 单选按钮

如果我们想要判断radio按钮是否有一个被选中了,我们页面的输出可能就是一个男、女性别的选择,但是也可能一个15岁大的无聊小孩,一手拿着http协议的书,
另一只手通过telnet客户端向你的程序在发送请求呢,你设定的性别男值是1,女是2,他给你发送一个3,你的程序会出现异常吗？
因此我们也需要像下拉菜单的判断方式类似,判断我们获取的值是我们预设的值,而不是额外的值。
```
<input type="radio" name="gender" value="1">男
<input type="radio" name="gender" value="2">女
```

那我们也可以类似下拉菜单的做法一样。

```
slice:=[]string{"1","2"}

for _, v := range slice {
	if v == r.Form.Get("gender") {
		return true
	}
}
return false
```

### 复选框

有一项选择兴趣的复选框,你想确定用户选中的和你提供给用户选择的是同一个类型的数据。

```
<input type="checkbox" name="interest" value="football">足球
<input type="checkbox" name="interest" value="basketball">篮球
<input type="checkbox" name="interest" value="tennis">网球
```

对于复选框我们的验证和单选有点不一样,因为接收到的数据是一个slice

```
slice:=[]string{"football","basketball","tennis"}
a:=Slice_diff(r.Form["interest"],slice)
if a == nil{
	return true
}
return false
```

上面这个函数`Slice_diff`开源的一个库[https://github.com/astaxie/beeku](https://github.com/astaxie/beeku)里面(操作slice和map的库))。

### 日期和时间

你想确定用户填写的日期或时间是否有效。例如,用户在日程表中安排8月份的第45天开会,或者提供未来的某个时间作为生日。

Go里面提供了一个time的处理包,我们可以把用户的输入年月日转化成相应的时间,然后进行逻辑判断。

```
t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
fmt.Printf("Go launched at %s\n", t.Local())
```

获取time之后我们就可以进行很多时间函数的操作。具体的判断就根据自己的需求调整。

### 身份证号码

如果我们想验证表单输入的是否是身份证,通过正则也可以方便的验证,但是身份证有15位和18位,我们两个都需要验证

```
// 验证15位身份证,15位的是全部数字
if m, _ := regexp.MatchString(`^(\d{15})$`, r.Form.Get("usercard")); !m {
	return false
}

// 验证18位身份证,18位前17位为数字,最后一位是校验位,可能为数字或字符X。
if m, _ := regexp.MatchString(`^(\d{17})([0-9]|X)$`, r.Form.Get("usercard")); !m {
	return false
}
```

上面列出了我们一些常用的服务器端的表单元素验证,希望通过这个引导入门,能够让你对Go的数据验证有所了解,特别是Go里面的正则处理。


## 预防跨站脚本

现在的网站包含大量的动态内容以提高用户体验,比过去要复杂得多。所谓动态内容,就是根据用户环境和需要,Web应用程序能够输出相应的内容。
动态站点会受到一种名为“跨站脚本攻击”（Cross Site Scripting, 安全专家们通常将其缩写成 XSS）的威胁,而静态站点则完全不受其影响。

攻击者通常会在有漏洞的程序中插入JavaScript、VBScript、 ActiveX或Flash以欺骗用户。一旦得手,他们可以盗取用户帐户信息,修改用户设置,盗取/污染cookie和植入恶意广告等。

对XSS最佳的防护应该结合以下两种方法：一是验证所有输入数据,有效检测攻击(这个我们前面小节已经有过介绍);另一个是对所有输出数据进行适当的处理,以防止任何已成功注入的脚本在浏览器端运行。

那么Go里面是怎么做这个有效防护的呢？Go的html/template里面带有下面几个函数可以帮你转义:

* func HTMLEscape(w io.Writer, b []byte)  //把b进行转义之后写到w
* func HTMLEscapeString(s string) string  //转义s之后返回结果字符串
* func HTMLEscaper(args ...interface{}) string //支持多个参数一起转义,返回结果字符串


我们看之前的例子：

```
fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) // 输出到服务器端
fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
template.HTMLEscape(w, []byte(r.Form.Get("username"))) // 输出到客户端
```

如果我们输入的username是`<script>alert()</script>`,那么我们可以在浏览器上面看到输出如下所示：

![Javascript过滤之后的输出](images/form_escape.png?raw=true)

Go的html/template包默认帮你过滤了html标签,但是有时候你只想要输出这个`<script>alert()</script>`看起来正常的信息,该怎么处理？请使用text/template。请看下面的例子:

```
import "text/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")
```

输出

```
Hello, <script>alert('you have been pwned')</script>!
```

或者使用template.HTML类型


```
import "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", template.HTML("<script>alert('you have been pwned')</script>"))
```

输出

```
Hello, <script>alert('you have been pwned')</script>!
```

转换成`template.HTML`后,变量的内容也不会被转义。

转义的例子：

```
import "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")
```

转义之后的输出：

```
Hello, &lt;script&gt;alert(&#39;you have been pwned&#39;)&lt;/script&gt;!
```


## 防止多次递交表单

不知道你是否曾经看到过一个论坛或者博客,在一个帖子或者文章后面出现多条重复的记录,这些大多数是因为用户重复递交了留言的表单引起的。
由于种种原因,用户经常会重复递交表单。通常这只是鼠标的误操作,如双击了递交按钮,也可能是为了编辑或者再次核对填写过的信息,点击了浏览器的后退按钮,
然后又再次点击了递交按钮而不是浏览器的前进按钮。当然,也可能是故意的——比如,在某项在线调查或者博彩活动中重复投票。那我们如何有效的防止用户多次递交相同的表单呢？

解决方案是在表单中添加一个带有唯一值的隐藏字段。在验证表单时,先检查带有该唯一值的表单是否已经递交过了。如果是,拒绝再次递交；如果不是,则处理表单进行逻辑处理。
另外,如果是采用了Ajax模式递交表单的话,当表单递交后,通过javascript来禁用表单的递交按钮。

```
<input type="checkbox" name="interest" value="football">足球
<input type="checkbox" name="interest" value="basketball">篮球
<input type="checkbox" name="interest" value="tennis">网球	
用户名:<input type="text" name="username">
密码:<input type="password" name="password">
<input type="hidden" name="token" value="{{.}}">
<input type="submit" value="登陆">
```

我们在模版里面增加了一个隐藏字段`token`,这个值我们通过MD5(时间戳)来获取唯一值,然后我们把这个值存储到服务器端(session来控制,我们将在第六章讲解如何保存),以方便表单提交时比对判定。

```
func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) // 获取请求的方法
	if r.Method == "GET" {
		crutime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h, strconv.FormatInt(crutime, 10))
		token := fmt.Sprintf("%x", h.Sum(nil))

		t, _ := template.ParseFiles("login.gtpl")
		t.Execute(w, token)
	} else {
		//请求的是登陆数据,那么执行登陆的逻辑判断
		r.ParseForm()
		token := r.Form.Get("token")
		if token != "" {
			//验证token的合法性
		} else {
			//不存在token报错
		}
		fmt.Println("username length:", len(r.Form["username"][0]))
		fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) //输出到服务器端
		fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
		template.HTMLEscape(w, []byte(r.Form.Get("username"))) //输出到客户端
	}
}
```

上面的代码输出到页面的源码如下：

![增加token之后在客户端输出的源码信息](images/form_token.png?raw=true) 

我们看到token已经有输出值,你可以不断的刷新,可以看到这个值在不断的变化。这样就保证了每次显示form表单的时候都是唯一的,用户递交的表单保持了唯一性。

我们的解决方案可以防止非恶意的攻击,并能使恶意用户暂时不知所措,然后,它却不能排除所有的欺骗性的动机,对此类情况还需要更复杂的工作。


## 处理文件上传

你想处理一个由用户上传的文件,比如你正在建设一个类似Instagram的网站,你需要存储用户拍摄的照片。这种需求该如何实现呢？

要使表单能够上传文件,首先第一步就是要添加form的`enctype`属性,`enctype`属性有如下三种情况:

```
application/x-www-form-urlencoded   表示在发送前编码所有字符(默认)
multipart/form-data	                不对字符编码。在使用包含文件上传控件的表单时,必须使用该值。
text/plain	                        空格转换为 "+" 加号,但不对特殊字符编码。
```

所以,创建新的表单html文件, 命名为upload.gtpl, html代码应该类似于:

```
<html>
<head>
	<title>上传文件</title>
</head>
<body>
<form enctype="multipart/form-data" action="/upload" method="post">
  <input type="file" name="uploadfile" />
  <input type="hidden" name="token" value="{{.}}"/>
  <input type="submit" value="upload" />
</form>
</body>
</html>
```

在服务器端,我们增加一个handlerFunc:

```
http.HandleFunc("/upload", upload)
// 处理/upload 逻辑
func upload(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) // 获取请求的方法
	if r.Method == "GET" {
		crutime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h, strconv.FormatInt(crutime, 10))
		token := fmt.Sprintf("%x", h.Sum(nil))

		t, _ := template.ParseFiles("upload.gtpl")
		t.Execute(w, token)
	} else {
		r.ParseMultipartForm(32 << 20)
		file, handler, err := r.FormFile("uploadfile")
		if err != nil {
			fmt.Println(err)
			return
		}
		defer file.Close()
		fmt.Fprintf(w, "%v", handler.Header)
		f, err := os.OpenFile("./test/"+handler.Filename, os.O_WRONLY|os.O_CREATE, 0666)  // 此处假设当前目录下已存在test目录
		if err != nil {
			fmt.Println(err)
			return
		}
		defer f.Close()
		io.Copy(f, file)
	}
}
```

通过上面的代码可以看到,处理文件上传我们需要调用`r.ParseMultipartForm`,里面的参数表示`maxMemory`,
调用`ParseMultipartForm`之后,上传的文件存储在`maxMemory`大小的内存里面,如果文件大小超过了`maxMemory`,那么剩下的部分将存储在系统的临时文件中。
我们可以通过`r.FormFile`获取上面的文件句柄,然后实例中使用了`io.Copy`来存储文件。

> 获取其他非文件字段信息的时候就不需要调用`r.ParseForm`,因为在需要的时候Go自动会去调用。而且`ParseMultipartForm`调用一次之后,后面再次调用不会再有效果。

通过上面的实例我们可以看到我们上传文件主要三步处理：

1. 表单中增加enctype="multipart/form-data"
2. 服务端调用`r.ParseMultipartForm`,把上传的文件存储在内存和临时文件中。
3. 使用`r.FormFile`获取文件句柄,然后对文件进行存储等处理。

文件handler是multipart.FileHeader,里面存储了如下结构信息

```
type FileHeader struct {
	Filename string
	Header   textproto.MIMEHeader
	// contains filtered or unexported fields
}
```

我们通过上面的实例代码打印出来上传文件的信息如下:

![打印文件上传后服务器端接受的信息](images/form_upload.png?raw=true)

### 客户端上传文件

我们上面的例子演示了如何通过表单上传文件,然后在服务器端处理文件,其实Go支持模拟客户端表单功能支持文件上传,详细用法请看如下示例：

```
package main

import (
	"bytes"
	"fmt"
	"io"
	"io/ioutil"
	"mime/multipart"
	"net/http"
	"os"
)

func postFile(filename string, targetUrl string) error {
	bodyBuf := &bytes.Buffer{}
	bodyWriter := multipart.NewWriter(bodyBuf)

	//关键的一步操作
	fileWriter, err := bodyWriter.CreateFormFile("uploadfile", filename)
	if err != nil {
		fmt.Println("error writing to buffer")
		return err
	}

	// 打开文件句柄操作
	fh, err := os.Open(filename)
	if err != nil {
		fmt.Println("error opening file")
		return err
	}
	defer fh.Close()
	
	//iocopy
	_, err = io.Copy(fileWriter, fh)
	if err != nil {
		return err
	}

	contentType := bodyWriter.FormDataContentType()
	bodyWriter.Close()

	resp, err := http.Post(targetUrl, contentType, bodyBuf)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	resp_body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return err
	}
	fmt.Println(resp.Status)
	fmt.Println(string(resp_body))
	return nil
}

// sample usage
func main() {
	target_url := "http://localhost:9090/upload"
	filename := "./astaxie.pdf"
	postFile(filename, target_url)
}
```

上面的例子详细展示了客户端如何向服务器上传一个文件的例子,客户端通过multipart.Write把文件的文本流写入一个缓存中,然后调用http的Post方法把缓存传到服务器。

> 如果你还有其他普通字段例如username之类的需要同时写入,那么可以调用multipart的WriteField方法写很多其他类似的字段。