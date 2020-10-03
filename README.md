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



使用三个参数调用 `r.halt` 和使用 rack 对象（数组中有三个元素）响应的区别是使用 rack 对象会直接返回，而使用三个参数调用，首先会更新请求的响应头和 body，然后再进行返回。



#### r.run



Roda 支持在路由的任何地方直接调用其他 rack 应用程序（包括其他 Roda 应用程序），以及将这些应用程序作为子路径，挂载到当前 Roda 应用程序中。

假设我们开发一个管理系统前端，作为一个独立的 rack 应用程序。我们想挂载到当前应用的 `/admin` 路由上。我们将 `/admin` 设置为一个分支，并将任意对该地址的请求，发送到 `/admin` 应用中，并将 admin 的 应用作为响应返回。



```
route do |r|
  r.on "admin" do
    r.run AdminApp
  end

  # rest of application
end
```



### RodaResponse



假设我们有一个 Roda 应用，名为 `App`，（继承自Roda），Roda 会自动初始化 `App::RodaRequest` 作为 App 类的请求父类，这点我们之前讨论过。与之相似，Roda 也会自动初始化名为 `App::RodaResponse` 的类，作为 App 的响应类，`App::RodaResponse` 继承自  `Roda::RodaResponse`。创建响应的子类的理由，与创建请求的子类理由一样，都是为了插件有更大的自定义空间。



`Roda::RodaResponse` 示例比 `Roda::RodaRequest` 更简单一些。添加的方法要少很多。`Roda::RodaResponse` 实例可以访问 `status`（响应状态码）`headers`（响应头），以及body（响应内容）。在讲解 `r.halt` 的小节，我们看到了如何设置状态码和响应头。设置响应内容的方式也类似。只是响应内容要符合 rack 响应要求（对每个响应做出响应的字符串）。



```
route do |r|
  response.status = 403
  response.headers['My-Header'] = "header value"
  response.body = ["response body"]
end
```



`Roda::RodaResponse` 提供一些便捷方法。可以使用操作数组的方式来操作头。



```
route do |r|
  response['Other-Header'] = response['My-Header']
end
```



如在 `r.halt` 章节所示，响应支持写入内容。但是，如果使用手动方式写入内容，那么`Roda` 将忽略所在代码块的结果，并使用已经写入的内容作为响应的内容。



```
route do |r|
  response.write 'response body'
  'ignored'
end
```



`Roda::RodaResponse` 同样支持使用 `redirect` 方法重定向和设置重定向状态码（默认使用302）。我们一般不直接在 `response` 上调用。而是通常在请求上调用，他们具有一致的行为（和在 response 上调用），之后也会终止请求处理。

`

```
route do |r|
  r.is 'old-path'
    response.redirect '/new-path' # 302 status used
  end

  r.is 'other-old-path'
    response.redirect '/other-new-path', 303
  end
end

```

### Roda 类



我们已经了解了一些有关 Roda 运行实例的基础概念，现在让我们来讨论 Roda 中的类本身。



#### app，一个 rack 应用程序



在之前的例子中，我们看到过，Roda 可以当作 rack 应用程序。如果我们需要在 `config.ru` 中运行 Roda（或者运行 Roda 的子类），这个特性是必要的。事实上，Roda 在内部创建了一个 rack 应用，当它被调用的时候，它传递请求的相关环境信息给内部的这个 rack应用。我们可以直接通过 `app` 来访问内部的这个 rack 应用程序。因此，`config.ru` 文件可以被修改为：



```
require "./app"

run App.app
```



添加了 `.app`，此时的行为和之前没有不同，不过速度会稍微快一点。



#### `freeze`，避免意外修改



Roda 应用程序推荐在运行生产环境和测试的时候，将应用设为只读，这是为了避免意外造成的修改导致的运行错误或测试失败。在开发版本中，我们也可以将执行方式设为只读。但是有关代码热加载的库会依赖修改类本身。然而， `rerun` 不会用到这个特性。如果你使用 `rerun` 进行代码热加载，你可以在所有环境都使用一样的 `config.ru` 代码。



```
require "./app"

run App.freeze.app
```



如果你想让开发模式下保持可修改状态，可以通过检查 `RACK_ENV` 环境变量来实现。



```
require "./app"

unless ENV["RACK_ENV"] == "development"
  App.freeze
end

run App.app
```

#### 路由代码块



我们已经讨论了请求和响应，现在，我们继续讨论剩下的主要话题，这是路由块的范围。该路由块是在 Roda 应用程序类的新实例上下文中执行。



```
class App < Roda
  route do |r|
    self.class # App
  end
end
```



因此，对于上面给出的 Roda 应用程序（App），路由块范围是 App 的一个实例。按照设计，Roda 类（因此也是App 类）几乎没有公共实例方法，因此不会污染路由块的范围。有一些内部方法是以 `_roda_` 开头的，但除此之外仅添加了一些方法：



- request 是请求对象（App::RodaRequest 实例）
- response 是响应对象（App::RodaResponse 实例）
- opts 是类选项（我们将在下一节讨论）
- env 是 rack 环境的字典结构（等同于 request.env）
- session 是当前的会话（等同于request.session）



#### opts，类和插件的配置



相较于保存状态在多个实例变量，Roda 将所有类级别的状态保存在一个的字典中，这个字典我们可以通过 opts 来进行访问。插件的配置通常保存在 opts 更好，以方便访问。



在我们创建的 Roda 应用中，有一些设置是需要我们关心的，他们影响 Roda 本身的行为，或者 Roda 相关的多个插件的行为。



- :root 用于设置应用在文件系统中的根目录。应用的各个部分所设置的默认路径通过它设置。默认值为进程的当前工作目录，因此，如果我们的 Roda 应用程序是从另一个目录运行的，我们肯定应该设置这个。
- :freeze_middleware 会在构建应用程序的时候，冻结正在使用的每个中间件。仅当我们确定所有正在使用的中间件中冻结后，都可以正常工作时，我们才应该使用这个选项
- :add_script_name 会在构建绝对链接和 URL 时，在请求环境前添加 `SCRIPT_NAME`。如果我们从自路径而不是从根路径运行 Roda 应用程序，需要设计此选项。



#### plugin，加载插件



我们之前看到过，plugin 用于为 `Roda` 应用程序加载插件。有些插件不接收参数：



```
class App < Roda
  plugin :h
  plugin :flash
end
```



有些插件接收可选的参数：



```
class App < Roda
  plugin :render
  plugin :render, escape: true
end
```



还有一些必须要参数：



```
class App < Roda
  plugin :request_aref, :raise
  plugin :match_affix, "", /(?:\/\z|(?=\/|\z))/
end
```



当插件需要的时候，我们可以在插件加载的时候，传递代码块进去。这只在插件需要的时候会别使用。如果插件不需要，则会忽略（Ruby 的默认行为）：



```
class App < Roda
  plugin :not_found do
    "File Not Found"
  end

  plugin :error_handler do |e|
    "Internal Server Error"
  end
end
```



#### route，设置路由代码块



先前的例子已经演示，`route` 指令设置使用的路由快。我们还没讨论的是，路由快也被视为匹配快。与任何匹配快一样，如果尚未写入响应主体，并且路由块的返回值为字符串，则将其设为响应内容。因此，如果王默恩希望对所有请求使用相同响应，则无需使用 `r.on`，只需要让 `route` 块返回值即可。



```
class App < Roda
  route do |r|
    "Response body for all requests"
  end
end
```



在内部，出于性能的考虑，Roda 使用我们传递的块进行路由并由其创建一个实例方法，该方法在被当作 `rack` 应用调用的时候被使用。如果你想访问 `route` 块，我们可以使用 `route_block`。 



#### 中间件处理



我们可以再 `config.ru` 文件中，使用 `use` 来加载中间件，Roda 同样支持使用 `use` 在 Roda 中加载中间件。这对于依赖中间件的 Roda 应用程序来说，很有用。将中间件引用写在 `Roda` 中，可以被任意多个 `config.ru` 文件使用。



```
require 'logger'

class App < Roda
  use Rack::CommonLogger, Logger.new($stdout)
end
```



默认情况下，中间件是作为子类被继承的，我们可以通过 `inherit_middleware` 来关闭中间件。另外，如果我么要清楚中间件堆栈，可以使用 `clear_middleware!`。比如要配置中间件加载顺序这种更高级的中间件，我们可以使用 `middleware_stack` 插件。



值得注意的是，rack 中间件的工作方式与插件不同。每个 `rack` 中间件都会包装应用程序。因此，如果我们在应用程序中加载了 3 个 rack 中间件，则首先执行第一个，然后执行第二个和第三个，最后将其分发给应用程序。如果 Rack 中间件和 Roda 插件都可以满足需求，我们最好还是选择 `Roda` 插件来完成。



### 访问请求数据



截止到目前，我们已经学习了如何拿到客户端请求，并响应。那么如何给客户端请求传递数据呢？Roda 允许我们以多种方式读取客户端数据。第一种方式我们已经看过：在请求路径重传递信息。



#### 请求路径



请求路径中的可变数据通常起着双重作用：我们希望基于路径的结构来路由请求，我们也希望捕获路径的一部分以供以后使用。



让我们通过一个示例，来演示通过一个请求中匹配三个参数。第一个参数是 `posts`，第二段部分是一个数字ID，第三部分是表示操作的字符串。在 match 块中，我们将返回 ID 的检查值和操作的检查值。



```
class App < Roda
  route do |r|
    r.get "posts", Integer, String do |id, action|
      "#{id.inspect} - #{action.inspect}"
    end
  end
end
```



如果我们尝试访问路由，我们看到 ID 参数被当作 整数处理，操作参数被当作字符串处理：



```
require "lucid_http"

GET "/posts/1/show"
body                            # => "1 - \"show\""

GET "/posts/2/update"
body                            # => "2 - \"update\""
```



在上面的例子中，我们在一个 `r.get` 代码块中，接收了三个参数。但是，在许多情况下，最好使用每个段使用单独的匹配块，因为路由树上可能都有其他路由。



让我们为一组 model 添加模型。首先，我们已经有了一个匹配单个模型的路由。我们只想匹配被允许的模型名字，因为允许所有名字，可能会收到我们不想要的名字。我们要小心任何客户端传递过来的数据包含攻击信息，并尽可能的限制被允许的输入。我们使用数组限制模型名称，虽然对大多数模型，我们可能希望使用更有效的方法。如果我们还记得数组匹配器那一节的内容，，当匹配的成员是一个字符串时，将同时捕获该字符串。我们将据此找到被提交的字符串。



有关数组模型，需要注意的是，我们在 App 类中定义，在 `route` 块中引用。我们可以在在调用 `r.on` 时内联它，或在 `route` 块内使用局部变量，但那会让每个请求做一些额外工作。我们也可以使用常量而不是局部变量。那样可以工作的很好。但通常来说，局部变量是一种更简单的方法，我们仅在需要的时候才使用常量。



在 `match` 块中，我们使用模型名称去寻找对应的模型类。这时用到 `Object.const_get` 方法，这个方法接收一个字符串，返回对应的 class。`Object.const_get` 只接收可信的输入，这就是为什么我们确保限制允许的类名称的原因。请注意，只要只要知道模型名称，就可以使用它。我们不必等到路径完全路由即可获得对模型的引用。这使我们能够在任何嵌套路由中使用此模型，这正式我们想要的。



```
class App < Roda
  models = ["account", "post"]

  route do |r|
    r.on models do |model_name|
      model_class = Object.const_get(model_name.capitalize)

      # ...
    end
  end
end
```



然后，我们有了用来匹配 `/post/index` 的 `index` 路由，将返回 posts 的列表。



```
class App < Roda
  models = ["account", "post"]

  route do |r|
    r.on models do |model_name|
      model_class = Object.const_get(model_name.capitalize)

      r.get "index" do
        model_class.all.join(" | ")
      end
    end
  end
end
```



如果我们到了这里，但是剩余路径不是 `/index`，或者请求方法不是 `GET`，我们将跳过该块，并尝试与模型的 ID进行匹配。如果接下来的一节是数字，会假定为模型的 ID。这意味着我们有兴趣对先前的模型进行操作，因此，接下来的逻辑将访问对应的模型实例。



```
class App < Roda
  models = ["account", "post"]

  route do |r|
    r.on models do |model_name|
      model_class = Object.const_get(model_name.capitalize)

      r.get "index" do
        model_class.all.join(" | ")
      end

      r.on Integer do |id|
        model = model_class[id]

        # ...
      end
    end
  end
end
```



最后，一旦我们找到了对应的模型，且请求是一个 `GET` 类型，对于 `show` 的请求，我们将显示模型的信息。如果是对 `update` 的 `POST` 请求，我们将更新模型实例（在这个例子中，我们将只返回我们打算更新的内容）。



```
class App < Roda
  models = ["account", "post"]

  route do |r|
    r.on models do |model_name|
      model_class = Object.const_get(model_name.capitalize)

      r.get "index" do
        model_class.all.join(" | ")
      end

      r.on Integer do |id|
        model = model_class[id]

        r.get "show" do
          model.to_s
        end

        r.post "update" do
          "Updating #{model}"
        end
      end
    end
  end
end
```



这是一个简单的例子，它演示了如何设计路由树来一级一级的处理路径。如果需要，我们可以引入请求的数据在路径中。



#### 查询字符串参数



我们同样可以传递查询字符串。查询字符串在 URL 的末尾，以问号作为开头标记，使用 `key=value` 串作为内容，多个查询字符串以 `&` 进行分割。



来看个例子：我们将 `post` 参数值设为 42，`action` 值为设为 `show`



```
http://localhost:9292?post=42&action=show
```



这种请求方式，通常用于使用 `GET` 请求提交数据的时候。最常见的使用场景就是搜索。让我们来假设有一组文章，我们希望通过 q 参数传递搜索条件。



我们来添加一个 `GET` 请求，来处理 `/search` 路径，用来执行我们的搜索请求。我们调用请求对象的 `r.query_string` 实例方法，并作为响应内容返回（query_string 方法来自 `Rack::Request` 对象）。



```
class App < Roda
  route do |r|
    r.get "search" do
      r.query_string
    end
  end
end
```



此时当我们浏览 `http://localhost:9292/search?q=article`，我们看到 `r.query_string` 被返回。



```
require "lucid_http"

GET "/search?q=article"
body   
```



我们可以解析这段字符串为 key value 形式，这个方法已经内置，通过 `r.params` 可以使用（这个方法同样来自 Rack::Request）。



```
class App < Roda
  route do |r|
    r.on "search" do
      r.params.inspect
    end
  end
end
```



当我们浏览  http://localhost:9292/search，并传递 `q` 作为查询字符串参数时，我们可以在终端中看到解析为哈希结构的参数。如果我们添加另一个查询参数，我们将在终端中看到新的结构被输出。

```
require "lucid_http"

GET "/search?q=article"
body                    # => "{\"q\"=>\"article\"}"

GET "/search?q=article&category=video"
body                   # => "{\"q\"=>\"article\", \"category\"=>\"video\"}"
```



现在，我么可以动手修改我们的路由了。我们想搜索所有包含请求中 `q` 参数的文章，并将这些文章内容合并在一起，返回一个字符串。



```
class App < Roda
  ARTICLES = [
    "This is an article",
    "This is another article",
    "This is a post",
    "And this is whatever you want it to be",
  ]

  route do |r|
    r.on "search" do
      ARTICLES.filter do |article|
        article.include?(r.params["q"])
      end.join(" | ")
    end
  end
end
```



当我们查找 `article` 关键字，我们会收到所有包含 `article` 的文章。



```
require "lucid_http"

GET "/search?q=article"
body        
```



如果您之前是 Rails 开发者，您可能好奇是否可以使用符号的方式来获取参数，答案是不可以。`r.params` 返回一个 Ruby 哈希结构，哈希是区分符号和字符串的。这也提现了 Roda 是非侵入式的，他使用了 Ruby 核心类，并且没有修改其默认行为以适应一些场景。如果我们熟悉 Ruby 核心类是如何工作的，我们会发现 Roda 易于理解。



上面的例子还存在一个小问题。如果我们直接访问搜索页，而没有带 `q` 参数会发生什么？我们可以试一下，将会看到抛出了 `TypeError`，因为 `r.params[q]` 是 `nil`，而 `String#include?` 不能接收 `nil` 作为参数。类似问题也存在于 q 被解析为数组或者哈希（都有可能）



有多种方式可以修复这个问题。一种是将 `r.params["q"]` 转换为字符串。这样做的好处是可以处理所有输入而不会出错。但是由于 `nil` 会转换为空字符串，这将会导致不指定显示所有文章的 `q` 参数而直接进入搜索页面。



```
class App < Roda
  route do |r|
    r.on "search" do
      ARTICLES.filter do |article|
        article.include?(r.params["q"].to_s)
      end.join(" | ")
    end
  end
end
```



另一种方式是使用 Ruby  内置的标准类型判断：



```
class App < Roda
  route do |r|
    r.on "search" do
      case q = r.params["q"]
      when String
        ARTICLES.filter do |article|
          article.include?(q)
        end.join(" | ")
      else
        "Invalid q parameter"
      end
    end
  end
end
```



#### 请求内容的参数

另外一种方式处理请求中所提交的数据是通过 `POST` 方法。当浏览器通过 `POST` 提交时，数据在请求的`body` 中，这点与写在请求字符串中不同。



我们增加一个例子。我们想提交 `POST` 请求到 `http://local:9292/articles`，并接收 `content` 中的参数，以此来创建文章。



这个路由目前还不存在，我们调用会收到 `404` 状态码。



```
require "lucid_http"

POST "/articles", form: {content: Time.now.strftime("%H:%M:%S") }
body                            # => ""
status   
```



我们使用 `r.post`，以及 `articles` 这个地址进行匹配。在匹配块中，我们需要解析请求发来的请求中的数据。可以使用与查询时相同的方式进行解析，即通过 `r.params` 方法。`r.params` 同时可以处理查询字符串以及请求 body。



如果被匹配到，我们可以将内容中的参数创建新的文章，以及返回最新的文章创建时间，以及文章统计现有的文章。



```
r.post "articles" do
  ARTICLES << r.params["content"]
  "Latest: #{ARTICLES.last} | Count: #{ARTICLES.count}"
end
```



现在，我们再次尝试，我们将看到文章已经被添加。



```
require "lucid_http"

POST "/articles", form: {content: Time.now.strftime("%H:%M:%S")}
# => "Latest: 12:13:38 | Count: 5"

sleep 2

POST "/articles", form: {content: Time.now.strftime("%H:%M:%S")}
# => "Latest: 12:13:40 | Count: 6"
```



这个例子和之前的例子具有相同的问题，当我们提交的内容不包含 `content` 参数，将会发生意料之外的行为（本例中，`nil` 会被添加到数组中）我们想要确保 `r.params["content"]` 必须为一个有效的字符串。



我们可以看到，`r.params` 合并了查询字符串以及请求 `body` 参数。通常，我们不需要区分这些，如果想要区分，我们可以在 `r.GET` 中处理查询参数，在 `r.POST` 处理请求 body 参数（两种方式都来自 `Rack::Request`）。



`Roda` 依赖 rack 解析 body，因此，它仅能处理 rack 支持的请求内容解析，包括 `application/x-www-form-urlencoded` 和 `multipart/form-data`。`HTTP` 支持其他类型的请求，对于 `JSON` 请求，请求的类型通常是 `application/json`，如果我们使用 json_parser 插件，`Roda` 可以处理这些请求。



#### 请求头部



最后要访问请求的数据，是请求头部。内置访问请求头部的方式是通过环境变量哈希（env）。环境变量通过哈希的方式，包含所有请求的头部，但 rack 将头部的 keys 转换为大写，并在前面增加了 `HTTP_` 字符串（`Content-Type` 和 `Content-Length` 是个例外）。所以，如果我们想要访问 `My-Foo` 头，我们需要使用 `env["HTTP_MY_FOO"]`。



如果我们不能记住这条规则，我们可以使用 request_headers 插件，引入侯，我们就可以使用 `headers["My-Foo"]` 来访问了。



### Roda 插件



Roda 的哲学并非是所有功能开箱即用。如果我们理解这一点，，我们就已经理解了 95% 的 Roda 核心内容。



然而，Roda 并非只包括 Roda 核心。就代码而言，Roda 核心不到 Roda 整个项目的 15%。Roda 其余的 85%，是通过插件来实现的。有一些插件，在大多数应用程序中都会被用到。另外一些插件，只在某些特定应用中会被使用，除了这些，其他应用几乎不用。本章覆盖了一些最常用的插件，以及他们的用法。而没有包含一些特殊的插件的使用。



要想找到插件列表，我们可以访问 Roda 网站： [http://roda.jeremyevans.net](http://roda.jeremyevans.net/)，并点击文档连接。在那里能找到插件列表。点击一个插件将看到我们使用 `RDoc` 生成的插件信息及使用介绍。



我们现在开始讨论插件和插件的渲染，或者处理响应。



### 渲染



#### public



关于渲染部分，我们首先来看 `public` 插件。本插件位于文档页顶部，从标题看过去，已经大概明白要讨论的是什么了。我们要讨论的内容，涵盖足够的关于插件基本使用的内容。如果你想进一步了解关于插件的信息，可以查看插件具体的连链接，包含类和模块的内容。如果有插件需要配置，会在 `configure` 方法中。加载插件会调用 `configure` 方法，以根据加载插件时传递的参数配置 `Roda` 插件。



如果我们出于好奇或者调试，想看一下插件是如何实现的，可以访问 https://github.com/jeremyevans/roda，在 `lib/roda/plugins/<plugin name>.rb` 文件中进行查看。



在我们尝试 `public` 插件之前，我们先来构建一个空的 `app`



```
class App < Roda
  route do |r|
  end
end
```



对于这个 `Roda` 应用程序，任何请求都将收到 `404` 状态。可以尝试访问 `http://localhost:9292/dave.html `，验证是否存在。

```
require "lucid_http"

GET "/dave.html"
status                          # => "404 Not Found"
```

现在，我们将看到加载和使用 `public` 插件，行为所发生的变化。如之前章节所述，要添加插件，我们需要调用 `plugin` 类方法，并传递插件的名称（在本例中是 `:public`）的符号形式。



注意，在本项目中，我们没有添加任何新的 gems，`Roda` 已经包含了本插件，而 `public` 插件也不需要依赖任何其他包。我们也不需要`require` 语句。Roda 已经自动引入插件文件。



`public` 增加了 `r.public` 方法。当我们在 `route` 块中调用调用 `r.public` 方法，它会对这个路径增加一个静态文件服务。



```
class App < Roda
  plugin :public

  route do |r|
    r.public
  end
end
```



默认情况下，`public` 插件为 `<app root>/public` 目录（`<app root>` 由应用的 `:root`来进行配置的）。让我们创建 `public` 目录，然后添加一些文件。



```
  Dir.mkdir("public")
  Dir.chdir("public")

  %w[chris dave matt pete].each_with_index do |doc_name, i|
    doc_num = i + 9
    File.write("#{doc_name}.html", <<CONTENT)
<h2>My name is #{doc_name.capitalize} <h2>
<h3>and I'm ##{doc_num}</h3>
CONTENT
    end
  end
```



现在我们已经添加了 `public` 文件夹，尝试访问 `http://localhost:9292/dave.html`，我们将看到正常渲染的页面和 `200` 状态码。



```
require "lucid_http"

GET "/dave.html"
body               # => "<h2>My name is Dave <h2>\n<h3>and I'm #10</h3>\n"
status             # => "200 OK"
```



如果我们想用 `static` 目录替换 `public` 目录来提供文件服务，我们需要在加载插件的时候，传递 `:root` 选项。



```
class App < Roda
  plugin :public, root: "static"

  route do |r|
    r.public
  end
end
```



现在，重启我们的应用，然后插件配置生效。此时，`public` 目录被替换为 `static` 目录。当我们的变更生效时，再次访问，会受到 `404` 状态码，因为目录还不存在。



```
require "lucid_http"

GET "/dave.html"
body                          # => ""
status                        # => "404 Not Found"
```



然后我们将 `public` 目录改名为 `static`，



```
File.rename('public', 'static')
```



再次重试，一切恢复正常。



```
require "lucid_http"

GET "/dave.html"
body               # => "<h2>My name is Dave <h2>\n<h3>and I'm #10</h3>\n"
status             # => "200 OK"
```



使用 `public` 插件一个特性是，我们服务的目录来自子目录。所以如果我们想访问 `static` 目录的文件，但是想在路径前增加 `/static`，我们可以在 `r.on "static"` 中调用  `r.public` 来实现。



```
class App < Roda
  plugin :public, root: "static"

  route do |r|
    r.on "static" do
      r.public
    end
  end
end
```



我们再次发送相同的请求，会收到 `404` 状态码，因为目录还不存在。然而，如果我们在请求前面加上 `/static` 前缀，就可以找到正确的内容。



```
require "lucid_http"

GET "/dave.html"
body                          # => ""
status                        # => "404 Not Found"

GET "/static/dave.html"
body               # => "<h2>My name is Dave <h2>\n<h3>and I'm #10</h3>\n"
status             # => "200 OK"
```



`public` 插件另一个优秀的特性是，所服务的文件，支持使用 `gzip` 或者 `brotli` 进行压缩。如果请求表明支持 `gzip` 或 `brotli` 特性，则会收到压缩过的文件。未被压缩的文件，性能会更高一些，因为省去了压缩和解压缩的时间，相应的，可以通过想办法减小传输文件的体积，来增加传输速度。



让我们创建一个被压缩过的文件，以及删除原始文件。



```
require 'zlib'

Zlib::GzipWriter.open('static/dave.html.gz') do |gz|
  gz.write(File.read('static/dave.html'))
end
File.delete('static/dave.html')
```



然后修改插件的选项，使用 `:gzip` 设置。



```
class App < Roda
  plugin :public, root: "static", gzip: true

  route do |r|
    r.on "static" do
      r.public
    end
  end
end
```



然后我们来通过请求来检查这项特性是否可以正确工作，确保只有 `gzipped` 版的文件存在。



```
require "lucid_http"

GET "/static/dave.html"
body               # => "<h2>My name is Dave <h2>\n<h3>and I'm #10</h3>\n"
status             # => "200 OK"
```



需要注意的是，在生产环境中，同一个文件我们需要确保压缩过的和没压缩过的都存在，这样可以提供更好的兼容性。



### 生成 HTML



目前为止，我们一直使用 Roda 返回一小段字符串作为相应。这有点脱离真实环境。大多数现实场景使用 HTML 作为页面，或者 HTML 片段，或者 JSON。Roda 内核并不支持这些形式的返回值，正常情况下，我们返回 HTML 或者 JSON 二者其一。Roda 核心努力让自身变的更小，这些高级特性通过插件来实现。



在大多数 Web 应用生产环境，我们不会在路由处理中，直接返回响应主体。大多数应用程序将使用专用方法处理响应主体的创建。在本节中，我们将展示如何将生成主体的代码从路由树直接返回改为使用拆分的模板，并从模板中返回内容。从模板读取内容并返回主体的过程，通常被称为 `渲染`。



假设我们需要编写一个 to-do list 应用。我们以空行作为分割，每一行返回一项任务。



```
require "roda"
require "./models"

class App < Roda
  route do |r|
    r.root do
      Task.all.map(&:title).join("\n")
    end
  end
end
```



查看命令行输出内容，与设想的一致。



```
require "lucid_http"

GET "/"
puts body

# Play Battletoads
# Learn how to force-push
# Find radioactive spider
# Rescue April
# Add red setting to sonic screwdriver
# Fix lightsaber
# Shine claws
# Saw cape
# Buy Blue paint
# Repaint TARDIS
```



在浏览器中查看，展示上会有些不正常，因为 `Content-Type` 使 `text/html`（Roda 默认），但我们返回的内容是 `plain text`。所以换行被转换为空格。



如果我们想要在浏览器中显示的正常，我们需要返回 HTML 格式的内容。我们可以使用无序列表和复选框来展示任务。



一种方式是为每一个任务项目后面添加空白的字符串，并将该字符串作为响应正文返回。这是可行的，但是很难看，而且维护困难。



```
route do |r|
  r.root do
    result = String.new
    result << "<ul>"
    Task.all.each do |task|
      result << "<li class=\"#{task.done? ? :done : :todo}\">"
      result << "  <input type=\"checkbox\"#{" checked" if task.done?}>"
      result << "    #{task.title}"
      result << "</li>"
    end
    result << "</ul>"
    result
  end
end
```



我们可以将生成逻辑拆分为独立的方法，或者添加一些帮助方法。但是最终构建代码还是很长。Roda 通过 `render` 插件来支持模板，许多框架也有类似的机制。



#### render（渲染）



使用 `render` 插件，我们可以删除上面构建页面的冗余代码，替换为 `render` 方法，传递模板的名字。



开始使用 `render` 插件之前，我们首先需要安装 `tilt`。这个 `gem` 被 `render` 用来渲染模板。我们将 `tilt` 添加到 `Gemfile` 文件中，然后执行 `bundle install`。



```
source "https://rubygems.org"

gem "roda"
gem "puma"
gem "tilt"
```



安装 `gem` 之后，我们可以使用 `render` 插件了。我们将在应用程序中加载，然后尝试渲染模板。



```
class App < Roda
  plugin :render

  route do |r|
    r.root do
      render "tasks"
    end
  end
end
```



然后我们尝试一下，看看得到什么。由于尚未创建模板文件，会出现一个错误。错误消息很有用，现在我们知道插件在哪个文件夹寻找文件。正如扩展名所示，插件期望的默认模板语言是 erb，并且期望目录在 `views` 中。



```
require "lucid_http"

GET "/"
error
# => "Errno::ENOENT: No such file or directory @ " \
#    "rb_sysopen - /home/lucid/code/my_app/views/tasks.erb"
```



所以我们创建 `views` 目录和 `views/tasks.erb` 文件。在 `views/tasks.erb` 文件中，我们可以添加模板代码。我们不止可以返回 `HTML`片段，也可以在其中返回完整的 HTML 页面。



我们给 `HTML` 文件加上标题，对于列表中的任务，我们把他们逐项列出。条目的选中状态由 `done` 和 `todo` 来判断，配合 `li` 元素进行显示。在列表条目内部，增加 `checkbox`，当任务完成时，显示选中状态。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <h2>Tasks</h2>
    <ul>
      <% @tasks.each do |task| %>
        <li class="<%= task.done? ? :done : :todo %>">
          <input type="checkbox"<%= " checked" if task.done? %>>
          <%= task.title %>
        </li>
      <% end %>
    </ul>
  </body>
</html>
```



再次尝试，我们收到 `NoMethodError` 错误。这是意料之中，因为我们还没有传递 `@tasks` 实例变量。



```
require "lucid_http"

GET "/"
error
# => "NoMethodError: undefined method `each' " \
#    "for nil:NilClass for #<App:0x00000001bd6ba8>"
```



我们可以在渲染模板前，设置 `@tasks` 变量的值，来修复这个问题。



```
class App < Roda
  plugin :render

  route do |r|
    r.root do
      @tasks = Task.all
      render "tasks"
    end
  end
end
```



当我们访问这个页面，我们看到了一个标准的列表。



![img](.\assets\render_render_working.png)

在路由树中设置变量，即允许在视图中进行访问，因为视图与路由在同一代码作用域下。在路由树中设置实例变量后，无需再额外传递给视图。



除了上面介绍这种传递实例变量的方法，我们直接将实例变量作为参数传递给渲染方法也可以传递实例变量。



明确传递变量的方式是使用 `:locals` 作为 `render` 方法的参数名进行传递，`locals` 的内容是一个哈希结构，传递完之后，在视图中，即可访问。



```
class App < Roda
  plugin :render

  route do |r|
    r.root do
      render "tasks", locals: { tasks: Task.all }
    end
  end
end
```



使用 `locals` 参数传递后，我们将 `@tasks` 修改为 `tasks` 来进行访问。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <h2>Tasks</h2>
    <ul>
      <% tasks.each do |task| %>
        <li class="<%= task.done? ? :done : :todo %>">
          <input type="checkbox"<%= " checked" if task.done? %>>
          <%= task.title %>
        </li>
      <% end %>
    </ul>
  </body>
</html>
```



通常来说，使用实例变量进行传递的方式更加值得推荐。这种方式会有更好的性能，更少的代码。也有些开发者更习惯使用 `locals` 参数的方式明确传递变量。本书余下部分，大多数情况下都会使用实例变量的方式将变量传递给视图。后面也会演示一种适合使用 `locals` 传递变量的场景。



现在来美化一下列表。我们简单的将样式写在 `head` 标签里，增加一下颜色区分，删除前面的圆点。圆点在 `checkbox` 样式下，显得很奇怪。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <ul>
      <% @tasks.each do |task| %>
        <li class="<%= task.done? ? :done : :todo %>">
          <input type="checkbox"<%= " checked" if task.done? %>>
          <%= task.title %>
        </li>
      <% end %>
    </ul>
  </body>
</html>
```



重新加载，可以看到样式已经生效。



![img](.\assets\render_render_working_with_css.png)

现在，列表变得更加漂亮了，我们来增加下一个特性：显示指定任务的详情。我们先来添加新的路由，给要使用的实例变量赋值，然后渲染视图。



```
route do |r|
  r.root do
    @tasks = Task.all
    render "tasks"
  end

  r.get "tasks", Integer do |id|
    next unless @task = Task[id]
    render "task"
  end
end
```



然后添加 `views/task.erb` 文件。在这个文件中，我们需要展示任务标题，任务是否完成，还有它的截止日期。我们把之前的 `erb` 代码复制过来进行修改。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <h2><%= @task.title %></h2>
    <% if @task.done? %>
      <span class="done">[DONE]</span>
    <% else %>
      <span class="todo">[TODO]</span>
    <% end %>
    <h3>Due Date: <%= @task.due.strftime("%v") %></h3>
  </body>
</html>
```



我们再试一次，就会看到新的模板被渲染出来。这是已经完成的任务。



![img](.\assets\render_render_single_task_done.png)

这是个任务没完成时的样子。



![img](.\assets\render_render_single_task_undone.png)

### 布局



我们现在有两个不同的模板，代码有可优化的空间。我们使用相同的 HTML 结构，CSS 样式和标题，我们每个需要展示的页面都重复这些内容。



如果我们想改变基础的布局，比如 `done` 状态的颜色增加绿色阴影，我们需要把每个视图都改一遍。



我们可以通过将 `render` 替换为 `view`，来修复重复编写代码的问题。



```
route do |r|
  r.root do
    @tasks = Task.all
    view "tasks"
  end

  r.get "tasks", Integer do |id|
    next unless @task = Task[id]
    view "task"
  end
end
```



当尝试加载任意页面时，我们会收到 `layout.erb` 没有找到的问题。



```
require "lucid_http"

GET "/"
error
# => "Errno::ENOENT: No such file or directory @ " \
#    "rb_sysopen - /home/lucid/code/my_app/views/layout.erb"
```



接下来，我们将页面相同的代码，移动到布局视图中，在布局中，我们使用 `yield` 语句，来插入不同页面的内容。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%= yield %>
  </body>
</html>
```



我们可以更新 `tasks.erb` 模板。



```
<h2>Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <li class="<%= task.done? ? :done : :todo %>">
      <input type="checkbox"<%= " checked" if task.done? %>>
      <%= task.title %>
    </li>
  <% end %>
</ul>
```



接着更新 `task.erb` 模板。



```
<h2><%= @task.title %></h2>
<% if @task.done? %>
  <span class="done">[DONE]</span>
<% else %>
  <span class="todo">[TODO]</span>
<% end %>
<h3>Due Date: <%= @task.due.strftime("%v") %></h3>
```



最后，来检查一下是否工作正常。



配合 `view` 方法，来渲染视图时，特定于页面的视图在布局视图之前呈现。这很有用，它允许我们在页面特定的视图中设定实例变量，并在布局中访问他们，比如用于设置 HTML 页面标题。我们来修改一下布局文件，以使用 `@page_title` 实例变量来自定义页面标题，并将前一个标题用作备用。



```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%= yield %>
  </body>
</html>
```



更新 `tasks.erb` 视图，以设置 `@page_title`。

```
<% @page_title = "All Tasks" %>

<h2>Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <li class="<%= task.done? ? :done : :todo %>">
      <input type="checkbox"<%= " checked" if task.done? %>>
      <%= task.title %>
    </li>
  <% end %>
</ul>
```



同样可以更新 `task.erb` 来设置 `@page_title`。

```
<% @page_title = "Task: #{@task.title}" %>

<h2><%= @task.title %></h2>
<% if @task.done? %>
  <span class="done">[DONE]</span>
<% else %>
  <span class="todo">[TODO]</span>
<% end %>
<h3>Due Date: <%= @task.due.strftime("%v") %></h3>
```



#### content_for



在一些场景下，布局可能有一些地方，需要使用页面提供的内容，而不仅仅是一种内容。虽然我们可以给一个变量复制，然后传递给布局来实现，但这样会写许多丑陋的代码。如果要返回的内容比较多，尤其明显。



下面的例子，我们来假设我们想添加一个`footer` 到我们的页面。我们想为 `footer` 提供不一样的模板。让我们改变一下我们的视图。默认情况下，我们已经有了 `footer.erb` 来提供默认的 footer。我们可以在布局文件中使用 `render("footer")` 来渲染 `footer` 视图。



```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%= yield %>
    <div class="footer">
      <%= render("footer") %>
    </div>
  </body>
</html>
```

 

简化的 `footer.erb` 视图：

```
This is the footer.
```



这为所有页面添加了同样的 `footer`。假设我们想让页面的 `footer` 显示所有任务，以替代任务数量。我们需要传递 `footer` 的内容到 `tasks.erb` 视图，然后修改视图以使用它。要完成这一点，我们通过使用 `content_for` 插件来完成，本插件允许将内容存储在一个模板文件中，然后在另一个模板中显示。我们首先来激活这个插件。

```
class App < Roda
  plugin :render
  plugin :content_for

  route do |r|
    r.root do
      @tasks = Task.all
      view "tasks"
    end

    r.get "tasks", Integer do |id|
      next unless @task = Task[id]
      view "task"
    end
  end
end
```

接下来，让我们更新 `tasks.erb` 视图以存储自定义 `footer` 来展示多少总共有多少任务。在这个视图中，我们可以在代码块中，调用 `content_for` 方法，然后在其中返回内容。

```
<% @page_title = "All Tasks" %>

<h2>Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <li class="<%= task.done? ? :done : :todo %>">
      <input type="checkbox"<%= " checked" if task.done? %>>
      <%= task.title %>
    </li>
  <% end %>
</ul>
<% content_for(:footer) do %>
  There are <%= tasks.length %> tasks total.
<% end %>
```



最后，让我们更新 `layout.erb` 文件，如果页面提供了自定义的 `footer`，就显示自定义的 `footer`，否则就使用默认的 `footer`。在布局文件中，我们调用 `content_for`，而不用传递代码块，这将返回存储的内容，如果 `content` 没有存储，将返回空。所以，如果 `footer` 的内容被传递，它将被使用，如果没有传递，则使用默认的 `footer`。



```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%= yield %>
    <div class="footer">
      <%= content_for(:footer) || render("footer") %>
    </div>
  </body>
</html>
```



#### 过滤内容



当前模板存在一个问题，导致他们不能过滤某些内容而无法正常显示。如果未来内容被输入控制，会造成跨站脚本攻击（XSS）。要杜绝这种攻击，我们需要过滤输出。



在本书先前部分，我们了解了 `h` 插件，以及如何使用它在显示前过滤内容。在这里我们也可以使用它对每次想输出的内容进行过滤。我们来演示一下在 `task.erb` 文件中使用 `h` 插件。



```
<% @page_title = "Task: #{h @task.title}" %>

<h2><%=h @task.title %></h2>
<% if @task.done? %>
  <span class="done">[DONE]</span>
<% else %>
  <span class="todo">[TODO]</span>
<% end %>
<h3>Due Date: <%= @task.due.strftime("%v") %></h3>
```



糟糕的是，这种使用方式很容易出错，因为在大型应用程序中，我们很可能会忘记进行转义（请注意，上述模板中两个位置都使用了 `h`）。幸运的是，Roda 提供了更好的解决方式。除了过滤我们知道不好的内容以外，我们还可以过滤我们没有意识到的可能有问题的内容（好的内容是指我们已经过滤过的内容）。



我们通过 `render` 插件，以及 `:escape` 选项来开启。 这种自动转义过滤，依赖 `erubi` gem，所以我们需要在 `Gemfile` 中添加它，并执行 `bundle install`。



```
source "https://rubygems.org"

gem "roda"
gem "puma"
gem "tilt"
gem "erubi"
```



安装 `erubi` gem之后，我们可以使用 `render` 插件以及 `:escape` 选项了。



```
class App < Roda
  plugin :render, escape: true

  # ...
end
```



这种方式使我们可以删除 `task.erb` 视图中的手动过滤转义，`h2`标签中的 `@title` 也会被自动转义。即使内容不需要转义，`h3 `中的 `@task.due.strftime("%v")` 也会被输出。



```
<% @page_title = "Task: #{@task.title}" %>

<h2><%= @task.title %></h2>
<% if @task.done? %>
  <span class="done">[DONE]</span>
<% else %>
  <span class="todo">[TODO]</span>
<% end %>
<h3>Due Date: <%= @task.due.strftime("%v") %></h3>
```



在布局视图中，我们只需要做一个改变，调用一下 `yield，将已经处理过的内容输出插入到这里。我们可以通过增加一个 `=` 来实现。



```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <style>
      ul { list-style: none; }
      ul .todo { color: red;}
      ul .done { color: green;}
    </style>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%== yield %>
  </body>
</html>
```



在指定页面输出中，不会被过滤。但使用自动转义时，如果我们要输出已经转义过的值，我们只需要记住使用双等号输出标签而不是单等号。



`render` 插件包含更多可配置的高级项目，`:escape` 选项已经在前面展示过。可以在 [`render` 插件文档](http://roda.jeremyevans.net/rdoc/classes/Roda/RodaPlugins/Render.html)查看更多信息。



### 视图子目录



可以在 `views` 文件夹中放置所有模板文件，这在小型应用中，工作的很好。但是程序规模一大，就需要使用子目录进行组织模板文件了。我们接下来会演示如何来进行组织。让我们创建 `views/tasks` 子目录，并把它移动到 `views` 文件夹中。



```
Dir.mkdir('views/tasks')
File.rename('views/tasks.erb', 'views/tasks/index.erb')
File.rename('views/task.erb', 'views/tasks/task.erb')
```



然后更新我们的应用程序，来渲染子目录的文件，将子目录名称添加到模板名称前面。



```
route do |r|
  r.root do
    @tasks = Task.all
    view "tasks/index"
  end

  r.get "tasks", Integer do |id|
    next unless @task = Task[id]
    view "tasks/task"
  end
end
```



#### 视图选项



上面的例子可以正常工作，但是相关的子目录名字，在不同路由中，被返回了两次。让我们使用 `view_options` 插件修复这个问题。首先，我们来把例子修改的更加接近现实。通常，我们只会在有多个路由的情况下使用子目录模板。每个分支都使用相同的模板子目录是十分普遍的情况。因此，我们将扩展示例，使其具有用于任务的分支和用于帖子的分支。



```
class App < Roda
  plugin :render, escape: true

  route do |r|
    r.on "tasks" do
      r.get true do
        @tasks = Task.all
        view "tasks/index"
      end

      r.get Integer do |id|
        next unless @task = Task[id]
        view "tasks/task"
      end
    end

    r.on "posts" do
      r.get true do
        @posts = Post.all
        view "posts/index"
      end

      r.get Integer do |id|
        next unless @post = Post[id]
        view "posts/post"
      end
    end
  end
end
```



如前面的例子所示，在每一个路由中，都出现了重复的视图子目录。我们使用 `view_options` 插件来解决这个问题。它使用 `set_view_subdir` 方法来设置视图的子目录。使用这个插件时，我们需要设置 `render` 插件的 `:layout` 选项，记得要包含路径。这里使用 `views/layout.erb` 文件作为公共的布局文件，否则的话，会在每个子目录中寻找布局文件是否存在。



```
class App < Roda
  plugin :render, escape: true, layout: './layout'
  plugin :view_options

  route do |r|
    r.on "tasks" do
      set_view_subdir "tasks"

      r.get true do
        @tasks = Task.all
        view "index"
      end

      r.get Integer do |id|
        next unless @task = Task[id]
        view "task"
      end
    end

    r.on "posts" do
      set_view_subdir "posts"

      r.get true do
        @posts = Post.all
        view "index"
      end

      r.get Integer do |id|
        next unless @post = Post[id]
        view "post"
      end
    end
  end
end
```



程序执行的行为保持不变，但是现在已经不需要指定在每个路由中都明确指定视图的子目录了。`view_options` 插件还具有按分支更改任何视图或布局选项的功能。如果子目录之一中的模板使用与默认使用的模板不同的工程，则可以使用此方法。



#### 减少视图中重复的内容



在布局节中，我们学习了如何将每个页面中公共的内容拆分到布局文件中，以减少重复。然而，在许多复杂的应用中，遇到类似的问题，我们想要复用代码，但不想在多个视图文件中复制。在本节中，我们将学习如何将模板拆分为多个部分。



我们有一个 To-Do 应用，在 root 路由中，展示任务。



![img](.\assets\partials_original.png)



我们注意到，当我们的列表变得很长，必须滚动才能定位到未完成的任务。要解决这个问题，我们可以对列表进行排序，将未完成的任务排在前面。但是，我们不想丢失列表的实际顺序。



于是，我们决定让列表只包含未完成的任务。我们创建一个 `/todo` 路由。为了让例子更简单，我们这里就不创建子目录了。所以我们使用 `todo` 视图，以及设置 `@tasks` 实例变量，视图将可被访问。



```
route do |r|
  r.root do
    @tasks = Task.all
    view "index"
  end

  r.get "todo" do
    @tasks = Task.todo
    view "todo"
  end
end
```



然后我们需要创建 `views/todo.erb` 模板文件。由于这只是该实现的第一步，并且我们已经有了风格化的任务，因此我们决定在列表项中使用相同的代码。我们可以稍后再回来简化代码。



```
h2>Undone Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <li class="<%= task.done? ? :done : :todo %>">
      <input type="checkbox"<%= " checked" if task.done? %>>
      <%= task.title %>
    </li>
  <% end %>
</ul>
```



我们来检查一下是否工作正常。





![img](.\assets\partials_undone_tasks.png)



工作一切正常。要简化视图，我们决定将重复的代码拆分为一个模板文件。我们创建 `views/tasks.erb` 视图，并将处理指定任务的代码拷贝过去。



```
<li class="<%= task.done? ? :done : :todo %>">
  <input type="checkbox"<%= " checked" if task.done? %>>
  <%= task.title %>
</li>
```



现在，我们需要一种方式来渲染视图中的每个任务。幸运的是，我们已经在前面的章节潜移默化的学到了如何来实现，可能没有意识到。我们需要复用 `render` 方法以渲染模板。我们使用 `render` 替代 `view`，因为我们不想在渲染 `views/task.erb` 视图时候引入布局。



接着传递任务变量到视图，这是在循环中进行的，循环的代码块所接受的变量，就是任务变量。在这里我们推荐使用上下文变量而不是实例变量来进行渲染。



```
<h2>Undone Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <%== render("task", locals: { task: task }) %>
  <% end %>
</ul>
```



现在来确认一下是否正常工作。



![img](.\assets\partials_undone_tasks2.png)



#### render_each



为一组对象，渲染相同的模板，是 Web 应用程序很常见的操作，`Roda` 提供 `render_each` 插件，对这个操作提供支持。我们可以将插件加载到我们的应用程序中。



```
class App < Roda
  plugin :render, escape: true
  plugin :render_each

  # ...
end
```



然后，更新 `views/task` 视图来使用它。渲染过程的代码，和手动渲染很类似。然而，通过使用 `render_each` 插件，提供了更好的性能。

```
<h2>Undone Tasks</h2>
<ul>
  <%== render_each(tasks, "task") %>
</ul>
```



在名为 `task.erb` 的模板文件中，`render_each` 方法自动设置本地变量 `task`，在其中保存要渲染的任务。我们可以通过 `:local `选项来指定不同的本地变量。



#### partials



如果我们查看了前面示例的 `views/tasks` 目录，我们可以看到 `task.erb` 和 `todo.erb` 旨在为我们的网站呈现整个页面。在大多数 `Web 框架` 中，通常称为视图。另一方面，`tak.erb` 仅用于呈现页面的一部分，而且可能在一个请求执行上下文中，被执行多次。在 Web 开发中，这种类型的文件，通常被称为片段（partial template 缩写）



`Ruby on Rails` 提供了通过更改文件名，以区分视图和片段的简便方法。如果文件是片段，我们在文件名前面加上下划线。`partials`插件提供了类似 `Rails` 的方式。这是个使用例子。首先，我们需要重命名片段，在前面加上下划线。



```
File.rename('views/task.erb', 'views/_task.erb')
```



然后，我们来更新我们的应用程序，以使用 `partials` 插件。



```
class App < Roda
  plugin :render, escape: true
  plugin :partials

  # ...
end
```



接着，我们更新 `views/todo.erb` 视图，以使用 `partials` 插件。我们可以通过手动来调用 `partial` 方法，来使用。



```
<h2>Undone Tasks</h2>
<ul>
  <% tasks.each do |task| %>
    <%== partial("task", locals: { task: task }) %>
  <% end %>
</ul>
```



或者使用 `each_partial` 方法，来简单调用。两种方式返回的内容一致。

```
<h2>Undone Tasks</h2>
<ul>
  <%== each_partial(tasks, "task") %>
</ul>
```



`partial` 和 `each_partial` 只是在模板文件名称前面，加上下划线，然而进行加载。它们不做任何其他行为。所以，我们可以使用这两个方法来简化片段的加载。



#### symbol_views



`Roda` 的默认行为，是通过匹配代码块返回字符串作为响应内容，或者 `nil`，`false`。但是，这只是默认行为，插件可以对此进行修改。接下来两节，我们将学习如何使用这些插件。



让我们通过一个之前还没介绍视图模板的例子来讨论这一点。我们的代码看来类似这样：



```
class App < Roda
  plugin :render, escape: true

  route do |r|
    r.root do
      @tasks = Task.all
      view "index"
    end

    r.get "todo" do
      @tasks = Task.todo
      view "todo"
    end
  end
end
```



这里有些重复，我们调用了 `view` 方法多次。实际上，在大型路由树种，大多数 `r.get` 都会以调用 `view` 来结尾。要减少这些重复的代码，我们可以通过 `symbol_view` 插件。`symbol_views` 插件允许匹配返回的符号，如果匹配快返回符号，这个符号会被传递到 `view` 方法，并将被用于响应体。所以，上面的例子可以被这样修改：



```
class App < Roda
  plugin :render, escape: true
  plugin :symbol_views

  route do |r|
    r.root do
      @tasks = Task.all
      :index
    end

    r.get "todo" do
      @tasks = Task.todo
      :todo
    end
  end
end
```



`symbol_views` 也支持视图子目录，然而看起来没有那么美观：



```
class App < Roda
  plugin :render, escape: true
  plugin :symbol_views

  route do |r|
    r.root do
      @tasks = Task.all
      :"tasks/index"
    end

    r.get "todo" do
      @tasks = Task.todo
      :"tasks/todo"
    end
  end
end
```



但是大多数使用视图子目录的应用程序，通常使用前面介绍过的 `view_options` 插件，因此很少这样调用。



#### json



`symbol_views` 不是对匹配代码块返回类型进行扩展的唯一插件。`symbol_views` 帮助我们减少代码中用于返回重复的 `HTML` 的代码。`Roda` 提供类似的方式，来减少返回 `JSON` 的代码，使用 `json` 插件。



我们接下来通过一个展示电影上映时间的例子来进行演示。我们构建一个名为 `movies` 的本地变量，在其中保存有关电影信息的数组，这些信息包含电影的上映时间。我们使用两个路由。一个是 `/movies`，列出电影列表。另外一个，`/movies/<slug>`，现实电影的标题，时间，以及指定电影的描述信息。



```
class App < Roda
  movies = [
    {
      slug: "infinity-war",
      title: "Avengers Infinity War",
      times: ["15:30", "18:40", "21:45"],
      description: "The Avengers fight Thanos."
    },
    {
      slug: "the-usual-suspects",
      title: "The Usual Suspects",
      times: ["11:10", "15:45"],
      description: "A random police lineup leads to something deadly."
    },
    {
      slug: "the-matrix",
      title: "The Matrix",
      times: ["17:15", "22:10"],
      description: "Computer hacker finds he lives in a simulation."
    },
  ]

  route do |r|
    r.on "movies" do
      r.get true do
        movies.map do |movie|
          "#{movie[:title]}: /movies/#{movie[:slug]}"
        end.join("\n")
      end

      r.get String do |slug|
        movie = movies.find { |m| m[:slug] == slug }

        <<~EOF
          #{movie.title}
          Times: [ #{movie.times.join(" ")} ]
          Description: #{movie.description}
        EOF
      end
    end
  end
end
```



糟糕的事情总是不期而至，经理决定数据渲染放到前端进行，通过一个名为 `du jour` 的框架。所以，我们的 `Roda` 应用需要切换到返回 `JSON`，然后交给前端进行处理。



我们首先令 `/movies` 路由返回 `JSON`。我们并不打算手工构造 `JSON`，所以，我们将对 `movies` 数组调用 `to_json`，来使其转换成需要的格式。我们同样需要设置 `Content-Type` 响应头，来使浏览器知道我们返回的使 `JSON`。



```
r.get true do
  response['Content-Type'] = 'application/json'

  movies.map do |movie|
    {title: movie[:title], url: "/movies/#{movie[:slug]}"}
  end.to_json
end
```



现在，我们发现返回值已经是 `JSON` 结构了。



```
require "lucid_http"
require "json"

GET "/movies", json: true
# => [{"title"=>"Avengers Infinity War", "url"=>"/movies/infinity-war"},
#     {"title"=>"The Usual Suspects", "url"=>"/movies/the-usual-suspects"},
#     {"title"=>"The Matrix", "url"=>"/movies/the-matrix"}]
```



现在，我们将视线移回 `/movies/<slug>` 路由。我们首先需要找到正确的电影。如果我们不能找到电影，我们使用 `next` 方法返回 `404` 响应。如果我们找到了电影，我们设置 `Content-Type`，然后创建新的包含电影信息的 `hash` 结构，并通过调用 `to_json` 方法来返回 `JSON` 结构。



这是当前应用程序的完整路由。 



```
route do |r|
  r.on "movies" do
    r.get true do
      response['Content-Type'] = 'application/json'

      movies.map do |movie|
        {title: movie[:title], url: "/movies/#{movie[:slug]}"}
      end.to_json
    end

    r.get String do |slug|
      next unless movie = movies.find { |m| m[:slug] == slug }
      response['Content-Type'] = 'application/json'

      {
        title:       movie[:title],
        times:       movie[:times],
        description: movie[:description]
      }.to_json
    end
  end
end
```



我们来检查一下 `/movies/<slug>` 路由是否工作。



```
require "lucid_http"
require "json"

GET "/movies/infinity-war", json: true
# => {"title"=>"Avengers Infinity War",
#     "times"=>["15:30", "18:40", "21:45"],
#     "description"=>
#      "The Avengers fight Thanos."}
```

这种情况有两种情况导致重复。首先，我们需要设置 `Content-Type` 响应头，以便前端将响应作为 `JSON`处理。仅在确定要返回 `JSON` 时才设置 `Content-Type` 标头，而不是在返回空 `404` 响应的情况下才设置。其次，我们需要在要用作 `JSON` 响应主体的哈希或数组上调用 `to_json`。



`Roda` 的 `json` 插件，设计就是为了删除不同路由中，重复的返回内容。我们首先需要引入并加载这个插件。然后匹配代码块可以直接返回数组或者哈希，这些内容会被转换成 `JSON` 结构并作为响应内容返回。`json` 插件也会设置 `Content-Type` 头，所以，我们无需再设置响应头。



在本例中，我们的 `r.get true` 路由返回一个数组，我们的 `r.get String` 路由返回一个哈希。他们都会都被转换为 `JSON`。



```
class App < Roda
  plugin :json

  movies = [
    # ...
  ]

  route do |r|
    r.on "movies" do
      r.get true do
        movies.map do |movie|
          {title: movie[:title], url: "/movies/#{movie[:slug]}"}
        end
      end

      r.get String do |slug|
        next unless movie = movies.find { |m| m[:slug] == slug }

        {
          title:       movie[:title],
          times:       movie[:times],
          description: movie[:description]
        }
      end
    end
  end
end
```



我们再次检测输出，确保一切正常工作。



```
require "lucid_http"
require "json"

GET "/movies", json: true
# => [{"title"=>"Avengers Infinity War", "url"=>"/movies/infinity-war"},
#     {"title"=>"The Usual Suspects", "url"=>"/movies/the-usual-suspects"},
#     {"title"=>"The Matrix", "url"=>"/movies/the-matrix"}]

GET "/movies/infinity-war", json: true
# => {"title"=>"Avengers Infinity War",
#     "times"=>["15:30", "18:40", "21:45"],
#     "description"=>
#      "The Avengers fight Thanos."}
```



希望 `symbol_views` 和 `json` 插件可以展示给你 `Roda` 的默认行为，是如何通过插件进行改变和扩展的，这些插件用起来也很简单。



#### Assets



到目前为止，我们把焦点放在了渲染视图上，以及为请求构造响应。我们将扩大讨论，在视图中使用的 CSS 及 JavaScript 文件。我们现在的例子，是在布局文件中内联了 CSS。通常来说，我们并不这样做。我们会把样式文件拆分，然后进行引用。JavaScript 文件也是如此。



#### 静态资源



使用样式文件和 JavaScript 最简单的方式静态引用。通常来说，静态资源文件放在 `public` 目录中，然后在 Web 服务器中进行配置，直接访问，或者使用 `public` 插件来完成这些操作。在开发过程中，我们可能想使用 `public` 插件，接下来让我们在例子中进行添加。



让我们创建目录结构，以及资源的空文件。



```
Dir.mkdir("public/css")
Dir.mkdir("public/js")
File.write("public/css/app.css", "")
File.write("public/js/app.js", "")
```



我们打开 `public/css/app.css` 文件，从布局文件中拆分 CSS 代码到其中，此时格式化一下会看起来更舒服。



```
ul {
  list-style: none;
}
ul .todo {
  color: red;
}
ul .done {
  color: green;
}
```



然后打开 `public/js/app.js` 文件，添加 checkbox CSS 类发生变化时响应的 javascript 代码。



```
(function() {
  Array.from(document.getElementsByTagName("input")).
    forEach(function(element) {
      element.onchange = function() {
          element.parentNode.classList.toggle("done");
          element.parentNode.classList.toggle("todo");
        }
    });
})();
```



我们需要更新路由树，以响应对 `public` 目录的请求。



```
class App < Roda
  plugin :render, escape: true
  plugin :symbol_views
  plugin :public

  route do |r|
    r.public

    r.root do
      @tasks = Task.all
      :index
    end

    r.get "todo" do
      @tasks = Task.todo
      :todo
    end
  end
end
```



然后编辑 `views/layout.erb` 文件，链接需要的 CSS 和 JS 文件。

```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <link rel="stylesheet"  href="/css/app.css" />
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%== yield %>
    <script type="text/javascript" src="/js/app.js"></script>
  </body>
</html>
```



现在页面正常显示，如之前使用内联样式时一样。当变更 checkbox 样式时，表现得更好。



使用静态资源，可以使我们来组织更复杂得项目。如果我们足够万股，我们甚至可以在大型网站中使用静态资源。然而，静态资源存在一些问题：



- 缺乏对资源编译得支持。我们只能直接使用 CSS 和 JavaScript，而无法使用可编译为 JavaScript 和 CSS 得语言，而使 Web 开发更加轻松。对于 JavaScript，这还不是个主要的问题，但是对于任何具有中等复杂样式需求的网站，使用 SCSS 替代 CSS 是一个巨大的进步。

- 缺乏对合并资源的支持。如果我们在多个 CSS 或者 JavaScript 文件中组织我们的代码，浏览器需要请求多次。合并资源可以提高性能，因为他减少带宽并减少延迟。

- 缺乏压缩资源的支持。通常，我们为了性能，在生产环境，只提供压缩过的资源以减少传输带宽。但是我们不会在开发环境中使用压缩过的资源，这会使调试变得复杂。



庆幸的是，Roda 通过 `assets` 插件来解决这些我们关心得问题。



### 介绍 assets



`assets` 插件用来编译资源，以及在生产环境中合并和压缩资源。`Roda` 尝试让设置资源尽可能的简单。默认情况下，`Roda` 认为资源存储在 `assets` 目录中，`assets/css` 存储 CSS 文件（以及其他可被编译为 CSS 的文件），`assets/js` 存储 javascript 文件（或者可被编译为 javascript 的文件）。我们首先创建 `assets` 目录，然后将我们的静态资源移动进去。



```
Dir.mkdir("assets")
Dir.mkdir("assets/css")
Dir.mkdir("assets/js")
File.rename("public/css/app.css", "assets/css/app.css")
File.rename("public/js/app.js", "assets/js/app.js")
```



然后我们可以更新我们的路由，以对 `assets` 提供响应。需要注意的是，我们只是把 `public` 插件替换为 `assets` 插件，使用 `r.assets` 替代 `r.public` 在路由树中。



```
class App < Roda
  plugin :render, escape: true
  plugin :symbol_views
  plugin :assets, css: ["app.css"], js: ["app.js"]

  route do |r|
    r.assets

    r.root do
      @tasks = Task.all
      :index
    end

    r.get "todo" do
      @tasks = Task.todo
      :todo
    end
  end
end
```



我们接着编辑 `views/layout.erb` 文件，调用 `assets` 方法，替换掉硬编码的链接。`assets` 方法返回的内容，已经过滤了 `HTML` 标签（或者多个标签），所以只需要直接使用 `yield`，其中插入的内容已经是过滤后的。我们在这里调用 `assets`两次，以提供对 `CSS` 和 `JavaScript` 文件的资源插入。

```
<html>
  <head>
    <title><%= @page_title || "To-Do or not To-Do" %></title>
    <%== assets(:css) %>
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    <%== yield %>
    <%== assets(:js) %>
  </body>
</html>
```



现在一切可以正常工作，页面显示如之前一样，而已经改用 `assets` 插件。



### 资源编译



如前面的例子所示，`assets` 插件只提供静态文件的服务，而不做其他处理。它比 `public` 插件的处理速度要慢一些，似乎没有什么优势。让我们实际使用一下 `assets` 插件，以便了解其优点。



首先，我们将 `assets.css` 文件改为 `assets.scss` 文件。将 `CSS` 文件改为 `SCSS` 文件。



```
File.rename("public/css/app.css", "assets/css/app.scss")
```



然后，我们更新 `assets` 插件选项，将文件名的变化修改进去。



```
plugin :assets, css: ["app.scss"], js: ["app.js"]
```



将 `SCSS` 格式的文件转换为 `CSS` 格式，需要引入 `sassc` gem（或者已经弃用的 `sass` gem），所以让我们将 `sassc` gem 添加到 `Gemfile`，然后运行 `bundle install`。



```
source "https://rubygems.org"

gem "roda"
gem "puma"
gem "tilt"
gem "erubi"
gem "sassc"
```



安装完 `sassc` gem 后，我们来检查一下是否一些正常。再次访问，可能需要一点额外的时间，因为格式转换需要时间。现在已经可以使用 `SCSS` 的特性了，我们来试试。编辑 `assets/css/app.scss` 文件，修改它使用 `SCSS` 的语法。这对于 CSS 来说，是错误的语法，CSS 不支持嵌套样式标签。然而 SCSS 可以支持类似的标签。



```
ul {
  & {
    list-style: none;
  }
  .todo {
    color: red;
  }
  .done {
    color: green;
  }
}
```



再次刷新页面，会发现一切和之前工作的一样。如果我们请求 `/assets/css/app.scss` 文件，我们可以看到 `SCSS` 文件已经被编译为 `CSS`，内容以及 `Content-Type` 也是正确的。



```
require "lucid_http"

GET "/assets/css/app.scss"
status                          # => 200 OK
content_type                    # => "text/css; charset=UTF-8"
body
# >> ul {
# >>   list-style: none; }
# >> ul .todo {
# >>   color: red; }
# >> ul .done {
# >>   color: green; }
```



SCSS 不只支持标签嵌套，还支持变量，mixins，继承和一些计算操作。任何合法的 CSS 文件，都可以通过简单的把扩展名修改为 `SCSS` 来使用 SCSS 的特性。



除了支持编译 `CSS`，`assets` 插件也支持编译 `javascript`。`assets` 也提供对模板的支持，通过 `tilt`  gem，这个 `gem` 依赖 `render` 插件。对于 CSS，可以支持 `sass` 和 `less`。对于 JavaScript，支持包括 `coffee`，`ts`，`babel` 以及`rb`（Opal，允许我们使用 Ruby 编写前端代码）。



#### 资源合并



在前面的章节，我们讨论了 `assets` 插件提供的一项改进，将其他格式编译为 CSS 或者 JavaScript。现在，我们将看看该插件提供的另一项特性，将 CSS 和 JavaScript 进行合并。



我们将通过多个 CSS 文件和 JavaScript 文件来举例。对于 CSS，除了我们先前使用的 `app.scss` 文件之外，我们还将假设我们包含了 `Bootstrap CSS` 库的修改版本。对于 JavaScript，我们假设要添加一个 `TypeScript` 文件，该文件将处理任务页面之上的动态页面操作。需要注意的是，我们使用了两种不同的格式来转换为 `CSS` 和 `JavaScript`。



```
plugin :assets,
  css: ["bootstrap.css", "app.scss"],
  js: ["app.js", "tasks.ts"]
```



如果我们预览页面内容，我们将看到调用 `assets(:css)` 和 `assets(:js)` 的返回结果生成 `link` 和 `script` 分别生成标签。我们可以据此推断出 `assets` 插件默认不会合并资源。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
    <link rel="stylesheet"  href="/assets/css/bootstrap.css" />
    <link rel="stylesheet"  href="/assets/css/app.scss" />
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    # ...
    <script type="text/javascript" src="/assets/js/app.js"></script>
    <script type="text/javascript" src="/assets/js/tasks.ts"></script>
  </body>
</html>
```



让我们查看资源编译部分，哪些地方发生了改变。在加载资源插件后，我们将调用 `complie_assets` 方法。



```
plugin :assets,
  css: ["bootstrap.css", "app.scss"],
  js: ["app.js", "tasks.ts"]
compile_assets
```



当我们调用 `complie_assets` 时，Roda 将转换所有的 `CSS` 资源，并将他们合并在一起。`Roda` 也将转换 `javascript` 资源为`javascript` 文件，以及进行合并。做了这些修改后，如果我们再次请求这个页面，我们可以发现内容已经改变，此时，存在单个的 `link` 和 `script` 标签。在此示例中，字符串被中间截断。



```
<html>
  <head>
    <title>To-Do or not To-Do</title>
    <link rel="stylesheet"
      integrity="sha256-5uh7i+PDFHCJmpP9ef8B8TTTS8x7A2jeM/IvQH9Togs="
      href="/assets/app.e6e87b8be3c31470899a93fd79ff01f134d.css" />
  </head>
  <body>
    <h1>To-Do or not To-Do</h1>
    # ...
    <script type="text/javascript"
      integrity="sha256-M1Y2mm2BRge3II4oUCyNKCQDthqZ3YTCQlfGTKP7bW8="
      src="/assets/app.3356369a6d814607b7208e28502c8d282403.js">
    </script>
  </body>
</html>
```



在 `link` 标签 `href` 属性和 `script` 标签 `src` 属性中，我们看到了用于资源文件路径的 `/assets/app.*.css` 和 `/assets/app.*.js`。字符串中间的内容是什么呢？`assets` 插件使用 `SHA-256` 对内容进行哈希，然后将哈希字符串插入到资源的文件中去。这保证了在任意资源文件被修改后，文件名被更新，任何时候都能引用正确的资源文件，HTTP 客户端不使用旧的文件缓存。



每个 tag 都有的 `integrity` 属性的内容也是哈希，但是是进行 `base64` 加密过的。`integrity` 属性允许我们无法控制的其他服务器上，并确保仅在未修改的情况下才加载。



#### 资源预编译



`compile_assets` 对生产环境很重要。然而，我们不想在开发环境中使用，因为一旦调用 `compile_assets`，插件将仅提供合并的资源文件，而不会获取对资源文件的更改。通过使用条件控制，我们可以很容易跳过资源合并。



```
plugin :assets,
  css: ["bootstrap.css", "app.scss"],
  js: ["app.js", "tasks.ts"]
compile_assets unless ENV["RACK_ENV"] == "development"
```



但是，在某些情况下，需要在程序启动之前调用 `compile_assets`。一个原因是在只读文件系统上运行。`assets` 可以使用被称为资源预编译的进程来处理。通过资源预编译，可以在应用程序启动之前对资源进行编译，并将元数据保存在 `JSON` 文件中。当应用程序启动时，它会检查 JSON 文件是否存在。如果存在，则启用，如果不存在则在默认模式下运行，并对每个资源文件产生单独的链接和脚本标签。



使用资源预编译，我们需要在加载 `assets` 插件的时候，带上 `:precompiled` 选项，以及预编译的 `metadata` 文件。另外，我们需要删除对 `compile_assets` 方法的调用。



```
plugin :assets,
  css: ["bootstrap.css", "app.scss"],
  js: ["app.js", "tasks.ts"],
  precompiled: File.expand_path('../compiled_assets.json', __FILE__)
```



预编译资源，通常使用 `rake` 任务。`rake` 任务也可以部署到 Heroku，进行预编译资源。

 

```
namespace :assets do
  desc "Precompile the assets"
  task :precompile do
    require './app'
    App.compile_assets
  end
end
```



当调用 `compile_assets`时，如果 `:precompiled` 选项传递给了 `assets` 插件，`metadata` 相关的资源将被写入到 JSON 格式的文件，所以，下次应用启动的时候，`assets` 插件将工作在 `compiled` 模式。