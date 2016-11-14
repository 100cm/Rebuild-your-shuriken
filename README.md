#打造你自己的Rails


*捡起你的知识并且把他们拼凑成你自己的rails吧！


##第一章 

###### 为什么要打造自己的rails
&nbsp;&nbsp;了解一个框架越深，就越能让你掌握它，并且使用它，如果你能够自己打造一个自己的‘rails’，岂不是一件非常值得去做的事情吗？
&nbsp;&nbsp; 当然,RAILS 是一个非常顽固的框架，甚至他的制作团队也是这么说的。（约束>配置）但是现在你有机会自己制作一个你认定的约束框架，并且在这基础之上能够添加一些小插件。

###### 谁适合看这份文档（书籍）
&nbsp;&nbsp;如果你觉得自己能够熟练使用rails，但是对内部一些机制不大了解，阅读本分文档会对你有一些很大的帮助。

#### 开始吧！

###### 以下环境(mac os)是你应该搭建好的
- Ruby 2.0或者更高版本
- 一个编辑器（ruby mine ／ atom）
- 一个终端工具（iterm2）

以上工具如在其他环境 可以找合适的替代品。在此不多做赘述。

#### 让他开始工作
 想一个名字是很难的，暂时就以Shuriken 作为我们的"框架"的名字
首先在你的终端创建一个gem

```ruby
bundle gem Shuriken
create  rulers/Gemfile
          create  shuriken/Rakefile
          create  shuriken/LICENSE.txt
          create  shuriken/README.md
          create  shuriken/.gitignore
          create  shuriken/shuriken.gemspec
          create  shuriken/lib/shuriken.rb
          create  shuriken/lib/shuriken/shuriken.rb
    Initializating git repo in src/rulers
```
其实rails 也是一个gem，我们创建了一个叫Shuriken（手里剑）的gem。并且目前的依赖都在shuriken.gemspec 中

```ruby
#shuriken.gemspec
	lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'Shuriken/version'

Gem::Specification.new do |spec|
  spec.name          = "Shuriken"
  spec.version       = Shuriken::VERSION
  spec.authors       = ["icepoint0"]
  spec.email         = ["351711778@qq.com"]

  spec.summary       = "RUBY FRAME WORK"
  spec.description   = "Gay BING"
  spec.homepage      = "http://thunderjava.com"
  spec.license       = "MIT"

  # Prevent pushing this gem to RubyGems.org by setting 'allowed_push_host', or
  # delete this section to allow pushing this gem to any host.
  if spec.respond_to?(:metadata)
    spec.metadata['allowed_push_host'] = "TODO: Set to 'http://github.com/icepoint0'"
  else
    raise "RubyGems 2.0 or newer is required to protect against public gem pushes."
  end

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^bin/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]
  spec.add_development_dependency "bundler", "~> 1.11"
  spec.add_dependency "rake", "~> 10.0"
  spec.add_development_dependency "rspec", "~> 3.0"
  spec.add_dependency "rack"
  spec.add_dependency "erubis"
```

并且我们加入了 rack,  erubis 等依赖。这个后续会提到作用。


####创建我们的应用项目
&nbsp;&nbsp;现在我们的gem已经创建好了。接下来就是创建我们的应用项目来使用这个gem。

在终端创建一个文件夹
```ruby
> mkdir shuriken-demo
> cd shuriken-demo
> mkdir config
> mkdir app
> touch Gemfile #创建gemfile
```

在你的Gemfile里加上我吗现在要用的这个gem

```ruby
#shuriken-demo/Gemfile
source 'https://rubygems.org'
gem 'Shuriken', path: '~code/Shuriken'
```
gem之后的path为 本地gem路径 由于还在开发阶段 所以暂不上传到你的git ／ ruby gems网站

然后执行bundle

```ruby
> cd shuriken-demo
> bundle
```

学过java的同学都知道。java 有servelt , 那么ruby有什么呢？对了。是rack。

我们现在暂时要打造的是一个很小的基于rack的框架 因此少不了rack。

在shuriken-demo 里创建config.ru 并且添加如下内容

```ruby
#shuriken-demo/config.ru
run proc{
[200,{'Content-Type'=>'text/html'},
["Hello,Shuriken"]]
}
```

Rack 的启动代表着 对于每个请求执行这个脚本，在这个例子里所有的请求都返回hello shuriken 的200 返回html

在 **shuriken-demo** 项目目录下 启动他
```ruby
rackup -p 3001
```
打开你的浏览器 输入 **localhost:3001** 就可以看到 hello shuriken的字样了！


### 为你的Rack 请求加上规则

&nbsp;&nbsp;进入你的Shuriken 路径下 打开lib/Shuriken.rb

```ruby
# Shuriken/lib/Shuriken.rb
	require 'Shuriken/version'
	module Shuriken
	  class Application
	    def call(env)
		     [200, {'Content-Type' => 'text/html'},
            ["Hello from Ruby on Shuriken!"]
         end
       end
     end    
```

然后在你的shuriken-demo 项目里 创建 config/application.rb 并且添加

```ruby
require 'Shuriken'
	module ShurikenDemo 
	   class Application < Shuriken::Application
	   end
	end
```
  
然后修改你的 conifg.ru

```ruby
#shuriken-demo/config.ru
require './config/application'
run ShurikenDemo::Application.new
```

然后执行 **rack up -p 3001** 就可以看到"Hello from ruby on shuriken"的字样了----第一章结束 -2016.11.14

##第二章

#### 你的第一个控制器

首先，我们梳理一下思路，路由从浏览器进入。到我们的rack run env里。这些路由带着参数。带着特有的path_info . 所以我们首先要拿到path_info 并且去进行解析。然后去和我们的action 以及controller 进行匹配。此外 ，我们还要能确保controller能够自动加载。不用手动去require。还要能够渲染页面模板。而不是单纯的渲染html 字符串。并且能够像rails那样去使用action里
的实例变量。让我们一步一步来吧。

#####rails里的自动加载

首先我们要理解自动加载，通常我们用const_get() 去获取contstant。但是如果没有require 进来则会抛出异常。现在我们在
**lib/Shuriken/loader** 里加入如下代码

```ruby
class Object
  def self.const_missing(c)
    require Shuriken.to_underscore(c.to_s)
    Object.const_get(c)
  end
end
```
然后在我们的 **lib/Shuriken.rb** 里require进去

```
# lib/Shuriken.rb
require 'Shuriken/loader'
```

然后我们现在要根据路由去执行对应的controller以及action

还是修改这个文件

```ruby
#lib/Shuriken.rb
module Rulers
      class Application
        def call(env)
          klass, act = get_controller_and_action(env)
          controller = klass.new(env)
          text = controller.send(act)
          [200, {'Content-Type' => 'text/html'},
              [text]]

         end
       end
 end
```
然后还是在Application class 里添加get_controller_and_action 这个方法

```ruby

 def get_controller_and_action(env)
     _, cont, action, after =
      env["PATH_INFO"].split('/', 4)
     cont = cont.capitalize # "User"
     cont += "Controller" # "UserController"
     [Object.const_get(cont), action]
 end
        
```

我们需要一个基础controller 来让我们的应用进行继承，所以创建 **lib/Shuriken/controller.rb** 并且在**lib/Shuriken.rb** require进去

```ruby
#lib/Shuriken/controller.rb
	module Shuirken
	 class Controller
	   def initialize(env)
	   end
	 end
	end	
```

打开我们的shuriken-demo 创建我们的控制器 **app/controllers/user_controller

```ruby
class UserController < Shuriken::Controller
  def hello
  'hello'
  end
end
```
现在我们的控制器加好了 自动加载的代码也写好了。那么怎么像rails那样自动加载这个controller呢。你需要在**config/application.rb** 下加这些内容

```ruby
#config/application.rb
#在 require 'Shuirken'之后添加以下代码
  $LOAD_PATH << File.join(File.dirname(__FILE__),
"..", "app","controllers")
```

然后执行 **rackup -p 3001** 

打开你的浏览器 输入 "localhost:3001/user/hello"
你会发现一个错误:

```
NameError: wrong constant name
    Favicon.icoController
      .../gems/rulers-0.0.3/lib/rulers/routing.rb:9:in
    `const_get'
      .../gems/rulers-0.0.3/lib/rulers/routing.rb:9:in
    `get_controller_and_action'
      .../gems/rulers-0.0.3/lib/rulers.rb:7:in `call'
      .../gems/rack-1.4.1/lib/rack/lint.rb:48:in
    `_call'
      (...more lines...)

```
是的。我们有一个站点图标请求忘记了。马上补上：
打开Shuriken项目 里的 lib/Shuriken.rb

```ruby
 # rulers/lib/Shuriken.rb
    module Rulers
      class Application
        def call(env)
          if env['PATH_INFO'] == '/favicon.ico'
            return [404,
              {'Content-Type' => 'text/html'}, []]
          end
          klass, act = get_controller_and_action(env)
          controller = klass.new(env)
          text = controller.send(act)
          [200, {'Content-Type' => 'text/html'},[text]]
        end
     end
   end
```
也许你会觉得这是一个糟糕的设定。不过最后我们会修复它。至少现在让他可以先工作起来。


##第三章

######渲染模板

我们现在成功的执行了我们的路由方法。并且成功的返回了我们执行的方法的结果字符串。现在我们要渲染我们的页面模板

###Erb 和 Erubis

rails 中使用的默认页面模板就是erb 对应的模板渲染引擎为erubis,回顾下之前我们已经在我们的shuriken.gemspec中加入了erubis 依赖 ，忘记添加的同学可以回第一章回顾一下。

######现在开始！
打开你的lib/Shuriken/controller.rb
它看上去应该是这样的

```ruby
    module Shuriken
      class Controller
      
        def initialize(env)
            @env = env 
        end
        
        def env 
           @env
        end
      end
   end
```
我们在文件顶部require 引擎 ： "require 'erubis'" 并且添加一个简单的方法

```ruby
    def render_view(view_name, locals = {})
      # 渲染模板
      html_filename = File.join "app", "views", controller_name,
                                "#{view_name.to_s}.html.erb"
      erb_filename = File.join "app", "views", controller_name, "#{view_name.to_s}.erb"

      #erb ／ html 渲染
      begin
        template = File.read html_filename
      rescue
        template = File.read erb_filename
      end
      v = View.new
      v.set_vars locals
      v.evaluate template
    end
    
     #渲染对应的controller下面的view
    def controller_name
      klass = self.class
      klass = klass.to_s.gsub /Controller$/, ""
      Shuriken.to_underscore klass
    end

```

我们定义了一个render_view 方法去渲染我们的view 并且把参数进行传递到页面模板中使用，默认渲染师对应controller下的action 名字的模板。


#### Request && Response

同样的 在lib/Shuriken/controller.rb 下加入以下方法

```ruby 
def request
  @request ||= Rack::Request.new(@env)
end

 def params
   request.params
 end
```

以上代码是确保我们可以在controller 中使用 params[:xx] 获取我们的请求参数。

然后我们添加response

```ruby
 def response(text, status = 200, headers = {})
      raise "already responsed" if @response
      res = [text].flatten
      @response = Rack::Response.new(res, status,
                                     headers)
 end

 def get_response
      @response
 end
```

然后再添加我们的render 方法 进行渲染response

```ruby
def render(view_name, locals={})
      response(render_view(view_name, locals))
end

```

到此为止 我们已经封装了rack 的request 和response的用法 现在让我们尝试一下是否可以正确渲染

打开我们的 **lib/Shuriken.rb**

```ruby
#修改以下部分代码
 class Application
        def call(env) 
          if env['PATH_INFO'] == '/favicon.ico'
            return [404,
              {'Content-Type' => 'text/html'}, []]
          end
          klass, act = get_controller_and_action(env)
          controller = klass.new(env)
          text = controller.send(act)
          r = controller.get_response
         if r
           [r.status, r.headers, [r.body].flatten]
         else
            [200,{'Content-Type' => 'text/html'}, [text]]
         end 
        end
  end
```

然后在我们的UserController  中尝试一波：

```ruby
class UserController < Shuriken::Controller
  def hello
   render 'hello'
  end
end
```
如果成功返回了 user/hello.html.erb 中的内容。ok。成了！

### 在页面模板中使用实例变量

怎么在我们的页面模板中使用实例变量呢。之前我们可以观察到，我们可以传递参数到我们的locals 变量并且进行渲染

就像这样

hello html中

```html
<%= hello %>
```

user controller 中

```ruby
def hello
render 'hello',hello: "你好"
end
```

如果 localhost:3001/user/hello 能够成功返回 页面模板中的你好，就表示我们的代码没问题

接下来我们要使用我们的实例变量,这样就不用繁琐的进行方法传递


还是打开 lib/Shuriken/controller.rb

```ruby
    def instance_vars
      vars = {}
      instance_variables.each do |i|
        vars[i] = instance_variable_get i
      end
      vars
    end
```

然后在我们的render_view方法中添加一行，把我们获取到的实例变量merge进去

```ruby
    def render_view(view_name, locals = {})
      # 渲染模板
      html_filename = File.join "app", "views", controller_name,
                                "#{view_name.to_s}.html.erb"
      erb_filename = File.join "app", "views", controller_name, "#{view_name.to_s}.erb"

      #erb ／ html 渲染
      begin
        template = File.read html_filename
      rescue
        template = File.read erb_filename
      end
      v = View.new
      #这里是最新的添加的一行，用来合并实例变量
      locals.merge! instance_vars
      v.set_vars locals
      v.evaluate template
    end
```

现在就可以在我们的页面模板中愉快的使用实例变量咯！
 

## 第四章

### 在本章我们要做一个对路由分发请求的简单处理。

以下内容请到[我的github查看源码](http://github.com/icepoint0/Shuriken)

或有时间在整理出来。


参考书籍[rebuilding rails](http://rebuilding-rails.com/) 
这份文档算半翻译的一份文档
















