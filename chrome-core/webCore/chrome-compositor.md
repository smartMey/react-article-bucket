#### 主要内容
在本章节，主要是一些渲染引擎相关内容。通过这部分内容我希望自己能够更多的了解浏览器，不仅仅是JS引擎，也包括渲染引擎和图像图形学基础知识。如果你有任何问题欢迎issue,同时也欢迎star！

#### 1.计算机图形学基础知识
##### 1.1 什么是[bitmap](https://baike.baidu.com/item/Bitmap/6493270?fr=aladdin)
**位图文件（Bitmap）**，扩展名可以是.bmp或者.dib。位图是**Windows标准格式图形文件**，它将图像定义为由**点（像素）**组成，每个点可以由多种色彩表示，包括2、4、8、16、24和32位色彩。例如，一幅1024×768分辨率的32位真彩图片，其所占存储字节数为：1024×768×32/(8\*1024)=3072KB(其中1个字节=8个比特位)。

位图文件图像效果好，但是`非压缩格式`的，需要**占用较大存储空间**，不利于在网络上传送。jpg格式则恰好弥补了[位图文件这个缺点](https://baike.baidu.com/item/%E4%BD%8D%E5%9B%BE%E5%9B%BE%E5%83%8F/1451321)。

![](./images/bitmap.jpg)

##### 1.2 什么是[渲染到纹理](http://blog.csdn.net/a812073479/article/details/49208119)
常规的渲染操作是将场景送至backbuffer，而backbuffer实际上是一个Surface，而纹理恰恰又包含了Surface，所以我们只需要取得纹理的Surface，其次将场景送至这个Surface，最后再把这个纹理渲染到backbuffer中即可。举个例子，假设你要在一面墙壁上画一幅画，你有两种方法:

- 方法1
  直接在墙上画，这个很好理解，就对应常规的backbuffer渲染
- 方法2
  先将画绘制在纸上，然后将纸贴到墙上，这就对应渲染到纹理的过程。这里**墙壁**相当于backbuffer，而纸张相当于纹理的Surface，在纸上作画相当于渲染到纹理，把纸贴到墙上相当于把纹理渲染到backbuffer。具体的步骤如下:

<pre>
1 创建纹理并获得纹理的表面（Surface）
2 向纹理的表面渲染场景
3 渲染纹理本身
</pre>

##### 1.3 双缓冲(Double Buffer)原理和使用
- 双缓冲作用

  双缓冲甚至是多缓冲，在许多情况下都很有用。一般需要使用双缓冲区的地方都是**由于“生产者”和“消费者”供需不一致**所造成的。这样的情况在很多地方可能会发生，使用**多缓冲**可以很好的解决。我举几个常见的例子：

  例1. 在网络传输过程中数据的接收，有时可能数据来的太快来不及接收导致数据丢失。这是由于“发送者”和“接收者”速度不一致所致，在他们之间安排一个或多个缓冲区来存放来不及接收的数据，让速度较慢的“接收者”可以慢慢地取完数据不至于丢失。

  例2. 再如，计算机中的**三级缓存结构**：外存（硬盘）、内存、高速缓存（**介于CPU和内存之间，可能由多级**）。从左到右他们的存储**容量不断减小，但速度不断提升**，当然价格也是越来越贵。作为“生产者”的 CPU 处理速度很快，而内存存取速度相对CPU较慢，如果直接在内存中存取数据，他们的速度不一致会导致 CPU能力下降。因此在他们之间又增加的高速缓存来作为缓冲区平衡二者速度上的差异。

  例3. 在图形图像显示过程中，计算机从**显示缓冲区**取数据然后显示，很多图形的操作都很复杂需要大量的计算，很难访问一次**显示缓冲区**就能写入待显示的完整图形数据，通常需要多次访问显示缓冲区，每次访问时写入**最新计算**的图形数据。而这样造成的后果是一个需要复杂计算的图形，你看到的效果可能是**一部分一部分地**显示出来的，造成很大的闪烁不连贯。而使用双缓冲，可以使你先将计算的**中间结果存放在另一个缓冲区中，待全部的计算结束，该缓冲区已经存储了完整的图形之后，再将该缓冲区的图形数据一次性复制到显示缓冲区**。

  例1中使用双缓冲是为了**防止数据丢失**，例2中使用双缓冲是为了提高**CPU的处理效率**，而例3使用双缓冲是为了**防止显示图形时的闪烁延迟**等不良体验。

- 双缓冲原理

  ![](./images/double-buffer.png)

  注意，**显示缓冲区是和显示器**一起的，显示器**只负责从显示缓冲区取数据显示**。**我们通常所说的在显示器上画一条直线**，其实就是往该显示缓冲区中写入数据。显示器通过不断的刷新（从显示缓冲区取数据），从而使显示缓冲区中数据的改变及时的反映到显示器上。

  这也是显示**复杂图形时造成延迟**的原因，比如你现在要显示从屏幕中心向外发射的一簇射线，你开始编写代码用一个循环从0度开始到360度，每隔一定角度画一条从圆心开始向外的直线。你**每次画线其实是往显示缓冲区写入数据**，如果你还没有画完，显示器就从显示缓冲区取数据显示图形，此时你看到的是一个不完整的图形，然后你继续画线，等到显示器再次取显示缓冲区数据显示时，图形比上次完整了一些，依次下去直到显示完整的图形。**你看到图形不是一次性完整地显示出来，而是每次显示一部分，从而造成闪烁**。

##### 1.4 什么是OpenGL
[OpenGL](https://baike.baidu.com/item/OpenGL/238984?fr=aladdin)（全写Open Graphics Library）是指定义了一个跨编程语言、跨平台的编程接口规格的专业的**图形程序接口**。它用于三维图像（二维的亦可），是一个功能强大，调用方便的底层图形库。

OpenGL是Khronos Group开发维护的一个规范，它主要为我们定义了[用来操作图形和图片的一系列函数的API](https://www.zhihu.com/question/51867884)，需要注意的是OpenGL本身并非API。GPU的硬件开发商则需要提供满足OpenGL规范的实现，这些实现通常被称为“驱动”，它们负责将OpenGL定义的API命令**翻译为GPU指令**。

当然，如果硬件开发商的某款显卡**无法在硬件上支持**OpenGL所定义的所有功能(OpenGL的实现没说一定要有GPU的，像mesa 3d可以在纯软件的环境下运行)，那么硬件开发商就必须通过**软渲染的方式**提供这种功能。综上，OpenGL并非一个能够直接安装的库或包，**它只是一个规范**。我们只需要安装显卡的驱动即可，因为**显卡驱动**中就包括了对OpenGL规范的实现。


##### 1.5 OpenGL的渲染管道
![](./images/rax.png)

这个图的最开始是图像的真实数据，即01010的比特数据;第二步就是获取顶点数据，即所谓的vertex shader;接着就是光栅化Rasterization，此时将点连接起来得到具体的线，进而得到区域;接着进一步处理，得到每一个点的具体颜色;最后根据这些有颜色的点生成具体的图像。

###### 1.5.1 Vertex Shader
**输入**：顶点坐标（Position），该坐标值是由glVertex* 或者是glDraw*传入的。

**输出**：顶点坐标，这个是经过几何变换后的坐标。

功能：简单的说就是把输入的**顶点坐标乘以（一系列）几何变换矩阵**转化为屏幕上的具体位置。每输入一个顶点（也就是glVertex\*每调用一次），Vertex shader都会被调用一次。Vertex Shader只知道处理顶点，它不知道这些顶点是做什么用的，也就是不知道这些顶点将来会被装配成什么图元（因为Vertex shader后面才会有图元装配的过程）。
  
当然，VS还可以接收**颜色，纹理坐标，雾坐标**等属性，并在内部对他们做一点点变化，然后再输出

###### 1.5.2 Fragment Shader
Vertex Shader主要对**顶点的属性**起作用，即多边形的转角点;而我们的Fragment Shader主要处理的是多个顶点之间的像素，它们是使用特定的规则在顶点间进行**插值**。比如:你想要多边形是完全的红色，此时你会将所有的顶点都设置为红色，但是如果你想要在顶点间设置渐变效果，你可以通过Fragment Shader来完成。

###### 1.5.3 Rasterization
光栅化（Rasterization）是把**顶点数据转换为片元**的过程，具有将图**转化为一个个栅格**组成的图像的作用，特点是每个元素对应帧缓冲区中的一像素。更加通俗的讲:光栅化是将几何数据经过一系列变换后最终转换为**像素**，从而呈现在显示设备上的过程。具体光栅化的过程可以[点击这里](https://www.jianshu.com/p/54fe91a946e2?open_source=weibo_search)，变换如下图:

![](./images/raster-process.png)

###### 1.5.4 [Frame Buffer Object](http://blog.csdn.net/wangdingqiaoit/article/details/52423060)帧缓冲对象
一直以来，我们在使用OpenGL渲染时，最终的目的地是**默认的帧缓冲区**，实际上OpenGL也允许我们创建自定义的帧缓冲区。使用自定义的帧缓冲区，可以实现**镜面，离屏渲染**，以及很酷的后处理效果。

在OpenGL中，渲染管线中的**顶点、纹理**等经过一系列处理后，最终显示在2D屏幕设备上，渲染管线的最终目的地就是帧缓冲区。帧缓冲包括OpenGL使用的**颜色缓冲区(color buffer)、深度缓冲区(depth buffer)、模板缓冲区(stencil buffer)**等缓冲区。默认的帧缓冲区由窗口系统创建，这个默认的帧缓冲区，就是目前我们一直使用的绘图命令的作用对象，称之为窗口系统提供的帧缓冲区(window-system-provided framebuffer)。

OpenGL也允许我们手动创建一个帧缓冲区，并将渲染结果**重定向**到这个缓冲区。在创建时允许我们自定义帧缓冲区的一些特性，这个自定义的帧缓冲区，称之为**应用程序帧缓冲区(application-created framebuffer object )**。

同默认的帧缓冲区一样，自定义的帧缓冲区也包含颜色缓冲区、深度和模板缓冲区，这些逻辑上的缓冲区（logical buffers）在FBO中称之为**可附加的图像**(framebuffer-attachable images)，他们是可以附加到FBO的二维像素数组（2D arrays of pixels ）。

FBO中包含两种类型的附加图像(framebuffer-attachable): **纹理图像和RenderBuffer图像**(texture images and renderbuffer images)。附加纹理时OpenGL渲染到这个纹理图像，在着色器中可以访问到这个纹理对象；附加RenderBuffer时，OpenGL执行离屏渲染(offscreen rendering)。

之所以用附加这个词，表达的是**FBO可以附加多个缓冲区**，而且可以灵活地在缓冲区中切换，一个重要的概念是附加点(attachment points)。FBO中包含一个以上的颜色附加点，但只有一个深度和模板附加点。

![](./images/fbo.png)

##### 1.6 [离屏渲染](http://blog.csdn.net/qq_29846663/article/details/68960512)
OpenGL中，GPU屏幕渲染有两种方式:

###### 1.6.1 On-Screen Rendering (当前屏幕渲染) 
指的是GPU的渲染操作是在**当前用于显示的屏幕缓冲区**进行。

###### 1.6.2 Off-Screen Rendering (离屏渲染)
指的是在GPU在**当前屏幕缓冲区以外开辟一个缓冲区**进行渲染操作。

当前屏幕渲染不需要额外创建新的**缓存**，也不需要**开启新的上下文**，相对于离屏渲染性能更好。但是受当前屏幕渲染的局限因素限制(只有自身上下文、屏幕缓存有限等)，当前屏幕渲染有些情况下的渲染解决不了的，就使用到离屏渲染。相比于当前屏幕渲染，离屏渲染的代价是很高的，主要体现在两个方面：
<pre>
(1）创建新缓冲区:要想进行离屏渲染，首先要创建一个新的缓冲区。
(2)上下文切换:离屏渲染的整个过程，需要多次切换上下文环境：先是从当前屏幕（On-Screen）切换到离屏（Off-Screen），等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的。
</pre>

#### 2.深入GPU百科全书
##### 2.1 方程与几何
对于图形学来说，外形的改变就是多边形的改变，进一步来讲就是**顶点坐标**的变化。而顶点坐标的变化，是可以通过方程来描述的。只要通过改变这些方程的参量,就能够产生不同的图形。

![](./images/vector.png)

其实在有关方程的问题上，要做的事情不止翻译这么简单。**首先**要把方程的结果转化成坐标，**然后**还要对这些坐标进行处理，把它们转化成实实在在看得见的点，再把这些点放在正确的位置上，对他们进行细腻的调整以达到最佳的效果，**最后**再把它们连起来形成表面。比如下面是DX10的几何处理过程:

![](./images/dx10.jpg)

而下面将讲解我们的主角GPU:

![](./images/gpu1.png)

现代GPU的几何单元具备**完整的setup、顶点调节、多变性调节、基本光照调整以及相关的材质调节**等多种功能。通过几何单元的动作，程序员创造的抽象数学将被还原成实在的可视空间几何体，这些几何体还会进一步的根据程序要求被调整到适当的位置，以达到微调模型外形实现不同效果的目的。

![](./images/dme.gif)

用正常的汉语来说，就是**几何单元**替我们看懂了那些晦涩的数学公式，计算了它们的结果，然后在我们可以看到的地方把那些结果所表示的点全部标了出来，接着再根据程序的要求调整它们的位置，最后把这些点全部连起来形成了物体。

##### 2.2 GPU的角色转化
###### 2.2.1 概述
要说真正的的几何处理能力，其实GPU一开始是不具备的。**GPU的前身显卡**所充当的角色曾经是单纯的画笔，CPU指哪里，显卡就画哪里，CPU让画什么显卡就画什么。第一代GPU革命虽然让CPU**丧失了对Tranform和lighting**的霸占，但第一代GPU依旧**不具备**完整的几何调节能力。顶点的位置依旧是CPU说了算，所有关于**几何形状的计算**全部由CPU来完成，而且**一旦确定就不能再行更改**。比如下图,顶点的生成就不在图形流水线过程之中，即不由GPU控制:

![](./images/no-vertex.png)

###### 2.2.2 Vertex Shader阶段
随着模型精度的一步步提升，**顶点的数目**也跟着多边形的增长一步步的激增，终于到了CPU无法负载的地步。于是，第二代GPU非常彻底的断灭了CPU复辟夺权的企图——它通过**Vertex Shader彻底拿走了CPU调节几何的能力。现在，CPU就只剩下最原始的生成顶点这一功能了**。Vertex Shader的出现，让GPU第一次有了独立自主的调节模型内顶点的能力，也让GPU内部出现了像素处理和Trangle Setup之外的新鲜事物。

###### 2.2.3 Geometry Shader阶段
第一代的VS(Vertex Shader)虽然让GPU具备了一定的几何处理能力，但它并不出色，时不时的还要CPU过来扶一把。直到DirectX 10将所有Shader执行单元统一，并由此带来了更加广泛的Shader应用领域之后，**Geometry Shader**的出现才让GPU彻底替我们背了跟数学缠斗的黑锅。
  
到了DirectX 11 GPU当中，我们发现了几何部分的重大变化——除了熟悉的**setup**(在顶点被摆放妥当之后，setup单元接下来要做的事情就是连接。按照正确的规则将顶点连接起来之后，物体的外形也就被确定下来了。正如当你抬头仰望星空时，你看到的永远都只是一颗一颗的星星，只有把它们连起来，**星座**才会出现，setup输出三角形的目的也在于此)之外，Tesselation和功能更加完善的Geometry Shader来了。

区别于以往的传统的处理方式，新的**Geometry Shader+Tesselation**允许**GPU自己生成新的顶点并完成设置工作**。现在CPU还是干以前一样的工作，基本模型的顶点还是由他来生成，而且数量也没有变化，只不过当顶点再次流入到GPU之后情况会发生些许变化。现在的几何单元会先查看程序中是否包含一种叫做**镶嵌系数**的新东西，如果没有，则几何单元重复常规过程；如果有，则几何单元将会进入一个全新的状态:

**首先**，CPU生成的模型依旧不变，几何单元在阅读了同顶点模型信息一起传过来的**镶嵌系数及方程**之后，会先让**Geometry Shader中的Hull Shader**对曲面要求进行解析，按照要求寻找其中的**控制点**，然后让Tesselator单元根据系数的要求生成全新的点(注意:setup一旦将顶点连接完成几何结构的模型构筑，就等于房子原有的结构已经造好。这时候程序如果有更高的几何外形要求而需要更多的多边形，比如实现浮雕效果，Tessellation单元就会根据需求重新生成新的顶点，并与Domain Shader一起将新的顶点放置到合适的位置上，以完成对旧有多边形的“碎裂”过程)，新生成的点全部在旧有顶点范围之内，当这些新生成的点被Tesselator单元送出之后，Hull Shader会根据程序的镶嵌说明把它们送到新的位置上并完成必要的调整，最后再**由Domain Shader**把Hull shader调整好的镶嵌点变成新的顶点输送给setup。下面给出Hull Shader和Tesselation Pipeline的处理过程:

![](./images/hull-shader.png)

![](./images/tessellation.png)

##### 2.3 顶点引证（Vertex Instancing)
对大数量的拥有**重复性但却有不能完全相同**的模型来说，程序可以先创建一个通用模型的基本信息，然后将模型的基本信息封装成一个`位于基础索引内`的只读信息包，这就是网络水军们得通用帖子模板。几何单元可以通过`SV_InstanceID`的索引来快速读取通用模型信息，并跳过重复的创建过程，这就是著名动作ctrl-V。接下来只要对通用模型进行一定的LOD修改等常规动作，即可完成一个全新模型的输出。

##### 2.4 三维转换
在图形流水线过程中，真正的具备3维空间意义的处理过程，却**只有几何单元一处而已**，当几何单元处理过图形数据之后，后续的所有步骤将`全部在2维空间中进行`，再也没有实在对应空间的3D坐标了。我们看到的所谓3D图形效果的处理，其实几乎都是在2维空间中完成的。

##### 2.5 Vertex Texture和Vertex Lighting
Vertex Texture和Vertex Lighting会按照程序要求提前访问**材质库**，并为整个模型附上最基本的**带光照信息的底层纹理**，就好像把框架模型变成一座石膏雕像一样。到这一步，常规的几何过程基本上就算完成了。

##### 2.6 光栅化
###### 2.6.1 光栅化原因
把三维世界变成二维，就因为你面前的显示器**只能显示二维的画面**。同时，Rasterizer（光栅化）收割3D世界所要面对的第一个问题，就是西方写实派美术的基础——**线透视关系**。

###### 2.6.2 Transform操作处理透视关系
Transform词如其意，就是指对立体物体的变形。三维空间中的物体会自然地呈现三维坐标系的特征，其上的每一个点均会带有完整的三维坐标向量。想要将这些点固定在二维世界中**同时还要符合透视关系**，这就是Transform的工作。

透视关系，其实归根结底还是数学关系，物体到投影面的形态属于标准欧式几何，人们可以用典型规范的数学手段加以描述，于是将三维物体二维化的过程，便可以通过方程变换的形式加以完成了。经过计算，**所有三维空间的点都可以在保证与观察者也就是摄像机位置存在正确关系的前提下，变成一个没有真实Z轴的二维点**。简单地说，就是可以被装进画里了。

由于这是整个光栅化过程的核心，因此，**光栅化也经常被称作坐标变换**。不难发现，这种基于数学关系的投影转变跟摄影有着非常接近的关系。如果没有Transform过程，我们甚至可以直接将坐标变换过程理解成是在拍照。

###### 2.6.3 为什么有进行光栅化
由于透视关系的缘故，我们显示器所在的固定视野会导致视线中**大量遮蔽关系**的出现，立体世界中那些被遮挡的以及因为角度问题背对摄像机视野的面将无法被我们所看到，`如果不进行光栅化过程`，不考虑视觉关系就直接将几何单元生成的空间全部根据需要进行**材质和像素填充**，渲染那些看不到的部分对于GPU来说显然是一种极大的资源浪费。

另外，光栅化之后GPU所要处理的操作，将会是**像素和材质**，**由于整个渲染过程所操作的最基本单位是像素，所以光栅化操作所转化的二维画面，其实就是一个以像素为基本单位构成的平面。后续的颜色处理，将围绕着这个平面内所有的像素展开。如果不进行光栅化操作，而是继续维持可操作的顶点和几何模型，未被冻结的几何外形将无法为像素和材质的操作提供保护**。世界原本是美丽的，但错误的构筑世界的工作配合却可能会毁掉这种美丽。下面是光栅化的过程:

![](./images/texure-process.jpg)


#### 2.Chrome官方文档涉及的基础知识点
<pre>
bitmap:内存中像素值的缓冲区(主内存或者GPU的RAMA)
texture:GPU中3D模型的一块bitmap   
texture quad:其对应于一个四点多边形，比如正方形。如果你想要以平面矩形的方式去展示纹理的时候，可以是2D或者3D，这也是我们在合成的时候做的事情。
invalidation:document中某一个区域被标注为脏数据，也就意味着它需要渲染(painting)。
painting：渲染阶段，RenderObjects调用GraphicsContext的API去重新绘制自身的阶段
rasterization：渲染阶段，当RenderLayers的bitmap的后端存储已经写好。在RenderObjects调用GraphicsContext的时候会被立即调用，但是如果在使用SkPicture监听渲染painting的时候可能延迟，而栅格化将交给SkPicture。
compositing：将RenderLayer的纹理上传到最终的屏幕上作为图片(final screen image)显示的时候
drawing：渲染的一个阶段，用于将像素绘制到屏幕上。即将最终的屏幕图像(final screen image)绘制到屏幕上。
backbuffer:当使用双缓存的时候，代表需要写入的屏幕缓存，而不是当前正在展示的这个缓存
frontbuffer:当使用双缓存的时候，代表正在展示的这个缓存，而不是需要写入的屏幕缓存
swapbuffers:调换缓存
Frame Buffer Object:OpenGL团队提供的机制能够在当前屏幕以外的地方写入，就像常规屏幕缓存一样(backbuffer)。这对于我们很有用，因为我们能够渲染到纹理中，然后合成这些纹理。
damage:因为用户的操作或者程序原因(JS改变了style)导致屏幕的某一个区域已经失效(脏化)。这个区域在repaint的时候需要更新
retained mode:图层系统通过该方法来维持一个将来用于渲染的对象的完整模式。web平台会保持对该模式下DOM的引用。平台(比如浏览器)会时刻监控DOM的状态以及相应的API(JS操作该DOM)能够修改或者查询当前的状态，浏览器能够在任何时候通过JS来修改该模式的数据。
immediate mode:图层系统不会监听整个屏幕的状态，而是在接受到指令的时候立即执行并不在关注该指令。为了redraw整个场景，所有的命令都必须重新发送。Direct3D就是这种模式，因为它是Canvas2D
context virtualization:GPU进程不会为每一个指定的命令缓冲区客户端创建一个驱动级别的GL上下文。多个客户端可能有一个共享的真实的上下文，同时在解析GL命令的时候将特定的客户端的GL状态保存为一个期望的状态。我们将这个阴影状态(shadowed state)叫做“virtual context”即虚拟上下文。这在Android平台上的某些驱动中能够避免很多bugs和一些性能问题(GL中慢上下文切换,多场景FBO渲染中的同步问题,使用共享组的crash)。Chrome在一些驱动中通过GPU黑名单文件启动了上下文虚拟化。
</pre>


#### 1.Chrome的合成器
Chrome的合成器是一个软件库，其用于管理GraphicsLayer树，同时协调**每一帧**的生命周期,它独立于Blink。我们知道渲染存在于两个阶段:首次渲染和合成。这允许合成器在每一个合成层的基础上做一些额外的工作。比如，合成器在合成bitmap之前通过css的transform为每一个合成层的bitmap实现相应的动画。而且，因为每一个合成层的渲染和合成操作是解耦的，某一些合成层失效只会造成这些失效的层的内容单独被渲染，然后和以前没有变化的合成层一起被合成。

每次浏览器需要一个新的帧的时候，合成器就会合成这些bitmap。这里指的**合成(drawing)**是:合成器将这些bitmap合成为屏幕上的最终图像，而**渲染(painting)**指的是:某一个层的后端存储被填充，例如bitmap的软件栅格化或者纹理的硬件栅格化。但是注意:最后生成的bitmap都是通过Browser进行绘制的。

#### 2 Chrome软件渲染通过[Browser进程完成](https://sites.google.com/a/chromium.org/dev/developers/design-documents/gpu-accelerated-compositing-in-chrome)
Webkit在软件渲染网页的时候是从根层级遍历整个RenderLayer层级。而Webkit代码库包含两个不同的方式来渲染页面的内容，一个是软件路径还有一个硬件加速路径。软件路径是传统的渲染模式。

软件路径是遵循**从后到前(先negative后positive)**的顺序渲染所有的RenderLayers，RenderLayer层级树是从顶层递归调用，然后在每一个节点上都会执行**RenderLayer::paintLayer()**方法，而该方法会执行下面的步骤:
<pre>
1.判断该layer是否和已经变更的层级有重叠，如果没有重叠直接进行下面的步骤
2.递归调用该层级的negZOrderList集合中每一个layer的paintLayer()方法
3.该RenderLayer下的所有RenderObjects开始绘制自己。这是通过首先递归调用创建该层级的RenderObject的方法，而如果在递归调用的时候发现了新的RenderLayer则停止
4.递归调用该层级的posZOrderList集合中每一个layer的paintLayer()方法
</pre>

在这个模型中，RenderObjects将自己的内容绘制到目标bitmap是通过单个共享的GraphicsContext来完成(在Chrome中通过Skia完成)。注意:GraphicsContext本身没有层级的概念，但是为了正确绘制半透明的层级，我们需要知道:半透明的renderLayer下的RenderObjects绘制之前会调用GraphicsContext::beginTransparencyLayer()方法。在Skia的实现中,beginTransparencyLayer()调用后将会导致所有的后续绘制调用渲染到一个独立的bitmap中，这个独立的bitmap在半透明的RenderLayer下的所有RenderObjects被渲染后，同时调用了GraphicsContext的endTransparencyLayer()时将会和前一个bitmap合并。具体流程如下图:

![](./images/soft-rendering.png)

当所有的RenderLayers都已经被绘制到一个**共享的bitmap**之后，bitmap依然需要被绘制到屏幕中。在chrome中bitmap存在于共享内存中，而对共享内存的控制是通过IPC传递给Browser进程的。**Browser进程通过OS的窗口API(比如window上的HWND)负责将这些bitmap绘制到指定的Tab或者窗口中**。

#### 3.GPU渲染
合成器可以使用GPU去绘制页面到屏幕上，这和传统的软件渲染有很大的区别，在传统的软件渲染中Renderer进程将将页面内容产生的bitmap通过IPC/共享内存传递给Browser进程，由Browser进程控制页面的展示。

在硬件加速模式下，合成操作在GPU中通过调用特定的3D APIs完成(在windows上是D3D，否则是GL)。渲染进程的合成器使用GPU绘制页面中的特定区域(所有合成层，都是相对于视口进行定位的)到一个特定的bitmap中，这个bitmap对应于最终的屏幕内容。不过，**但是**最后还是通过IPC传递给Browser进程去渲染。


#### 4.GPU进程
在我们深入研究合成器产生的GPU命令之前，我们需要理解渲染进程是如何向**GPU发送命令**的。在Chrome的多进程模型中，我们有一个独立的GPU进程。GPU进程之所以存在，主要是因为安全原因。而Android是一个特例，在Android中GPU进程是做一个**线程存**在于Browser进程中的，但是Android中该GPU线程和其他平台的原理是一样的。

因为渲染线程的**sandbox特性**，渲染进程无法直接调用OS提供的3D APIs(GL / D3D)。而GPU进程能够在一个独立的进程中访问3D APIs，同时渲染进程虽然遵循sandbox特性，也将能够通过GPU进程访问它们。GPU进程遵循客户端-服务器(client-server)模式:
<pre>
client(渲染进程中执行的代码):不能直接访问3D APIs，产生命令并序列化放入到命令缓冲区。这个缓冲区是client/server进程共享的内存空间
server(GPU进程，运行于相对宽松的sandbox环境,允许访问3D APIs):从共享内存中获取序列化的命令，解析它并调用特定的方法
</pre>

模式如下图:

![](./images/server-client.png)

下面主要针对上图的概念做深入说明:

- 命令缓冲区(Command Buffer)

  GPU进程获取到的命令格式和GL ES 2.0 API指定的几乎是相同的。大多数GL调用都没有返回值，因此server(GPU进程)和client(渲染进程)可以异步执行，此时性能是非常低的。任何client和server的**同步化(synchronization)**，例如client通知server有新的任务需要处理，这种机制都是通过IPC机制来完成的。
  从**client的角度**来说，可以选择直接往命令缓冲区写入命令或者使用GL ES 2.0 API的客户端库，该库能够提供序列化。为了方便，目前合成器和WebGL都使用GL ES的客户端库。
  从**server的角度**来说，从命令缓冲区获取到的命令都会被ANGLE转化为对桌面OpenGL或者Direct3D的调用。

- 资源共享与同步

  除了为命令缓冲区提供存储空间,Chrome使用共享内存来解决在client与server端传递大量的资源，比如构建textures的bitmap,vertex arrays等。

- 命令缓冲区多路复用(Command Buffer Multiplexing)

  当前Chrome实例只有单个GPU进程，它会处理所有来自于**渲染进程和插件进程**的请求。GPU进程能够在**多个命令缓冲区之间复用**，而此时每一个命令缓冲区都有自己独立的渲染上下文(Rendering Context)。
  每一个渲染进程可以有多个GL来源，比如WebGL Canvas元素直接创建一个GL命令流。对于那些内容直接在GPU中创建的合成层来说他的工作原理如下:
 他们的内容直接被渲染到纹理(texture,使用Frame Buffer Object)中，因此合成器可以在渲染GraphicsLayer时候直接使用它，而不是将它的内容直接渲染到后备缓冲区。
 注意:为了让合成器的GL上下文能够直接访问非屏幕产生的GL context纹理(texture)，所有GPU进程使用的GL contexts在创建的时候就能够共享资源。如下图


![](./images/buffer.png)

GPU进程设计模式有如下优点:
<pre>
安全(Security):所有的渲染逻辑依然在sanbox化的渲染进程中，对于3D APIs的访问只能通过GPU进程完成
健壮性(Robustness):GPU进程的crash(错误的驱动)不会导致浏览器崩溃
均匀性(Uniformity):以OpenGL ES 2.0标准作为浏览器渲染API,因此在不同OS平台上Chrome能够使用同一个易于维护的代码库。
并行性(Parallelism):渲染进程能够很快的将命令发送到命令缓冲区并立即返回CPU密集型的活动。这样能更好的在多核机器上使用两个进程。
</pre>

#### 5.线程化的合成器
合成器是基于GL ES 2.0客户端库来实现的，它会代理对GPU进程的访问。如果一个页面是通过合成器渲染的,那么它所有的像素都是直接**通过GPU进程被渲染(drawing)到window的后端存储(drawing!=painting)(绘制到屏幕的过程依然是Browser进程做的)**。

理论上，每一个线程化的合成器能够从主线程获取足够的信息，进而独立的产生帧响应用户的输入，即使主线程处于繁忙状态，无法获取足够的信息。但是实际上也就意味着，它存有cc LayerTree的副本，SkPicture将会监听当前viewPort特定区域的RenderLayer的变化

- 监听:从Blink角度的重绘(painting)
  我们关心的区域是视口中SkPictures监听的部分。当DOM改变的时候，比如某些元素的style和以前主线程帧中的值不一样，而且处于无效状态。Blink将会重绘无效的RenderLayer区域到基于SkPicture的GraphicsContext。这不会产生新的像素pixel，而是产生一系列的Skia命令，使用这些命令就可以产生这些新的像素pixel。这些Skia命令主要用于合成器调配。
- 提交:交由合成线程
  线程化的合成器的主要属性是对主线程状态的操作，这样为了产生一个新的帧就不需要询问主线程任何数据。线程化的合成器主要包括两个部分，主线程部分以及"impl"部分。主线程有一个LayerTreeHost，他是LayerTree的一个副本，而impl部分是LayerTreeHostImpl，其也是LayerTree的一个副本。

  概念上说两个LayerTree是独立的，合成器(impl)进程的数据副本能够产生帧，而且不需要和主线程产生任何交互。这也意味着:主线程能够长时间支持javascript，同时合成器能够重新绘制以前在GPU中提交的内容而不用打扰主进程。

  为了产生新的帧，合成线程需要知道如何修改他的状态，比如更新Layer的状态来响应浏览器的scroll事件。因此，一些input事件，比如scroll,需要从Browser进程传递到合成层，继而传递到渲染进程的主进程。因为input/output事件都是线程化的合成器来管理，其能够保证用户输入的视觉响应。除了滚动事件，合成器能够处理任何页面更新，而不需要询问Blink去重新绘制。因此，css动画或者css filter都可能造成合成器驱动的页面更新。
  
  两个LayerTree是通过一系列的消息机制，比如commit来保证内容的一致性，而内部是通过合成器的调度这来完成的。每次commit都能把主线程的状态转移到合成器的线程(包括更新后LayerTree,SkPicture的监听数据等等)，每次同步都会阻塞主线程。这是主线程更新特定的帧的最后一步，也是它唯一需要参与的一步。

  合成器在一个独立的线程中运行，这允许合成器自动更新LayerTree层级数据而不需要通过主线程来完成。但是主线程最终也是需要，比如scroll offset等数据(Javascript需要知道当前视口的滚动位置)。因此，commit也将负责将基于合成器线程的layerTree数据更新到主线程的树中，同时执行其他的任务。

  有意思的是,这种架构使得Javascript的touch事件阻止合成层的滚动，但是scroll事件却不会。JS能够在touch事件中调用preventDefault()，但是在scroll事件中是不允许的。因此合成器在没有询问JS(主线程)之前是不能滚动页面的，如果这个JS将会取消接下来的touch事件。Scroll事件在另一方面，是不允许被阻止的，而且它是被异步发送到JS中的。因此，合成线程能够立即滚动，而不需要关心主进程是否立即处理scroll事件了。

  - 树的激活(Tree Activation)
    当合成线程从主线程获取到了一个新的LayerTree，它会检查这个新的树来看那一部分已经失效了，进而继续栅格化这部分LayerTree。此时处于活动态的树依然是先前的那一棵LayTree，而pending Tree指的就是当前的新的LayTree,也就是内容刚被栅格化的这个LayTree树。
    为了保证展示的内容的一致性，pending Tree只有在它可见的时候(处于视口中)才会被激活。某一棵树从激活态被转化为pending态就是激活过程。等待栅格化的内容准备完成的过程中，用户能够看到少量的内容，但是这些内容很可能是过期的。如果没有内容可用，Chrome就会显示一个空页面。
    需要注意，我们很可能滚动到一个已经处于激活态的栅格区域，因为Chrome只会监听RenderLayer的SkPictures部分。如果用户正在滚动到一个未被监听的区域，此时合成线程将会通知主线程去监听和提交额外的内容，但是如果这个新的内容不能在指定的时间被监听，提交和栅格化，用户将会看到checkerboard区域。
    为了减少checkerboard出现的机会，Chrome能够为pending tree在获取到高分辨率的内容之前快速栅格化低分辨率的内容。如果当前pending tree视口中低分辨率的内容比当前视口中的内容要好，那么就会被替换。
    这个架构将会使得栅格化和其他的帧的产生流分离。它启动了很多技术用于改善图层系统(graphics system)的响应性。图像解码和resize操作都是异步的，而这些操作在以前都是通过主线程在patting的时候产生的，而且是非常昂贵的。异步的纹理传输系统是在impl方实现的。
    
  - Tiling
    栅格化页面中所有的layer是很耗CPU和内存(RAM，每一个软件渲染的bitmap，VRAM用于纹理存储)的。为了避免栅格化整个页面，合成器将大多数的页面内容layer绘制为不同的tiles，然后基于每一个tile来栅格化layer层。
    网页内容的层级tiles通过多个元素进行优先级排序，比如距离视口的距离和出现在屏幕中的大概时间。然后GPU内存根据这些指标进行分配。所有的tiles将会根据优先级从SkPicture的监听数据获取内存分配。注意：tiling对那些内容已经存在于GPU的不是必须的，比如硬件加速的video或者WebGL

  - 栅格化:**渲染(painting)**从cc/Skia角度
    SkPicture对于合成线程的监听会收到相应的GPU返回的bitmap主要通过两个方式:通过Skia’s软件渲染将它绘制到bitmap中，进而作为纹理上传到GPU中。还有一种方式就是通过Skia’s OpenGL的后端的存储，继而直接传递给GPU作为纹理。
    对于Ganesh-rasterized的Layer来说，SkPicture和Ganesh一起，而产生的GL命令流将会通过GPU进程的命令缓存进行处理。任何时候，合成线程只要决定去栅格化任何tiles将会产生GL命令，这些tiles将会打包到一起进而避免在GPU中过高的栅格化开销。这部分内容可以查看GPU加速栅格化设计文档。
    对于软件栅格化的Layer来说，整个渲染是以渲染进程和GPU进程的内存共享为基础的。Bitmap将会通过资源传送机制上传到GPU进程。因为软件栅格化是非常昂贵的，所以合成并不会通过合成线程自己完成(否则可能阻塞active Tree产生一个新的帧)，而是在一个合成栅格化的工作线程中。多个栅格化工作线程能够加速软件栅格化。每一个工作线程会从tile优先级队列的头部获取到数据。而完成以后的tiles将会作为渲染纹理传递给GPU。
    bitmap产生的渲染纹理上传开销在内存带宽大的平台上是微不足道的。这回导致软件渲染Layer的性能低下，进而对于上传用于硬件栅格化bitmap产生一定的性能损耗(比如图片数据或者CPU渲染的遮罩)。以前Chrome有很多不同的纹理上传机制，而最成功的是异步传输，整个过程是在GPU进程的worker线程中完成的(或者Browser进程中的一个额外的线程)。
    一个好的解决纹理传输问题的方法就是使用0复制的缓冲，整个缓冲区是CPU和GPU共享的，这是大多数设备以前统一的内存模式。而Chrome目前并没有使用该方式，未来可能会采用这种实现。更多内容可以[查看这里](https://docs.google.com/document/d/1SaTYTBvHWWDKA3MPJPpQ-79RNgdS4Xu4g3KiD39VQjU/edit?usp=sharing)。
    同时你也需要注意:我们也可以采用第三种方式来处理，即采用GPU来栅格化。将每一个Layer的内容在painting的时候直接栅格化到一个后端缓冲区，而不是提前栅格化到一个纹理(texture)中。这种方式有利于节省内存空间，因为没有产生额外的中间纹理，同时也会提升性能(在painting的时候节省了后端缓冲中的纹理副本)。但是，这种方式对于性能也有损失，特别是当纹理texture有效缓存了Layer内容的情况下(因为现在它需要重绘re-paint每一个帧)。这种"直接写到后端缓存的情况"或者"直接写入到Ganesh的模式"在2014.5月之前并没有实现。你可以查看[GPU栅格化设计方案](https://docs.google.com/document/d/1Vi1WNJmAneu1IrVygX7Zd1fV7S_2wzWuGTcgGmZVRyE/edit)了解更多信息。

![](./images/raster.png)

- 在GPU中合成(drawing), Tiling, and Quads
  当所有的纹理已经填充了，渲染页面内容的过程就是深度遍历Layer层级的过程，同时发出GL命令去绘制每一个Layer的纹理到特定的帧缓冲中(Frame Buffer)。
  合成(drawing)屏幕中的某一个层的过程其实就是合成页面中的每一个tiles。tiles其实就是2\*2的方格。合成器会产生quads和一系列的渲染passes(是一个简单的数据结构，其包括一系列的quads)。而真实合成的GL命令是针对quads来完成的。这是quad实现的抽象方式，因此针对合成器我们可以写入到non-GL的后端存储(一个显著的non-GL实现就是软件合成器)。为每一个render pass合成适量的quads来产生视口内容，进而产生动画(transform)，同时在render pass的quad集合中绘制每一个quad。
  注意:通过深度优先遍历的方式能够为CC Layer产生正确的z-index顺序，同时cc Layer中的多个RenderLayers的z-index顺序是通过RenderLayer中的RenderObject的绘制顺序遍历来保证的。




 参考资料:

[双缓冲(Double Buffer)原理和使用](http://blog.csdn.net/xiaohui_hubei/article/details/16319249)

[GPU Accelerated Compositing in Chrome](https://sites.google.com/a/chromium.org/dev/developers/design-documents/gpu-accelerated-compositing-in-chrome)

[OpenGL是什么?](https://www.zhihu.com/question/51867884)

[Textures objects and parameters](https://open.gl/textures)

[OpenGL学习脚印: 帧缓冲对象(Frame Buffer Object)](http://blog.csdn.net/wangdingqiaoit/article/details/52423060)

[OpenGL基本概念](http://blog.shengbin.me/posts/opengl-concepts)

[OpenGL ES基础视频](http://v.youku.com/v_show/id_XNjQzMTkyODUy)

[OpenGL 各个shader的作用和区别](http://www.360doc.com/content/18/0204/19/52580135_727714374.shtml)

[Graphics pipeline](https://en.wikipedia.org/wiki/Graphics_pipeline)

[光栅化](https://baike.baidu.com/item/%E5%85%89%E6%A0%85%E5%8C%96/10008122?fr=aladdin)

[图形学 光栅化详解（Rasterization）](https://www.jianshu.com/p/54fe91a946e2?open_source=weibo_search)

[光栅化的深入理解](http://blog.csdn.net/waitforfree/article/details/10066547)

[图形学 坐标系空间变换](https://www.jianshu.com/p/09095090c07f)

[OpenGL学习脚印: 帧缓冲对象(Frame Buffer Object)](http://blog.csdn.net/wangdingqiaoit/article/details/52423060)

[离屏渲染](http://blog.csdn.net/qq_29846663/article/details/68960512)

[GPU大百科全书 第一章：美女 方程与几何](http://blog.csdn.net/pizi0475/article/details/7523490)

[GPU大百科全书索引（有助于理解openGL工作流程）](https://www.cnblogs.com/BigFeng/p/5006014.html)

[GPU大百科全书 前传 看图形与装修的关系](http://blog.csdn.net/pizi0475/article/details/7523481)

[GPU大百科全书 第二章 凝固生命的光栅化](http://blog.csdn.net/pizi0475/article/details/7523491)