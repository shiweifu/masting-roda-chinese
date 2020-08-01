### 介绍

2011年，我成为了一名 Ruby on Rails 开发者。Rails 是一个伟大的框架。它也是一个庞大而应用广泛的框架。它尽量允许我们开箱即用尽可能多的功能（而无论我们是否用得到），这些功能甚至不用配置或者只需要极少的配置。

在使用 Rails 一段时间后，我决定转向更小型的框架，以获得更简约的使用方式。在尝试众多 Gem 之后，我选定了`Roda`。它由 Jeremy Evans 创建，我十分喜爱它，我想通过分享有关它的有关知识和使用方法，来表达我对它的喜爱。

### 关于 Roda

Roda 是一个树状路由 Web 工具。Roda 的设计哲学包括简约，可读性扩展性以及注重性能，它的默认配置只会开启一些最常用的基本特性。Roda 通过扩展的方式实现了大量特性。这些特性均可以通过插件进行开启。

可以把 Roda 的每个特性（插件），想象成一个工具，不同类型的 Web 应用依赖不同的工具。Roda 让我们自己选择我们打算用来构建 Web 应用的工具。Roda 常常被拿来与其他 Web 框架比较，事实上，比起 Web 框架，Roda 更像一个提供不同工具的库。

Roda 的路由实现，被开发者成为`树状路由`。如我们在上面提到的，树状路由为 Roda 提供了可扩展性和可用性。树状路由将路由配置与请求的响应集成在了一起，所以当路由被捕获的时候，我们也处理了它，这是树状路由的主要特点。利用这一特性，我们可以消除许多 Web 框架中，将路由与请求分开的固有代码。

Roda 是一个轻量级的库。默认开启的基础特性实现还不到 800 行代码。同时，Roda 提供了将近 100 个插件来实现大部分 Web 应用程序的需求。

Roda 设计之初就考虑了性能，被广泛认为是最快的 Ruby Web 框架。Roda 的某些性能优化代码难以理解，但 Roda 大多数代码是容易理解的。使用 Roda 构建的应用程序也容易理解，因为它可以清晰的追踪处理逻辑，并准确查看请求路由的处理方式。

Roda 特别注意应用程序范围内的污染，尽量避免在应用内部使用实例变量，常量或方法。这有助于避免因为名称冲突而产生的异常错误。Roda 内部的实例变量，均以下划线为前缀，常量使用 Roda 为前缀。而在应用范围内定义的方法很少。

在本书中，我们将介绍 Roda 基础概念和所提供的工具。这些理念和好的实践将帮助我们入门这个让人惊喜的库。

### 如何阅读这本书？

本书是由案例驱动的。每当一个概念介绍完毕后，都会抛出一个需要解决的问题或情况，来进行描述。

本书对略读是友好的。每个例子您都可以在家自己练习，我也建议您这样做。通过执行示例代码，可以很快弄明白书中的概念。

我还建议您进一步尝试这些代码。如果有些内容让你感到困惑，在联系我们之前，可以尝试修改代码来弄清到底缺的使哪些。如果您发现有些内容需要修改，请(提交一个 Merge Request)[https://gitlab.com/fiachetti/mastering-roda/merge_requests]。


### 快速介绍 lucid_http

在内容开始涉及 Roda 之前，我想先介绍 `lucid_http`，这是我为了展示 HTTP 的交互而创建的一个 gem。在本书其余的部分，将会使用该 gem 来完成发送请求和接收请求响应的示例代码。这里不详细展开。有关更详细的介绍，请参阅附录 `lucid_http gem`。

`lucid_http` 是在 `http.rb` 之上做了一层薄薄的封装。`http.rb` 提供了非常简单和一致的 API，用来执行 HTTP 请求。`lucid_http`在此之上，提供了更高层次的抽象表示。

我创建了一个小应用，用来展示这个 gem 如何工作。代码包含在 Master Roda 的仓库中，(地址在这里)[https://gitlab.com/fiachetti/mastering-roda/]，(在 appendix_lucid_http_app.ru 文件中)[https://gitlab.com/fiachetti/mastering-roda/]。在附录部分，我讲介绍如何运行起这个项目。

我们先构建一个 GET 请求，访问 `hello`。

要完成这个操作，我们需要调用 GET 方法，将要访问的地址作为一个参数传递进去

```
require "lucid_http"

GET "/hello"
# => "<h1>Hello World!<h1>"
```

这个方法将返回响应的 body，如注释中所示。

默认情况下，base URL 是 `http://localhost:9292`。注意，结尾没有 `/`，所以我们访问相对路径时，要以 `/` 作为请求开头，如 `/hello`。

通过调用 GET 方法，我们得到了响应的 body。那如果想要获取其他相关信息呢？`lucid_http` 同样提供了相应的方法：

```
require "lucid_http"

GET "/hello/you"
status                          # => 200 OK
status.to_i                     # => 200
content_type                    # => "text/html"
path                            # => "http://localhost:9292/hello/you"
```

当我们进行下一次请求时，当前请求的上下文都会被清理掉，新请求会有一个干净的执行环境。

```
require "lucid_http"

GET "/hello/you"
status                          # => 200 OK
content_type                    # => "text/html"
path                            # => "http://localhost:9292/hello/you"
body[/\>(.+)\</, 1]             # => "Hello, You!"

GET "/403"
status                          # => 403 Forbidden
content_type                    # => "text/html"
path                            # => "http://localhost:9292/403"
body                            # => "The request returned a 403 status."
```

我们可以通过传递 `follow: true` 参数，来使请求响应重定向。

```
require "lucid_http"

GET "/redirect_me"
status                          # => 302 Found

GET "/redirect_me", follow: true
status                          # => 200 OK
body                            # => "You have arrived here due to a redirection."
```

如果我们得到错误，我们可以通过调用 `error` 来查看错误信息。这个方法会把 body 中的第一行作为错误描述信息。

```
require "lucid_http"

GET "/500"
status                          # => 500 Internal Server Error
error                           # => "SocketError: SocketError"
```

当然，如果请求的 `code` 不是 `500`，那么 `lucid_http` 也不会胡乱返回错误信息。

```
require "lucid_http"

GET "/not_500"
status                          # => 200 OK
error                           # => "No 500 error found."
```

默认情况下，`lucid_http` 不会对返回 `JSON` 格式的端点做特殊处理，所以当返回 `JSON` 数据时，显示的不是很友好。

```
require "lucid_http"

GET "/hello_world"
# => "You said: hello_world"

GET "/hello_world.json"
# => "{\"content\":\"You said: hello_world\",\"keyword\":\"hello_world\",\"timestamp\":\"2016-12-31 15:00:42 -0300\",\"method\":\"GET\",\"status\":200}"
```

我们可以通过传递 `json: true` 参数，来美化 `JSON` 的显示：

```
require "lucid_http"

GET "/hello_world"
# => "You said: hello_world"

GET "/hello_world.json", json: true
# => {"content"=>"You said: hello_world",
#     "keyword"=>"hello_world",
#     "timestamp"=>"2016-12-31 15:01:06 -0300",
#     "method"=>"GET",
#     "status"=>200}
```

`lucid_http` 当然也支持多种 HTTP 请求类型，供我们使用。

```
require "lucid_http"

GET     "/verb"                  # => "<GET>"
POST    "/verb"                  # => "<POST>"
PUT     "/verb"                  # => "<PUT>"
PATCH   "/verb"                  # => "<PATCH>"
DELETE  "/verb"                  # => "<DELETE>"
OPTIONS "/verb"                  # => "<OPTIONS>"
```

最后要展示的，使通过 `:form` 参数来构建一个传递 `form` 表单的请求。

```
require "lucid_http"

POST "/params?item=book", json: true
# => {"item"=>"book"}

POST "/params", json: true, form: { item: "book", quantity: 1, price: 50.0, title: "The complete guide to doing absolutely nothing at all."  }
# => {"item"=>"book",
#     "quantity"=>"1",
#     "price"=>"50.0",
#     "title"=>"The complete guide to doing absolutely nothing at all."}
```

OK，我们已经简单介绍了一些阅读本书的前置知识，以及代码展示风格，现在可以开始了。

### Roda 核心

在本节，我们将探索 Roda 的核心类，以及 Roda 默认的一些行为和功能。我们会介绍许多 Roda 的核心特性，以及相关插件。

我们将在本节学习到 Roda 应用程序的基本结构、用户发起的请求是如何被 Roda 代码处理的，如何返回适当的内容作为请求的响应，以及处理用户会话。

#### 一个非常小的 hello world

我们将通过构建一个非常小的 Web 应用程序来看看它是如何工作的。我们首先需要创建一个 Roda 项目。

很多用户有重量级 Web 框架的使用经验，比如 Rails。这些框架往往提供命令行工具用来新建项目。Roda 更像一个库而不是框架，它并没有提供命令行工具。

所以，我们的第一步是创建一个新的空文件夹。

```
mkdir my_app
```

然后添加 `Gemfile`，来控制我们项目中用到的 Rubygems。

理所应当的，我们第一个添加的 Gem 是 `roda`。然后我们需要一个 Web 应用服务器。我们使用 `puma`，它是一个简洁的，成熟的，快速的选择。现在，我们的 `Gemfile` 看起来是这样的：

```
source "https://rubygems.org"

gem "roda"
gem "puma"
```

现在来执行 `bundle install`，来安装我们添加的gems。

```
bundle install
```

对于本书其余部分，除非另有说明，否则我们的每一个例子，都使用此配置。从现在开始，这些 Gem 将会出现在我们例子的每一个 Gemfile 中。现在，让我们开始写一些代码吧。

几乎所有的 Ruby web 应用框架，都构建在一个统一的兼容层上，即 Rack。Roda 也是 Rack 兼容的，所以我们创建 `config.ru` 文件，这是标准的 `rackup 文件`。

接着我们引入 `roda`，创建新的 class 来构建我们的应用程序。这个应用程序继承自 `Roda` 类。

Roda 围绕着 `routing tree` 来构建，通过添加路由来增加“分叉”。所以，我们创建一个 `route` 块。这个块将收到的请求作为一个参数，为了方便，这个变量一般叫 `r`。

我们第一个路由写在 `/hello`，用来检查是否可以正常的处理 HTTP 请求。如果一切正常，我们写在块中的代码将会被执行。这个块，执行完毕后，只会返回`"hello"`。

通过继承自 Roda 类，我们的 App 类隐式的成为了一个 `Rack application`。为了告诉 `Rack` 执行 HTTP 请求，我们还得告诉它运行 `App` 类。

```
require "roda"

class App < Roda
  route do |r|
    r.get "hello" do
      "hello!"
    end
  end
end

run App
```


然后再命令行中，运行 `rackup` 命令，开启一个 Web 服务器，开始接收请求。

```
rackup
```

现在，当在浏览器访问 `http://localhost:9292/hello`，我们已经可以看到消息啦。

让我们修改一下这个小应用，我们修改 block 的返回值.

```
require "roda"

class App < Roda
  route do |r|
    r.get "hello" do
      "Hello, world!"
    end
  end
end

run App
```

再次访问 `http://localhost:9292/hello`，我们将会发现毫无变化。为什么呢？因为服务器还在执行着旧代码。

这再次题型我们，我们不是建立在精美的 web 框架上。如果我们想要诸如代码修改，自动重新加载这样的功能，我们必须动一动手。

实现重新加载代码比较简单的方式是，使用 `rerun` gem，(RubyTapas Episode #320)[https://www.rubytapas.com/2015/06/29/episode-320-rerun/?ref=federico] 介绍了这一工具。

执行：
```
rerun rackup
```

现在，当我们再次访问 ()[http://localhost:9292/hello]，我们将会看到最新代码的输出。

本文接下来的所有例子，都假设使用 rerun 来执行 rackup，来实现代码变更后的重新加载。

我们已经有了一个可以工作的 web 应用，接下来给它增加一些交互。让我们给它增加一个回复请求的能力。我们创建第二个路由。和之前一样，我们再次匹配字符串 "hello"。不同的是，我们将后面增加 String，这表示后面任意的字符串都能被匹配到，比如 `/hello/Frank` 和 `/hello/Nancy`。

当我们使用 `String` 作为后面的匹配对象时，我们提供的 block 对象将会被调用，匹配到的字符串会作为 block 的参数。我们传递一个合适的名称，然后将这个传递的值输出。

```
require "roda"

class App < Roda
  route do |r|
    r.get "hello", String do |name|
      "<h0>Hello #{name}!</h1>"
    end
  end
end

run App
```

当我们访问 ()[http://localhost:9291/hello/Roda] 时，我们会发现字符串 Roda 被捕获到了。

到目前为止，我们一直在使用 `config.ru` 这个 rackup 文件。实际上 rackup 文件只是用于配置应用服务器的启动流程，实际的应用程序并不应包含其中。让我们来更好的组织代码，将应用的代码移出 config.ru 文件。

我们删掉了整个 App 类，以及上方的 `require` 语句，然后增加一个 `require` 语句，这次 `require` 的是 `app.rb` 文件。

```
require "./app"

run App
```

然后创建一个名为 `app.rb` 的新文件，将刚刚移除的代码放进去。

```
require "roda"

class App < Roda
  route do |r|
    r.get "hello", String do |name|
      "<h0>Hello #{name}!<h1>"
    end
  end
end
```

接着我们检查一下是否一切正常。

现在，想象我们编写一个网站，该网站介绍一位神秘访客。用户预先不知道这个神秘访客是谁，但是他们可以通过访问 `/mystery_guest` 路径来看到它。

由于超出本书范围，我们让这个神秘访客为 `mozzarella pizza`。我们使用一个 Ruby 类结构来创建 Pizza 类，然后创建一个 Pizza 的实例。接着在我们的 mystery_guest 路径，使用 Pizza 实例。为了好玩，我们将会在页面拼写错误 `Guest`。

```
Pizza = Struct.new(:flavor)

class App < Roda
  mystery_guest = Pizza.new("Mozzarella")

  route do |r|
    r.get 'mystery_guest' do
      "The Mystery Gest is: #{mystery_guest}"
    end
  end
end
```

那并没有成功。这是为什么？如果我们使用 `LucidHttp`访问这个页面，我们会看到结果返回的是正确的，是 `Pizza`！

```
require "lucid_http"

GET "/mystery_guest"
# => "#<struct Pizza flavor=\"Mozzarella\">"
```

请求已经得到正确的响应，为什么我们无法在浏览器中看到它？仔细观察，会发现 `#` 后面的字符串`<struct Pizza flavor="Mozzarella">`因为包含 HTML tag，所以无法正确显示。那么如何解决呢？一个方案是在 Pizza 类中实现 `to_s` 方法。不过这种方法有些无聊。即使对于这种解决方案，有两个主要因素，也需要特殊考虑：

- 我们所讨论的秘密客人案例，有点像一个人为构建的例子，有点脱离现实，但事实并非如此。有时，我们要处理更长的响应，并且试图生成一个变量或方法的结果，而该结果实际上返回的类，可能类似 Pizza 类。我们可以使用 `to_s`，来返回一些格式化的信息，但我们无法写出一个广泛使用的`to_s`方法，因为不知道要处理的是什么类。

- 我们实际上，是想要呈现 HTML 风格的字符串。比如<>这种，会被浏览器当作标签。这种情况下，我们需要对特殊内容转义。

`h` 插件为以上种种情况提供了解决方案。我们可以使用 `plugin :h` 来加载该插件。引入该插件后，我们可以把想要处理的字符串传给 `h` 方法。

```
Pizza = Struct.new(:flavor)

class App < Roda
  plugin :h

  mystery_guest = Pizza.new("Mozzarella")

  route do |r|
    r.get 'mystery_guest' do
      "The Mystery Gest is: #{h mystery_guest}"
    end
  end
end
```

这时，响应已经被处理。

```
require "lucid_http"

GET "/mystery_guest"
# => "The Mystery Gest is: #&lt;struct Pizza flavor=&quot;Mozzarella&quot;&gt;"
```

原始字符串可能有些奇怪，但是浏览器显示的一切正常。


