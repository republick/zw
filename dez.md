#**第二章 内存寻址**

我们知道，操作系统是一组软件的集合。但它和一般软件不同，因为它是充分挖掘硬件潜能的软件，也可以说，操作系统是横跨软件和硬件的桥梁。因此，要想深入解析操作系统内在的运作机制，就必须搞清楚相关的硬件机制——尤其是内存寻址的硬件机制。

操作系统的设计者必须在硬件相关的代码与硬件无关的代码之间划出清楚的界限，以便于一个操作系统很容易地移植到不同的平台。Linux的设计就做到了这点，它把与硬件相关的代码全部放在arch(architecture一词的缩写，即体系结构相关)的目录下，在这个目录下，可以找到Linux目前版本支持的所有平台，例如，支持的平台有arm、alpha,、i386、m68k、mips等十多种。在这众多的平台中，大家最熟悉的就是i386，即Intel80x86体系结构。因此，我们所介绍的内存寻址也是以此为背景。

##**2.1内存寻址**

曾经有一个叫“阿兰.图灵”的天才1，它设想出了一种简单但运算能力几乎无限发达的理想机器，这不是一个具体的机械设备，而是一个思想模型，可以用来计算能想象得到的所有可计算函数。这个有趣的机器由一个控制器，一个读写头和一条假设两端无限长的带子组成。工作带相当于存储器，被划分成大小相同的格子，每格上可写一个字母，读写头可以在工作带上随意移动，而控制器可以要求读写头读取其下方工作带上的字母。

这听起来仅仅是纸上谈兵，但它却是当代冯.诺依曼计算机体系的理论鼻祖。它带来的“数据连续存储和选择读取思想”是目前我们使用的几乎所有机器运行背后的灵魂。计算机体系结构中的核心问题之一就是如何有效地进行内存寻址，因为所有运算的前提都是先要从内存中取得数据，所以内存寻址技术从某种程度上代表了计算机技术。
> 1. 据说他16岁开始研究相对论，虽然英年早逝，但才气纵横逻辑学、物理学、数学等多个领域，尤其是数学逻辑上的所作所为奠定了现代计算技术的理论基础 。后来以他名字命名的“图灵奖”被看作计算机学界的最高荣誉。

###**2．1．1 Intel X86 CPU寻址的演变**

在微处理器的历史上，第一款微处理器芯片4004是由Intel推出的，那是一个4位的微处理器。在4004之后，intel推出了一款8位处理器8080，它有1个主累加器（寄存器A）和6个次累加器（寄存器B,C,D,E,H和L）,几个次累加器可以配对（如组成BC, DE或HL）用来访问16位的内存地址，也就是说8080可访问到64K内的地址空间。另外，那时还没有段的概念，访问内存都要通过绝对地址，因此程序中的地址必须进行硬编码（给出具体地址），而且也难以重定位，这就不难理解为什么当时的软件大都是些可控性弱,结构简陋，数据处理量小的工控程序了。

几年后，intel开发出了16位的处理器8086，这个处理器标志着Intel X86王朝的开始，这也是内存寻址的第一次飞跃。之所以说这是一次飞跃，是因为8086处理器引入了一个重要概念—段

8086处理器的寻址目标是1M大的内存空间，于是它的地址总线扩展到了20位。但是，一个问题摆在了Intel设计人员面前，虽然地址总线宽度是20位的，但是CPU中“算术逻辑运算单元（ALU）”的宽度，即数据总线却只有16位，也就是可直接加以运算的指针长度是16位的。如何填补这个空隙呢？可能的解决方案有多种，例如，可以像一些8位CPU中那样，增设一些20位的指令专用于地址运算和操作，但是那样又会造成CPU内存结构的不均匀。又例如，当时的PDP－11小型机也是16位的，但是其内存管理单元（MMU）可以将16位的地址映射到24位的地址空间。受此启发，Intel设计了一种在当时看来不失为巧妙的方法，即分段的方法。 在微处理器的历史上，第一款微处理器芯片4004是由Intel推出的，那是一个4位的微处理器。在4004之后，intel推出了一款8位处理器8080，它有1个主累加器（寄存器A）和6个次累加器（寄存器B,C,D,E,H和L）,几个次累加器可以配对（如组成BC, DE或HL）用来访问16位的内存地址，也就是说8080可访问到64K内的地址空间。另外，那时还没有段的概念，访问内存都要通过绝对地址，因此程序中的地址必须进行硬编码（给出具体地址），而且也难以重定位，这就不难理解为什么当时的软件大都是些可控性弱,结构简陋，数据处理量小的工控程序了。

几年后，intel开发出了16位的处理器8086，这个处理器标志着Intel X86王朝的开始，这也是内存寻址的第一次飞跃。之所以说这是一次飞跃，是因为8086处理器引入了一个重要概念—段

为了支持分段，Intel在8086 CPU中设置了四个段寄存器：CS、DS、SS和ES，分别用于可执行代码段、数据段、堆栈段及其他段。每个段寄存器都是16位的，对应于地址总线中的高16位。每条“访内”指令中的内部地址也都是16位的，但是在送上地址总线之前，CPU内部自动地把它与某个段寄存器中的内容相加。因为段寄存器中的内容对应于20位地址总线中的高16位(也就是把段寄存器左移4位)，所以相加时实际上是内存总线中的高12位与段寄存器中的16位相加，而低4位保留不变，这样就形成一个20位的实际地址，也就实现了从16位内存地址到20位实际地址的转换，或者叫**“映射”**。

段式内存管理带来了显而易见的优势，程序的地址不再需要硬编码了，调试错误也更容易定位了，更可贵的是支持更大的内存地址。程序员开始获得了自由。

技术的发展不会就此止步。intel的80286处理器于1982年问世了，它的地址总线位数增加到了24位，因此可以访问到16M的内存空间。更重要的是从此开始引进了一个全新理念—**保护模式**。这种模式下内存段的访问受到了限制。访问内存时不能直接从段寄存器中获得段的起始地址了，而需要经过额外转换和检查（从此你不能再随意存取数据段,具体保护和实现我们后面讲述）。

为了和过去兼容，80286内存寻址可以有两种方式，一种是先进的保护模式，另一种是老式的8086方式，被成为**实模式**。系统启动时处理器处于实模式，只能访问1M空间，经过处理可进入保护模式，访问空间扩大到16M，但是要想从保护模式返回到实模式，你只有重新启动机器。还有一个致命的缺陷是80286虽然扩大了访问空间，但是每个段的大小还是64k，程序规模仍受到限制。因此这个先天低能儿注定命不会很久。很快它就被天资卓越的兄弟——80386代替了。

80386是一个32位的CPU，也就是它的ALU数据总线是32位的，同时它的地址总线与数据总线宽度一致，也是32位，因此，其寻址能力达到4GB。对于内存来说，似乎是足够了。从理论上说，当数据总线与地址总线宽度一致时，其CPU结构应该简洁明了。但是，80386无法做到这一点。作为X86产品系列的一员，80386必须维持那些段寄存器的存在，还必须支持实模式，同时又要能支持保护模式，这给Intel的设计人员带来很大的挑战。

Intel选择了在段寄存器的基础上构筑保护模式，并且保留段寄存器16位。在保护模式下,它的段范围不再受限于64K，可以达到4G（参见段机制一节）。这一下真正解放了软件工程师,他们不必再费尽心思去压缩程序规模，软件功能也因此迅速提升。

从8086的16位到80386的32位处理器，这看起来是处理器位数的变化，但实质上是处理器体系结构的变化，从寻址方式上说，就是从“实模式”到“保护模式”的变化。从80386以后，Intel的CPU经历了80486、Pentium、PentiumII、PentiumIII等型号，虽然它们在速度上提高了好几个数量级，功能上也有不少改进，但基本上属于同一种系统结构的改进与加强，而无本质的变化，所以我们把80386以后的处理器统称为**80x86**。

###**2．1．2  80X86寄存器简介**

80386作为80X86系列中的一员，必须保证向后兼容，也就是说，既要支持16位的处理器，也要支持32位的处理器。在8086中，所有的寄存器都是16位的，我们来看一下80x86中寄存器有何变化：



- 	把16位的通用寄存器、标志寄存器以及指令指针寄存器扩充为32位的寄存器

- 	段寄存器仍然为16位。

- 	增加4个32位的控制寄存器

- 	增加4个系统地址寄存器

- 	增加8个调式寄存器

- 	增加2个测试寄存器

 下面介绍几种常用的寄存器。

**1.通用寄存器**

8个通用寄存器是8086寄存器的超集，它们分别为：EAX ，EBX ，ECX ，EDX ，EBP ，EBP， ESI及 EDI。这8个通用寄存器中通常保存32位数据，但为了进行16位的操作并与16为机保持兼容，它们的低位部分被当成8个16位的寄存器，即AX、BX…DI。为了支持8位的操作，还进一步把EAX、EBX、ECX、EDX这四个寄存器低位部分的16位，再分为8位一组的高位字节和低位字节两部分，作为8个8位寄存器。这8个寄存器分别被命名为AH、BH、CH、DH和AL、BL、CL、DL。因此，这8个通用寄存器既可以支持1位、8位、16位和32位数据运算，也支持16位和32位存储器寻址。

**2. 段寄存器**

8086中有4个16位的段寄存器：CS、DS、SS、ES，分别用于存放可执行代码的代码段、数据段、堆栈段和其他段的基地址。在80x86中，有6个16位的段寄存器，但是，这些段寄存器中存放的不再是某个段的基地址，而是某个段的选择符（Selector）。因为16位的寄存器无法存放32位的段基地址，段基地址只好存放在一个叫做描述符表（Descriptor）的表中。因此，在80x86中，我们把段寄存器叫做选择符。有关段选择符、描述符表将在段机制一节进行描述。

**3.指令指针寄存器和标志寄存器**

指令指针寄存器EIP中存放下一条将要执行指令的偏移量（offset ），这个偏移量是相对于目前正在运行的代码段寄存器CS而言的。偏移量加上当前代码段的基地址，就形成了下一条指令的地址。EIP中的低16位可以被单独访问，给它起名叫指令指针IP寄存器，用于16位寻址。标志寄存器EFLAGS存放有关处理器的控制标志，很多标志与16位FLAGS中的标志含义一样。 

**4．控制寄存器**

80x86有四个32位的控制寄存器，它们是CR0，CR1，CR2和CR3，主要用于操作系统的分页机制（参见下节）。其结构如图 2.1所示。

![](http://i.imgur.com/5DMLAKS.png)
 
这几个寄存器中保存全局性和任务无关的机器状态。

CR0中包含了6个预定义标志，这里介绍内核中主要用到的0位和31位。0位是保护允许位PE(Protedted Enable)，用于启动保护模式，如果PE位置1，则保护模式启动，如果PE=0，则在实模式下运行。CR0的第31位是分页允许位(Paging Enable)，它表示芯片上的分页部件是否被允许工作。由PG位和PE位定义的操作方式如图2.2所示。

![](http://i.imgur.com/5c48Tlq.png)	

使用以下代码就可以允许分页（AT＆T汇编语言参考2.5节）：

    movl %cr0, %eax
    orl  $0x80000000, %eax
    movl %eax, %cr0

CR1是未定义的控制寄存器，供将来的处理器使用。

CR2是缺页线性地址寄存器，保存最后一次出现缺页的全32位线性地址（将在内存管理一章介绍）。

CR3是页目录基址寄存器，保存页目录的物理地址，页目录总是放在以4K字节为单位的存储器边界上，因此，其地址的低12位总为0，不起作用，即使写上内容，也不会被理会。

这几个寄存器是与分页机制密切相关的，因此，在后面分页机制和第四章虚拟内存管理中会涉及到，读者要记住CR0、CR2及CR3这三个寄存器的作用。 
###**2．1．3 物理地址、虚拟地址及线性地址**

在硬件工程师和普通用户看来，内存就是插在或固化在主板上的内存条，它们有一定的容量，比如128MB。但在应用程序员看来中，并不过度关心插在主板上的内存容量，而是他们可以使用的内存空间，比如，他们可以开发一个占用1 GB内存的程序，并让其在操作系统下运行，哪怕这台机器上只有128 MB的物理内存条。而对于操作系统开发者而言，则是介于二者之间，它既需要知道物理内存的地址，也需要提供一套机制，为应用程序员提供另一个内存空间，这个内存空间的大小可以和实际的物理内存大小之间没有多大关系。

我们将主板上的物理内存条所提供的内存空间定义为物理内存空间，其中每个内存单元的实际地址就是物理地址；将应用程序员看到的内存空间2定义为虚拟地址空间(或地址空间)，其中的地址就叫虚拟地址(或逻辑地址)， 一般用“段：偏移量”的形式来描述，比如在8086中A815:CF2D就代表段首地址为A815，段内偏移位为CF2D的虚地址。
> 2 因为高级语言不涉及内存空间，因此，这里指的是从汇编语言的角度看。

线性地址空间是指一段连续的，不分段的，范围为0到4GB的地址空间，一个线性地址就是线性地址空间的一个绝对地址。

那么，这几种地址之间如何转换？例如，当程序执行“mov  ax，[1024]”这样一条指令时，在8086的实模式下，把某一段寄存器(比如ds)左移4位，然后与16位的偏移量(1024)相加后被直接送到内存总线上，这个相加后的地址就是内存单元的物理地址，而程序中的地址（例如ds:1024）就叫虚拟地址。在80X86保护模式下，这个虚拟地址不是被直接送到内存总线，而是被送到内存管理单元（MMU）。MMU由一个或一组芯片组成，其功能是把虚拟地址映射为物理地址，即进行地址转换，如图2.3所示。
![](http://i.imgur.com/0mpJQxn.png)

其中，MMU是一种硬件电路，它包含两个部件，一个是分段部件，一个是分页部件，在本书中，我们把它们分别叫做分段机制和分页机制，以利于从逻辑的角度来理解硬件的实现机制。分段机制把一个虚拟地址转换为线性地址；接着，分页机制把一个线性地址转换为物理地址，如图2.4所示。

![](http://i.imgur.com/AjpT74k.png)

下一节对段机制和分页机制进行具体介绍。
 
 
##**2.2 段机制**

段是虚拟地址空间的基本单位，段机制必须把虚拟地址空间的一个地址转换为线性地址空间的一个线性地址。

###**2.2.1 段描述符**

为了实现地址映射，仅仅用段寄存器来确定一个基地址是不够的，至少还得描述段的长度，并且还需要段的一些其他信息，比如访问权之类。所以，这里需要的是一个数据结构，这个结构包括三个方面的内容：

(1)	段的基地址(Base Address)：在线性地址空间中段的起始地址。

(2)	段的界限(Limit)：在虚拟地址空间中，段内可以使用的最大偏移量。

(3)	段的保护属性(Attribute)： 表示段的特性。例如，该段是否可被读出或写入，或者该段是否作为一个程序来执行，以及段的特权级等等。

如图2.5所示，虚拟地址空间中偏移量从0到limit范围内的一个段，映射到线性地址空间中就是从Base到Base+Limit。

![](http://i.imgur.com/pQ8tic1.png)
                  
把图2.5用一个表描述则如图2.6:
  ![](http://i.imgur.com/s23iRtc.png)

这样的表就是段描述符表（或叫段表），其中的表项叫做段描述符（Segment Descriptor）。

所谓描述符(Descriptor)，就是描述段的属性的一个8字节存储单元。在实模式下，段的属性不外乎是代码段、堆栈段、数据段、段的起始地址、段的长度等等，而在保护模式下则复杂一些。将它们结合在一起用一个8字节的数表示，称为描述符 。80x86通用的段描述符的结构如图2.7所示。

![](http://i.imgur.com/O4ZbGaB.png)

从图可以看出，一个段描述符指出了段的32位基地址和20位段界限(即段长)。

第六个字节的G位是粒度位，当G=0时，以节长为单位表示段的长度，即一个段最长可达220（1M）字节。当G=1时，以页（4K）为单位表示段的长度，即一个段最长可达1M×4K=4G字节。D位表示缺省操作数的大小，如果D=0，操作数为16位，如果D=1，操作数为32位。第六个字节的其余两位为0，这是为了与将来的处理器兼容而必须设置为0的位。

第5个字节是存取权字节，它的一般格式如图2.8所示：
![](http://i.imgur.com/r55VWWk.png)

第7位P位(Present) 是存在位，表示这个段是否在内存中，如果在内存中。P=1；如果不在内存中，P=0。

DPL(Descriptor Privilege Level)，就是描述符特权级，它占两位，其值为0～3，用来确定这个段的特权级即保护等级。

S位(System)表示这个段是系统段还是用户段。如果S=0，则为系统段，如果S=1，则为用户程序的代码段、数据段或堆栈段。

类型占3位，第三位为E位，表示段是否可执行。当E=0时，为数据段描述符，这时的第2位ED表示扩展方向。当ED=0时，为向地址增大的方向扩展，这时存取数据段中的数据的偏移量必须小于等于段界限，当ED=1时，表示向地址减少的方向扩展，这时偏移量必须大于界限。当表示数据段时，第1位(W)是可写位，当W=0时，数据段不能写，W=1时，数据段可写入。在80x86中，堆栈段也被看成数据段，因为它本质上就是特殊的数据段。当描述堆栈段时，ED=0，W=1,即堆栈段朝地址增大的方向扩展。


在保护模式下，有三种类型的描述符表，分别是全局描述符表GDT（Gloabal Descriptor Table）、中断描述符表IDT（Interrupt Descriptor Table）及局部描述符表LDT（Local Descriptor Table）。为了加快对这些表的访问，Intel设计了专门的寄存器gdtr,ldtr和idtr，以存放这些表的基地址及表的长度界限。各种描述表的具体内容详参见相关参考书。

由此可以推断，在保护模式下段寄存器中该存放什么内容了，那就是图2.6中的索引。因为索引表示段描述符在描述符表中位置，因此，把段寄存器也叫选择符，其结构如图2.9所示：

![](http://i.imgur.com/5pH4ARL.png)


可以看出，选择符有三个域。其中，第15～13位是索引域，第2位TI（Table Indicator）为选择域，决定从全局描述符表（TI=0）还是从局部描述符表（TI=1）中选择相应的段描述符。这里我们重点关注的是RPL域，RPL表示请求者的特权级（Requestor Privilege Level）。
 
保护模式提供了四个特权级，用0～3四个数字表示，但很多操作系统（包括Linux）只使用了其中的最低和最高两个，即0表示最高特权级，对应内核态；3表示最低特权级，对应用户态。保护模式规定，高特权级可以访问低特权级，而低特权级不能随便访问高特权级。

###**2．2．1 地址转换及保护**

程序中的虚拟地址可以表示为“选择符：偏移量”这样的形式，通过以下步骤可以把一个虚拟地址转换为线性地址：

（1） 在段寄存器中装入段选择符，同时把32位地址偏移量装入某个寄存器(比如ESI、EDI等)中。

（2） 根据选择符中的索引值、TI及RPL值，再根据相应描述符表中的段地址和段界限，进行一系列合法性检查(如特权级检查、界限检查)，如果该段无问题，就取出相应的描述符放入段描述符高速缓冲寄存器3中。

（3）将描述符中的32位段基地址和放在ESI、EDI等中的32位有效地址相加，就形成了32位线性地址。

注意，在上面的地址转换过程中，从两个方面对段进行了保护：


> （1）	在一个段内，如果偏移量大于段界限，虚拟地址将没有意义，系统将产生异常。

> （2）	如果要对一个段进行访问，系统会根据段的保护属性检查访问者是否具有访问权限，如果没有，则产生异常。例如，如果要在只读段中进行写入，系统将根据该段的属性检测到这是一种违规操作，则产生异常。


###**2．2．2 Linux中的段**

Intel微处理器的段机制是从8086开始提出的，那时引入的段机制解决了从CPU内部16位地址到20位实地址的转换。为了保持这种兼容性，386仍然使用段机制，但比以前复杂得多。因此，Linux内核的设计并没有全部采用Intel所提供的段方案，仅仅有限度地使用了一下分段机制。这不仅简化了Linux内核的设计，而且为把Linux移植到其他平台创造了条件，因为很多RISC处理器并不支持段机制。但是，对段机制相关知识的了解是进入Linux内核的必经之路。

从2.2版开始，Linux让所有的进程（或叫任务）都使用相同的逻辑地址空间，因此就没有必要使用局部描述符表LDT。但内核中也用到LDT，那只是在VM86模式中运行Wine，也就是说在Linux上模拟运行Winodws软件或DOS软件的程序时才使用。


在80X86上任意给出的地址都是一个虚拟地址，即任意一个地址都是通过“选择符:偏移量”的方式给出的，这是段机制存访问模式的基本特点。所以在80X86上设计操作系统时无法回避使用段机制。一个虚拟地址最终会通过“段基地址＋偏移量”的方式转化为一个线性地址。但是，由于绝大多数硬件平台都不支持段机制，只支持分页机制，所以为了让Linux具有更好的可移植性，我们需要去掉段机制而只使用分页机制。

但不幸的是，80X86规定段机制是不可禁止的，因此不可能绕过它直接给出线性地址空间的地址。万般无奈之下，Linux的设计人员干脆让段的基地址为0，而段的界限为4GB，这时任意给出一个偏移量，则等式为“0+偏移量=线性地址”，也就是说“偏移量＝线性地址”。另外由于段机制规定“偏移量 < 4GB”，所以偏移量的范围为0H～FFFFFFFFH，这恰好是线性地址空间范围，也就是说虚拟地址直接映射到了线性地址，我们以后所提到的虚拟地址和线性地址指的也就是同一地址。看来，Linux在没有回避段机制的情况下巧妙地把段机制给绕过去了。

另外，由于80X86段机制还规定，必须为代码段和数据段创建不同的段，所以Linux必须为代码段和数据段分别创建一个基地址为0，段界限为4GB的段描述符。不仅如此，由于Linux内核运行在特权级0，而用户程序运行在特权级别3，根据80X86的段保护机制规定，特权级3的程序是无法访问特权级为0的段的，所以Linux必须为内核和用户程序分别创建其代码段和数据段。这就意味着Linux必须创建4个段描述符——特权级0的代码段和数据段，特权级3的代码段和数据段。

Linux在启动的过程中设置了段寄存器的值和全局描述符表GDT的内容，内核代码中可以这样定义段：

       #define __KERNEL_CS     0x10   ／*内核代码段，index=2,TI=0,RPL=0*／
       #define __KERNEL_DS     0x18    ／*内核数据段, index=3,TI=0,RPL=0*／
       #define __USER_CS       0x23    ／*用户代码段, index=4,TI=0,RPL=3*／
       #define __USER_DS       0x2B    ／*用户数据段, index=5,TI=0,RPL=3*／

从定义看出，没有定义堆栈段，实际上，Linux内核不区分数据段和堆栈段，这也体现了Linux内核尽量减少段的使用。因为这几个段都放在GDT中，因此，TI=0 , index就是某个段在GDT表中的下标。内核代码段和数据段具有最高特权，因此其RPL为0，而用户代码段和数据段具有最低特权，因此其RPL为3。可以看出，Linux内核再次简化了特权级的使用，使用了两个特权级而不是4个。


内核代码中可以这样定义全局描述符表：

    ENTRY(gdt_table)
        .quad 0x0000000000000000        /* NULL descriptor */
        .quad 0x0000000000000000        /* not used */
            .quad 0x00cf9a000000ffff        /* 0x10 kernel 4GB code at 0x00000000 */
        .quad 0x00cf92000000ffff       /* 0x18 kernel 4GB data at 0x00000000 */
        .quad 0x00cffa000000ffff       /* 0x23 user   4GB code at 0x00000000 */
        .quad 0x00cff2000000ffff       /* 0x2b user   4GB data at 0x00000000 */
        .quad 0x0000000000000000        /* not used */
        .quad 0x0000000000000000        /* not used */
       …

从代码可以看出，GDT放在数组变量gdt_table中。按Intel规定，GDT中的第一项为空，这是为了防止加电后段寄存器未经初始化就进入保护模式而使用GDT的。第二项也没用。从下标2到5共4项对应于前面的4种段描述符值。对照图2.7，从描述符的数值可以得出：

**·**	段的基地址全部为0x00000000

**·**	段的上限全部为0xffff

**·**	段的粒度G为1，即段长单位为4KB

**·**	段的D位为1，即对这四个段的访问都为32位指令

**·**	段的P位为1，即四个段都在内存。

通过上面的介绍可以看出，Intel的设计可谓周全细致，但Linux的设计者并没有完全陷入这种沼泽，而是选择了简洁而有效的途径，以完成所需功能并达到较好的性能为目标。

但是，如果这么定义段，则上一节所说的段保护的第一个作用就失去了，因为这些段使用完全相同的线性地址空间（0～4GB），它们互相覆盖。可以设想，如果不使用分页的话，线性地址空间直接被映射到物理空间，则你修改任何一个段的数据，都会同时修改其它段的数据，段机制所提供的通过“基地址：界限”方式本来将线性地址空间分割，以让段与段之间完全隔离，这种实现段保护的方式根本就不起作用了。那么，这是不是意味着用户可以随意修改内核数据？显然不是的，这是因为，一方面用户段和内核段具有不同的特权级别，另一方面，Linux之所以这么定义段，正是为了实现一个纯的分页，而分页机制会提供给我们所需要的保护。

##**2.3 分页机制**

分页机制在段机制之后进行，以完成线性——物理地址的转换过程。段机制把虚拟地址转换为线性地址，分页机制进一步把该线性地址再转换为物理地址。

如果不允许分页(CR0的最高位置0)，那么经过段机制转化而来的32位线性地址就是物理地址。但如果允许分页(CR0的最高位置1)，就要将32位线性地址通过一个地址变换机构转化成物理地址。80X86规定，分页机制是可选的，但很多操作系统主要采用分页机制。

###**2．3．1页与页表**

**1．页、物理页面及页大小**

为了效率起见，将线性地址空间划分成若干大小相等的片，称为页（Page），并给各页加以编号，从0开始，如第0页、第1页等。相应地，也把物理地址空间分成与页大小相等的若干存储块，称为（物理）块或页面（Page Frame），也同样为它们加以编号，如0＃页面、1＃页面等。如图 2.10所示，图中用箭头把线性地址空间中的页，与对应的物理地址空间中的页面联系起来，表示把线性地址空间中若干页将分别装入到多个可以不连续的物理页面中。例如第0页将装入到第2页面，第1页将装入到第0页面，但是第2页也将装入到第2个页面，这似乎是一种错误，但学过内存管理一章后会理解这是一种正常现象。本节只涉及分页机制的一般原理，更多的内容将在第四章内存管理一章讲述。

![](http://i.imgur.com/AAQzURm.png)



那么，页的大小应该为多少？页过大或过小都会影响内存的使用率。其大小在设计硬件分页机制时就必须确定下来，例如80X86支持的标准页大小为4KB（也支持4MB），从后面的内容可以看出，选择4KB大小既巧妙又高效。

**2．页表**

   
页表是把线性地址映射到物理地址的一种数据结构。参照段描述符表，页表中应当包含如下内容：

（1）	物理页面基地址：线性地址空间中的一个页装入内存后所对应的物理页面的起始地址。

（2）	页的属性：表示页的特性。例如该页是否在内存，是否可被读出或写入等。

由于页面的大小为4KB，它的物理页面基地址（32位）必定是4K的倍数，因此其地址的最低12位总是0，那么就可以用这12位存放页的属性，这样用32位完全可以描述页的映射关系，也就是页表中每一项（简称页表项）占4个字节就足够。

不过，4 GB的线性空间可以被划分为1M个4K大小的页，每个页表项占4个字节，则1M个页表项的页表就需要占用4 MB空间，而且还要求是连续的，显然这是不现实的。我们可以采用两级页表来解决这个问题。

**3．两级页表**

所谓两级页表就是对页表再进行分页。第一级称为页目录，其中存放的是关于页表的信息。4MB的页表再次分页（4MB／4K）可以分为1K个页，同样对每个页的描述需要4个字节，于是可以算出页目录最多占用4KB个字节，正好是一个页，其示意图如2.11所示。

![](http://i.imgur.com/KSeEC29.png)

页目录共有1K个表项， 于是，线性地址的最高10位(即22位~ 31位)用来产生第一级的索引。两级表结构的第二级称为页表，每个页表也刚好存放在一个4K字节的页中，包含1K个字节的表项。第二级页表由线性地址的中间10位(即21位~ 12位)进行索引，最低12位表示页内偏量。具有两级页表的线性地址结构如图2.12所示。

![](http://i.imgur.com/nhzAhu6.png)
   
 **4．页表项结构**


不管是页目录还是页表，每个表项占四个字节，其表项结构基本相同，如图2.13所示：

![](http://i.imgur.com/XsTFcGM.png)
       
物理页面基地址： 对页目录而言，指的是页表所在的物理页面在内存的起始地址，对页表而言，指的是页所对应的物理页面在内存的起始物理地址。因为其最低12位全部为0，因此用高20位来描述32位的地址。
属性包括：

（1）	第0位是P（Present），如果P=1，表示页装入到内存中，如果P=0，表示不在内存中。

（2）	第1位是R/W（Read/Write），第2位是U/S（User/Supervisor）位，这两位为页表或页提供硬件保护。

（3）	第3位是PWT（Page Write-Through）位，表示是否采用写透方式，写透方式就是既写内存（RAM）也写高速缓存,该位为1表示采用写透方式

（4）	第4位是PCD（Page Cache Disable）位，表示是否启用高速缓存,该位为1表示启用高速缓存。

（5）	第5位是访问位，当对相应的物理页面进行访问时，该位置1。

（6）	第7位是Page Size标志，只适用于页目录项。如果置为1，页目录项指的是4MB的页

（7）	第9~11位由操作系统专用，Linux也没有做特殊之用。

  **5．硬件保护机制**

对于页表，页的保护是由U/S标志和R/W标志来控制的。当U/S标志为0时，只有处于内核态的操作系统才能对此页或页表进行寻址。当这个标志为1时，则不管在内核态还是用户态，总能对此页进行寻址。

此外，与段的三种存取权限（读、写、执行）不同，页的存取权限只有两种（读、写）。如果页目录项或页表项的读写标志为0，说明相应的页表或页是只读的，否则是可读写的。

###**2．3．2线性地址到物理地址的转换**

当访问线性地址空间的一个操作单元时，如何把32位的线性地址通过分页机制转化成32位物理地址呢？过程如图2.14所示。

![](http://i.imgur.com/bbnzBns.png)

第一步，用32位线性地址的最高10位第31~22位作为页目录项的索引，将它乘以4，与CR3中的页目录的起始地址相加，获得相应目录项在内存的地址。

第二步，从这个地址开始读取32位页目录项，取出其高20位，再给低12位补0，形成的32位就是页表在内存的起始地址。

第三步，用32位线性地址中的第21~12位作为页表中页表项的索引，将它乘以4，与页表的起始地址相加，获得相应页表项在内存的地址。

第四步，从这个地址开始读取32位页表项，取出其高20位，再将线性地址的第11~0位放在低12位，形成最终32位页面物理地址。
    
###**2．3．3 分页举例**

下面举一个简单的例子，这将有助于读者理解分页机制是怎样工作的。

假如操作系统给一个正在运行的进程分配的线性地址空间范围是0x20000000 到 0x2003ffff。这个空间由64页组成。我们暂且不关心这些页所在的物理页面的地址，只关注页表项中的几个域。

我们从分配给进程的线性地址的最高10位（分页硬件机制把它自动解释成页目录域）开始。这两个地址都以2开头，后面跟着0，因此高10位有相同的值，即十六进制的0x080或十进制的128。因此，这两个地址的页目录域都指向进程页目录的第129项。相应的目录项中必须包含分配给进程的页表的物理地址,如图2.15。如果给这个进程没有分配其它的线性地址，则页目录的其余1023项都为0，也就是这个进程在页目录中只占一项。

![](http://i.imgur.com/68AVQKS.png)
 
中间10位的值（即页表域的值）范围从0到0x03f，或十进制的从0到63。因而只有页表的前64个表项是有意义的，其余960表项填为0。

假设进程需要读线性地址0x20021406中的内容。这个地址由分页机制按下面的方法进行处理:

1．目录域的0x80用于选择页目录的第0x80目录项,此目录项指向页表。

2．页表域的第0x21项用于选择页表的第0x21表项,此表项指向页所对应的内存物理页面。

3．最后,偏移量0x406用于在目标物理页面中读偏移量为0x406中的字节。

如果页表第0x21表项的Present标志为0，说明此页还没有装入内存中；在这种情况下，分页机制在转换线性地址的同时产生一个缺页异常（参见内存管理一章）。无论何时，当进程试图访问限定在0x20000000到0x2003ffff范围之外的线性地址时，都将产生一个缺页异常，因为这些页表项都填充了0，尤其是，它们的Present标志都为0。  
                       
###**2．3．4 页面高速缓存**

由于在分页情况下，页表是放在内存中的，这使CPU在每次存取一个数据时,都要至少两次访问内存,从而大大降低了访问速度。所以，为了提高速度，在80X86中设置一个最近存取页的高速缓存硬件机制，它自动保持32项处理器最近使用的页表项，因此，可以覆盖128K字节的内存地址。当访问线性地址空间的某个地址时，先检查对应的页表项是否在高速缓存中，如果在，就不必经过两级访问了，如果不在，再进行两级访问。平均来说，页面高速缓存大约有90%的命中率，也就是说每次访问存储器时，只有10%的情况必须访问两级分页机构。这就大大加快了速度，页面高速缓存的作用如图2.16所示。有些书上也把页面高速缓存叫做“联想存储器”或“转换旁视缓冲器（TLB）”。

![](http://i.imgur.com/3YLFk45.png)

##**2．4  Linux中的分页机制**
 
如前所述，Linux主要采用分页机制来实现虚拟存储器管理。这是因为：

（1）	Linux的分段机制使得所有的进程都使用相同的段寄存器值，这就使得内存管理变得简单，也就是说，所有的进程都使用同样的线性地址空间（0～4G）。

（2）	Linux设计目标之一就是能够把自己移植到绝大多数流行的处理器平台。但是，许多RISC处理器支持的段功能非常有限。

为了保持可移植性，Linux采用三级分页模式而不是两级，这是因为许多处理器（如康柏的Alpha，Sun的UltraSPARC，Intel的Itanium）都采用64位结构的处理器，在这种情况下，两级分页就不适合了，必须采用三级分页。图2.17为三级分页模式，为此，Linux定义了三种类型的页表：

- 	页总目录PGD（Page Global Directory）
- 	页中间目录PMD（Page Middle Derectory）
- 	页表PT（Page Table）
             
![](http://i.imgur.com/ua66uMo.png)

尽管Linux采用的是三级分页模式，但我们的讨论还是以80X86的两级分页模式为主，因此，Linux忽略中间目录层，以后，我们把页总目录就叫页目录。

我们将在第四章看到，每一个进程有它自己的页目录和自己的页表集。当进程切换发生时（参见第三章“进程切换”一节），Linux把cr3控制寄存器的内容保存在前一个执行进程的PCB中，然后把下一个要执行进程的PCB的值装入cr3寄存器中。因此，当新进程恢复在CPU上执行时，分页单元指向一组正确的页表。

当三级分页模式应用到只有两级页表的奔腾处理器上时会发生什么情况？Linux使用一系列的宏来掩盖各种平台的细节。例如，通过把PMD看作只有一项的表，并把它存放在pgd表项中（通常pgd表项中存放的应该是pmd表的首地址）。页表的中间目录(pmd)被巧妙地“折叠”到页表的总目录(pgd)中，从而适应了二级页表。


###**2.4.1 模拟页表初始化**

在了解页表基本原理之后，通过代码来模拟内核初始化页表的过程：

     #define NR_PGT 0x4
     #define PGD_BASE (unsigned int*)0x1000
     #define PAGE_OFFSET (unsigned int)0x2000

     #define PTE_PRE 0x01 /* Page present */
     #define PTE_RW  0x02 /* Page Readable/Writeable */
     #define PTE_USR 0x04 /* User Privilege Level*/ 

     void page_init()
     {
       int pages = NR_PGT; // 系统初始化创建4个页表

       unsigned int page_offset = PAGE_OFFSET;
       unsigned int* pgd = PGD_BASE;// 页目录表位于物理内存的第二个页框内
       unsigned int phy_add = 0x0000; // 在物理地址的最低端建立页机制所需的表格

       // 页表从物理内存的第三个页框处开始
       // 物理内存的头8KB没有通过页表映射
       unsigned int* pgt_entry = (unsigned int*)0x2000;
  
        while (pages--)// 填充页目录表,这里依次创建4个页目录表
         {   
           *pgd++ = page_offset |PTE_USR|PTE_RW|PTE_PRE;
           page_offset += 0x1000;
         }   

       pgd = PGD_BASE;

  
       while (phy_add < 0x1000000) {// 在页表中填写页到物理地址的映射关系，
     //映射了4M大小的物理内存
         *pgt_entry++ = phy_add |PTE_USR|PTE_RW|PTE_PRE;
         phy_add += 0x1000;
       }

  
     __asm__ __volatile__ ("movl %0, %%cr3;"
                        "movl %%cr0, %%eax;"
                        "orl  $0x80000000, %%eax;"
                        "movl %%eax, %%cr0;"::"r"(pgd):"memory","%eax");
     }

从代码中可以看出在物理内存的第二个页框设置了页目录，然后的while循环初始化了页目录中的四个页目录项，即四个页表。紧接着的第二个while循环初始化了四个页表中的第一个，其它三个没有用到，映射了4MB的物理内存，至此页表已初始化好，剩下的工作就是将页目录的地址pgd传递给cr3寄存器，这由gcc嵌入式代码部分完成，并且设置了cr0寄存器中的分页允许。最后一句以__asm开头的嵌入式汇编参见2.5.3节。

虽然上述代码比较简单，但却描述了页表初始化的过程，以此为模型我们可以更容易理解Linux内核中关于页表的代码。

##**2.5 Linux中的汇编语言**

在Linux源代码中，大部分是用C语言编写的，但也涉及到汇编语言，有些汇编语言出现在以.S为扩展名的汇编文件中，在这种文件中，整个程序全部由汇编语言组成。有些汇编命令出现在以.c为扩展名的C文件中，在这种文件中，既有C语言，也有汇编语言，我们把出现在C代码中的汇编语言叫做“嵌入式”汇编。

尽管C语言已经成为编写操作系统的主要语言，但是，在操作系统与硬件打交道的过程中，在需要频繁调用的函数中以及某些特殊的场合中，C语言显得力不从心，这时，繁琐但又高效的汇编语言必须粉墨登场。因此，在了解80X86内存寻址的基础上，有必要对相关的汇编语言知识有所了解。

读者可能有过在DOS操作系统下编写汇编程序的经历,也具备一定的汇编知识。但是，在Linux的源代码中，你可能看到了与Intel的汇编语言格式不一样的形式，这就是AT&T的i386汇编语言。

###**2.5.1 AT&T与Intel汇编语言的比较**

我们知道，Linux是Unix家族的一员，尽管Linux的历史不长，但与其相关的很多事情都发源于Unix。就Linux所使用的i386汇编语言而言，它也是起源于Unix。Unix最初是为PDP-11开发的，曾先后被移植到VAX及68000系列的处理器上，这些处理器上的汇编语言都采用的是AT&T的指令格式。当Unix被移植到i386时，自然也就采用了AT&T的汇编语言格式，而不是Intel的格式。尽管这两种汇编语言在语法上有一定的差异，但所基于的硬件知识是相同的，因此，如果你非常熟悉Intel的语法格式，那么你也可以很容易地把它“移植”到AT&T来。下面我们通过对照Intel与AT&T的语法格式，以便于你把过去的知识能很快地“移植”过来。

 **1．前缀**

在Intel的语法中，寄存器和和立即数都没有前缀。但是在AT&T中，寄存器前冠以“％”，而立即数前冠以“$”。在Intel的语法中，十六进制和二进制立即数后缀分别冠以“h”和“b”，而在AT&T中，十六进制立即数前冠以“0x”，表2.2给出几个相应的例子。

表2.1 Intel与AT&T前缀的区别

|Intel语法|	AT&T语法|
|-----|-----|
| Mov	eax,8|	 movl	$8,%eax|
|Mov	ebx,0ffffh| movl	$0xffff,%ebx|
 |int	80h	 |  int 	$0x80|

**2. 操作数的方向**

Intel与AT&T操作数的方向正好相反。在Intel语法中，第一个操作数是目的操作数，第二个操作数源操作数。而在AT&T中，第一个数是源操作数，第二个数是目的操作数。由此可以看出，AT&T 的语法符合人们通常的阅读习惯。

例如：在Intel中， `mov	eax,[ecx]`

   在AT&T中，   `movl	(%ecx),%eax`

**3．内存单元操作数**

从上面的例子可以看出，内存操作数也有所不同。在Intel的语法中，基寄存器用“［］”括起来，而在AT&T中，用“（）”括起来。
    
例如： 在Intel中，`mov	eax,[ebx+5]`

   在AT&T，`movl	5(%ebx),%eax`

**4．操作码的后缀**

在上面的例子中你可能已注意到，在AT&T的操作码后面有一个后缀，其含义就是指出操作码的大小。“l”表示长整数（32位），“w”表示字（16位），“b”表示字节（8位）。而在Intel的语法中，则要在内存单元操作数的前面加上byte ptr、 word ptr和dword ptr，“dword”对应“long”。表2.4给出几个相应的例子。

> 表2.2 操作码的后缀举例

|Intel语法|	AT&T语法|
|------|------|
 |Mov	al,bl|	 movb	%bl,%al|
 |Mov	ax,bx|	 movw	%bx,%ax|
| Mov	eax,ebx	 |movl	%ebx,%eax|
| Mov	eax, dword ptr [ebx]|	 movl	(%ebx),%eax|

###**2.5.2 AT&T汇编语言的相关知识**

在Linux源代码中，以.S为扩展名的文件是“纯”汇编语言的文件。这里，我们结合具体的例子再介绍一些AT&T汇编语言的相关知识。

   **1．GNU汇编程序GAS（GNU ASsembly和连接程序**

当你编写了一个程序后，就需要对其进行汇编（assembly）和连接。在Linux下有两种方式，一种是使用汇编程序GAS和连接程序ld，一种是使用gcc。我们先来看一下GAS和ld:

GAS把汇编语言源文件（.o）转换为目标文件（.o），其基本语法如下：

    as filename.s -o filename.o

一旦创建了一个目标文件，就需要把它连接并执行，连接一个目标文件的基本语法为：

    ld filename.o -o filename

这里 filename.o是目标文件名，而filename 是输出(可执行) 文件。
 
GAS使用的是AT&T的语法而不是Intel的语法，这就再次说明了AT&T语法是Unix世界的标准，你必须熟悉它。

如果要使用GNC的C编译器gcc，就可以一步完成汇编和连接，例如：

    gcc -o example example.S2 .  **AT&T中的节（Section）**

在AT&T的语法中，一个节由.section关键词来标识，当你编写汇编语言程序时，至少需要有以下三种节：

.section .data： 这种节包含程序已初始化的数据，也就是说，包含具有初值的那些变量，例如：

              hello     : .string "Hello world!\n"
              hello_len : .long 13

 .section .bss：这个节包含程序还未初始化的数据，也就是说，包含没有初值的那些变量。当操作系统装入这个程序时将把这些变量都置为0，例如：

      name      : .fill 30   # 用来请求用户输入名字
              name_len  : .long  0   # 名字的长度 (尚未定义)
当这个程序被装入时，name 和 name_len都被置为0。如果你在.bss节不小心给一个变量赋了初值，这个值也会丢失，并且变量的值仍为0。

使用.bss比使用.data的优势在于，.bss节不占用磁盘的空间。在磁盘上，一个长整数就足以存放.bss节。当程序被装入到内存时，操作系统也只分配给这个节4个字节的内存大小。
  
注意：编译程序把.data和.bss在4字节上对齐（align），例如，.data总共有34字节，那么编译程序把它对其在36字节上，也就是说，实际给它36字节的空间。

.section .text ：这个节包含程序的代码，它是只读节，而.data 和.bss是读／写节。

当然，关于AT&T的汇编内容还很多，感兴趣的读者可以参看相关文档。

###**2.5.3 Gcc嵌入式汇编**

在Linux的源代码中，有很多C语言的函数中嵌入一段汇编语言程序段，这就是gcc提供的“asm”功能，例如内核代码中，读控制寄存器CR0的一个宏read_cr0()：
 
    #define read_cr0() ({ \
         unsigned int __dummy; \
         __asm__( \
                 "movl %%cr0,%0\n\t" \
                 :"=r" (__dummy)); \
         __dummy; \
     })
 
这种形式看起来比较陌生，因为这不是标准C所定义的形式，而是gcc 对C语言的扩充。其中__dummy为C函数所定义的变量；关键词__asm__表示汇编代码的开始。括弧中第一个引号中为汇编指令movl，紧接着有一个冒号，这种形式阅读起来比较复杂。

一般而言，嵌入式汇编语言片段比单纯的汇编语言代码要复杂得多，因为这里存在怎样分配和使用寄存器，以及把C代码中的变量应该存放在哪个寄存器中的问题。为了达到这个目的，就必须对一般的C语言进行扩充，增加对编译器的指导作用，因此，嵌入式汇编看起来晦涩而难以读懂。

**1. 嵌入式汇编的一般形式：**
 
    __asm__ __volatile__ ("<asm routine>" : output : input : modify);
 
其中，__asm__表示汇编代码的开始，其后可以跟__volatile__（这是可选项），其含义是避免“asm”指令被删除、移动或组合；然后就是小括弧，括弧中的内容是我们介绍的重点：
·      "<asm routine>"为汇编指令部分，例如，"movl %%cr0,%0\n\t"。数字前加前缀“％“，如％1，％2等表示使用寄存器的样板操作数。可以使用的操作数总数取决于具体CPU中通用寄存器的数量，如Intel可以有8个。指令中有几个操作数，就说明有几个变量需要与寄存器结合，由gcc在编译时根据后面输出部分和输入部分的约束条件进行相应的处理。由于这些样板操作数的前缀使用了”％“，因此，在用到具体的寄存器时就在前面加两个“％”，如%%cr0。

·      输出部分（output），用以规定对输出变量（目标操作数）如何与寄存器结合的约束（constraint）,输出部分可以有多个约束，互相以逗号分开。每个约束以“＝”开头，接着用一个字母来表示操作数的类型，然后是关于变量结合的约束。例如，上例中：

:"=r" (__dummy)

“＝r”表示相应的目标操作数（指令部分的%0）可以使用任何一个通用寄存器，并且变量__dummy 存放在这个寄存器中，但如果是：

：“＝m”（__dummy）

“＝m”就表示相应的目标操作数是存放在内存单元__dummy中。

表示约束条件的字母很多，表 2.3 给出几个主要的约束字母及其含义：

> 表2.3  主要的约束字母及其含义

   |字母|	含义|
   |-----|-----|
  | m, v,o	|表示内存单元
   |R|	表示任何通用寄存器
   |Q	|表示寄存器eax, ebx, ecx,edx之一
   |I, h	|表示直接操作数
   |E, F	|表示浮点数
   |G	|表示“任意”
   |a, b.c d	|表示要求使用寄存器eax/ax/al, ebx/bx/bl, ecx/cx/cl或edx/dx/dl
   |S, D	|表示要求使用寄存器esi或edi
  | I	|表示常数（0～31）|



-     输入部分（Input）：输入部分与输出部分相似，但没有“＝”。如果输入部分一个操作数所要求使用的寄存器，与前面输出部分某个约束所要求的是同一个寄存器，那就把对应操作数的编号（如“1”，“2”等）放在约束条件中，在后面的例子中，我们会看到这种情况。
-       修改部分（modify）:这部分常常以“memory”为约束条件，以表示操作完成后内存中的内容已有改变，如果原来某个寄存器的内容来自内存，那么现在内存中这个单元的内容已经改变。
 
注意，指令部分为必选项，而输入部分、输出部分及修改部分为可选项，当输入部分存在，而输出部分不存在时，分号“：“要保留，当“memory”存在时，三个分号都要保留，例如system.h中的宏定义__cli()：

       #define __cli()                 __asm__ __volatile__("cli": : :"memory")

**2.  Linux源代码中嵌入式汇编举例**

   Linux源代码中，在arch目录下的.h和.c文件中，很多文件都涉及嵌入式汇编，下面以system.h中的C函数为例，说明嵌入式汇编的应用。

（1）简单应用

     #define __save_flags(x)         __asm__ __volatile__("pushfl ; popl %0":"=g" (x): /* no input */)
     #define __restore_flags(x)      __asm__ __volatile__("pushl %0 ; popfl": /* no output */
      :"g" (x):"memory", "cc")

第一个宏是保存标志寄存器的值，第二个宏是恢复标志寄存器的值。第一个宏中的pushfl指令是把标志寄存器的值压栈。而popl是把栈顶的值（刚压入栈的flags）弹出到x变量中，这个变量可以存放在一个寄存器或内存中。这样，你可以很容易地读懂第二个宏。

(2) 较复杂应用

     static inline unsigned long get_limit(unsigned long segment)
     {
         unsigned long __limit;
         __asm__("lsll %1,%0"
                 :"=r" (__limit):"r" (segment));
        return __limit+1;
     }

这是一个设置段界限的函数，汇编代码段中的输出参数为__limit（即%0），输入参数为segment（即%1）。lsll是加载段界限的指令，即把segment段描述符中的段界限字段装入某个寄存器（这个寄存器与__limit结合），函数返回__limit加1，即段长。

（3）复杂应用

在Linux内核代码中，有关字符串操作的函数都是通过嵌入式汇编完成的，因为内核及用户程序对字符串函数的调用非常频繁，因此，用汇编代码实现主要是为了提高效率（当然是以牺牲可读性和可维护性为代价的）。在此，我们仅列举一个字符串比较函数strcmp，其代码在arch/i386／string.h中。

     static inline int strcmp(const char * cs,const char * ct)
     {
     int d0, d1;
     register int __res;
     __asm__ __volatile__(
         "1:\tlodsb\n\t"
         "scasb\n\t"
         "jne 2f\n\t"
         "testb %%al,%%al\n\t"
         "jne 1b\n\t"
         "xorl %%eax,%%eax\n\t"
         "jmp 3f\n"
         "2:\tsbbl %%eax,%%eax\n\t"
         "orb $1,%%al\n"
         "3:"
         :"=a" (__res), "=&S" (d0), "=&D" (d1)
                      :"1" (cs),"2" (ct));
     return __res;
     }

其中的“\n”是换行符，“\t”是tab符，在每条命令的结束加这两个符号，是为了让gcc把嵌入式汇编代码翻译成一般的汇编代码时能够保证换行和留有一定的空格。例如，上面的嵌入式汇编会被翻译成：

**1：**

**lodsb**        //装入串操作数,即从[esi]传送到al寄存器，然后esi指向串中下一个元素

   **scasb**        //扫描串操作数，即从al中减去es:[edi]，不保留结果，只改变标志

   **jne2f**          //如果两个字符不相等，则转到标号2 
  
   **testb %al  %al**  

   **jne 1b**

   **xorl %eax %eax**

   **jmp 3f**

**2:**

 **sbbl %eax %eax**

 **3**

 **orb $1 %al**



 这段代码看起来非常熟悉，读起来也不困难。其中1f 表示往前（forword）找到第一个标号为1的那一行，相应地，1b表示往后找。其中嵌入式汇编代码中输出和输入部分的结合情况为：



-     返回值__res，放在al寄存器中，与%0相结合；
-     局部变量d0，与％1相结合，也与输入部分的cs参数相对应，也存放在寄存器ESI中，即ESI中存放源字符串的起始地址。
-     局部变量d1， 与％2相结合，也与输入部分的ct参数相对应，也存放在寄存器EDI中，即EDI中存放目的字符串的起始地址。

通过对这段代码的分析我们应当体会到，万变不利其本，嵌入式汇编与一般汇编的区别仅仅是形式，本质依然不变。

##**2．6 Linux系统地址映射举例**

Linux采用分页存储管理。虚拟地址空间划分成固定大小的“页”，由MMU在运行时将虚拟地址映射（变换）成某个物理页面中的地址。从80X86系列的历史演变过程可知，分段管理在分页管理之前出现，因此，80X86的MMU对程序中的虚拟地址先进行段式映射（虚拟地址转换为线性地址），然后才能进行页式映射（线性地址转换为物理地址）。既然硬件结构是这样设计的，Linux内核在设计时只好服从这种选择，只不过，Linux巧妙地使段式映射实际上不起什么作用。

本节通过一个程序的执行来说明地址的映射过程。

  假定我们有一个简单的C程序Hello.c

     # include <stdio.h>
     greeting ( )
     {
		   printf(“Hello,world!\n”);
     }
     main()
      {
            greeting();
      }

之所以把这样简单的程序写成两个函数，是为了说明指令的转移过程。我们用gcc和ld对其进行编译和连接，得到可执行代码hello。然后，用Linux的实用程序objdump对其进行反汇编：

    % objdump –d hello

得到的主要片段为：

     08048568 <greeting>:
     8048568:     pushl  %ebp
     8048569:     movl  %esp, %ebp
     804856b:     pushl  $0x809404
     8048570:     call    8048474  <_init+0x84>
     8048575:     addl   $0x4, %esp
     8048578:     leave
     8048579:     ret
     804857a:     movl  %esi, %esi
     0804857c <main>:
     804857c:     pushl  %ebp
     804857d:     movl  %esp, %ebp
     804857f:     call    8048568  <greeting>
     8048584:     leave
     8048585:     ret
     8048586:     nop
     8048587:     nop

最左边的数字是连接程序ld分配给每条指令或标识符的虚拟地址,其中分配给greeting()这个函数的起始地址为0x08048568。Linux最常见的可执行文件格式为elf(Executable and Linkable Format)。在elf格式的可执行代码中，ld总是从0x8000000开始安排程序的“代码段”，对每个程序都是这样。至于程序执行时在物理内存中的实际地址，则由内核为其建立内存映射时临时分配，具体地址取决于当时所分配的物理内存页面。

假定该程序已经开始运行，整个映射机制都已经建立好，并且CPU正在执行main()中的“call 08048568”这条指令，于是转移到虚地址0x08048568。Linux内核设计的段式映射机制把这个地址原封不动地映射为线性地址，接着就进入页式映射过程。

每当调度程序选择一个进程运行时，内核就要为即将运行的进程设置好控制寄存器CR3，而MMU的硬件总是从CR3中取得指向当前页目录的指针。

当我们的程序转移到地址0x08048568的时候，进程正在运行中，CR3指向我们这个进程的页目录。根据线性地址0x08048568最高10位，就可以找到相应的目录项。把08048568按二进制展开：

0000 1000 0000 0100 1000 0101 0110 1000

最高10位为0000 1000 00，即十进制32，这样以32为下标在页目录中找到其目录项。这个目录项中的高20位指向一个页表，CPU在这20位后填12个0就得到该页表的物理地址。

找到页表之后，CPU再来找线性地址的中间10位，为0001001000，即十进制72，于是CPU就以此为下标在页表中找到相应的页表项，取出其高20位，假定为0x840，然后与线性地址的最低12位0x568拼接起来，就得到greeting()函数的入口物理地址为0x840568, greeting()的执行代码就存储在这里。

##**2.7 小结**

本章从寻址方式的演变入手，给出与操作系统设计密切相关的概念，比如，实模式，保护模式，各种寄存器，物理地址，虚拟地址以及线性地址等。然后对保护模式的分段机制和分页机制给予简要描述，并从Linux设计的角度分析了这些机制的具体落实。接着介绍了Linux中的汇编以及嵌入式汇编，最后给出了Linux系统的地址映射示例，这是在第二章就引入内存寻址的根本目的，就是操作系统如何借助硬件把虚地址转化为物理地址。

   
习题

1.	Intel微处理器从4位、8位、16位到32位的演变过程中，什么起了决定作用？其演变过程继承了什么？同时又突破了什么？
2.	在80X86的寄存器中，哪些寄存器供一般用户使用？哪些寄存器只能操作系统使用？
3.	什么是物理地址？什么是虚地址？什么又是线性地址？举例说明。
4.	在保护模式下，MMU如何把一个虚地址转换为物理地址？
5.	你是如何认识段的？请用C语言描述段描述符表。
6.	为什么把80X86下的段寄存器叫段选择符？
7.	保护模式主要保护什么？通过什么进行保护？
8.	Linux是如何利用段机制又巧妙地绕过段机制？在内核代码中如何表示各种段，查找最新源代码进行阅读和分析。
9.	页的大小是由硬件设计者决定还是操作系统设计者决定？过大或过小会带来什么问题？
10.	为什么对32位线性地址空间要采用两级页表？
11.	编写程序，模拟页表的初始化。
12.	为什么在设计两级页表的线性地址结构时，给页目录和页表各分配10位？如果不是这样，举例说明会产生什么样的结果？
13.	页表项属性中各个位的定义是由硬件设计者决定还是操作系统设计者决定？如果通过页表项的属性对页表及页中的数据进行保护？
14.	深入理解图2.12，并结合图叙述线性地址到物理地址的转换？
15.	假定一个进程分配的线性地址范围为0x00e80000～0xc0000000，又假定这个进程要读取线性地址0x00faf000中的内容，试按分页原理描述其处理过程。
16.	页面高速缓存起什么作用？如何置换其中的内容，以使其命中率尽可能高？
17.	Linux为什么主要采用分页机制来实现虚拟存储管理？它为什么采用三级分页而不是两级？
18.	Intel的汇编语言与AT&T的汇编语言有何主要区别？
19.	分析源代码文件system.h中read_cr3()函数和string_32.h中的memcpy()函数。

