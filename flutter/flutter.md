
# flutter调研
## 优点： 
1. 一致性
和 react-native 、 weex 不同，Flutter 的控件不是通过原生控件去实现的渲染，而是由 Flutter Engine 提供的平台无关的渲染能力，也就是Flutter 的控件和平台没关系。
为了尽可能不依赖平台特性， Flutter在Dart虚拟机之上实现了全套的UI组件、手势管理、资源管理、并采用GPU直接渲染的方式。随之而来的结果就是高效的渲染性能及比源生UI更加灵活的表现力。在大大减少了适配工作的同时，很好的保障了低端设备的流畅度。
 react-native 通过将 JS 里的控件转化为原生控件进行渲染，所以 rn 里的控件是需要依赖原生平台的控件，所以不同系统之间原生控件的差异，同个系统的不同版本在控件上的属性和效果差异，组合起来在后期开发过程中就是很大的维护成本,而 Flutter 的控件特性决定了它没有这些问题。
Flutter 这样实现也有坏处，那就是当你需要使用平台的控件作为混合开发时，Flutter 的成本和体验无疑被放大 ，这一点上 react-native 反而有着先天的优势。
2. 性能
Flutter 的性能一般情况下是比 react-native 好。
[ 《Flutter vs React Native vs Native：深度性能比较》](https://mp.weixin.qq.com/s?__biz=Mzg3NTA3MDIxOA%3D%3D&chksm=cec65278f9b1db6e806f8ce6383f3b5c5de3e8ec313bed83b3e321b07459901829c1630a0dc3&idx=1&mid=2247484445&scene=21&sn=a9b7d5a5ef2b62cb3328adb440ba7f15#wechat_redirect)
不同于端上UI库的分层渲染，然后叠加的思路，flutter采用的是内存计算最终渲染指令，GPU栅格化后直接渲染，通过统一的图形库接口(skia)，在Android端和iOS端都可以获得非常高的渲染性能。另一方面在CPU的使用上，由于release模式下Dart使用AOT模式，所有的代码都会编译成机器码，没有类似Weex的JSCore中间层，直接在CPU上运行，代码执行性能损耗极少。
3. 快速开发
作为一种在虚拟机上运行的语言，Dart天然支持两种运行方式,即实时编译的JIT和预编译为机器码的AOT。通过在Debug环境中使用JIT模式，flutter可以实现秒级的Hot Reload。搭配完备的IDE插件及**一套代码，多端运行**的设计理念，开发确实非常敏捷迅速。
4. 灵活、极具表现力的UI
得益于从底层全新构建的UI组件，Flutter涵盖了包括Android和iOS两种风格组件。通过组合大于集成的构建思路，Flutter可以搭建起想要的任何界面。在此基础上，不受平台框架限制的动画能力及完全接管的滚动、转场效果、手势分发能力，使得开发者可以获得比源生UI库更大的灵活性。
- - - -
## 缺点
1. 技术新:
	* 目前的Flutter迭代速度非常频繁。
	* 社区不成熟，国内各大公司各自为战，还没有较为权威的开发社区。
	* 技术栈较深， Flutter技术栈涉及Dart语言、Dart Framework、 Dart VM、 skia渲染引擎等多层庞大的代码库，对于想要使用Flutter快速开发的团队显然与出发点不同，任何的踩坑或许只能依赖于官方的修复和处理。
2. 动态化能力的缺失:
	* Google最初的设想只是跨端，尽管社区很多加入动态化能力(Dart在Release包中使用JIT模式运行)的呼声，但是官方似乎顾虑重重，明确表示动态化不是当前开发方向。
	* 苹果的审核机制不会允许Dart的动态化能力。
	* Flutter的高性能很大程度上源自Dart语言AOT模式下的机器码运行方式，即便有一天官方支持了动态化能力，运行性能肯定也要大打折扣。
3. 混合开发的困境:
	* Flutter并未考虑过Native&Flutter混合开发的场景，导致混合开发有非常多的问题。 这些问题包括混合转场、Flutter容器的复用、手势的分发、Native UI组件的参与渲染等；同时还有CI/CD工程化方案的缺失，闲鱼做了大量的改造和定制，才使得Flutter能够胜任混合开发。
	* 包大小问题: 在一些大型App中，包大小是很敏感的指标。 在iOS端，最简单的Flutter项目将增加10M以上的包大小，随着业务逻辑的增加和资源的加入(icon、font等)包大小仍会上升；在Android端，问题相对没有那么严重，由于skia库的内置，Android端大概增加4M+的包大小。
- - - -
## 技术栈分解
![01](/var/folders/vm/tgw9fg194vdcd86gmt4rpt640000gn/T/net.shinyfrog.bear/BearTemp.nSLzbL/01.png)
1. **Dart语言**
Dart语言产生较晚，充分参考了其它语言，加入了很多高级特性。典型的有支持async/await关键字的并发线程模型、优秀的内存管理机制、支持AOT和JIT两种运行方式等。
2. **Dart Framework**
纯 Dart实现的 SDK
Dart Framework提供了各种内置UI组件的实现，允许开发者通过组合构建业务界面，原生支持Android和iOS两套UI风格；同时拥有包管理机制，支持社区Package的开发和使用。
在显示环节， UI组件需要经过layout和build两步， Flutter中的Dart组件库采用了类似React的响应式架构，再此之上又通过优化，将layout和build的时间复杂度降低到了亚线性的量级。
底下两层：底层UI库，提供动画、手势及绘制能力
Rendering层：构建UI树，当UI树有变化时，会计算出有变化的部分，然后更新UI树，最终将UI树绘制到屏幕上
Widgets层：基础组件库，提供了 Material 和Cupertino两种视觉风格的组件库
3. **Flutter Engine：纯 C++实现的 SDK**
4. **engine层 skia渲染库**
Skia是一套开源的2D绘图库引擎，在不同的平台有不同的backend，在iOS上是openGL ES，而Android上面则是Vulkan。基于skia可以提供一致的高性能GPU渲染能力。
5. **engine层 Dart VM**
DartVM是Dart语言的运行时环境，支持AOT和JIT两种运行模式。内部实现了Dart的线程/多线程模型、内存管理、及Dart语言的多种高级特性。
6. Text:文字排版引擎
- - - -
更多详细信息及相关资源：[[Flutter技术调研报告](https://juejin.im/post/5c4e6dc66fb9a049eb3c516a)]



#ASSIGNMENT/IMD

