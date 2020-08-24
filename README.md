原文：https://fiachetti.gitlab.io/mastering-roda/

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

### 基本路由

从底层来看， HTTP 协议只是一些请求和请求的响应。如果我们只需要响应一种请求，我们可以用一大堆代码来处理这一种请求。

Web 应用程序往往需要支持许多不同种类的请求。为了让开发人员能不陷入混乱，我们通常会将请求进行分类。相应的，我们也会尽量避免重复的代码。

在 Web 应用中，`路由`的意思是接受客户端请求，并将其转发到响应的处理代码。本节我们来讨论 Roda 中的路由。

如我们在前面章节所见，创建 Roda app 的第一步，我们要继承自 Roda 类。

为了给我们的应用添加路由，我们进入 Roda 类的内部，并调用 `route` 类方法。我们传递一个 block，参数为当前请求封装的对象。

我们这里暂停一下，看一下 r 参数的内容。我们使用 `Kernel#p` 方法进行调试输出。我们对这行不进行缩进，这作为个标识，等下进行删除。我们在这个请求返回空字符串，避免输出错误信息（我们将在后面解释为什么需要此步骤）。

```
 require "roda"

  class App < Roda
    route do |r|
p r
      ""
    end
  end
```

此时，我们访问任何路径，都会进入相同的路由，返回相同的结果。

```
#<App::RodaRequest GET />
127.0.0.1 - - [19/Sep/2016:19:46:26 -0300] "GET / HTTP/1.1" 404 - 0.0016
```

`Roda::RodaRequest` 是 `Rack::Request` 的子类，添加了用于处理路由的方法。`App::RodaRequest` 是 `Roda::RodaRequest` 的子类，在创建 App 时自动生成，允许在插件中，自定义 App 请求实例。

`Roda::RodaRequest` 是 `Rack::Request` 的子类，增加了有关路由的方法。`App::RodaRequest`是 `Roda::RodaRequest` 的子类，它在 App 对象创建时被生成。这允许我们通过插件自定义 App 请求实例。

我们可以从 p 的输出种看到对 `/` 路径发起的 GET 请求。

我们可以在路由处理方法中，调用请求对象的方法，接下来的章节会探索这些方法。现在，让我们定义一个路由。

在这里，我们调用请求对象的 `on` 该方法，传递字符串参数和 block。在 block 中，我们返回一段字符串

```
class App < Roda
  route do |r|
    r.on "hello" do
      "Hello Lucid!"
    end
  end
end
```

`r.on` 方法，在 Roda 中，被成为匹配方法，这类方法接收参数，然后将参数与请求进行比对，看看是否匹配。如果匹配成功，匹配方法就调用传递过来的匹配block，到此，请求算是处理结束。如果与参数的比对没有成功，匹配方法就不会调用匹配 block，只会继续向下执行。

`r.on` 方法是最简单的一种匹配方法。它只会与当前请求进行比对，没有什么其他的比对逻辑。

让我们看看它都返回了什么。页面 body 是通过匹配 block 进行返回的。当 Roda 调用 match block 时，match block 返回了字符串，这个字符串就被用于响应对象的 body。我们同样可以通过响应对象的状态码是 200，内容类型为 `text/html`。

```
require "lucid_http"

GET "/hello"

body                            # => "Hello Lucid!"
status                          # => "200 OK"
content_type                    # => "text/html"
```

如果 block 返回 nil 或者 false，

```
class App < Roda
  route do |r|
    r.on "hello" do
      nil
    end
  end
end
```

这时 Roda 会认为路由没有被匹配到，然后返回空的 body，状态码为 404.

```
require "lucid_http"

GET "/hello"

body                      # => ""
status                    # => "404 Not Found"
```

如果返回一些 Roda 无法处理的内容，比如 `Integer`

```
class App < Roda
  route do |r|
    r.on "hello" do
      1
    end
  end
end
```

Rails 抛出 `Roda::RodaError` 异常，大多数 WebServer 将会认为服务端内部错误：

```
require "lucid_http"

GET "/hello"

body                      # => "..."
status                    # => "500 Internal Server Error"
```

Roda 的匹配 block，默认只支持 string, nil 和 false。可以通过插件来支持额外的返回值类型，稍后我们会讨论。


让我们回过去看之前的例子。仔细观察可以发现，生成内容的代码，实际上被包在一个 block中。

让我们增加一些调试语句在 route 块中，另一些在 hello 块中。

```
  require "roda"

  class App < Roda
    route do |r|
p "ROUTE block"

      r.on "hello" do

p "HELLO block"

        "Hello Lucid!"
      end
    end
  end
```

当我们访问应用的根节点（[](http://localhost:9292)） ，我们可以看到 `route` 块中的调试信息被打印了出来。

而当我们访问[](http://localhost:9292/hello)，两个调试信息都被打印了。

```
"ROUTE block"
127.0.0.1 - - [16/Nov/2016:12:56:22 -0300] "GET / HTTP/1.1" 404 - 0.0020
"ROUTE block"
"HELLO block"
127.0.0.1 - - [16/Nov/2016:12:58:35 -0300] "GET /hello HTTP/1.1" 200 5 0.0011
```

这个演示提现了 Roda 的一个核心概念。在许多 Web 框架中，路由的定义，是在应用程序启动时进行的，然后被编译为某些内部的数据结构。Roda 不同，每当收到请求，路由定义块都会被执行。

这是一个非常给力的想法。一方面，它让 Roda 的路由系统容易理解。我们写路由的时候，不必先在脑海中，将路由定义映射为某种内部表现形式。我们看到的，就是我们得到的：通过阅读项目代码，我们可以很清楚知道每一步路由要做什么。

很多用户会对 Roda 路由设计的性能有所疑虑。毕竟，Roda 中的每个请求，都会执行路由代码。当 Roda 的路由定义变得庞大时，是否会变慢呢？事实上，完全不用担心。Roda 中的路由执行时，会跳过没有匹配到的分支，只有匹配到的分支会被执行。事实上，在路由方面，Roda 是最快的 Ruby Web 框架，具有很强的可用性。

从 Big-O 的角度来看，Roda 的路由设计，执行效率是 O(N)。Roda的默认路由设计要求在路由树的每个分支（包括根）上进行O（N）查找，其中N是该级别上可能的分支数量。 但是，在大多数应用程序中，这会导致应用程序的O（log（N））路由，其中N是应用程​​序中路由的总数。 稍后我们会看到Roda附带了一个插件，该插件允许在路由树的每个分支上进行O（1）路由，从而实现O（N）路由性能，其中N是请求中的段数（无论 应用程序中的路由总数）。

我们将在之后章节中看到，访问请求与 Roda 的路由定义一起工作。匹配过程就像毛虫爬树一样，只有与请求匹配的路由代码，才会执行。

通过添加第二个路由，以及一些调试代码，我们将会看到这个过程。

```
  require "roda"

  class App < Roda
    route do |r|
p "ROUTE block"
      r.on "hello" do
p "HELLO block"
        "hello"
      end

      r.on "goodbye" do
p "GOODBYE block"
        "goodbye"
      end
    end
  end
```


如果我们现在访问[](http://localhost:9292/goodbye)，我们会期待 `hello` 和 `goodbye` 字符串都被输出，但是并不会，只有匹配道的路由部分会被执行。即使我们首先定义了 `hello` 路由。

```
"ROUTE block"
"GOODBYE block"
127.0.0.1 - - [16/Nov/2016:13:06:14 -0300] "GET /goodbye HTTP/1.1" 200 7 0.0017
```

事实上，这种树状风格的路由，正是 Roda 被称为“路由树 Web 工具包”的原因。

也可以把路由想象成一种惰性评估，只有被匹配道的块才会被执行。

这种树状的路由，还有另外一个优势。在大多数框架中，当要针对进来的请求进行一些初始化时，我们必须等待路由匹配完成。在 Roda 中，请求的初始化与配置，可以在路由匹配时就进行一些配置。这样就无需使用“过滤器”或“钩子”这种机制在路由之前或之后执行某些操作。

这是什么意思呢？举个例子，同时存在两个路由，`hello` 和 `goodbye`，在进入这两个路由前，我们都需要在数据库中找到某个用户。让我们模拟这种场景，并将找到的用户赋值给某个变量。接着，我们在最后的输出字符串中，使用这个变量。这种情况下，我们需要在两个路由中，编写相同的代码两次。

```
require "roda"

class App < Roda
  route do |r|
    r.on "hello" do
      name = "Lucid"
      "Hello, #{name}!"
    end

    r.on "goodbye" do
      name = "Lucid"
      "Goodbye, #{name}!"
    end
  end
end
```

如果按照这个例子来看，假设我们继续添加需要访问用户的子路由，那我们就需要继续获取这个用户，对吗？事实上是不用的。这种情况下，Ruby 代码块中的共享作用域性质提现了价值。我们可以只在路由块中获取一次这个用户，接着在每个嵌套的块中，都可以访问这个用户。

```
require "roda"

class App < Roda
  route do |r|
    name = "Lucid"

    r.on "hello" do
      "Hello, #{name}!"
    end

    r.on "goodbye" do
      "Goodbye, #{name}!"
    end
  end
end
```

现在访问 ()[http://localhost:9292/hello]，我们可以得到和我们预想一样的输出。访问 ()[http://localhost:9292/goodbye] 也会得到类似的输出。

```
require "lucid_http"

GET "/hello"                    # => "Hello, Lucid!"
GET "/goodbye"                  # => "Goodbye, Lucid!"
```

Roda 的路由的优势是将路由存储在一个代码块中，而不是某个数据结构。事实上，这是一种权衡过的设计，它限制了路由的自省能力，比如打印路由表。有一些办法可以优化自省能力，但是需要额外的设置。

### 匹配函数

#### r.on 和 r.is

在前面的章节，我们已经概览了一下 Roda 中的路由。现在我们继续看 Roda 为我们提供的不同种类的 match 函数。

我们来拿一个 Roda Blog 应用来进行说明（非常原始，对吧？）。我们想让 `/posts` 路由返回我们所有的 Posts。我们使用 `r.on` 来匹配。我们添加一个变量来存储我们的数据，该变量是我们所有帖子的哈希值。我们使用 join 方法，将数据的值连接成一个字符串，并在处理块结束时返回。

```
class App < Roda
  route do |r|
    r.on "posts" do
      post_list = {
        1 => "Post[1]",
        2 => "Post[2]",
        3 => "Post[3]",
        4 => "Post[4]",
        5 => "Post[5]",
      }

      post_list.values.join(" | ")
    end
  end
end
```

如果我们现在访问 (http://localhost:9292/posts)[http://localhost:9292/posts]，我们将看到帖子列表。我们访问 (http://localhost:9292/posts/)[http://localhost:9292/posts/] 或者 (http://localhost:9292/posts/whatever)[http://localhost:9292/posts/whatever]，我们将看到相同的列表。在真实的应用中，这并不是正确的行为，我们来修复它。

```
require "lucid_http"

GET "/posts"
body
# => "Post[1] | Post[2] | Post[3] | Post[4] | Post[5]"

GET "/posts/"
body
# => "Post[1] | Post[2] | Post[3] | Post[4] | Post[5]"

GET "/posts/whatever"
body
# => "Post[1] | Post[2] | Post[3] | Post[4] | Post[5]"
```

`r.on` 会将所有的用于匹配的块与请求（本例中的"posts"）进行比对。由于这三个请求都以 `/posts` 开头，因此匹配成功。

让我们来看一下路由在更复杂的情况下如何工作，假设我们希望 (http://localhost:9292/posts/1)[http://localhost:9292/posts/1] 返回 id 为 1 的帖子。要完成这一点，我们需要在 `r.on` "posts" 块中，嵌套调用 `r.is`。`r.is` 是一个类似 `r.on` 的匹配方法，区别在于除了 r.on 完成的匹配外，还要求处理匹配器后消耗剩余的路径，才算匹配成功（我们将在稍后解释消耗的含义）。

换言之，`r.on` 是一个非终端匹配方法，而 `r.is` 是终端匹配方法。非终端匹配方法，不需要完全消耗剩余路径即可匹配成功。终端匹配方法，需要完全消耗剩余路径，才算是匹配成功。

我们接下来传递 `Integer` 匹配器给 `r.is` 匹配方法。`Integer` 匹配器的操作类似 `String` 匹配器，但是 String 匹配器可以匹配任意字符串，而 `Integer` 只能匹配（0-9）。匹配成功后， Roda 将调用匹配块。Integer 类型的匹配器，将会把参数转换为整数。

```
class App < Roda
  route do |r|
    r.on "posts" do
      post_list = {
        1 => "Post[1]",
        2 => "Post[2]",
        3 => "Post[3]",
        4 => "Post[4]",
        5 => "Post[5]",
      }

      r.is Integer do |id|
        post_list[id]
      end

      post_list.values.map { |post| post }.join(" | ")
    end
  end
end
```

再次尝试，我们将看到我们请求的帖子内容。如果我们请求的帖子不存在，post_list[id] 将会是 nil，返回 404 响应。这是一种理想行为，请求不存在的帖子将会返回不存在的响应。

```
require "lucid_http"

GET "/posts/1"
body                            # => "Post[1]"
status                          # => "200 OK"

GET "/posts/5"
body                            # => "Post[5]"
status                          # => "200 OK"

GET "/posts/6"
body                            # => ""
status                          # => "404 Not Found"
```

添加 `r.is` 调用，不会影响其他路由。`/posts`，`/posts/` 和 `/posts/whatever` 的表现将不变。但是通常会对资源访问的地址做一些限制。我们将让 `/posts` 返回所有的日志，但 `/posts/` 和 `/posts/whatever` 返回 404 响应。我们可以通过将所有帖子的返回包装在 `r.is` 块中，来进行更改。

```
class App < Roda
  route do |r|
    r.on "posts" do
      post_list = {
        1 => "Post[1]",
        2 => "Post[2]",
        3 => "Post[3]",
        4 => "Post[4]",
        5 => "Post[5]",
      }

      r.is Integer do |id|
        post_list[id]
      end

      r.is do
        post_list.values.map { |post| post }.join(" | ")
      end
    end
  end
end
```


接下来，我们可以检查是否达到预期的效果。

```
require "lucid_http"

GET "/posts"
body
# => "Post[1] | Post[2] | Post[3] | Post[4] | Post[5]"

GET "/posts/"
body                            # => ""
status                          # => "404 Not Found"

GET "/posts/whatever"
body                            # => ""
status                          # => "404 Not Found"
```

回到我们的例子，这看起来有点奇怪，因为 `r.is` 后面没有跟匹配器。随着我们使用 Roda，这将变得自然而然。我们可以将匹配器视为过滤器。Roda 的默认行为是将 match 方法进行匹配，除非存在匹配器或其他原因导致匹配不成功。如果没有匹配器，则 `r.on` 一直成功；`r.is` 只在剩余路径已经被消耗完毕时才匹配。

现在，我们来讨论请求路径的消耗是什么意思。当匹配器对路径进行匹配时，他们将消耗匹配地址的路径部分。Roda 在路由匹配期间，不会修改请求的路径（r.path 和 r.path_info 保持不变）。它存储路径中有尚未路由的部分，被成为匹配路径（可在 r.matched_path 中找到）。匹配器成功匹配后，他们会消耗剩余路径的一部分。剩余路径在路由过程中变小，如果为空，则剩余路径已被完全消耗。

通过一个`剩余路径`和`匹配路径`的例子，很容易说明清楚它是如何工作的。让我们在不同的树状路由块中，输出`剩余路径`和`匹配路径`。

```
 class App < Roda
    route do |r|
p [0, r.matched_path, r.remaining_path]
      r.on "posts" do
p [1, r.matched_path, r.remaining_path]
        post_list = {
          1 => "Post[1]",
          2 => "Post[2]",
          3 => "Post[3]",
          4 => "Post[4]",
          5 => "Post[5]",
        }

        r.is Integer do |id|
p [2, r.matched_path, r.remaining_path]
          post_list[id]
        end

        r.is do
p [3, r.matched_path, r.remaining_path]
          post_list.values.map { |post| post }.join(" | ")
        end
      end
    end
  end
```

下面是 `/`，`/posts` 和 `/posts/1` 的请求结果，显示了匹配器如何使用剩余的路径以及每个步骤中的路径是什么。

```
[0, "", "/"]
127.0.0.1 - - [19/Sep/2016:19:46:26 -0300] "GET / HTTP/1.1" 404 - 0.0016
[0, "", "/posts"]
[1, "/posts", ""]
[3, "/posts", ""]
127.0.0.1 - - [19/Sep/2016:19:46:36 -0300] "GET /posts HTTP/1.1" 200 47 0.0016
[0, "", "/posts/1"]
[1, "/posts", "/1"]
[2, "/posts/1", ""]
127.0.0.1 - - [19/Sep/2016:19:46:46 -0300] "GET /posts/1 HTTP/1.1" 200 7 0.0016
```

在刚才的大多数例子，使用单个匹配器，但其实匹配方法可以接受多个匹配参数。我们可以方便的接受年、月、日为参数，然后用于创建 `Date` 对象和传递给 Post.posts_for_date 方法。

```
# ...

r.on "posts", "date", Integer, Integer, Integer do |year, month, day|
  date = Date.new(year, month, day)
  posts = Post.posts_for_date(date)

  # ...
end

#...
```

这里 `捕获` 的意思是与 `消费` 有关。当匹配器消耗一节路径时，这段路径可能被捕获并执行响应的匹配块代码。在上面的例子中，"posts"，"date" 和 Integer 是整体匹配器，余下的路径都会被匹配到。但是，"posts" 和 "date" 匹配器不会捕获他们所使用的那节路径，而整数捕获器会捕获。由于有三个整数匹配器，因此块有三个参数。

让我们展开讨论一下之前的例子，聚焦到 `/posts/1/show` 和 `/posts/1/show/detail` 路由。他们都用于显示日志，但是 `detail` 路由返回的内容包含最后访问的信息。因为我们需要处理 `/posts/1` 下的路由，所以我们不能使用 `r.is Integer`，我们需要换用 `r.on Integer`，因为此时路由树不再需要终端匹配。

在 `r.on Integer` 匹配块中，我们找到日志，并存储为一个变量。然后我们调用 `r.on "show"`，并在其中使用两个 `r.is`，第一个不带有参数，表示匹配 `show` 本身，第二个带上 `"/detail"`，用于匹配剩余的路径。

```
require "roda"

class App < Roda
  route do |r|
    r.on "posts" do
      # ...
      r.on Integer do |id|
        post = post_list[id]

        r.on "show"  do
          r.is do
            "Showing #{post}"
          end

          r.is "detail" do
            "Showing #{post} | Last access: #{Time.now.strftime("%H:%M:%S")}"
          end
        end
      end
      # ...
    end
  end
end
```

简而言之，`r.on` 被用于路由树树枝的路径处理，该分支中有多个要处理的路径。`r.is` 用于处理路由树种的叶子路径，其余的路径已经被路由树匹配完了。

`r.on` 和 `r.is` 可能是 Roda 种最通用的路由匹配方法。然而，一个 HTTP 请求不仅是对路径的请求，还是对使用特定请求方法的路径的请求，响应取决于所使用的请求方法。在下一节中，我们讲讨论如何处理请求方法。


#### r.get 和 r.post

现在，我们已经知道了如何为不同路径编写路由，现在，我们来讨论如何处理各种请求方法。对于浏览器，我们只需要考虑两种请求方法：`GET` 和 `POST`。通常，`GET` 用于导航到某个页面，一般用于幂等的请求。而 `POST` 用于可能修改状态的表单。

一般来说，成功处理请求，需要完整的请求路径（消耗整个剩余路径），以及特定于请求的方法。默认情况下，Roda 包含两种用于处理特定请求的匹配方法，`r.get` 用于处理 GET 请求，`r.post` 用于处理 POST 请求。

除了匹配的方法不同，`r.get` 和 `r.post` 具有相同的行为。如果没有传递任何匹配器，则 `r.get` 和 `r.post` 均作为非终端匹配方法运行。如果将任何参数传递给它们，则 `r.get` 和 `r.post` 都将作为终端匹配方法。为什么通过传递参数来决定行为的差异？好吧，虽然 Roda 的行为看起来不一致，但却完全符合我们的要求。

通常，使用 `r.is` 完全匹配路由路径之后，不再使用任何参数检查请求方法，这种情况下，不需要对终端匹配进行重复检查。

```
require "roda"

class App < Roda
  route do |r|
    r.on "posts" do
      r.is Integer do |id|
        r.get do
          # Handle GET /posts/$ID
        end

        r.post do
          # Handle POST /posts/$ID
        end
      end
    end
  end
end
```

另一种情况，如果路由需要在检查参数之前检查请求方法，我们就不能使用终端式请求。

```
require "roda"

class App < Roda
  route do |r|
    r.get do
      r.on "posts" do
        r.is Integer do |id|
          # Handle GET /posts/$ID
        end
      end
    end

    r.post do
      r.on "posts" do
        r.is Integer do |id|
          # Handle POST /posts/$ID
        end
      end
    end
  end
end
```

#### 提示

首先通过检查 HTTP 请求的路径，再检查方法，可以减少编写重复代码。在实际的应用中，首先检查请求方法的情况很少。这种情况往往出现在绝大多数路由都使用一种请求方法，而很少的路由使用另一种请求方法。

因此，当我们传递 `r.get` 和 `r.post` 匹配器的时候，通常表明剩余的路由路径仅有特定的请求方法处理。

```
require "roda"

class App < Roda
  route do |r|
    r.on "posts" do
      r.is Integer do |id|
        r.get "show" do
          # Handle GET /posts/$ID/show
        end

        r.post "update" do
          # Handle POST /posts/$ID/update
        end
      end
    end
  end
end
```

如果 `r.get` 和 `r.post` 在通过匹配器猴，没有进行终端匹配，我们需要 `r.is` 进行包装。

```
require "roda"

class App < Roda
  route do |r|
    r.on "posts" do
      r.is Integer do |id|
        r.get "show" do
          r.is do
            # Handle GET /posts/$ID/show
          end
        end

        r.post "update" do
          r.is do
            # Handle POST /posts/$ID/update
          end
        end
      end
    end
  end
end
```

如上所示，这只会导致编写冗余的代码。通过使 `r.get` 和 `r.post`（如果通过任何匹配器）用作终端匹配的方法，Roda 通过牺牲了一些一致性，提高了可用性。

如果我们想让 `r.get` 或 `r.post` 使用终端匹配呢？我们需要传递一个始终可以匹配成功的匹配器。

```
require "roda"

class App < Roda
  route do |r|
    r.on "posts" do
      r.on Integer do |id|
        r.get true do
          # Handle GET /posts/$ID
        end

        r.is "manage" do
          r.get do
            # Handle GET /posts/$ID/manage
          end

          r.post do
            # Handle POST /posts/$ID/manage
          end
        end
      end
    end
  end
end
```

到目前位置，我们都在讨论 `GET` 和 `POST` 请求。那么其他请求方法呢？事实上，Roda 为了保持精简，默认只支持浏览器直接可以响应的方法。因此，Roda 提供了 `all_verbs` 插件添加了额外的请求类型，这样 `r.head`，`r.put`，`r.patch` 和 `r.delete` 这些 HTTP 请求方法都可以被支持了。这些方法的行为与 `r.get` 和 `r.post` 一致， 他们用于过滤不同情况所需要的请求。

```
require "roda"

class App < Roda
  plugin :all_verbs

  route do |r|
    r.on "posts" do
      r.is Integer do |id|
        r.head do
          # Handle HEAD /posts/$ID
        end

        r.get do
          # Handle GET /posts/$ID
        end

        r.post do
          # Handle POST /posts/$ID
        end

        r.put do
          # Handle PUT /posts/$ID
        end

        r.patch do
          # Handle PATCH /posts/$ID
        end

        r.delete do
          # Handle DELETE /posts/$ID
        end
      end
    end
  end
end
```

另外，如果我们想将 `HEAD` 请求与 `GET` 请求一样对待，除了忽略响应内容，我们还可以使用 `head` 插件。

```
require "roda"

class App < Roda
  plugin :head

  route do |r|
    r.on "posts" do
      r.is Integer do |id|
        r.get do
          # Handle HEAD /posts/$ID (response body will be empty)
          # Handle GET /posts/$ID
        end

        r.post do
          # Handle POST /posts/$ID
        end
      end
    end
  end
end
```

托管公共网站时，建议使用 `head` 插件，除非应用程序需要单独处理 `HEAD` 请求。否则，使用 `HEAD` 的网络爬虫可能会认为相关页面不再存在，因为对 HEAD 的请求将导致 404 请求响应。

#### r.root

到目前为止，我们已经看到了如何处理多个路径的请求，和处理请求的方法。那么，如何处理网站的 `root` 路径呢？



![img](/assets/root_redirect_lucidcode_root_site.png)



如果我们看过之前的章节，我们会发现可以使用 `r.get` 来实现这种定义，后面跟一个空的匹配器（匹配 GET / 请求）。



现在，我们返回一个无意义的字符串。



```ruby
class App < Roda
  route do |r|
    r.get "" do
      "Root Path"
    end

    r.get "posts" do
      posts = (0..5).map {|i| "Post #{i+1}"}
      posts.join(" | ")
    end
  end
end
```



当请求这个页面的时候，我们得到一个字符串。



```ruby
require "lucid_http"

GET "/"
path                            # => "http://localhost:9292/"
body                            # => "Root Path"
```



使用这种方式，我们需要为每个 App 编写类似的路由，它的语法也不怎么漂亮。幸运的是，Roda 为这个特定的路由提供一个便利的匹配方法，称为 `r.root`。



我们修改这个例子，来使用它：



```ruby
class App < Roda
  route do |r|
    r.root do
      "Root Path"
    end

    # ...
  end
end
```



如果我们再次发起相同的请求，`r.root`将可以匹配到它，并返回相同的内容。



```ruby
require "lucid_http"

GET "/"
path                            # => "http://localhost:9292/"
body                            # => "Root Path"
```



需要注意的是，`r.root` 等同于 `r.get ""`，而不是 `r.is ""`。其被设计为只匹配 `GET` 类型的请求。如果我们想要处理其他类型的请求， 我们需要使用 `r.is ""`。`r.root` 的优势是，它在处理 `GET` 请求时，它更清晰。



假设我们现在想要使用 `/posts/:id` 路径来获取日志。我们需要添加一个路由。需要注意的是，我们修改 `posts` 匹配器，从 `r.get` 改为 `r.on`，因为现在我们不再需要终端处理了。在 `r.on "posts"`，我们有两个路由，一个返回所有日志（`GET /posts/`，注意结尾的`/`），另外一个返回指定的日志（比如 `GET /posts/1`）。



```ruby
require "roda"

class App < Roda
  route do |r|
    # ...
    r.on "posts" do
      posts = (0..5).map {|i| "Post #{i}"}

      r.get "" do
        posts.join(" | ")
      end

      r.get Integer do |id|
        posts[id]
      end
    end
  end
end
```



检查一下，看看是否正常工作。

```ruby
require "lucid_http"

GET "/posts/"
# => "Post 1 | Post 2 | Post 3 | Post 4 | Post 5"

GET "/posts/1"
# => "Post 1"
```



根据前面的介绍，使用 `r.root` 和 传递一个空字符串，给 `r.get` ，匹配方法，结果是一样的。我们修改成 `r.root`。



```
r.on "posts" do
  posts = (0..5).map {|i| "Post #{i}"}

  r.root do
    posts.join(" | ")
  end
end
```



看看能否正常工作



```
require "lucid_http"

GET "/posts/"
# => "Post 1 | Post 2 | Post 3 | Post 4 | Post 5"
```



可以看到，代码可以正常工作，即使我们不在应用程序的根目录，它也可以正常工作。这是为什么呢？这与Roda 中的匹配器工作方式有关。事实证明，Roda 附带的所有 match 方法都在剩余路径上进行匹配，因此匹配时将不考虑路径已消耗的任何部分。



`r.root` 方法在这里有效，但是在子路由中使用是否有效？这是个趣味问题，单通常只有在我们要支持带有斜杠的路由并且仅处理这些路由的 `GET` 请求时才有意义。对于根路由，强制使用尾部斜杠，并且`GET` 通常是唯一使用的请求方法，单是对于所有的其他路由，我们不得不使用使用尾部斜杠的应用程序路径结构设计。通常，最好避免这种情况。因此，我们将使用 `/posts` 和 `/posts/1` 代替 `/POSTS/` 和 `/posts/1`。这种情况下，我们将不再使用 `r.root`。相反，我们使用 `r.get true`。





### 自定义匹配方法



在前面的章节，我们我们已经看到了 Roda 内置的全部类型的匹配器和匹配方法。本节中，我们尝试编写一个简单的自定义类型的匹配方法。



如你所见，内置的匹配器和匹配方法主要是处理不同类型的方法，和请求的路径。这也是用于匹配的主要比对部分。下面的例子中，我们匹配请求的其他部分，或者与请求自身本身无关的内容。（比如时间和日期）。



假设我们想写一个匹配方法，接受一组哈希参数，进行比对，只在键和值正确的情况下才匹配成功。



```
require 'roda'

class App < Roda
  route do |r|
    r.with_params "secret"=>"Um9kYQ==\n" do
    end
  end
end
```



这段代码还不能正确工作，因为`r.with_params`比对方法还不存在。我们要在哪里定义 `r.with_params` 方法呢？在前面的章节，我们学习到，r 变量，是 `App::RodaRequest` 的实例，所以我们可以在类中直接添加。然而，我们目前可能不知道如何编写 `with_params` 方法。



```
require 'roda'

class App
  class RodaRequest
    def with_params(hash, &block)
      #
    end
  end

  route do |r|
    r.with_params "secret"=>"Um9kYQ==\n" do
    end
  end
end
```



首先，我们先弄明白如何编写这个方法。我们传递进的参数，是一个哈希而不是匹配器，所以我们不想将它传递给另一个需要匹配器的方法。最好检查哈希是否与预期参数匹配。如果不符合预期，我们就不需要做任何事情。如果成功匹配到了，我们要将其视为匹配快。



通过调用 params 方法，我们知道了提交参数。这个方法实际上来自于 Rack::Request。因此检查参数是否匹配的简单办法是如果迭代提供的哈希，然后从方法返回。



```
def with_params(hash, &block)
  hash.each do |key, value|
    return unless params[key] == value
  end

  #
end
```



这只是会处理匹配失败的情况。我们该如何成功个匹配？这种情况下，如果在 `hash.each` 调用之后，我们仍在执行该方法，则我们要将给定的块视为匹配块。这是可以通过不带任何参数的块传递给 on 来处理的。（请记住on是r.on，因为r是App :: RodaRequest的实例）



```
def with_params(hash, &block)
  hash.each do |key, value|
    return unless params[key] == value
  end

  on(&block)
end
```



on 将该块视为匹配快，将控制权传递给它，并且在该块执行后，将返回响应。



#### 匹配块条件控制



让我们使用一个类似之前的例子，来讲解。



```
require "roda"

class App < Roda
  route do |r|
    # ...
    r.on "posts" do
      posts = (0..5).map {|i| "Post #{i}"}

      r.get true do
        posts.join(" | ")
      end

      r.get Integer do |id|
        posts[id]
      end
    end
  end
end
```



我们做了一个小的修改。让 `/posts/:id` 返回一个字符串，包含日志名称和访问时间。

```
class App < Roda
  route do |r|
    # ...
    r.on "posts" do
      # ...

      r.get Integer do |id|
        post        = posts[id]
        access_time = Time.now.strftime("%H:%M")

        "Post: #{post} | Accessing at #{access_time}"
      end
    end
  end
end
```



现在，当 id 为 2 的日志，将正确获取这些信息。



```
require "lucid_http"

GET "/posts/2"
body                             # => "Post: Post 2 | Accessing at 09:53"
```



在我们刚刚添加新功能时，我们引入了一个 bug。如果我们访问不存在的日志，会发生什么？



```
require "lucid_http"

GET "/posts/12"
body                            # => "Post:  | Accessing at 09:55"
```



结果返回了一个空的日志名称，这与我们想要的相差甚远。



我们可以通过添加对不存在的日志进行条件判断，来修复这个问题。

```
r.get Integer do |id|
  if post = posts[id]
    access_time = Time.now.strftime("%H:%M")
    "Post: #{post} | Accessing at #{access_time}"
  end
end
```



如果日志存在，执行 `if` 表达式，可以获取正确的日志值，string  和 access_time 都会返回正确的值。如果日志不存在，`if` 表达式返回 `nil`，Roda 返回 404 状态码，以及空结构。



```
require "lucid_http"

GET "/posts/12"
status                            # => "404 Not Found"
```



还有一种方式，我们可以使用 `using` 方法，跳过当前 block 的其余部分，这适用于任意 Ruby 代码块。`next` 通常被用于跳过当前的迭代器，移动到下一个迭代器，但是由于每个路由块最多只能执行一次，因此它具有提前返回的行为。



```
r.get Integer do |id|
  next unless post = posts[id]
  access_time = Time.now.strftime("%H:%M")
  "Post: #{post} | Accessing at #{access_time}"
end
```



next 如果不带参数，相当于当前代码块返回 `nil`，表示日志不存在，求将返回 404 状态码（和上面使用 if 进行判断的例子一样）。但是，我们可以给 `next` 带上参数，这相当于带着参数提早返回代码块。



```
r.get Integer do |id|
  next "No matching post" unless post = posts[id]
  access_time = Time.now.strftime("%H:%M")
  "Post: #{post} | Accessing at #{access_time}"
end
```



这看起来可以正常工作，虽然没有找到日志，但是因为代码块返回了 `string`，请求被当成了成功响应。



```
require "lucid_http"

GET "/posts/12"
body                            # => "No matching post"
status                          # => "200 OK"
```



body 的内容是正确的，但是响应码是 200，我们需要在调用 `next` 之前手动设置状态码。



```
r.get Integer do |id|
  unless post = posts[id]
    response.status = 404
    next "No matching post"
  end

  access_time = Time.now.strftime("%H:%M")
  "Post: #{post} | Accessing at #{access_time}"
end
```



然后我们可以检查一下当前所使用的状态码和 body 的内容。



```
require "lucid_http"

GET "/posts/12"
body                            # => "No matching post"
status                          # => "404 Not Found"
```



#### 元编程路由



在 Roda 的设计中，我们对路由具有完全的控制权，这允许我们使用多种方式定义路由，尽量使用方便的方式定义，以此来减少代码。我们甚至可以使用元编程。



考虑下面的路由树。响应根据请求的参数，来读取不同的视图文件，然后返回内容。（view 依赖 render 插件，我们稍后讨论）。我们写了一些重复的定义代码，这块似乎可以简化一下。



```
route do |r|
  r.get "about"      { view("about") }
  r.get "contact_us" { view("contact_us") }
  r.get "license"    { view("license") }
end
```



像移除 Ruby 中其他重复的代码一样，我们可以移除这块重复定义的代码，然后把他们放进循环中。



```
route do |r|
  %w[about contact_us license].each do |route_name|
    r.get(route_name) { view(route_name) }
  end
end
```



让我们来想一下这块是如何工作的。我们在`运行时`创建了一组路由。这看起来充满了魔法，或者使用了高级元编程的技术，但其实并没有。这只是在数组上调用了 each 方法。这是 `Roda` 赋予的力量。我在 `运行时` 上做了标注。如果这组路由没有被匹配到，那么循环不会被触发。



在后面的章节，我们会看到如何使用内置的数组匹配器来进一步简化上面的循环。



### 匹配器



之前我们讨论了 Roda 默认内置的匹配器。然而那些只覆盖了很少一部分。让我们先简单看一下之前讨论过的匹配器，然后再讨论一些新的。



#### 字符串匹配器



字符串匹配器是最通用的匹配器。它与剩余路径中的下一节路径进行比对。

```
route do |r|
  r.get "posts" do
    # GET /posts
  end
end
```



如果字符串中包含 `/`，我们就可以在一串字符串中处理多节路径。



```
route do |r|
  r.get "posts/today" do
    # GET /posts/today
  end
end
```



当使用字符串匹配器时，它所匹配的段不会被捕获，但会被消耗（斜杠算在内）。



字符串匹配器将匹配整个段，而不会匹配前面。

```
route do |r|
  r.on "posts" do
    # Requests for /posts and any path starting with /posts/
    # Would not match /posts-today
  end
end
```



如果剩余路径不是以 `/` 开头，字符串匹配器不会进行匹配。



#### 类型匹配器



默认状态下，Roda 只支持两种类型的匹配器，我们前面已经看过了，分别是整数型和字符串。



值得注意的是，字符串类型的匹配器，捕获并使用路径中的下一个非空段。如果剩余路径不是以斜杠开头，或者为空，则它将不匹配。



```
route do |r|
  r.on "posts" do
    r.on String do |seg|
      "0 #{seg} #{r.remaining_path}"
    end
  end

  r.on String do |seg|
    "1 #{seg} #{r.remaining_path}"
  end
end
```



这是不同请求路径的例子。



```
require "lucid_http"

GET "/posts"
status                          # => "404 Not Found"

GET "/posts/"
status                          # => "404 Not Found"

GET "/posts/new"
body                            # => "0 new "

GET "/posts/new/"
body                            # => "0 new /"

GET "/posts/new/recent"
body                            # => "0 new /recent"

GET "/topics"
body                            # => "1 topics "

GET "/topics/"
body                            # => "1 topics /"

GET "/topics/new"
body                            # => "1 topics /new"
```



整形匹配器，匹配并捕获下一节路径中的整数字符（0-9）。当捕获成功，参数会被转换为整数类型，所以在块结构中，收到的参数为整数类型。



```
route do |r|
  r.on Integer do |seg|
    "#{seg.inspect} #{r.remaining_path}"
  end
end
```



下面是一些请求调用的例子。



```
require "lucid_http"

GET "/"
status                          # => "404 Not Found"

GET "/posts"
status                          # => "404 Not Found"

GET "/1a"
status                          # => "404 Not Found"

GET "/1"
body                            # => "1 "

GET "/2/"
body                            # => "2 /"

GET "/3/b"
body                            # => "3 /b"
```



#### 自定义匹配器

虽然 Roda 默认只包含 `String` 和 `Integer` 类型的匹配器，但它可以通过 `class_matchers` 插件，来使任意类型成为匹配器。加载这个插件后，我们需要调用 `class_matcher`，并带上想要注册的类型，然后通过一个正则表达式来进行匹配，最后跟一个代码块结构，来接收正则表达式处理过的数据，并在代码块中返回匹配器要传递出去的内容。（如果返回空，则不会继续执行）



```
class_matcher(Date, /(\d\d\d\d)-(\d\d)-(\d\d)/) do |y, m, d|
 [Date.new(y.to_i, m.to_i, d.to_i)]
end

route do |r|
  r.on Date do |date|
    date.strftime('%m/%d/%Y')
  end
end
```



这个匹配器，接收年-月-日的格式，返回年/月/日类型。



```
require "lucid_http"

GET "/2020-04-23"
body                           # => "04/23/2020"
```



### 布尔类型匹配器



我们已经看过布尔类型匹配器，`true`，`r.get` 或者 `r.post` 通常用于终端匹配。在布尔类型匹配器中，`true` 表示匹配成功，`false` 和 `nil` 表示匹配失败。



使用 `r.get` 和 `r.post` 时，会有多个理由返回 `true` 来进行强制终端匹配。然而很少有理由返回 `false` 或者 `nil`。我们可以调用方法或者访问可能的返回值本地变量。



```
def allow?
  request.ip == '127.0.0.1'
end

def allowed_prefix
  "let-me-in" if allow?
end

route do |r|
  r.on allowed_prefix do
    "Allowed #{r.remaining_path}"
  end

  r.on allow? do
    "Also Allowed #{r.remaining_path}"
  end
end
```



如果请求来自 IP 地址 `127.0.0.1`，将返回如下值：



```
require "lucid_http"

GET "/"
body                            # => "Also Allowed "

GET "/posts"
body                            # => "Also Allowed /posts"

GET "/let-me-in"
body                            # => "Allowed "

GET "/let-me-in/please"
body                            # => "Allowed /please"
```



如果请求不来自 `127.0.0.1`，`allowed_prefix` 将返回 `nil`， `allow?` 将返回false，请求收到 404 响应。



### 正则表达式匹配



Roda 支持正则表达式匹配器。正则表达式匹配器必须匹配完整段，并且他们消耗匹配的段，并捕获任何正则表达式捕获的内容（如果正则表达式没有任何捕获，则为空）



```
route do |r|
  r.on /posts/ do
    # Same as "posts" string matcher
  end

  r.on /posts/i do
    # Similar to a case insensitive string matcher
  end

  r.on /(posts|topics)/ do |seg|
    # Match either of the two segments and yield the matched segment
  end

  r.on /posts(?:\.html)/ do
    # Match with or without .html extension
  end

  r.on /(\d\d\d\d)-(\d\d)-(\d\d)/ do |year, month, day|
    # Handle multiple captures (arguments yielded are strings)
  end
end
```



类似字符串匹配器，如果剩余路径不是以 `/` 正则表达式匹配器不会工作。



### 数组匹配器



使用数组作为匹配器，其中的元素必须包含其他类型的匹配器。数组中的每个成员，将会以此进行捕获，如果有一个元素捕获成功，则数组捕获成功。



```
route do |r|
  r.get "posts", [Integer, true] do |id|
    # GET /posts/1 matches as:
    # * the Integer matcher matches
    # * the remaining path is fully consumed
    # * id is 1
    #
    # GET /posts matches as
    # * the true matcher matches
    # * the remaining path is fully consumed
    # * no argument is yielded (id is nil)
    #
    # GET /posts/new does not match as
    # * the true matcher matches
    # * but the remaining path is not fully consumed
  end

  r.on [/members/, /topics/] do
    # Match either of the regexp matchers
  end
end
```



为了提高可用性，有一个小的不一致之处。如果成员匹配器是一个字符串，则该字符串是生成的。这是为了使用捕获正则表达式的替代方法。



```
route do |r|
  r.on ['posts', 'topics'] do |seg|
    # Match either of the two segments and yield the matched segment
  end
end
```



### 哈希匹配器



哈希匹配器依赖匹配器的 key，key 表示哈希匹配器中，key 所对应匹配器的类型。默认支持 `:all` 和 `:method` 两种。



#### :all

`:all` 匹配器接受可遍历类型的值，行为与 `r.on`， `r.is`，`r.get` 和 `r.post` 相同，进行比对的路径，必须与其成员类型都匹配。通常使用 `:all` 来将多个匹配步骤，合并为一组匹配器。



```
route do |r|
  r.get ['post', {all: ['posts', Integer]}] do |id|
    # GET /post matches as
    # * the first array member matches
    # * the remaining path is fully consumed
    # * no argument is yielded (id is nil)
    #
    # GET /posts/1 matches as
    # * the first array member does not match
    # * the second array member is then tried
    #   * the second array member is a :all hash matcher
    #   * all members of the :all hash matcher match
    # * the remaining path is fully consumed
    # * id is 1
    #
    # GET /posts/new does not match as
    # * the first array member does not match
    # * the second array member is then tried
    #   * the second array member is a :all hash matcher
    #   * the second member of the :all hash matcher does not match
  end
end
```



#### :method

`:method` 哈希匹配器匹配指定的 HTTP 请求类型。当我们想要执行一个非终端匹配，同时为`r.on` 提供其他参数时，我们使用它。



```
route do |r|
  r.on "posts", method: :post do
    # POST requests for /posts or starting with /posts/
  end
end
```



我们也可以使用数组，来匹配多种 HTTP 请求类型。



```
route do |r|
  r.on "posts", method: ['put', 'patch'] do
    # PUT or PATCH requests for /posts or starting with /posts/
  end
end
```



如上所示，我们可以使用符号或者字符串，这里不区分大小写。（比对时会转换为大写）



### 自定义哈希匹配器



Roda 默认只提供了 `:all` 和 `:method` 两种类型的哈希匹配器，作为补充，它提供了 `hash_matcher` 插件，允许我们自定义哈希匹配器。



在先前的章节，我们使用如下例子来自定义匹配方法：



```
require 'roda'

class App < Roda
  route do |r|
    r.with_params "secret"=>"Um9kYQ==\n" do
    end
  end
end
```



让我们修改这段代码，使用自定义哈希匹配器来替代自定义方法匹配器。简单来说，相较于比对参数，我们可以创建哈希匹配器，只需要简单的哈希写法，来判断名为 `secret` 的参数。



```
require 'roda'

class App < Roda
  route do |r|
    r.on(secret: "Um9kYQ==\n") do
    end
  end
end
```



与前面一样，这段代码暂时还无法工作，我们还没有添加 `secret` 哈希匹配器。我们现在来添加。在加载插件后，我们需要调用 `hash_matcher` 方法，并传递相关符号，作为哈希键值。在后面跟的代码块中，返回 `nil` 或者 `false` 来表示没有匹配成功，返回任意其他的值都表示成功。

```
require 'roda'

class App < Roda
  plugin :hash_matcher

  hash_matcher(:secret) do |v|
    params['secret'] == v
  end

  route do |r|
    r.on(secret: "Um9kYQ==\n") do
    end
  end
end
```



`hash_matcher` 插件不处理捕获。但是我们可以将捕获追加到请求的捕获中，来手动添加捕获。例如我们想在 `secret` 参数匹配成功时，操作 `key` 参数，我们可以这样做：



```
require 'roda'

class App < Roda
  plugin :hash_matcher

  hash_matcher(:secret) do |v|
    if params['secret'] == v
      request.captures << params['key']
    end
  end

  route do |r|
    r.on(secret: "Um9kYQ==\n") do |key|
    end
  end
end
```



#### 其他哈希匹配器插件



Roda 附带了多个插件，这些插件添加了自己的哈希匹配器。`path_matchers` 包括 `:prefix`，`:suffix` 和 `:extension` 哈希匹配器。`header_matchers` 包括 `:header`，`:host`，`user_agent` 和 `:accept` 哈希匹配器。`param_matchers` 包括 `:param`，`:param!`，`:params` 和 `:params!` 哈希匹配器。



#### 符号匹配器



符号匹配器与字符串类型匹配器相同。他们是历史遗留产物，在新的 Roda 应用程序中，不再被推荐使用，因为字符串类型的匹配器更加直观且冗余性更低。



```
route do |r|
  r.on :segment do |seg|
    # same as r.on String do |seg|
  end
end
```



#### 自定义符号匹配器



虽然默认情况下，推荐使用字符串类型的匹配器来替代符号类型匹配器，Roda 还是提供了 `symbol_matchers` 插件，允许不同的符号匹配不同的节。让我们假设我们有不同的路由接受用户名，我们的用用户名只允许6到20个字符串。我们可以使用自定义的符号匹配器，来接受用户名并对格式进行验证。



```
require 'roda'

class App < Roda
  plugin :symbol_matchers

  symbol_matcher :username, /([a-z0-9]{6,20})/

  route do |r|
    r.on :username do |username|
    end
  end
end
```



`symbol_matchers` 插件内置许多符号匹配器，`:d` 用于整数匹配（类似于 `Integer` 类型匹配器），`:w` 用于字母匹配，`:rest` 用于比对剩余路径的其余部分。所有这些都消耗并捕获匹配的段（如果是 :rest，则是剩余路径的其余部分）。



#### Proc 匹配器

Roda 允许我们使用 Ruby proc 作为匹配器。proc 返回非 nil 或者非 false，均视为匹配成功。我们可以使用 proc 匹配器替代条件匹配。



```
r.get Integer do |id|
  post = posts[id]
  r.on(proc { post }) do
    access_time = Time.now.strftime("%H:%M")

    "Post: #{post} | Accessing at #{access_time}"
  end
end
```



前面章节的代码，仍然可以正常工作。



```
require "lucid_http"

GET "/posts/2"
body                            # => "Post: Post 2 | Accessing at 10:09"

GET "/posts/10"
status                          # => "404 Not Found"
```



这种匹配方式让代码更复杂，而没有增加什么价值。通常，我们只在从外部获取 proc 并希望该 proc 的值用于匹配时，才使用这种方式。



#### 其他



默认情况下，当我们使用未定义的值作为匹配器时，Roda 将抛出 `Roda::RodaError` 异常。这是为了使用未定义的匹配器时，防止未定义的行为发生。



### 其他 RodaRequest 方法



#### rredirect



当我们浏览到根目录的时候，我们希望页面显示日志列表。我们可以通过复制粘贴 `posts` 的处理代码，来实现。当然我们并不希望代码重复，我们来想办法解决这个问题。



```
class App < Roda
  route do |r|
    r.root do
      posts = (0..5).map {|i| "Post #{i}"}
      posts.join(" | ")
    end

    r.get "posts" do
      posts = (0..5).map {|i| "Post #{i}"}
      posts.join(" | ")
    end
  end
end
```



我们可以通过重定向地址，来避免这个问题。可以通过调用 `r.redirect` 方法来重定向。



```
class App < Roda
  route do |r|
    r.root do
      r.redirect "/posts/"
    end

    r.get "posts" do
      posts = (0..5).map {|i| "Post #{i}"}
      posts.join(" | ")
    end
  end
end
```



我们再次通过浏览器访问网站根目录。可以发现，浏览器马上跳转到了日志列表地址。



通过通过浏览器直接访问地址，浏览器会收到 `200` 状态码，而使用这种方式，浏览器会收到 `302`，并响应重定向。



```
require "lucid_http"

GET "/"
path                            # => "http://localhost:9292/"
body                            # => ""
status                          # => "302 Found"
```



如果我们的请求响应跳转，我们将收到期望的日志列表。



```
require "lucid_http"

GET "/", follow: true
path                            # => "http://localhost:9292/"
body                            # => "Post 1 | Post 2 | Post 3 | Post 4 | Post 5"
status.to_s                     # => "200 OK"
```



#### 重定向路径



通常来说，如果我们想重定向到不同路径，我们将为 `r.direct` 提供单独的路径。但是，如果我们遵循 `URL` 设计，对路径的 `GET` 请求显示表单，与 `POST` 请求提交表单，使用同一路径，并且在 `POST` 之后，我们希望通一页面执行 `GET` 请求，以查看当前更新后的页面，Roda 允许我们省略路径。



```
class App < Roda
  route do |r|
    r.is "posts", Integer do |id|
      @post = Post[id]

      r.get do
        @post.inspect 
      end

      r.post do
        @post.update(updated_at: Time.now)
        r.redirect
      end
    end
  end
end
```



#### 重定向状态码



默认情况下，`r.direct` 使用 302 状态码。我们可以指定重定向方法的第二个参数，来作为状态码。



```
class App < Roda
  route do |r|
    r.root do
      r.redirect "/posts/", 303
    end

    r.get "posts" do
      posts = (0..5).map {|i| "Post #{i}"}
      posts.join(" | ")
    end
  end
end
```



如果我们想要将重定向状态码改为 303，我们可以使用 `status_303` 插件。



#### r.halt



Roda 允许在路由执行的任何时刻，进行返回。因为 Roda 的设计，匹配代码块的存在，这种返回也是必要的。要停止（或终止）请求的处理过程，我们可以在路由树的执行种的任意时刻，调用 `r.halt`。默认情况下，我们调用 `r.halt` 不带任何参数，这使用当前的 response 作为响应。我们可以设置状态，头，或者响应的 body，然后调用 r.halt 来停止请求的处理，并响应。



```
route do |r|
  r.get "posts" do
    if r.params['forbid']
      response.status = 403
      response.headers['My-Header'] = 'header value'
      response.write 'response body'
      r.halt
    end

    # not reached if forbid parameter submitted
  end
end
```



当 `r.halt` 使用请求的默认响应时，我们可以传递符合 rack 规范的 响应对象给 `r.halt`，来替代使用请求当前的响应。一个符合 rack 规范的响应，是一个有三个成员的数组，包含状态码（整数），头（哈希），body（一组字符串）。



```
route do |r|
  r.get "posts" do
    if r.params['forbid']
      r.halt [
        403,
        {
          'Content-Type'=>'text/html',
          'Content-Length'=>'13',
          'My-Header'=>'header value',
        },
        ['response body']
      ]
    end

    # not reached if forbid parameter submitted
  end
end
```



`r.halt` 的默认行为，只支持使用请求的默认响应和符合 rack 规范的对象（一个数组中包含三个成员）。 Roda 也提供了 `halt` 插件，来扩展 `r.halt` ，以在返回当前响应之前，做一些修改。



如果我们加载了 `halt` 插件，我们可以在返回之前，带着响应状态码调用 `r.halt`。



```
r.halt 403
```



或者带着字符串调用 `r.halt`，来作为响应的内容。



```
r.halt 'response body'
```

 

或者带着三个参数调用，同时传递状态码，响应头和内容。



```
r.halt(403, {'My-Header'=>'header value'},  'response body')
```





























