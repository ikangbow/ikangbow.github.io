---
title: Java异常及使用
date: 2019-02-02 11:37:08
tags: java
category: java
---
# Java异常

## Java异常类层次结构图

java 异常是程序运行过程中出现的错误。Java把异常当作对象来处理，并定义一个基类java.lang.Throwable作为所有异常的超类。在Java API中定义了许多异常类,分为两大类，错误Error和异常Exception。其中异常类Exception又分为运行时异常(RuntimeException)和非运行时异常(非runtimeException)，也称之为不检查异常（Unchecked Exception）和检查异常（Checked Exception）。

![](throwable.png)


## Error与Exception
	
Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。
这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。
Exception是程序本身可以处理的异常，这种异常分两大类运行时异常和非运行时异常。程序中应当尽可能去处理这些异常。
	
## 运行时异常和非运行时异常

1.运行时异常: 都是RuntimeException类及其子类异常：
 
	IndexOutOfBoundsException：索引越界异常

	ArithmeticException：数学计算异常

	NullPointerException：空指针异常

	ArrayOutOfBoundsException：数组索引越界异常

	ClassNotFoundException：类文件未找到异常

	ClassCastException：造型异常（类型转换异常）

这些异常是不检查异常（Unchecked Exception），程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的。

 2.非运行时异常:是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如：

	IOException：文件读写异常

	FileNotFoundException：文件未找到异常

	EOFException：读写文件尾异常
	
	MalformedURLException：URL格式错误异常

	SocketException：Socket异常

	SQLException：SQL数据库异常

## 异常的捕获和处理

Java异常的捕获和处理是一个不容易把握的事情，如果处理不当，不但会让程序代码的可读性大大降低， 而且导致系统性能低下，甚至引发一些难以发现的错误。

1.try、catch方式：

    try{   
    //（尝试运行的）程序代码   
    }catch(异常类型 异常的变量名){   
    //异常处理代码   
    }finally{   
     //异常发生，方法返回之前，总是要执行的代码   
    }  

try、catch、finally三个语句块应注意的问题
第一、try、catch、finally三个语句块均不能单独使用，三者可以组成 try...catch...finally、try...catch、 try...finally三种结构，catch语句可以有一个或多个，finally语句最多一个。
第二、try、catch、finally三个代码块中变量的作用域为代码块内部，分别独立而不能相互访问。如果要在三个块中都可以访问，则需要将变量定义到这些块的外面。
第三、多个catch块时候，只会匹配其中一个异常类并执行catch块代码，而不会再执行别的catch块，并且匹配catch语句的顺序是由上到下。

2.抛异常给上一级方式：

    public static void demo() throws Exception{   
      //抛出一个检查异常   
      throw new Exception("方法demo中的Exception");   
    }

上面的代码可以看到两个关键字，throw和throws关键字
throw：是用于方法体内部，用来抛出一个Throwable类型的异常。
throws：是用于方法体外部的方法声明部分，用来声明方法可能会抛出某些异常。仅当抛出了检查异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出。
异常处理是为了程序的健壮性。异常能处理就处理，不能处理就抛出，最终没有处理的异常JVM会进行处理。对于一个应用系统来说，应该有自己的一套异常处理框架，这样当异常发生时，也能得到统一的处理风格，将优雅的异常信息反馈给用户。

# Java异常使用

在开发业务系统中，目前绝大多数采用MVC模式。
考虑如下场景：系统提供一个API，用于修改用户信息，服务器端采用json数据交互.首先我们定义BaseException，用来表示业务逻辑受理失败，它仅表示我们处理业务的时候发现无法继续执行下去。

    /**
	 * 业务受理失败异常
	 */
	public class BaseException extends RuntimeException{
		public ServiceException() {
	        super();
	    }

	    public BaseException(String arg0, Throwable arg1) {
	        super(arg0, arg1);
	    }

	    public BaseException(String arg0) {
	        super(arg0);
	    }

	    public BaseException(Throwable arg0) {
	        super(arg0);
	    }
	}

接下来Controller层

    @RequestMapping
    pulic ReturnStatus updateUser(User user){
		userService.updateUser(user);
		ReturnStatus status = new ReturnStatus(ture,"更新成功");
		status.setEntity(user)
		return status;
    }

关于上述Controller写法乍一看会有一些冗余，如果无法理解，请仔细研读MVC设计模式. 先不管service，我们来考虑下。 一个业务系统不可能不对用户提交的数据进行验证，验证包括两方面：有效性和合法性
- 有效性: 比如用户所在岗位,是否属于数据库有记录的岗位ID,如果不存在,无效.
- 合法性: 比如用户名只允许输入最多12个字符,用户提交了20个字符,不合法.

有效性检查，可以交给java的校验框架执行，比如JSR303。假设用户提交的数据经过验证都合法，还是有一些情况是不能调用修改逻辑的。

1. 要修改的用户ID不存在.
2. 用户被锁定,不允许修改.
3. 乐观锁机制发现用户已经被被人修改过.
4. 由于某种原因,我们的程序无法保存到数据库.
5. 一些程序员错误的开发了代码,导致保存过程中出现异常,比如NPE.

对于前3种，我们认为是有效性检查失败，第4种属与我们无法处理的异常，第5种就是程序员bug。

现在的问题是，前三种情况我们如何通知用户呢?

1.在ccontroller 调用userService的checkUserExist()方法.
2.在controller直接书写业务逻辑.
3.在service响应一个状态码机制,比如1 2 3表示错误信息,0 表示没有任何错误.

显然前2种方法都不可取，因为***MVC设计模式告诉我们,controller是用来接收页面参数，并且调用逻辑处理，最后组织页面响应的地方。我们不可以在controller进行逻辑处理，controller只应该负责用户API入口和响应的处理(如若不然,思考一下如果有一天service的代码打包成jar放到另一个平台，没有controller了,该怎么办？)***

状态码机制是个不错的选择，可是如此一来，用户保存逻辑变了，比如增加一个情况，不允许修改已经离职的用户，那么我们还需要修改controller的代码，代码量增加，维护成本增高，并且还耦合了service，不符合MVC设计模式。

那么怎么办呢？现在我们来看下service代码如何编写

    public void updateUser(User user){
		User userOrig = userDao.getUserById(user.getUserById);
		if(null == userOrig){
			throw new BaseException("用户不存在");
		}
		if(userOrig.isLocked()){
			throw new BaseException("用户被锁定,不允许修改");
		}
		if(!user.getVersion.equals(userOrig.getVersion)){
			throw new BaseException("用户已经被别人修改,请刷新");
		}
		//TODO保存用户数据...
    }

这样一来只要我们检查到不允许保存的项目，我们就可以直接throw 一个新的异常，异常机制会帮助我们中断代码执行。

接下来有2种选择:
1. 在controller 使用try-catch进行处理.
2. 直接把异常抛给上层框架统一处

第1种方式是不可取的，注意我们抛出的BaseException，它仅仅逻辑处理异常,并且我们的方法前面没有声明throws BaseException，这表示他是一个非受查异常.controller也没有关心会发生什么异常。

为什么不定义成受查异常呢？如果是一个受查异常，那么意味着controller必须要处理你的异常。并且如果有一天你的业务逻辑变了，可能多一种检查项，就需要增加一个异常，反之需要删除一个异常，那么你的方法签名也需要改变，controller也随之要改变,这又变成了紧耦合，这和用状态码123表示处理结果没有什么不同。

我们可以为每一种检查项定义一个异常吗？可以，但是那样显得太多余了。因为业务逻辑处理失败的时候，根据我们需求，我们只需要通知用户失败的原因(通常应该是一段字符串)，以及服务器受理失败的一个状态码(有时可能不需要状态码,这要看你的设计了)，这样这需要一个包含原因属性的异常即可满足我们需求。

最后我们决定这个异常继承自RuntimeException。并且包含一个接受一个错误原因的构造器，这样controller层也不需要知道异常，只要全局捕获到BaseException做统一的处理即可，这无论是在struct1,2时代，还是springMVC中，甚至servlet年代，都是极为容易的！

异常不提供无参构造器，因为绝对不允许你抛出一个逻辑处理异常，但是不指明原因，想想看，你是必须要告诉用户为什么受理失败的！

如此一来，我们只需要全局统一处理下 BaseException 就可以了，很好，spring为我们提供了ControllerAdvice机制，有关ControllerAdvice，可以查阅springMVC使用文档，下面是一个简单的示例：

    @ControllerAdvice(basePackages = {"com.xxx.xxx.bussiness.xxx"})
    public class ModuleControllerAdvice{
		public static final Logger LOGGER = LoggerFactory.getLogger(ModuleControllerAdvice.class);
		public static final Logger BASE_LOGGER = LoggerFactory.getLogger(BaseException.class);
		/**
		 * 业务受理失败
		 */
		 @ResponseBody
		 @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
		 @ExceptionHandler(BaseException.class)
		 private ReturnStatus handleServiceException(BaseException exception){
			String message = "业务受理失败,原因："+exception.getLocalizedMessage();
			SERVICE_LOGGER.info(message);
			ReturnStatus status = new ReturnStatus(MErrorCode.e2001.code(), MErrorCode.e2001.message());//自定义枚举异常类
			return status;
		 }
    }

在这个时候，我们就可以很轻松的处理各种情况了.

注意一点，在这个类中，我们定义了2个log对象，分别指向 BaseException.class 和 ModuleControllerAdvice.class。并且处理 BaseException的时候使用了info级别的日志输出，这是很有用的。

首先,BaseException一定要和其他的代码错误分离,不应该混为一谈.
其次,BaseException并不一定要记录日志,我们应该提供独立的log对象,方便开关.
接下来你可以在修改用户的时候想客户端响应这样的JSON

    {
		code : e2001,
		message : "用户不存在"
    }
如此一来没有任何地方需要关心异常，或者业务逻辑校验失败的情况。用户也可以得到很友好的错误提示。
如果你只需要一句概括，那么直接定义一个简单的异常，用于中断处理，并且与用户保持友好交互即可。
如果不可能一句话描述清楚，并且包含附加信息，比如需要在日志或者数据库记录消息ID，此时可能专门针对这种重要/复杂业务创建独立异常。
上述两种情况因为web系统，是用户发起请求之后需要等待程序给予响应结果的。

如果是后台作业，或者复杂业务需要追溯性。这种通常用流程判断语句控制，要用异常处理。我们认为这些流程判断一定在一个原子性处理中。并且检查到(不是遇到)的问题(不是异常)需要记录到用户可友好查看的日志。这种情况属于处理反馈，并不叫异常。

综上，笔者通常分为如下几类:
逻辑异常，这类异常用于描述业务无法按照预期的情况处理下去，属于用户制造的意外。
代码错误，这类异常用于描述开发的代码错误，例如NPE，ILLARG，都属于程序员制造的BUG。
专有异常，多用于特定业务场景，用于描述指定作业出现意外情况无法预先处理。
各类异常必须要有单独的日志记录，或者分级，分类可管理。有的时候仅仅想给三方运维看到逻辑异常。

注意

异常设计的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多。
上面这句话出自<java编程思想>，但是我们思考如下几点：
业务逻辑检查，也是意外情况
UnknownHostException，表示找不到这样的主机，这个异常和NoUserException有什么区别么？换言之，没有这样的主机是异常，没有这样的用户不是异常了么？所以一定要弄明白什么是用异常来控制逻辑，什么是定义程序异常。

异常处理效率很低

书中所示的例子，是在循环中大量使用try-catch进行检查，但是业务系统，用户发起请求的次数与该场景天壤地别。淘宝的11`11是个很好的反例。但是请你的系统上到这个级别再考虑这种问题。
系统有千万并发，不可能还去考虑这些中规中矩的按部就班的方式，别忘了MVC本来就浪费很多资源，代码量增加很多。
业务系统也存在很多巨量任务处理的情况。但是那些任务都是原子性的，现在MVC中的controller和service可不是原子性的，不然为什么要区分这么多层呢。
如果那么在乎效率，考虑下重写Throwable的fillStackTrace方法。你要知道异常的开销大到底大在什么地方，fillStackTrace是一个native方法，会填充异常类内部的运行轨迹。

不要用异常进行业务逻辑处理

我们先来看一个例子:

    public void processMessage(Message<String> message){
		try{
			//处理消息验证
			//处理消息解析
			//处理消息入库
		}catch(ValidateException e){
			//验证失败
		}catch(ParseException e){
			//解析失败
		}catch(PersistException e){
			//入库失败
		}
}

上述代码就是典型的使用异常来处理业务逻辑。这种方式需要严重的禁止！上述代码最大的问题在于，我们如何利用异常来自动处理事务呢？
然而这和我们的异常中断service没有什么冲突.也并不是一回事.
我们提倡在 业务处理 的时候，如果发现无法处理直接抛出异常即可。
而并不是在 逻辑处理 的时候，用异常来判断逻辑进行的状况。
改正后的逻辑

    public void processMessage(Message<String> message){
	
		if(!message.isValud){
			MessageLogService.log("消息验证失败"+message.errors());
			return;
		}
		...
		//TODO ...
    }














