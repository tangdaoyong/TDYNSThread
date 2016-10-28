#ios多线程
[参考文章](http://www.cnblogs.com/kenshincui/p/3983982.html)
#####概念
无论是哪种语言开发的程序最终往往转换成汇编语言进而解释成机器码来执行。但是机器码是按顺序执行的，一个复杂的多步操作只能一步步按顺序逐个执行。改变这种状况可以从两个角度出发：对于单核处理器，可以将多个步骤放到不同的线程，这样一来用户完成UI操作后其他后续任务在其他线程中，当CPU空闲时会继续执行，而此时对于用户而言可以继续进行其他操作；对于多核处理器，如果用户在UI线程中完成某个操作之后，其他后续操作在别的线程中继续执行，用户同样可以继续进行其他UI操作，与此同时前一个操作的后续任务可以分散到多个空闲CPU中继续执行（当然具体调度顺序要根据程序设计而定），既解决了线程阻塞又提高了运行效率。

在单线程中一个线程只能做一件事情，一件事情处理不完另一件事就不能开始，这样势必影响用户体验。早在单核处理器时期就有多线程，这个时候多线程更多的用于解决线程阻塞造成的用户等待（通常是操作完UI后用户不再干涉，其他线程在等待队列中，CPU一旦空闲就继续执行，不影响用户其他UI操作），其处理能力并没有明显的变化。如今无论是移动操作系统还是PC、服务器都是多核处理器，于是“并行运算”就更多的被提及。一件事情我们可以分成多个步骤，在没有顺序要求的情况下使用多线程既能解决线程阻塞又能充分利用多核处理器运行能力。

下图反映了一个包含8个操作的任务在一个有两核心的CPU中创建四个线程运行的情况。假设每个核心有两个线程，那么每个CPU中两个线程会交替执行，两个CPU之间的操作会并行运算。单就一个CPU而言两个线程可以解决线程阻塞造成的不流畅问题，其本身运行效率并没有提高，多CPU的并行运算才真正解决了运行效率问题，这也正是并发和并行的区别。
![](/Users/tangdaoyong/Desktop/TDY_iOSGit/TDY_NSThread/202333349252647.png)

>1.线程同步：是多个线程同时访问同一资源，等待资源访问结束，浪费时间，效率低 ,串行执行任务。
>2.线程异步：访问资源时在空闲等待时同时访问其他资源，实现多线程机制,并行执行任务 1.2.3模式
>3.串行队列：只有一个线程，加入到队列中的操作按添加顺序依次执行。
>4.并发(并行)队列：有多个线程，操作进来之后它会将这些队列安排在可用的处理器上，同时保证先进来的任务优先处理。

可以认为串行列中的线程是同步的，需要等待。而并行队列中的线程是异步的，不需要等待返回，通过其他方式控制线程的结束顺序，以确保队列的特性。
![](/Users/tangdaoyong/Desktop/TDY_iOSGit/TDY_NSThread/1898690-8046a20064ccebc1.png)
同步任务： 优先级高，在线程中有执行顺序，不会开启新的线程

dispatch_sync和 dispatch_async需要两个参数，一个是队列，一个是block,它们的共同点是block都会在你指定的队列上执行(无论队列是并行队列还是串行队列)，不同的是dispatch_sync会阻塞当前调用GCD的线程直到block结束，而dispatch_async异步继续执行。

###线程的同步和异步

#####多线程和异步操作的异同

　　多线程和异步操作两者都可以达到避免调用线程阻塞的目的，从而提高软件的可响应性。甚至有些时候我们就认为多线程和异步操作是等同的概念。但是，多线程和异步操作还是有一些区别的。而这些区别造成了使用多线程和异步操作的时机的区别。

#####异步操作的本质

　　所有的程序最终都会由计算机硬件来执行，所以为了更好的理解异步操作的本质，我们有必要了解一下它的硬件基础。 熟悉电脑硬件的朋友肯定对DMA这个词不陌生，硬盘、光驱的技术规格中都有明确DMA的模式指标，其实网卡、声卡、显卡也是有DMA功能的。DMA就是直接内存访问的意思，也就是说，拥有DMA功能的硬件在和内存进行数据交换的时候可以不消耗CPU资源。只要CPU在发起数据传输时发送一个指令，硬件就开始自己和内存交换数据，在传输完成之后硬件会触发一个中断来通知操作完成。这些无须消耗CPU时间的I/O操作正是异步操作的硬件基础。所以即使在DOS这样的单进程（而且无线程概念）系统中也同样可以发起异步的DMA操作。

#####线程的本质
　　线程不是一个计算机硬件的功能，而是操作系统提供的一种逻辑功能，线程本质上是进程中一段并发运行的代码，所以线程需要操作系统投入CPU资源来运行和调度。

#####异步操作的优缺点

　　因为异步操作无须额外的线程负担，并且使用回调的方式进行处理，在设计良好的情况下，处理函数可以不必使用共享变量（即使无法完全不用，最起码可以减少共享变量的数量），减少了死锁的可能。当然异步操作也并非完美无暇。编写异步操作的复杂程度较高，程序主要使用回调方式进行处理，与普通人的思维方式有些初入，而且难以调试。

#####多线程的优缺点
　　多线程的优点很明显，线程中的处理程序依然是顺序执行，符合普通人的思维习惯，所以编程简单。但是多线程的缺点也同样明显，线程的使用（滥用）会给系统带来上下文切换的额外负担。并且线程间的共享变量可能造成死锁的出现。

#####适用范围

　　在了解了线程与异步操作各自的优缺点之后，我们可以来探讨一下线程和异步的合理用途。我认为：当需要执行I/O操作时，使用异步操作比使用线程+同步I/O操作更合适。I/O操作不仅包括了直接的文件、网络的读写，还包括数据库操作、Web Service、HttpRequest以及.Net Remoting等跨进程的调用。
　　而线程的适用范围则是那种需要长时间CPU运算的场合，例如耗时较长的图形处理和算法执行。但是往往由于使用线程编程的简单和符合习惯，所以很多朋友往往会使用线程来执行耗时较长的I/O操作。这样在只有少数几个并发操作的时候还无伤大雅，如果需要处理大量的并发操作时就不合适了。

#####线程同步与异步区别

1.线程同步：是多个线程同时访问同一资源，等待资源访问结束，浪费时间，效率低
2.线程异步：访问资源时在空闲等待时同时访问其他资源，实现多线程机制

异步处理就是,你现在问我问题,我可以不回答你,等我用时间了再处理你这个问题.同步不就反之了，同步信息被立即处理 -- 直到信息处理完成才返回消息句柄；异步信息收到后将在后台处理一段时间 -- 而早在信息处理结束前就返回消息句柄

#####iOS多线程

在iOS中每个进程启动后都会建立一个主线程（UI线程），这个线程是其他线程的父线程。由于在iOS中除了主线程，其他子线程是独立于Cocoa Touch的，所以只有主线程可以更新UI界面（新版iOS中，使用其他线程更新UI可能也能成功，但是不推荐）。iOS中多线程使用并不复杂，关键是如何控制好各个线程的执行顺序、处理好资源竞争问题。常用的多线程开发有三种方式：

    1.NSThread
    2.NSOperation
    3.GCD
三种方式是随着iOS的发展逐渐引入的，所以相比而言后者比前者更加简单易用，并且GCD也是目前苹果官方比较推荐的方式（它充分利用了多核处理器的运算性能）。做过.Net开发的朋友不难发现其实这三种开发方式 刚好对应.Net中的多线程、线程池和异步调用，因此在文章中也会对比讲解。

###ios多线程之NSThread
NSThread是轻量级的多线程开发，使用起来也并不复杂，但是使用NSThread需要自己管理线程生命周期。可以使用对象方法:
    + (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(id)argument
直接将操作添加到线程中并启动，也可以使用对象方法:
    - (instancetype)initWithTarget:(id)target selector:(SEL)selector object:(id)argument
创建一个线程对象，然后调用start方法启动线程。

    将数据显示到UI控件,注意只能在主线程中更新UI,
    //performSelectorOnMainThread方法是NSObject的分类方法，每个NSObject对象都有此方法，更新UI的时候使用UI线程，可以调用NSObject的这个分类扩展方法，进行UI线程完成更新。
    //它调用的selector方法是当前调用控件的方法，例如使用UIImageView调用的时候selector就是UIImageView的方法
    //Object：代表调用方法的参数,不过只能传递一个参数(如果有多个参数请使用对象进行封装)
    //waitUntilDone:是否线程任务完成执行
    [self performSelectorOnMainThread:@selector(updateImage:) withObject:data waitUntilDone:YES];

    //方法1：使用对象方法
    //创建一个线程，第一个参数是请求的操作，第二个参数是操作方法的参数
    NSThread *thread=[[NSThread alloc]initWithTarget:self selector:@selector(loadImage) object:nil];
    //启动一个线程，注意启动一个线程并非就一定立即执行，而是处于就绪状态，当系统调度时才真正执行
    // [thread start];

    //方法2：使用类方法
    [NSThread detachNewThreadSelector:@selector(loadImage) toTarget:self withObject:nil];
通过NSThread的currentThread可以取得当前操作的线程，其中会记录线程名称name和编号number，需要注意主线程编号永远为1。

    [NSThread currentThread];//得到当前线程的信息
多个线程虽然按顺序启动，但是实际执行未必按照顺序，因为线程启动后仅仅处于就绪状态，实际是否执行要由CPU根据当前状态调度。

    thread.threadPriority = 1.0;//线程优先级范围为0.0~1.0之间，值越大优先级越高，每个线程的优先级默认为0.5
在线程操作过程中可以让某个线程休眠等待，优先执行其他线程操作，而且在这个过程中还可以修改某个线程的状态或者终止某个指定线程。

    [NSThread sleepForTimeInterval:2.0];//休眠2秒
线程状态分为isExecuting（正在执行）、isFinished（已经完成）、isCancellled（已经取消）三种。其中取消状态程序可以干预设置，只要调用线程的cancel方法即可。但是需要注意在主线程中仅仅能设置线程状态，并不能真正停止当前线程，如果要终止线程必须在线程中调用exist方法，这是一个静态方法，调用该方法可以退出当前线程。

    [thread cancel];////注意设置为取消状态仅仅是改变了线程状态而言，并不能终止线程
    [thread exit];//退出当前线程
使用NSThread在进行多线程开发过程中操作比较简单，但是要控制线程执行顺序并不容易。

######*扩展--NSObject分类扩展方法*

为了简化多线程开发过程，苹果官方对NSObject进行分类扩展(本质还是创建NSThread)，对于简单的多线程操作可以直接使用这些扩展方法。

    - (void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg：在后台执行一个操作，本质就是重新创建一个线程执行当前方法。

    - (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait：在指定的线程上执行一个方法，需要用户创建一个线程对象。

    - (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait：在主线程上执行一个方法（前面已经使用过）。

###NSOperation

使用NSOperation和NSOperationQueue进行多线程开发类似于C#中的线程池，只要将一个NSOperation（实际开中需要使用其子类NSInvocationOperation、NSBlockOperation）放到NSOperationQueue这个队列中线程就会依次启动。NSOperationQueue负责管理、执行所有的NSOperation，在这个过程中可以更加容易的管理线程总数和控制线程之间的依赖关系。

NSOperation有两个常用子类用于创建线程操作：NSInvocationOperation和NSBlockOperation，两种方式本质没有区别，但是是后者使用Block形式进行代码组织，使用相对方便。
#####NSOperation的子类NSInvocationOperation

    /*创建一个调用操作
     object:调用方法参数
    */
    NSInvocationOperation *invocationOperation=[[NSInvocationOperation alloc]initWithTarget:self selector:@selector(loadImage) object:nil];
    //创建完NSInvocationOperation对象并不会调用，它由一个start方法启动操作，但是注意如果直接调用start方法，则此操作会在主线程中调用，一般不会这么操作,而是添加到NSOperationQueue中
    //[invocationOperation start];
    //创建操作队列
    NSOperationQueue *operationQueue=[[NSOperationQueue alloc]init];
    //注意添加到操作队后，队列会开启一个线程执行此操作
    [operationQueue addOperation:invocationOperation];

#####NSOperation的子类NSBlockOperation

	//创建操作队列
    NSOperationQueue *operationQueue=[[NSOperationQueue alloc]init];
    operationQueue.maxConcurrentOperationCount=5;//设置最大并发线程数

    //方法1：创建操作块添加到队列
    //创建多线程操作
    NSBlockOperation *blockOperation=[NSBlockOperation blockOperationWithBlock:^{
         [self loadImage:[NSNumber numberWithInt:i]];//线程的操作
    }];
    //将NSBlockOperation添加到操作队列
    [operationQueue addOperation:blockOperation];

    //方法2：直接使用操队列添加操作
    [operationQueue addOperationWithBlock:^{
        [self loadImage:[NSNumber numberWithInt:i]];//线程的操作
    }];
在主队列中进行UI的刷新

	//更新UI界面,此处调用了主线程队列的方法（mainQueue是UI主线程）
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        [self updateImageWithData:data andIndex:i];
    }];
1.使用NSBlockOperation方法，所有的操作不必单独定义方法，同时解决了只能传递一个参数的问题。
2.调用主线程队列的addOperationWithBlock:方法进行UI更新，不用再定义一个参数实体。
3.使用NSOperation进行多线程开发可以设置最大并发线程，有效的对线程进行了控制。

#####NSOperation控制线程执行顺序

前面使用NSThread很难控制线程的执行顺序，但是使用NSOperation就容易多了，使用addDependency:可以对每个NSOperation可以设置依赖线程。假设操作A依赖于操作B，线程操作队列在启动线程时就会首先执行B操作，然后执行A。

        [blockOperation addDependency:lastBlockOperation];//设置依赖操作为最后一张图片加载操作，设置了依赖关系再添加到线程池。（这里设置blockOperation依赖于lastBlockOperation）

###GCD
GCD(Grand Central Dispatch)是基于C语言开发的一套多线程开发机制，也是目前苹果官方推荐的多线程开发方法。前面也说过三种开发中GCD抽象层次最高，当然是用起来也最简单，只是它基于C语言开发，并不像NSOperation是面向对象的开发，而是完全面向过程的。对于熟悉C#异步调用的朋友对于GCD学习起来应该很快，因为它与C#中的异步调用基本是一样的。这种机制相比较于前面两种多线程开发方式最显著的优点就是它对于多核运算更加有效。

GCD中也有一个类似于NSOperationQueue的队列，GCD统一管理整个队列中的任务。但是GCD中的队列分为并行队列和串行队列两类：

>1.串行队列：只有一个线程，加入到队列中的操作按添加顺序依次执行。
>2.并发队列：有多个线程，操作进来之后它会将这些队列安排在可用的处理器上，同时保证先进来的任务优先处理。

其实在GCD中还有一个特殊队列就是主队列，用来执行主线程上的操作任务（从前面的演示中可以看到其实在NSOperation中也有一个主队列）。

更新UI界面,此处调用了GCD主线程队列的方法：

    dispatch_queue_t mainQueue= dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{
        [self updateImageWithData:data andIndex:i];
    });//同步调用

#####GCD串行队列
使用串行队列时首先要创建一个串行队列，然后调用异步调用方法，在此方法中传入串行队列和线程操作即可自动执行。

	/*创建一个串行队列
     第一个参数：队列名称
     第二个参数：队列类型
    */
    dispatch_queue_t serialQueue=dispatch_queue_create("myThreadQueue1", DISPATCH_QUEUE_SERIAL);//注意queue对象不是指针类型
     //异步执行队列任务
    dispatch_async(serialQueue, ^{
        [self loadImage:[NSNumber numberWithInt:i]];
    });
    //如果在串行队列中会发现当前线程打印变化完全一样，因为他们在一个线程中。（例如在loadImage:方法中打印）
    //非ARC环境请释放
	//dispatch_release(seriQueue);

#####并发队列

并发队列同样是使用dispatch_queue_create()方法创建，只是最后一个参数指定为DISPATCH_QUEUE_CONCURRENT进行创建，但是在实际开发中我们通常不会重新创建一个并发队列而是使用dispatch_get_global_queue()方法取得一个全局的并发队列（当然如果有多个并发队列可以使用前者创建）。

	/*取得全局队列
     第一个参数：线程优先级
     第二个参数：标记参数，目前没有用，一般传入0
    */
    dispatch_queue_t globalQueue=dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    //异步执行队列任务
    dispatch_async(globalQueue, ^{
        [self loadImage:[NSNumber numberWithInt:i]];
    });
异步调用方法和同步调用方法

    dispatch_async()异步调用方法。
    dispatch_sync()同步调用方法。
>1.在GDC中一个操作是多线程执行还是单线程执行取决于当前队列类型和执行方法，只有队列类型为并行队列并且使用异步方法执行时才能在多个线程中执行。
>2.串行队列可以按顺序执行，并行队列的异步方法无法确定执行顺序。
>3.UI界面的更新最好采用同步方法，其他操作采用异步方法。

#####GCD其他任务执行方法

GCD执行任务的方法并非只有简单的同步调用方法和异步调用方法，还有其他一些常用方法：

    dispatch_apply():重复执行某个任务，但是注意这个方法没有办法异步执行（为了不阻塞线程可以使用dispatch_async()包装一下再执行）。

    dispatch_once():单次执行一个任务，此方法中的任务只会执行一次，重复调用也没办法重复执行（单例模式中常用此方法）。

    dispatch_time()：延迟一定的时间后执行。

    dispatch_barrier_async()：使用此方法创建的任务首先会查看队列中有没有别的任务要执行，如果有，则会等待已有任务执行完毕再执行；同时在此方法后添加的任务必须等待此方法中任务执行后才能执行。（利用这个方法可以控制执行顺序，例如前面先加载最后一张图片的需求就可以先使用这个方法将最后一张图片加载的操作添加到队列，然后调用dispatch_async()添加其他图片加载任务）

    dispatch_group_async()：实现对任务分组管理，如果一组任务全部完成可以通过dispatch_group_notify()方法获得完成通知（需要定义dispatch_group_t作为分组标识）。

###线程同步
说到多线程就不得不提多线程中的锁机制，多线程操作过程中往往多个线程是并发执行的，同一个资源可能被多个线程同时访问，造成资源抢夺，这个过程中如果没有锁机制往往会造成重大问题。

要解决资源抢夺问题在iOS中有常用的有两种方法：
>1.一种是使用NSLock同步锁
>2.另一种是使用@synchronized代码块。两种方法实现原理是类似的，只是在处理上代码块使用起来更加简单（C#中也有类似的处理机制synchronized和lock）。

#####NSLock

iOS中对于资源抢占的问题可以使用同步锁NSLock来解决，使用时把需要加锁的代码（以后暂时称这段代码为”加锁代码“）放到NSLock的lock和unlock之间，一个线程A进入加锁代码之后由于已经加锁，另一个线程B就无法访问，只有等待前一个线程A执行完加锁代码后解锁，B线程才能访问加锁代码。需要注意的是lock和unlock之间的”加锁代码“应该是抢占资源的读取和修改代码，不要将过多的其他操作代码放到里面，否则一个线程执行的时候另一个线程就一直在等待，就无法发挥多线程的作用了。

    //初始化锁对象
    NSLock *lock=[[NSLock alloc]init];
	//加锁
    [lock lock];
    中间是需要加锁的代码块（线程之间抢夺的资源）
    //使用完解锁
    [lock unlock];

    另外还有一个lockBeforeData:方法指定在某个时间内获取锁，同样返回一个BOOL值，如果在这个时间内加锁成功则返回YES，失败则返回NO。

#####@synchronized代码块

使用@synchronized解决线程同步问题相比较NSLock要简单一些，日常开发中也更推荐使用此方法。首先选择一个对象作为同步对象（一般使用self），然后将”加锁代码”（争夺资源的读取、修改代码）放到代码块中。@synchronized中的代码执行时先检查同步对象是否被另一个线程占用，如果占用该线程就会处于等待状态，直到同步对象被释放。下面的代码演示了如何使用@synchronized进行线程同步：

	//线程同步
    @synchronized(self){
	//加锁代码
    }
#####扩展--使用GCD解决资源抢占问题

在GCD中提供了一种信号机制，也可以解决资源抢占问题（和同步锁的机制并不一样）。GCD中信号量是dispatch_semaphore_t类型，支持信号通知和信号等待。每当发送一个信号通知，则信号量+1；每当发送一个等待信号时信号量-1,；如果信号量为0则信号会处于等待状态，直到信号量大于0开始执行。根据这个原理我们可以初始化一个信号量变量，默认信号量设置为1，每当有线程进入“加锁代码”之后就调用信号等待命令（此时信号量为0）开始等待，此时其他线程无法进入，执行完后发送信号通知（此时信号量为1），其他线程开始进入执行，如此一来就达到了线程同步目的。

    dispatch_semaphore_t _semaphore;//定义一个信号量

    /*初始化信号量
     参数是信号量初始值
     */
    _semaphore=dispatch_semaphore_create(1);

    /*信号等待
     第二个参数：等待时间
     */
    dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);//-1
    //加锁代码
    //信号通知
    dispatch_semaphore_signal(_semaphore);//+1

#####扩展-NSCondition-控制线程通信

由于线程的调度是透明的，程序有时候很难对它进行有效的控制，为了解决这个问题iOS提供了NSCondition来控制线程通信(同前面GCD的信号机制类似)。NSCondition实现了NSLocking协议，所以它本身也有lock和unlock方法，因此也可以将它作为NSLock解决线程同步问题，此时使用方法跟NSLock没有区别，只要在线程开始时加锁，取得资源后释放锁即可，这部分内容比较简单在此不再演示。当然，单纯解决线程同步问题不是NSCondition设计的主要目的，NSCondition更重要的是解决线程之间的调度关系（当然，这个过程中也必须先加锁、解锁）。NSCondition可以调用wati方法控制某个线程处于等待状态，直到其他线程调用signal（此方法唤醒一个线程，如果有多个线程在等待则任意唤醒一个）或者broadcast（此方法会唤醒所有等待线程）方法唤醒该线程才能继续。

    NSCondition *_condition;//定义
    //初始化锁对象
    _condition=[[NSCondition alloc]init];
    //加锁
    [_condition lock];
    	//唤醒所有等待线程
        [_condition broadcast];
        //线程等待
        [_condition wait];
 		//发出信号唤醒其他等待线程
        [_condition signal];
    //解锁
    [_condition unlock];
#####iOS中的其他锁
在iOS开发中，除了同步锁有时候还会用到一些其他锁类型，在此简单介绍一下：

NSRecursiveLock ：递归锁，有时候“加锁代码”中存在递归调用，递归开始前加锁，递归调用开始后会重复执行此方法以至于反复执行加锁代码最终造成死锁，这个时候可以使用递归锁来解决。使用递归锁可以在一个线程中反复获取锁而不造成死锁，这个过程中会记录获取锁和释放锁的次数，只有最后两者平衡锁才被最终释放。

NSDistributedLock：分布锁，它本身是一个互斥锁，基于文件方式实现锁机制，可以跨进程访问。

pthread_mutex_t：同步锁，基于C语言的同步锁机制，使用方法与其他同步锁机制类似。

在开发过程中除非必须用锁，否则应该尽可能不使用锁，因为多线程开发本身就是为了提高程序执行顺序，而同步锁本身就只能一个进程执行，这样不免降低执行效率。
###总结
1>无论使用哪种方法进行多线程开发，每个线程启动后并不一定立即执行相应的操作，具体什么时候由系统调度（CPU空闲时就会执行）。

2>更新UI应该在主线程（UI线程）中进行，并且推荐使用同步调用，常用的方法如下：

    - (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait (或者-(void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL) wait;方法传递主线程[NSThread mainThread])

    [NSOperationQueue mainQueue] addOperationWithBlock:

    dispatch_sync(dispatch_get_main_queue(), ^{})
3>NSThread适合轻量级多线程开发，控制线程顺序比较难，同时线程总数无法控制（每次创建并不能重用之前的线程，只能创建一个新的线程）。

4>对于简单的多线程开发建议使用NSObject的扩展方法完成，而不必使用NSThread。

5>可以使用NSThread的currentThread方法取得当前线程，使用 sleepForTimeInterval:方法让当前线程休眠。

6>NSOperation进行多线程开发可以控制线程总数及线程依赖关系。

7>创建一个NSOperation不应该直接调用start方法（如果直接start则会在主线程中调用）而是应该放到NSOperationQueue中启动。

8>相比NSInvocationOperation推荐使用NSBlockOperation，代码简单，同时由于闭包性使它没有传参问题。

9>NSOperation是对GCD面向对象的ObjC封装，但是相比GCD基于C语言开发，效率却更高，建议如果任务之间有依赖关系或者想要监听任务完成状态的情况下优先选择NSOperation否则使用GCD。

10>在GCD中串行队列中的任务被安排到一个单一线程执行（不是主线程），可以方便地控制执行顺序；并发队列在多个线程中执行（前提是使用异步方法），顺序控制相对复杂，但是更高效。

11>在GDC中一个操作是多线程执行还是单线程执行取决于当前队列类型和执行方法，只有队列类型为并行队列并且使用异步方法执行时才能在多个线程中执行（如果是并行队列使用同步方法调用则会在主线程中执行）。

12>相比使用NSLock，@synchronized更加简单，推荐使用后者。