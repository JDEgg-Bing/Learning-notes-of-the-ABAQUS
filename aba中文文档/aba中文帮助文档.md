# Abaqus脚本用户指南
## 第一部分：Abaqus脚本接口简介  

Abaqus脚本接口是用于访问Abaqus模型和数据的应用程序编程接口（API）。该接口是基于Python面向对象编程语言的扩展，因此Abaqus脚本本质上就是Python脚本。通过Abaqus脚本接口，您可以实现以下功能：  
- 创建和修改Abaqus模型的组件，例如部件、材料、载荷和分析步；  
- 创建、修改并提交Abaqus分析任务；  
- 读写Abaqus输出数据库；  
- 查看分析结果。  

您可以通过Abaqus脚本接口以脚本（或程序）形式调用Abaqus/CAE的功能（Abaqus/CAE的可视化模块也可单独授权为Abaqus/Viewer，因此该接口同样支持调用Abaqus/Viewer的功能）。由于Abaqus脚本接口是标准Python的定制化扩展，不允许用户通过扩展基础类型来创建自定义类。  

**本部分将介绍Abaqus脚本接口的基础知识，涵盖以下主题：**  
- 第1章《Abaqus脚本用户指南概述》  
- 第2章《Abaqus脚本接口入门》  
- 第3章《简单示例》

### 1.《Abaqus脚本用户指南》概述

《Abaqus脚本用户指南》将引导您理解Python编程语言和Abaqus脚本接口，从而编写自定义程序。本指南还介绍了如何通过Abaqus脚本接口和C++应用程序接口（API）访问Abaqus输出数据库。指南包含以下章节：

**Abaqus脚本接口简介**

本节概述Abaqus脚本接口，并说明Abaqus/CAE如何执行脚本。

**简单示例**

提供两个简单示例帮助您入门：

- 创建部件
- 从输出数据库读取数据

**Python语言入门**

本节是Python编程语言的基础介绍，并非全面描述。市场上有多种Python相关书籍，本指南列出了这些参考书目以及Python相关网站资源。

**Python与Abaqus脚本接口应用**

本节详细讲解Abaqus脚本接口，解释命令参考文档风格，介绍数据类型、错误处理等重要概念。

**Abaqus/CAE中的脚本接口应用**

说明如何通过脚本接口控制Abaqus/CAE模型和分析任务，介绍Abaqus对象模型、区域指定方法及分析产品（Abaqus/Standard、Abaqus/Explicit或Abaqus/CFD）消息读取技术。若不使用Abaqus/CAE可跳过本节。

**示例脚本**

提供一组与《Abaqus/CAE入门指南》附录B"在Abaqus/CAE中创建并分析简单模型"悬臂梁教程配套的脚本案例。额外示例包括：从输出数据库读取数据、显示云图、打印各分析步云图。最终案例演示如何读取Abaqus/CAE创建的模型数据库、参数化模型、提交分析任务集合并生成结果。

**通过脚本接口访问输出数据库**

分析任务执行后，结果将存储于输出数据库（.odb文件），可通过Abaqus/CAE可视化模块或Abaqus/Viewer查看。本节说明如何通过脚本接口访问输出数据库数据，包括：
- 读取部件和装配体几何模型数据（节点坐标、单元连接、单元类型等）
- 读取截面和材料及其装配位置数据
- 读取选定步骤、帧和区域的场输出数据
- 读取历史输出数据
- 处理场输出和历史输出数据
- 向现有或新建输出数据库写入模型数据及输出数据

**通过C++访问输出数据库**

本节介绍如何使用C++语言通过API访问输出数据库数据。C++ API功能与脚本接口API相同，但脚本接口的交互特性和与Abaqus/CAE的集成使其更易使用。C++接口面向有经验的开发者，适用于需要绕过脚本接口以提升性能的场景，能更快访问大型输出数据库数据。

### 2.Abaqus脚本接口简介

涵盖以下主题：
• “Abaqus/CAE与Abaqus脚本接口”，第2.1节
• “Abaqus脚本接口如何与Abaqus/CAE交互？”，第2.2节

#### 2.1 Abaqus/CAE与Abaqus脚本接口

当您使用Abaqus/CAE图形用户界面（GUI）创建模型并可视化结果时，每个操作后Abaqus/CAE都会在内部发出命令。这些命令反映了您创建的几何结构以及从每个对话框中选择的选项和设置。GUI使用名为Python的面向对象编程语言生成命令。GUI发出的命令被发送到Abaqus/CAE内核。内核解释这些命令，并使用选项和设置来创建模型的内部表示。内核是Abaqus/CAE的核心。GUI是用户与内核之间的接口。

Abaqus脚本接口允许您绕过Abaqus/CAE GUI，直接与内核通信。包含Abaqus脚本接口命令的文件称为脚本。您可以使用脚本执行以下操作：

- 自动化重复性任务。例如，您可以创建一个在用户启动Abaqus/CAE会话时执行的脚本。此类脚本可用于生成标准材料库。因此，当用户进入属性模块时，这些材料将可用。同样，脚本可用于创建用于运行分析作业的远程队列，这些队列将在作业模块中可用。
- 进行参数化研究。例如，您可以创建一个逐步修改零件几何形状并分析结果模型的脚本。同一脚本可以读取生成的输出数据库，显示结果，并从每个分析中生成带注释的硬拷贝。
- 创建和修改模型数据库以及在使用Abaqus/CAE时交互式创建的模型。Abaqus脚本接口是模型数据库和模型的应用程序编程接口（API）。有关模型数据库和模型的讨论，请参阅《Abaqus/CAE用户指南》第9.1节“什么是Abaqus/CAE模型数据库？”和第9.2节“什么是Abaqus/CAE模型？”
- 访问输出数据库中的数据。例如，您可能希望对分析结果进行自己的后处理。您可以将自己的数据写入输出数据库，并使用Abaqus/CAE的可视化模块查看其内容。

Abaqus脚本接口是流行的面向对象语言Python的扩展。任何关于Abaqus脚本接口的讨论同样适用于Python的一般情况，并且Abaqus脚本接口使用Python所需的语法和运算符。

#### 2.2 Abaqus脚本接口如何与Abaqus/CAE交互？

图2-1展示了Abaqus脚本接口命令如何与Abaqus/CAE内核交互。

![Screenshot](.\\images\\acl-all-schematic-nls.png)

图2-1 Abaqus脚本接口命令与Abaqus/CAE

可通过以下方式向Abaqus/CAE内核发送脚本接口命令：

- 图形用户界面(GUI)。例如当点击对话框中的"确定"或"应用"按钮时，GUI会根据对话框中的选项设置生成相应命令。您可以使用宏管理器将生成的脚本命令序列记录为宏文件。详见《Abaqus/CAE用户指南》第9.5.5节"创建和运行宏"。
- 点击主窗口左下角的图标![Screenshot](.\\images\\afxI_commandLine.png)可调出命令行界面(CLI)。您可以输入单条命令或粘贴来自其他窗口的命令序列，按[Enter]键执行。命令行支持输入任何Python命令，例如可作为简易计算器使用。

注：使用Abaqus/CAE时，错误和消息会显示在消息区。点击主窗口左下角图标![Screenshot](.\\images\\afxI_messageArea.png)可查看消息区。

- 当需要执行多条命令或重复执行相同命令时，可将语句集保存为脚本文件。脚本是采用纯ASCII格式存储的Python语句序列。例如可创建脚本实现：打开输出数据库、显示选定变量的云图、自定义图例并将图像输出到PostScript打印机。脚本还可用于以预设状态启动Abaqus/CAE，例如定义标准打印配置、创建远程队列、设置标准材料属性等。

运行脚本的方法包括：

**【启动时运行脚本】**

通过以下命令在启动Abaqus/CAE会话时运行脚本：```abaqus cae script=myscript.py```其中```myscript.py```为脚本文件名。Abaqus/Viewer对应命令为：```abaqus viewer script=myscript.py```通过在命令行输入"--"加空格分隔的参数可向脚本传递参数，这些参数将被Abaqus/CAE执行过程忽略但可在脚本内部调用。详见《Abaqus分析用户指南》第3.2.7和3.2.8节。

**【无GUI运行脚本】**

通过以下命令在不启动GUI的情况下运行脚本：```abaqus cae noGUI=myscript.py```Abaqus/Viewer对应命令为：```abaqus viewer noGUI=myscript.py```此模式可节省显示开销，适用于自动化前后处理任务。脚本运行结束后内核即终止，此时无法进行用户交互、作业监控或动画生成。无界面运行时作业始终以交互模式执行，指定的作业队列将被忽略。

**【从启动屏幕运行】**

在Abaqus/CAE启动界面点击"运行脚本"，通过对话框选择脚本文件。

**【通过文件菜单运行】**

从主菜单栏选择"文件>运行脚本"，通过对话框选择脚本文件。

**【通过命令行界面运行】**
在CLI中输入：```execfile('myscript.py')```其中```myscript.py```为当前目录下的脚本文件。图2-2展示了通过CLI运行脚本的示例。

![Screenshot](.\\images\\cmd-int-cli-nls.png)

图2-2 通过命令行界面运行脚本

点击主窗口左下角图标![Screenshot](.\\images\\afxI_commandLine.png)可在消息区与命令行界面间切换。

### 3. 简单示例

使用Abaqus脚本接口进行编程既直观又符合逻辑。为了展示编写自定义程序的便捷性，以下部分将介绍两个简单的Abaqus脚本接口示例：
- "创建零件" 第3.1节
- "从输出数据库读取数据" 第3.2节

您无需立即理解示例中的每一行代码；随着后续章节的详细讲解，相关术语和语法会逐渐清晰。第3.3节"小结"将阐述使用Python和Abaqus脚本接口编程的部分基本原理。

#### 3.1 创建零件

第一个示例展示了如何使用Abaqus/CAE脚本来复现Abaqus/CAE的功能。该脚本执行以下操作：
- 在模型数据库中创建新模型
- 创建二维草图
- 创建三维可变形零件
- 拉伸二维草图以创建零件的第一个几何特征
- 创建新视口
- 在新视口中显示新零件的着色图像

新建视口与着色零件如图3-1所示

![Screenshot](.\\images\\3\\cmd-int-aexample-nls.png)

图3-1 示例创建的新视口和零件

本指南中的示例脚本可通过Abaqus fetch工具复制到用户工作目录：```abaqus fetch job=scriptName```
其中```scriptName.py```是脚本名称（参见《Abaqus分析用户手册》第3.2.17节"获取示例输入文件"）。使用以下命令获取本示例脚本：
```abaqus fetch job=modelAExample```
注意：Abaqus默认不会在安装过程中安装示例脚本。因此如果fetch工具找不到示例脚本，可能是您的Abaqus安装缺少该脚本。您需要重新运行安装程序并在安装项目列表中选择Abaqus示例问题。

运行程序步骤如下：
1. 输入abaqus cae启动Abaqus/CAE
2. 在启动界面选择Run Script
3. 在弹出的对话框中选择modelAExample.py
4. 点击OK运行脚本
注意：若Abaqus/CAE已在运行，可通过主菜单栏选择File > Run Script来执行脚本

##### 3.1.1 示例脚本
```python
"""
modelAExample.py
简单示例：创建零件
"""

from abaqus import *
from abaqusConstants import *
backwardCompatibility.setValues(includeDeprecated=True,
                                reportDeprecated=False)

import sketch
import part

myModel = mdb.Model(name='Model A')

mySketch = myModel.ConstrainedSketch(name='Sketch A',
                                     sheetSize=200.0)

xyCoordsInner = ((-5 , 20), (5, 20), (15, 0),
    (-15, 0), (-5, 20))

xyCoordsOuter = ((-10, 30), (10, 30), (40, -30),
    (30, -30), (20, -10), (-20, -10),
    (-30, -30), (-40, -30), (-10, 30))

for i in range(len(xyCoordsInner)-1):
    mySketch.Line(point1=xyCoordsInner[i],
        point2=xyCoordsInner[i+1])

for i in range(len(xyCoordsOuter)-1):
    mySketch.Line(point1=xyCoordsOuter[i],
        point2=xyCoordsOuter[i+1])

myPart = myModel.Part(name='Part A', dimensionality=THREE_D,
    type=DEFORMABLE_BODY)

myPart.BaseSolidExtrude(sketch=mySketch, depth=20.0)

myViewport = session.Viewport(name='Viewport for Model A',
    origin=(10, 10), width=150, height=100)

myViewport.setValues(displayedObject=myPart)

myViewport.partDisplay.setValues(renderStyle=SHADED)
```
##### 3.1.2 脚本工作原理
本节解析示例脚本的各个部分

```from abaqus import *```

该语句使脚本可访问基本Abaqus对象，并通过变量mdb提供对默认模型数据库的访问。

```from abaqusConstants import *```

语句使脚本可访问Abaqus脚本接口定义的符号常量。

```
import sketch
import part
```

这些语句提供对草图和零件相关对象的访问权限。sketch和part被称为Python模块。

图3-2所示的下一条语句可从右向左解读：
1. 创建名为Model A的新模型
2. 将新模型存储到模型数据库mdb中
3. 将新模型赋值给变量myModel

![Screenshot](.\\images\\3\\cmd-righttoleft-nls.png)

图3-2 创建新模型

```
mySketch = myModel.ConstrainedSketch(name='Sketch A', sheetSize=200.0)
```

该语句在myModel中创建名为Sketch A的新草图对象，并将变量mySketch赋给该草图。草图将放置在200单位见方的图纸上。注意：
- 创建对象（面向对象编程术语）的命令称为构造函数并以大写字母开头。您已看到分别创建Model对象和Sketch对象的Model和Sketch命令
- 该命令使用前一条语句创建的myModel变量。在脚本中使用具有描述性的变量名可提高代码可读性

```xyCoordsInner```和```xyCoordsOuter```两个语句定义了构成字母"A"内外轮廓顶点的XY坐标。```xyCoordsInner```对应内轮廓顶点，```xyCoordsOuter```对应外轮廓顶点。

for循环在```mySketch```中创建字母"A"的内轮廓，使用```xyCoordsInner```中的顶点坐标定义每条线的起点和终点。注意：
- Python仅用缩进表示循环起止，不使用C/C++的大括号{}
- ```len()```函数返回```xyCoordsInner```中的坐标对数（本例为5个）
- ```range()```函数返回整数序列。与C/C++相同，Python数组索引从0开始

类似地，第二个循环创建字符"A"的外轮廓。

```myPart```语句在```myModel```中创建名为```Part A```的三维可变形零件，并赋值给myPart变量。

```myPart.BaseSolidExtrude```语句通过拉伸mySketch深度20.0，在myPart中创建基础实体拉伸特征。

```myViewport```语句在```session```中创建名为```Viewport for Model A```的新视口，视口原点位于(20,20)，宽150高100。

```myViewport.setValues```语句指示Abaqus在新视口中显示新零件。

```myViewport.partDisplay.setValues```语句将视口中的零件显示渲染样式设置为着色模式。

#### 3.2 从输出数据库读取数据

第二个示例展示了如何使用Abaqus脚本接口读取输出数据库、处理数据，并通过Abaqus/CAE的可视化模块显示结果。即使未将数据写回输出数据库，Abaqus脚本接口仍可实现数据可视化。该脚本执行以下操作：
- 打开教程输出数据库
- 创建指向输出数据库中第一和第二步的变量
- 创建指向第一和第二步最后一帧的变量
- 创建指向第一和第二步最后一帧位移场输出的变量
- 创建指向第一和第二步最后一帧应力场输出的变量
- 计算两个步骤位移场输出的差值并存储到deltaDisplacement变量
- 计算两个步骤应力场输出的差值并存储到deltaStress变量
- 将deltaDisplacement设为默认变形变量
- 将deltaStress的von Mises不变量设为默认场输出变量
- 绘制结果云图
最终云图效果如图3-3所示。

![Screenshot](.\\images\\3\\cmd-super.png)

图3-3 生成的云图效果

使用以下命令获取脚本及其读取的输出数据库：

```
abaqus fetch job=odbExample
abaqus fetch job=viewer_tutorial
```

## 3.2.1 示例脚本
```python
"""
odbExample.py

用于打开输出数据库、叠加不同步骤最后一帧变量，
并显示结果云图的脚本
"""

from abaqus import *
from abaqusConstants import *
import visualization

myViewport = session.Viewport(name='叠加示例',
    origin=(10, 10), width=150, height=100)

# 打开教程输出数据库
myOdb = visualization.openOdb(path='viewer_tutorial.odb')

# 关联输出数据库与视口
myViewport.setValues(displayedObject=myOdb)

# 创建指向前两个步骤的变量
firstStep = myOdb.steps['Step-1']
secondStep = myOdb.steps['Step-2']

# 读取前两个步骤最后一帧的位移和应力数据
frame1 = firstStep.frames[-1]
frame2 = secondStep.frames[-1]

displacement1 = frame1.fieldOutputs['U']
displacement2 = frame2.fieldOutputs['U']

stress1 = frame1.fieldOutputs['S']
stress2 = frame2.fieldOutputs['S']

# 计算第二步载荷引起的位移和应力增量
deltaDisplacement = displacement2 - displacement1
deltaStress = stress2 - stress1

# 创建结果的Mises应力云图
myViewport.odbDisplay.setDeformedVariable(deltaDisplacement)

myViewport.odbDisplay.setPrimaryVariable(field=deltaStress,
    outputPosition=INTEGRATION_POINT,
    refinement=(INVARIANT, 'Mises'))

myViewport.odbDisplay.display.setValues(plotState=(
                                        CONTOURS_ON_DEF,))
```

3.2.2 脚本运行原理

本节解析示例脚本的各个部分。

```py
from abaqus import *
from abaqusConstants import *
```

这些语句使脚本可以访问基本Abaqus对象及脚本接口中定义的所有符号常量。

```py
import visualization
```

该语句提供访问Abaqus/CAE可视化模块功能的命令。

```py
myViewport = session.Viewport(name='叠加示例')
```

该语句在会话中创建名为"叠加示例"的新视口，视口位置和尺寸采用默认值。

```py
odbPath = 'viewer_tutorial.odb'
```

该语句创建指向教程输出数据库的路径。

myOdb = session.openOdb(path=odbPath)
该语句使用路径变量打开输出数据库并赋值给myOdb变量。
myViewport.setValues(displayedObject=myOdb)
该语句在视口中显示输出数据库的默认绘图。
firstStep = myOdb.steps['Step-1']
secondStep = myOdb.steps['Step-2']
这些语句将输出数据库的第一和第二步赋值给对应变量。
frame1 = firstStep.frames[-1]
frame2 = secondStep.frames[-1]
这些语句将第一和第二步的最后一帧赋值给变量（Python中索引-1表示序列末项）。
displacement1 = frame1.fieldOutputs['U']
displacement2 = frame2.fieldOutputs['U']
这些语句将位移场输出赋值给对应变量。
stress1 = frame1.fieldOutputs['S']
stress2 = frame2.fieldOutputs['S']
类似地，这些语句处理应力场输出。
deltaDisplacement = displacement2 - displacement1
该语句计算位移场输出差值并存储。
deltaStress = stress2 - stress1
该语句对应力场执行相同操作。
myViewport.odbDisplay.setDeformedVariable(deltaDisplacement)
该语句设置用于显示模型变形形状的变量。
myViewport.odbDisplay.setPrimaryVariable(field=deltaStress,
    outputPosition=INTEGRATION_POINT,
    refinement=(INVARIANT, 'Mises'))
该语句设置用于云图显示的主变量。
myViewport.odbDisplay.display.setValues(plotState=(CONTOURS_ON_DEF,))
最后这条语句设置以变形模型为背景显示云图的绘图状态。


#### 3.3 总结

这些示例展示了脚本如何在模型数据库中的模型上运行，或对输出数据库中存储的数据进行操作。示例中命令的详细说明将在后续章节介绍，但需注意以下几点：

- 您可以在启动Abaqus/CAE会话时从启动界面运行脚本。会话开始后，可通过"文件→运行脚本"菜单或命令行界面执行脚本。
- 脚本是以ASCII格式存储的命令序列，可使用标准文本编辑器进行编辑。
- Abaqus附带一组示例脚本。使用"abaqus fetch"命令可获取脚本及其相关文件。
- 必须使用import语句导入所需的Abaqus脚本接口命令集。例如，"import part"语句提供了创建和操作零件的命令。
- 创建对象（面向对象编程术语中的"对象"）的命令称为构造函数，以大写字母开头。例如，以下语句使用Model构造函数创建模型对象：
```myModel = mdb.Model(name='Model A')```
创建的模型对象存储在：
```mdb.models['Model A']```
- 可使用变量引用对象。变量使脚本更易读易懂，如前述示例中的myModel即引用模型对象。
- Python脚本可包含循环结构，循环的开始和结束由脚本中的缩进控制。
- Python包含一组内置函数，例如len()函数可返回序列长度。
- 通过脚本命令可复现所有Abaqus/CAE交互操作，例如创建视口、显示云图、设置显示步骤和帧等。

