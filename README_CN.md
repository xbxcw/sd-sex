# Substance Expressions

Substance Designer 插件，用于从代码创建函数图。包含简单的编辑器和代码生成功能

## 安装
只需手动将插件路径添加到 SD 首选项中的搜索路径：
https://docs.substance3d.com/sddoc/plugin-search-paths-172825000.html（您需要添加 README.md 所在根目录的路径）

之后，打开任何函数图时您应该会看到此工具栏

![Toolbar](img/toolbar.png)

_(在 MacOS 上，创建的工具栏默认不活动，因此您需要先点击 IE 图标才能看到 Expression 按钮)_

或者您可以从 [发布页面](https://github.com/igor-elovikov/sd-sex/releases) 安装插件。只需从资源中下载 __sex.sdplugin__，然后通过 Substance Designer 中的插件管理器安装。_工具 -> 插件管理器_，然后点击 _安装_ 按钮并浏览下载的文件。

## 使用

只需点击 _Expression_ 按钮打开编辑器。

![Editor](img/editor.png)

要创建图形，只需点击 _COMPILE_ 按钮。就是这样。

当您打开编辑器时，插件会创建一个名为 _Snippet_ 的帧对象。不要删除它，因为它包含图形的实际代码。当您点击 _COMPILE_ 时，代码会保存到 snippet 对象中，因此在关闭编辑器之前要小心——即使您尚未完成，也请尝试编译以保存。


## 语言
插件使用 Python AST，因此在语法上是 Python，具有预期的结果。然而它只支持 Python 的非常有限的功能集，基本上只是简单的算术和逻辑表达式以及函数调用。

它看起来像这样：
```python
# 这是单行注释

x = 1.0 # 将浮点值 1.0 赋值给变量 [x]
y = 0.5
pos = get_float2("$pos") # 获取像素处理器的系统变量 $pos

sample = samplelum(pos + vector2(x, y), 0, 0) # 采样像素

_OUT_ = sample
```
## 内置函数节点
### 常量
![Constant](img/constant.png)

支持所有函数图类型
```python
# 布尔值
a = True
b = False

# 浮点数
f = 0.0
f2 = float2(1.0, 2.0)
f3 = float3(1.0, 2.0, 3.0)
f4 = float4(1.0, 2.0, 3.0, 4.0)

# 整数
i = 2
i2 = int2(1, 2)
i3 = int3(1, 2, 3)
i4 = int4(1, 2, 3, 4)

# 字符串
s = "foo"
```

对于向量类型，所有组件都必须显式指定。因此 `x = float3(0.0)` 无效，请改用 `x = float3(0.0, 0.0, 0.0)`

请注意，这些是常量声明，要创建变量请使用向量函数。
```python
one = 1.0
two = 2.0

v = float2(one, two) # 无效：float2 只接受常量
v = vector2(one, two) # 有效
```

### 向量
![Vector](img/vector.png)

#### 构造函数
浮点向量构造函数
```python
one = 1.0
two = 2.0
three = 3.0
four = 4.0
f2 = vector2(1.0, 2.0) # [f2] 是 float2(1.0, 2.0)
f3 = vector3(vector2(1.0, 2.0), 3.0) # [f3] 是 float3(1.0, 2.0, 3.0)
f4 = vector4(vector2(1.0, 2.0), vector2(3.0, 4.0)) # [f4] 是 float4(1.0, 2.0, 3.0, 4.0)
```
内置构造函数对于 float3 和 float4 有点繁琐。对于这些变量，您可以改用 function.sbs（SD 中包含的标准函数）中的 merge_float。
```python
f3 = merge_float3(one, two, three) # 比 vector3((vector2(1.0, 2.0), 3.0) 好得多
f4 = merge_float4(one, two, three, four)
```

#### Swizzling（分量重组）
Swizzling 对于整数和浮点类型的工作方式不同。浮点向量的分量是 `.xyzw`，整数的是 `.abcd`
```python
int_vector = int4(1, 2, 3, 4)
i = int_vector.ab # [i] 是 int2(1,2)
i = int_vector.aaa # [i] 是 int3(1, 1, 1)
i = int_vector.dbb # [i] 是 int3(3, 2, 2)

f_vector = float4(1.0, 2.0, 3.0, 4.0)
f = f_vector.zyx # [f] 是 float3(3.0, 2.0, 1.0)
f = f_vector.ww # [f] 是 float2(4.0, 4.0)

# 等等
```
目前 swizzling 仅支持作为右值。因此无法赋值给属性
```python
f3 = float3(1.0, 2.0, 3.0)

f3.x = 0.0 # 不支持！

f3 = merge_float3(0.0, f3.y, f3.z) # 请改用此方法
```

### 变量
![Variables](img/variables.png)

这个非常简单明了
```python
b = get_bool("my_boolean_var")

f = get_float("my_float_var")
f2 = get_float2("my_float2_var")
f3 = get_float3("my_float3_var")
f4 = get_float4("my_float4_var")

i = get_int("my_int_var")
i2 = get_int2("my_int2_var")
i3 = get_int3("my_int3_var")
i4 = get_int4("my_int4_var")
```

### 采样器
![Samplers](img/samplers.png)

采样时，灰度使用 `samplelum(uv, input, filtering)`，颜色使用 `samplecol(uv, input, filtering)`。


`samplelum` 返回 float，`samplecol` 返回 float4

* `uv` float2 变量，用于 uv 坐标
* `input` 整数常量，用于输入编号
* `filtering` 整数常量，用于采样方式。`0` 表示最近邻，`1` 表示双线性

请注意，input 和 filtering 必须使用显式常量。
```python
input_num = 5
filter = 0

# 由于函数图的工作方式，这不支持。Input 和 filtering 不能是变量
s = samplecol(float2(0.5, 0.5), input_num, filter) # 无效

s = samplecol(float2(0.5, 0.5), 5, 0) # 有效
```


### 类型转换
![Cast](img/cast.png)

相同大小向量的类型转换。SD 支持从 int 转换为 float，反之亦然。
```python
# 转 float 函数：tofloat(), tofloat2(), tofloat3(), tofloat4()
# 转 int 函数：toint(), toint2(), toint3(), toint4()

i = 2
f = tofloat(i) # [f] 是 2.0

f3 = float3(1.0, 2.0, 3.0)
i3 = toint3(f3) # [i3] 是 int3(1, 2, 3)
```

### 运算符
![Operator](img/operator.png)

与 SD 中一样，运算符处理相同大小和相同类型的变量。

```python
v1 = float3(1.0, 2.0, 3.0)
v2 = float3(1.0, 2.0, 3.0)

# 加法
v = v1 + v2

# 减法
v = v1 - v2

# 取反
v = -v1

# 除法（按分量）
v = v1 / v2

# 乘法（按分量）
v = v1 * v2

# 取模（按分量）
v = v1 % v2

# 标量乘法（仅 float）
# 表示法是 [向量] @ [标量]，顺序必须准确，标量始终在右侧
v = v1 @ 2.0 # [v] 是 float3(2.0, 4.0, 6.0)

# 点积
v = v1 ^ v2
# 或者
v = dot(v1, v2)
```

所有算术和逻辑表达式都遵循 Python 语法规则，因此您不仅限于使用一个运算符
```python
v = (v1 + v2 @ 2.0) * float3(5.0, 5.0, 5.0) - (v2 - v1) @ 5.0 
```

### 逻辑运算
![Logical](img/logical.png)

逻辑运算符与 Python 类似
```python
yes = True
no = False

b = yes and no # False
b = yes or no # True
b = not yes # False
```

### 比较运算
![Comparison](img/comparison.png)

比较运算符与 Python 类似
```python
one = 1.0
two = 2.0
four = 4

is_even = four % 2 == 0 # True
b = one > two # False
b = four <= 4 # True
b = four != 4 # False
# 等等
```

### 条件表达式
![Control](img/control.png)

SD 基本上只支持三元运算符进行条件控制。插件目前也有相同的限制，因此没有实际的分支表达式。

条件表达式
```python
b = True
f = 2.0
x = 5.0 if b else 0.0 # [x] 是 5.0
x = 5.0 if not b else 0.0 # [x] 是 0.0
x = 5.0 if b and (f * 3.0) < 4.0 else 0.0 # 条件中可以使用任何逻辑表达式
```

然而您仍然可以进行分支，只是不太方便。通常您只需计算所有分支（结果的所有值），然后通过条件表达式选择合适的分支
```python

branch1 = # ... 计算分支 1 ... #

branch2 = # ... 计算分支 2 ... #

condition = trigger > 0

result = branch1 if condition else branch2
```

模拟 switch 表达式
```python
switch = 3

x = 0.0
x = 1.0 if switch == 1 else x
x = 2.0 if switch == 2 else x
x = 3.0 if switch == 3 else x
x = 4.0 if switch == 4 else x

# 这里 [x] 是 3.0
```

### 函数
![Function](img/function.png)

支持所有 SD 内置函数。除 `min`、`max`、`abs` 外，所有函数只接受 float 参数。大多数函数支持标量和向量类型。如果函数接受向量，则按分量执行操作（与 SD 完全相同）
```python
v1 = float4(1.0, 2.0, 3.0, 4.0)
v2 = float4(5.0, 6.0, 7.0, 8.0)
t = 0.5

# 2 的幂
x = pow2(v1)

# 绝对值
x = abs(v1)

# 反正切 2 - 仅 float2 参数
x = atan2(v1.xy)

# 笛卡尔坐标 - 仅 2 个 float 标量参数
x = cartesian(v1.x, v1.y)

# 向上取整
x = ceil(v1)

# 余弦
x = cos(v1)

# 指数
x = exp(v1)

# 向下取整
x = floor(v1)

# 线性插值 - 最后一个参数是 float 标量
x = lerp(v1, v2, t)

# 对数
x = log(v1)

# 以 2 为底的对数
x = log2(v1)

# 最大值
x = max(v1, v2)
x = max(2, 5) # 也支持整数类型

# 最小值
x = min(v1, v2)

# 随机 - 仅标量 float 参数
x = rand(1.0) # [x] 是 0 到 1.0 之间的随机数

# 正弦
x = sin(v1)

# 平方根
x = sqrt(v1)

# 正切
x = tan(v1)
```

## 输出值

要将表达式标记为输出值，只需将其赋值给特殊变量 __`_OUT_`__
```python
_OUT_ = 10 
```

只需确保表达式结果的类型与图形预期的类型相似（如果有预期类型）。

## 类型检查

类型检查与 SD 类似，这意味着__没有隐式转换__

```python
x = 1.0 + 2 # 添加 float 和 integer 值
``` 

上面的示例无法编译成图形，并会在控制台显示错误消息。您必须根据所需的结果类型通过显式类型转换来解决此问题

```python
x = toint(1.0) + 2 # 有效 [x] 是整数值 3
x = 1.0 + tofloat(2) # 有效 [x] 是浮点值 3.0
```

## 导出变量

对于 FX-Maps，SD 允许您使用 Set/Sequence 节点输出多个值。请参阅 https://docs.substance3d.com/sddoc/using-the-set-sequence-nodes-102400025.html

插件也通过 `export` 关键字支持此功能。基本上您可以导出脚本中的任何变量，以便稍后在其他函数图中使用。只需确保它们在导出后被评估。如文档所述，通常的工作流程是在顶级参数（例如 "Color/Luminocity"）中创建超级函数，并在那里导出所有变量。

导出工作方式如下
 ```python
 # 顶层参数上的某些函数
 x = 1.0
 y = 10.0 * some_value
 
 vec = vector2(x, y)
 
 export(vec)
 
 _OUT_ = 1.0
 
 ```

在其他函数中，您只需获取导出的变量
```python
# 稍后评估的某个函数

vec = get_float2("vec") # 小心：您必须使用正确的类型获取器（在这种情况下是 float2）
```

## 声明图形输入
有一个辅助函数可以自动声明所有图形输入。如果您的图形有很多输入，这会非常方便。

假设您有一个名为 `My_Graph` 的图形，带有以下示例输入

![Inputs](img/params.png)

在任何函数子图中，您只需使用这个
```python
declare_inputs("My_Graph")

# 现在您可以直接使用输入

pos_with_offset = position + float2(0.5, 0.5)
x = 2.0 if trigger else 0.0

# 等等
```

## 导入外部函数

目前插件会自动从 SD 中包含的标准包 `function.sbs` 导入外部函数。`function.sbs` 中的所有函数都可以直接使用。请参阅 (https://github.com/igor-elovikov/sd-sex/blob/master/func_list.md) 查看所有别名。

有时当您打开编辑器时，会看到 `function.sbs` 在您的包中打开。目前这是解决依赖关系（加载包）的唯一方法。然而，如果您的包已经依赖于 `function.sbs`，则不会发生这种情况。因此，只有在从头开始处理某些图形时才会发生。

此外，当前打开的包中的任何函数图都会自动导入。因此，如果您的当前包中有函数图，您可以在这些包内的任何地方将它们用作脚本中的函数

![Function Example](img/import_func.png)

```python
x = My_Function() # 使用函数图 id 作为函数名
y = Other_Function(x, 2.0) # 您可以使用来自不同包的函数
_OUT_ = y + 2.0
```

基本上，如果您需要使用其他 .sbs 文件中的某个函数，只需打开该文件，使其列在您的资源管理器窗口中。保存包时，SD 会自动解析所有依赖关系。

## 插件设置

所有设置都存储在插件目录中的 settings.json 文件中。
您可以在那里设置编辑器的自定义字体大小，用它来根据您的 DPI 调整编辑器外观
还有其他设置：
* `"tab_spaces": 4` - 编辑器中制表符的空格数
* `"align_max_nodes": 150` - 编译后的图形可以对齐以形成更可读的结构。然而，对于复杂图形，这可能非常慢，因此仅当节点数少于 `align_max_nodes` 设置时才触发（如果不需要对齐，请将其设置为零）

## 元编程功能

这是一个非常强大的功能，允许您编写模块化且更具表现力的代码。它基于 Jinja 模板引擎：https://jinja.palletsprojects.com/

本质上，您编写的所有代码都是 Jinja 模板，它在进入编译器之前会展开。所以这就像编写代码来编写实际的脚本。一开始可能有点令人困惑，但实际上非常简单。基本上它只是文本处理。

要检查生成的代码，只需点击编辑器中的 _View Generated Code_ 选项卡。尝试不时切换到它，看看您的代码是什么样子，并确保您的模板没有任何错误。每次切换到此选项卡时，插件都会尝试展开您的模板，如果有任何错误，您会在控制台输出中看到它们。

查看 Jinja 文档，但开始时理解它的最佳方式是查看实际示例。

### Jinja 环境

Jinja 使用行语句 `::` 设置，因此 `{% set x = 2 %}` 与 `:: set x = 2` 相同。使用您喜欢的任何方式

此外，任何外部文件的路径都设置为包路径。因此，如果您有任何外部源文件，只需将它们放在与您的 .SBS 文件相同的目录中。

### 示例

这些是编写表达式的最常见做法。虽然您可以使用 Jinja 中包含的任何功能，但这只是我发现最有用的一些功能。

#### For 循环

循环可用于模拟数组
```python
:: for i in range(5)
    x{{ i }} = {{ i | float }}
:: endfor

# ---- 结果 ---- #

x0 = 0.0
x1 = 1.0
x2 = 2.0
x3 = 3.0
x4 = 4.0

```

也非常适合采样相邻像素。

基本的 3x3 方框模糊
```python
size = get_float2("$size")
pos = get_float2("$pos")

total_lum = 0.0

:: for x in range(-1, 2)
    :: for y in range(-1, 2)
        offset = vector2({{ x | float }}, {{ y | float }})
        total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
    :: endfor
:: endfor

_OUT_ = total_lum / 9.0

# ---- 结果 ---- #

size = get_float2("$size")
pos = get_float2("$pos")

total_lum = 0.0

offset = vector2(-1.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(-1.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(-1.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(0.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(0.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(0.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(1.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(1.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)
offset = vector2(1.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0)

_OUT_ = total_lum / 9.0
```

小心使用循环，并始终检查生成的代码。很容易创建一个巨大的图形，尤其是嵌套循环。

#### 模板变量

让我们扩展方框模糊示例以使用内核矩阵。

```python
size = get_float2("$size")
pos = get_float2("$pos")

# 高斯 3x3 内核
:: set kernel = [(0.0625, 0.125, 0.0625), (0.125, 0.25, 0.125), (0.0625, 0.125, 0.0625)]

total_lum = 0.0

:: for x in range(-1, 2)
    :: for y in range(-1, 2)
        offset = vector2({{ x | float }}, {{ y | float }})
        total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * {{ kernel[x + 1][y + 1] }}
    :: endfor
:: endfor

_OUT_ = total_lum 

# ---- 结果 ---- #
size = get_float2("$size")
pos = get_float2("$pos")

total_lum = 0.0

offset = vector2(-1.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.0625
offset = vector2(-1.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.125
offset = vector2(-1.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.0625
offset = vector2(0.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.125
offset = vector2(0.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.25
offset = vector2(0.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.125
offset = vector2(1.0, -1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.0625
offset = vector2(1.0, 0.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.125
offset = vector2(1.0, 1.0)
total_lum = total_lum + samplelum(pos + offset / size, 0, 0) * 0.0625

_OUT_ = total_lum 

```

当您需要使用显式常量并希望在多个地方使用相同的常量时，变量也很有用。
例如，您可以为采样过滤创建一个设置。

```python
# 0 表示最近邻，1 表示双线性
:: set filter = 1

sample = samplelum(pos, 0, {{ filter }})

# ---- 结果 ---- #

sample = samplelum(pos, 0, 1)
```

#### 宏

宏就像一个函数，可以帮助您减少代码重复。

假设您想将一个值乘以因子，但如果因子为零则保持不变。当然您可以为此创建一个函数，但有时编写宏更容易。

```python
:: macro apply_modifier(value, modifier)
{{ value }} = {{ value }} * {{ modifier }} if {{ modifier }} > 0.0 else {{ value }}
:: endmacro

x = 1.0
f = 0.0

{{ apply_modifier("x", "f") }}

# ---- 结果 ---- #

x = 1.0
f = 0.0

x = x * f if f > 0.0 else x
```

在这种特殊情况下，不使用宏可能更容易，但当操作变得更复杂时，宏是必需的。特别是当您迭代代码并突然决定对此类操作进行一些更改时。使用宏，您只需在一个地方调整它，然后重新编译。

#### 导入宏

您可以将常用的宏放在外部文件中，然后将其导入到您的代码中。它的工作方式类似于 Python 的导入。

因此，对于上面的示例，我们可以将宏放在某个文件中，如 `macros.sex`。然后我们可以导入它并使用

```python
:: import "macros.sex" as macros

x = 1.0
f = 0.0

{{ macros.apply_modifier("x", "f") }}

# ---- 结果 ---- #

x = 1.0
f = 0.0

x = x * f if f > 0.0 else x
```

请注意，`macros.sex` 必须与您的包在同一目录中

#### 包含外部文件

您也可以将任何外部文件包含到您的代码片段中。这实际上就像将文件内容粘贴到代码中。
当您在图形之间共享一些代码时，这非常有用。

```python
:: inlcude "blur.sex"
```

生成的代码将是 `blur.sex` 文件的内容

#### If-Else 块

使用 If-Else 块进行编译时分支。示例之一是在外部文件中创建一个可以通过某些设置更改的模板。

例如，我们可以创建一个文件 `color_or_grayscale.sex`，包含以下代码
```python
:: if color
    _OUT_ = uniform_f4_ab(float4(0.0, 0.0, 0.0, 0.0), float4(1.0, 1.0, 1.0, 1.0))
:: else
    _OUT_ = uniform_ab(0.0, 1.0)
:: endif
```

然后在代码片段中
```python
:: set color = true
:: include "color_or_grayscale.sex"

# ---- 结果 ---- #

_OUT_ = uniform_f4_ab(float4(0.0, 0.0, 0.0, 0.0), float4(1.0, 1.0, 1.0, 1.0))
```

根据 `color` 设置，我们可以为代码选择分支。