# Go Template language
> go template language广泛应用于helm charts中。

我们有个一个values.yaml文件。
```yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```

## A Starter Chart
简单的创建一个叫`mychart`的chart。

```bash
$ helm create mychart
Creating mychart
```
## A Quick Glimpse of ``mychart/templates/``
在`mychart/templates/`目录下：
  - `NOTES.txt`:  这个将会在你运行 `helm install`之后打印一些信息。
  - `deployment.yaml`: 一个简单的deployment 的manifest
  - `service.yaml`:  为deployment的创建 service endpoint
  - `_helpers.tpl`:  为chart重复使用的template 模板

上述文件中，`NOTES.txt`和`_helpers.tpl`广泛使用了go template。

## helm Built-in Objects
helm 提供，template可以直接使用的值。
包括：
  - Release
  - Values
  - Files
  - Capabilities
  - Template

这些内容不是本文的重点。请参考：  
[Build-In Objects](https://helm.sh/docs/chart_template_guide/builtin_objects/)  
[Values Files](https://helm.sh/docs/chart_template_guide/values_files/)

## Values Files
传递给helm的`values`。
来自于：
  - `values.yaml`
  - 如果是子chart，来至于父chart的`values.yaml`
  - `helm install` 或者`helm upgrade` 通过 `-f` flag
  - 通过 `--set` 传递的参数


## 关于. 和作用域
默认`.`指向 `root context`，可以全局引用，但是可以通过`range`，`with`变`.`的值和作用域。通过`range`和`with`的的`.`指向的是`当前的context`。
`$`永远指向`root context`。
## 字符和空格

format|作用  
| :=: | :-: |
{{- pipeline }}|去除pipeline左方空格
{{ pipeline -}} | 去除pipeline右方空格
{{- pipeline -}} | 去除pipeline左、右方空格




## 注释

{{/* 注释的内容 */}}

### 管道pipeline
不同于`go template language`中，**pipeline**是指产生数据的操作。比如{{.}}、{{.Name}}、funcname args等。helm中`pipelines are an efficient way of getting several things done in sequence`.  
pipeline 格式  
{{ .Values.favorite.drink | functionName  }}
一些常用的functions：
function name|作用|例子|Note
:-:|:-:|:-:|:-:
quote| 字段添加双引号符号||
squote 字段添加单引号符号||
upper/lower| 字段变大写/小写||
repeat n| 字段重复n次||
default| 默认字段| drink: {{ .Values.favorite.drink \| default "tea" }}等于 `default DEFAULT_VALUE GIVEN_VALUE`|如果`.Values.favorite.drink`为`false`或`nil`则值为“`tea`”或`DEFAULT_VALUE`
and| 返回两个参数的bool值|and .Arg1 .Arg2|
or|返回一个bool或两个值中的一个，返回第一个非空参数或者最后一个参数|or .Arg1 .Arg2
eq|Returns the boolean equality of the arguments (e.g., Arg1 == Arg2)|eq .Arg1 .Arg2
coalesce| takes a list of values and returns the first non-empty one|coalesce 0 1 2 返回1|
printf|格式化打印|printf "%s has %d dogs." .Name .NumberDogs|
trim|去除字符串首尾的空格|trim "   hello    "|
trimAll| 移除给定字符|trimAll "$" "$5.00"|
indent n| 在每行首添加n个空格|indent 4 $lots_of_text|
nindent n| 在新行缩进|nindent 4 $lots_of_text|
{{- "hello"}}| 去掉hello字符串前的空格||
{{"hello" -}}| 去掉hello字符串后的空格||
{{- "hello"  -}}| 去掉hello字符串前后的空格||


## 变量
在helm template中，一个变量可以使用 `$name` 去定义，并使用`:=`赋值。变量的作用域是当前作用域。

e.g:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```
{{- $relname := .Release.Name -}} 是在with之前定义的，所以在with内部， $relname 仍然指向release name。
变量也可以在range中应用：
```yaml
toppings: |-
  {{- range $index, $topping := .Values.pizzaToppings }}
    {{ $index }}: {{ $topping }}
  {{- end }}
```
## 流程控制
### `if/else` 条件判断
语法结构：

```yaml
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```
用pipeline替代value是因为这个流程控制可以执行整个pipeline而不仅仅是一个判断一个值。
当pipeline为下列值时，可以认为是false：
  - 布尔值 `false`
  - 数字 `0`
  - `空的字符串`
  - 一个 `nil`
  - 一个空的`集合(map,slice,tuple,dict,array)`

### `with` specify a scope
`with`控制变量的边界。将`.`指向正确的边界。
语法结构：

```yaml
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

e.g:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
```

需要注意的是，在 with 中，你将不能在引用父scope的变量。
e.g:
```yaml
{{- with .Values.favorite }}
drink: {{ .drink | default "tea" | quote }}
food: {{ .food | upper | quote }}
release: {{ .Release.Name }}
{{- end }}
```
将报错，因为在with中无法找到.Release 。
但是可以使用 $ 引用with边界之外的变量。
e.g:
```yaml
{{- with .Values.favorite }}
 drink: {{ .drink | default "tea" | quote }}
 food: {{ .food | upper | quote }}
 release: {{ $.Release.Name }}
 {{- end }}
```
### range provides a 'for each'-style loop
helm里使用range做循环操作。
语法结构：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}

```

### 内置函数和自定义函数
helm 有超过60个可用functions，一些使用`go template language`，另外一些则来源于 `Sprig template library`。  
[Template Functions and Pipelines](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)

## template应用：define、template、include
>template names are global,应该确保template的名字全局唯一，包括子charts。

在开始template之前有一些规则需要熟悉一下：
- 在template/目录下的文件如果包含kubernetes manifests，则会被当成template文件对待
- NOTES.txt是一个例外
- 以_开头的文件被认为不包含manifest。这些文件用来保存一些帮助信息。

###
define可以用来在一个template file中创建一个模板
```yaml
{{ define "MY.NAME" }}
  # body of template here
{{ end }}
```
我们可以用来定义一个封装k8s代码块的template
```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

template is an action, and not a function， there is no way to pass the output of a template call to other functions; the data is simply inserted inline.    
It is considered preferable to use include over template in Helm templates simply so that the output formatting can be handled better for YAML documents.


## 文件处理
[Accessing Files Inside Templates](https://helm.sh/docs/chart_template_guide/accessing_files/)
