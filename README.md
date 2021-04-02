# Go配置管理包
使用Go语言构建配置管理库，解决配置的注册、加载解析与使用问题。
## 背景描述（Background）

关于配置的管理，有一些通用的功能是很多项目都需要的，为了保证项目技术栈的统一，本项目组设计开发配置管理通用包，意在使配置能够便于管理，方便管理者进行注册、解析以及修改。目前Go语言没有对配置进行系统管理的标准库，因此本项目组决定参照OpenStack Oslo Config，Uber Go Configuration，spf13's Viper等目前市面上现存的配置管理库，实现属于自己的配置管理库，达到对配置项便捷管理的效果。</br>
## 方案描述（Solution Description）
本项目组意在使用Go语言进行配置管理库的开发。</br>
对于配置文件的注册，本项目组借鉴viper库进行进行配置的注册。</br>
使用标准库解析json格式的配置文件，使用gopkg.in/yaml.v2库进行yaml格式配置文件的加载与解析。</br>
设计简易的API接口风格进行配置的使用，每一个接口含义方便直接，传递值较少，便于用户管理使用。</br>
拥有完善的使用文档，其中包含其中将包含：1. 如何加载（初始化）配置；2. 如何注册配置项（配置组）；3. 如何获取配置项的值。使用户操作方便，各类接口用法一目了然，增强用户的使用满意度。

## 方案特点（Solution Features）

本方案开发出的配置管理库具有以下特点：

- 支持加载和解析 JSON、YAML 格式配置文件
- 支持默认配置值
- 拥有完善的使用文档

选用功能：

- 支持使用弃用配置项时，输出告警信息
- 支持从 Shell 环境变量读取配置项
- 支持从命令行参数读取配置项
- 支持 INI 格式配置文件


## 使用场景（Use Cases）

用户在进行文件配置时需要使用本项目配置管理通用包。

## 竞品分析（Alternatives）

本项目组主要和目前现有的三类配置管理库 OpenStack Oslo Config，Uber Go，Configuration，spf13's Viper进行对比。
</br>
## **spf13's viper</br>** ##
## 1. 配置如何进行注册 ##
<pre>
viper.RegisterAlias ( "loud" , "Verbose" )
 
viper.Set ( "verbose" , true ) / / same result as next line
viper.Set ( "loud" , true )    / / same result as prior line
 
viper.GetBool ( "loud" ) / / true
viper.GetBool ( "verbose" ) / / true
</pre>
## 2.如何加载解析json、yaml配置文件 ##
<pre>
viper.SetConfigName ( "config" ) //指定配置文件的文件名称 (不需要制定配置文件的扩展名 )
// viper.AddConfigPath ( "/etc/appname/" )    //设置配置文件的搜索目录
// viper.AddConfigPath ( "$HOME/.appname" )    // 设置配置文件的搜索目录
viper.AddConfigPath ( "." )      // 设置配置文件和可执行二进制文件在用一个目录
err : = viper.ReadInConfig ( ) // 根据以上配置读取加载配置文件
if err != nil {
log.Fatal ( err ) // 读取配置文件失败致命错误
}
</pre>
## 3.配置如何使用</br> ##
3.1设置配置项的默认值
<pre>
viper.SetDefault ( "ContentDir" , "content" )
viper.SetDefault ( "LayoutDir" , "layouts" )
viper.SetDefault ( "Taxonomies" , map [ string ] string { "tag" : "tags" , "category" : "categories" } )
</pre>
3.2修改配置项的值
<pre>
viper.Set ( "Verbose" , true )
viper.Set ( "LogFile" , LogFile ) / /命令行 flag
</pre>
3.3和环境变量一起使用
<pre>
AutomaticEnv() 
//当AutomaticEnv被调用时，任何viper.Get请求都会去获取环境变量.环境变量名为SetEnvPrefix设置的前缀
BindEnv(string…) : error 
//BindEnv需要一个或两个参数.第一个参数是键名，第二个参数是环境变量的名称
SetEnvPrefix(string)  
//为环境变量添加前缀，确保环境变量唯一
SetEnvKeyReplacer(string…) *strings.Replacer
//允许你使用一个strings.Replacer对象来将配置名重写为Env名
</pre>
## 总结 ##
Viper具有良好的API，方便扩展，不会妨碍正常代码的使用。并且Viper也可以处理很多种文件类型作为配置源，例如JSON, YAML,TOML等普通属性文件。另外Viper可以为我们从OS读取环境变量。一旦初始化并产生后，我们的配置总是可以使用各种的viper.Get函数来获取并使用，使用起来十分方便。

# Uber Go/Configuration #
## 1. 配置如何进行注册 ##
<pre>
func NewYAML(options ...YAMLOption) (*YAML, error) {
	cfg := &config{
		strict: true,
		name:   "YAML",
	}
	for _, o := range options {
		o.apply(cfg)
	}

	if cfg.err != nil {
		return nil, fmt.Errorf("error applying options: %v", cfg.err)
	}
	sourceBytes := make([][]byte, len(cfg.sources))
	for i := range cfg.sources {
		s := cfg.sources[i]
		if !s.raw {
			sourceBytes[i] = s.bytes
			continue
		}
		sourceBytes[i] = escapeVariables(s.bytes)
	}
	merged, err := merge.YAML(sourceBytes, cfg.strict)
	if err != nil {
		return nil, fmt.Errorf("couldn't merge YAML sources: %v", err)
	}
}
</pre>
## 2.如何加载解析配置文件 ##
<pre>
provider, err := NewYAML(Source(strings.NewReader("scalar: foo")))
require.NoError(t, err, "couldn't create provider")
</pre>
## 3.如何使用配置 ##
<pre>
assert.Equal(t, provider.Name(), v.Source(), "value source should be provider name")
assert.Equal(t, "foo", v.String(), "unexpected fmt.Stringer implementation")
assert.Equal(t, "foo", v.Value(), "unexpected Value")
assert.True(t, v.HasValue(), "unexpected HasValue")
assert.Equal(t, "foo", v.Get(Root).Value(), "unexpected Value after Get")
</pre>
## 总结 ##
Uber Go/Configuration只支持yaml配置文件的操作，涉及到的配置文件范围较为单一，无法具备较强的兼容性。并且其接口使用较为复杂，不够明了直接。
# OpenStack Oslo Config #
OpenStack的oslo项目旨在独立出系统中可重用的基础功能，oslo.config就是其中一个被广泛使用的库，该项工作的主要目的就是解析OpenStack中命令行（CLI）或配置文件（.conf）中的配置信息。
## 1. 配置如何进行注册 ##
- step1. 正确配置各个服务主配置文件（*.conf文件），本步骤在各个服务（如：keystone）中完成。
- step2. 在要使用到配置信息的模块中声明将用到的那些配置项的模式，包括配置项的名称、数据类型、默认值和说明等；
- step3. 创建一个对象，创建该对象的类充当配置管理器，这个对象作为容器以后将存储配置项的值。
- step4. 调用step3创建的对象中相应的注册方法（如：register_opt()），注册step2中声明的配置项模式。这个过程不会解析配置文件，只是为step3中创建的对象开辟相应的字段。
## 2.注册并加载解析文件实例 ##
先设置my.conf文件，在oslo.config语境下，[DEFAULT]字段不可省略。
<pre>
#-*-coding:utf-8-*-
# my.conf
 
[DEFAULT]
#[DEFAULT]不可省略
enabled_apis = ec2, osapi_keystone, osapi_compute
bind_host = 196.168.1.111
bind_port = 9999
 
[rabbit]
host = 127.0.0.1
port = 12345
use_ssl=true
user_id = guest
password = guest
</pre>
接着写一个脚本文件config.py，该脚本的功能非常简单，直接执行时打印该脚本使用到的配置项的值。
<pre>
#-*-coding:utf-8-*-
# config.py
# Author: D. Wang
 
from oslo.config import cfg
# 声明配置项模式
# 单个配置项模式
enabled_apis_opt = cfg.ListOpt('enabled_apis',
                                   default=['ec2', 'osapi_compute'],
                                   help='List of APIs to enable by default.')
# 多个配置项组成一个模式
common_opts = [
        cfg.StrOpt('bind_host',
                   default='0.0.0.0',
                   help='IP address to listen on.'),
                
        cfg.IntOpt('bind_port',
                   default=9292,
                   help='Port number to listen on.')
    ]
# 配置组
rabbit_group = cfg.OptGroup(
    name='rabbit',
    title='RabbitMQ options'
)
# 配置组中的模式，通常以配置组的名称为前缀（非必须）
rabbit_ssl_opt = cfg.BoolOpt('use_ssl',
                             default=False,
                             help='use ssl for connection')
# 配置组中的多配置项模式
rabbit_Opts = [
    cfg.StrOpt('host',
                  default='localhost',
                  help='IP/hostname to listen on.'),
    cfg.IntOpt('port',
                 default=5672,
                 help='Port number to listen on.')
]
 
# 创建对象CONF，用来充当容器
CONF = cfg.CONF
# 注册单个配置项模式
CONF.register_opt(enabled_apis_opt)
 
# 注册含有多个配置项的模式
CONF.register_opts(common_opts)
 
# 配置组必须在其组件被注册前注册！
CONF.register_group(rabbit_group)
 
# 注册配置组中含有多个配置项的模式，必须指明配置组
CONF.register_opts(rabbit_Opts, rabbit_group)
 
# 注册配置组中的单配置项模式，指明配置组
CONF.register_opt(rabbit_ssl_opt, rabbit_group)
 
# 接下来打印使用配置项的值
if __name__ =="__main__":
# 调用容器对象，传入要解析的文件（可以多个）
　　CONF(default_config_files=['my.conf'])
     
    for i in CONF.enabled_apis:
        print ("DEFAULT.enabled_apis: " + i)
     
    print("DEFAULT.bind_host: " + CONF.bind_host)
    print ("DEFAULT.bind_port: " + str(CONF.bind_port))
    print("rabbit.use_ssl: "+ str(CONF.rabbit.use_ssl))
    print("rabbit.host: " + CONF.rabbit.host)
    print("rabbit.port: " + str(CONF.rabbit.port))
</pre>
执行config.py，输出结果如下所示：
<pre>
DEFAULT.enabled_apis: ec2
DEFAULT.enabled_apis: osapi_keystone
DEFAULT.enabled_apis: osapi_compute
DEFAULT.bind_host: 196.168.1.111
DEFAULT.bind_port: 9999
rabbit.use_ssl: True
rabbit.host: 127.0.0.1
rabbit.port: 12345
</pre>
## 总结 ##
OpenStack的oslo.config库使用python构建的配置管理，为用户提供了一种开放的配置项解析工具，可以在其上实现自己需要的命令行和配置文件。由于其基于python构建，API接口风格较为简单，但是其调用速度可能受到影响。
# 综上所述： #
本项目组对比了现有的OpenStack Oslo Config，Uber Go Configuration，spf13's Viper三类配置管理库，发现这三类库的风格各有千秋，同时也各有优劣。</br>
其中基于Go语言实现的Uber Go Configuration，spf13's Viper拥有较快的响应速度，其使用Go语言进行开发也更符合本项目的基本需求。</br>
其中Uber Go Configuration接口使用较为复杂，只支持解析yaml文件，项目缺乏详细的(README)说明书，受众群体数量较少。</br>
而spf13's Viper具有较多的使用用户，功能强大，能够支持解析json、yaml等多类配置文件，并且接口风格简易，使用简单，适合Go语言开发者进行参考使用。</br>
对于OpenStack Oslo Config，其使用python进行构建，用来配置OpenStack各个服务通常以.conf结尾的ini风格的配置文件。其具有较为简易的API风格，便于使用，但是其在调用速度上相比Go语言会略逊一筹。</br>
因此项目组考虑综合以上三类配置管理库的优点，基于Go语言开发本项目配置管理库，使用较为简易的API接口风格，支持从 Shell 环境变量和命令行参数读取配置项，书写较为完善的使用说明，为客户提供良好的使用体验。
## 参考资料（References）

viper 配置文件解析库
[viper 配置文件解析库](https://www.cnblogs.com/jiujuan/p/13799976.html "viper 配置文件解析库")

viper-github地址:[https://github.com/spf13](https://github.com/spf13)

[OpenStack配置解析库oslo.config的使用方法](https://www.cnblogs.com/Security-Darren/p/3854797.html)

[OpenStack Oslo Config](https://docs.openstack.org/oslo.config/latest/)

[Uber Go Configuration](https://github.com/uber-go/config)
