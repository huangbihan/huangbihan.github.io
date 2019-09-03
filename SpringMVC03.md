### 1. 在SpringMVC的控制器中使用Session

当需要使用Session时，在处理请求的方法的参数列表中添加`HttpSession`类型的参数即可！然后，在方法体中，调用参数对象就可以向Session中存入数据、取出Session的数据等。

另外，在SpringMVC中，还提供了另一种使用Session的方式，其做法是：将需要保存到Session中的数据，直接存入到`ModelMap`中，然后，在类之前添加例如`@SessionAttributes({"uname"})`注解，用于表示**ModelMap中的哪些数据是需要放到Session中的**。这种做法的优点在于**便于执行单元测试**！但是，缺点在于**放在Session中的数据在转发时也都可以直接使用，即这些数据既在Session中存在，也在Request中存在，使用时不直观，并且会将Session中的数据暴露在URL中，使用这种方式操作的Session数据并不能通过HttpSession对象的invalidate()方法进行清除**！

在实际应用中，简化的操作方案还是推荐直接使用`HttpSession`，并不采用SpringMVC提供的解决方案，主要原因是：通常并不会对控制器中的方法执行单元测试，如果要测试控制器中的方法，可以通过在浏览器中直接输入网址进行测试，或者，使用专门的测试工具，所以，“是否便于执行单元测试”就不再是选择解决方案的重要依据！甚至，在大型应用中，根本就不使用Session！

### 2. SpringMVC中的拦截器(Interceptor)

在SpringMVC中，可以使用拦截器对指定的请求路径进行拦截，并在拦截过程中分析、处理，最终选择放行，或阻止继续向后执行！

当需要使用拦截器时，可以先自定义类，实现拦截器接口`HandlerInterceptor`，并重写接口中的抽象方法：

	public class LoginInterceptor implements HandlerInterceptor {
	
		public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
				throws Exception {
			System.out.println("LoginInterceptor.afterCompletion()");
		}
	
		public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
				ModelAndView modelAndView) throws Exception {
			System.out.println("LoginInterceptor.postHandle()");
		}
	
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
				throws Exception {
			System.out.println("LoginInterceptor.preHandle()");
			return false;
		}
	
	}

然后，需要在**spring.xml**中对拦截器进行配置：

	<!-- 配置拦截器链 -->
	<mvc:interceptors>
		<!-- 配置第1个拦截器 -->
		<mvc:interceptor>
			<!-- 拦截的请求路径 -->
			<mvc:mapping path="/index.do" />
			<!-- 指定拦截器类 -->
			<bean class="cn.tedu.spring.LoginInterceptor" />
		</mvc:interceptor>
	</mvc:interceptors>

真正能起到“拦截”效果的只有拦截器类中的`preHandle()`方法，该方法返回`false`时将阻止继续执行，返回`true`时将放行，可以重写该方法确定拦截逻辑：

	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("LoginInterceptor.preHandle()");
		HttpSession session = request.getSession();
		if (session.getAttribute("username") == null) {
			response.sendRedirect("user/login.do");
			return false;
		}
		// 返回true表示放行
		// 返回false表示阻止继续运行
		return true;
	}

在配置拦截器的拦截路径时，可以使用多个`<mvc:mapping />`节点，以配置多个需要拦截的路径，例如：

	<mvc:mapping path="/index.do" />
	<mvc:mapping path="/help.do" />

也可以使用星号`*`号作为通配符，例如：

	<mvc:mapping path="/*" />

在这里使用的星号`*`只能通配1层级的资源，例如以上配置可以匹配到`/index.do`、`/help.do`，却不可以匹配到`/user/reg.do`、`/user/login.do`等路径，如果一定要匹配多层级，需要使用2个星号，例如：

	<mvc:mapping path="/**" />

以上配置可以匹配到`/index.do`、`/help.do`，也可以匹配到`/user/reg.do`、`/user/login.do`，甚至还可以匹配`/a/b/c/d/e.do`这类层级更多的路径！

在配置时，还可以通过`<mvc:exclude-mapping />`节点配置例外，即白名单，如果将某些请求路径添加为例外，则拦截器对于这些请求路径也是不予处理的！与配置拦截路径相同，在配置例外时，也可以添加若干个`<mvc:exclude-mapping />`节点，也可以使用星号通配符！

所以，相比Java EE中的过滤器(Filter)来说，SpringMVC中的拦截器在使用和配置方面更加灵活！

### 3. Spring小结

1. 理解：Spring框架的作用；

2. 掌握：通过SET方式注入属性的值；

3. 了解：Spring管理的对象的作用域与生命周期；

4. 了解：通过构造方法注入属性的值；

5. 掌握：通过Spring中的配置，读取.properties文件；

6. 了解：通过Spring中的配置，注入集合类型的值；

7. 掌握：Spring表达式；

8. 理解：自动装配中的byType模式和byName模式；

9. 掌握：组件扫描与@Component / @Controller / @Service / @Repository注解；

10. 掌握：@Autowired和@Resource这2个自动装配的注解；

11. 了解：与作用域、生命周期相关的注解：@Scope、@Lazy、@PostConstruct、@PreDestroy；

12. 理解：IoC与DI。

13. SpringAOP：项目末期再讲。

### 4. SpringMVC小结

1. 理解：SpringMVC框架的作用；

2. 理解：SpringMVC框架的核心执行流程；

3. 掌握：创建SpringMVC的项目；

4. 掌握：@RequestMapping注解的使用；

5. 掌握：接收请求参数；

6. 了解：使用ModelMap封装需要转发的数据；

7. 理解：转发与重定向；

8. 掌握：@RequestParam注解的使用；

9. 掌握：通过HttpSession向Session中封装数据，并取出数据；

10. 掌握：拦截器的使用，拦截器与过滤器的区别。

### 5. 数据库知识的作业

**写出以下所有操作的SQL语句**

1. 创建名为`tedu_ums`的数据库(DataBase)；

2. 在该数据库中创建名为`t_user`的数据表，该表中至少包括：id、用户名、密码、年龄、手机号码、电子邮箱这些字段；

3. 向数据表插入不少于10条数据，每条数据的内容尽量完整且随机；

4. 删除id=7的数据；

5. 删除id=2、id=4、id=9的数据；

6. 将id=6的用户的邮箱改成`user06@tedu.cn`；

7. 将所有用户的密码改成`888888`；

8. 查询当前数据表中有多少条用户的数据；

9. 查询id=5的用户的所有数据；

10. 查询当前数据表中所有用户的数据；

11. 查询当前数据表中年龄最大的那个用户的所有数据(前提：保证当前数据表中年龄最大的用户只有1个)。

### -----------------------------------------

### 附1：使用Session

Session是在服务器端的内存中的数据。

哪些数据需要存储在Session中：

1. 用户的唯一标识，例如：用户的id等；

2. 使用频率极高的数据，例如：用户名等；

3. 不便于使用其它解决方案来存取或传输的数据。

### 附2：过滤器和拦截器的区别

过滤器(Filter)是JavaEE中的组件，而拦截器(Interceptor)是SpringMVC中的组件。

无论是过滤器，还是拦截器，都可以使得若干种请求都需要执行该组件中的代码，并且，都可以设计为放行或阻止继续运行，甚至都可以存在若干个，形成对应的“链”。

使用过滤器时，执行时间节点偏早期，会在所有的Servlet之前执行；拦截器会在`DispatcherServlet`接收到请求之后，在控制器进行处理之前被执行。

在配置方面，过滤器只能配置1个映射路径，而拦截器的配置非常灵活！

由于过滤器是Java EE中的组件，只要是使用Java语言开发的WEB项目，都可以使用过滤器，而拦截器是SpringMVC中的组件，只有开发环境添加了SpringMVC框架后，才可以使用拦截器，并且，只有被`DispatcherServlet`映射的路径，才可能被拦截器处理！

虽然拦截器的便于使用，配置简单，优点较多，但是，也不能完全取代过滤器，例如`CharacterEncodingFilter`实现设置请求与响应的字符编码，就只能在过滤器中完成，而不可以通过拦截器进行配置，其主要原因是在于两者的执行时间节点的区别！所以，如果需要执行时间节点在Servlet之前，就必须使用过滤器，否则，优先使用拦截器！

### 附3：单例模式

单例模式是一种**生产对象型**的设计模式。

单例模式能够**保证在同一时间，某个类的实例(对象)一定只有1个**。

假设存在`King`类：

	public class King {
	}

这样的类肯定不是单例的，因为默认存在公有的、无参数的构造方法，可以随意的创建对象，例如：

	King k1 = new King();
	King k2 = new King();

如果希望该类的对象唯一，首先，就应该限制构造方法的访问：

	public class King {
		private King() {
		}
	}

则以上创建对象的语句就是不成立的，但是，把构造方法私有，其目的是**不允许随意创建对象**，而不是**不允许对象**，依然可以在类的内部创建对象：

	public class King {
		private King king = new King();

		private King() {
		}
	}

由于仅当前类可以创建对象，且在代码中只创建1次对象，就可以保证**单例**效果！同时，为了保证类的外部能够获取该对象，还可以添加公有的方法来获取：

	public class King {
		private King king = new King();

		private King() {
		}

		public King getInstance() {
			return king;
		}
	}

则外部需要`King`类的对象时，调用`getInstance()`方法即可！当然，以上代码是矛盾的，因为调用`getInstance()`方法的前提是得先有`King`的对象，但是，获取对象的唯一途径就是调用该方法！所以，为了保证能够顺利的调用方法，需要在方法的声明中添加`static`关键字进行修饰，同时，因为**static成员不可以直接访问非static成员**，在被`static`修饰的`getInstance()`方法中不可以直接访问`king`变量，所以，还需要为`king`变量也添加`static`进行修饰：

	public class King {
		private static King king = new King();

		private King() {
		}

		public static King getInstance() {
			return king;
		}
	}

至此，一个简单的单例模式就已经完成！当需要获取`King`类的对象时，通过`King.getInstance()`方法就可以获取对象，并且，无论调用多少次该方法，获取到的都是同一个对象！

这种单例模式是**随时需要对象，随时就可以获取对象，因为对象早早的就创建好了，随时可以获取的**，所以，这种单例模式称之为**饿汉式单例模式(随时饿随时吃)**。

另外，还有**懒汉式单例模式**，其特征是**不到逼不得已，不会创建类的对象**，其大致代码是：

	public class King {
		private static King king;

		private King() {
		}

		public static King getInstance() {
			if (king == null) {
				king = new King();
			}
			return king;
		}
	}

以上单例模式的设计是存在线程安全问题的！为了解决该问题，可以：

	public class King {
		private static King king;

		private static final Object LOCK = new Object();

		private King() {
		}

		public static King getInstance() {
			synchronized(LOCK) {
				if (king == null) {
					king = new King();
				}
			}
			return king;
		}
	}

但是，每次获取对象时，都需要锁定对象进行判断，性能方面表现较差，还可以继续添加判断，以决定是否需要锁定并判断：
	
	public class King {
		private static King king;

		private static final Object LOCK = new Object();

		private King() {
		}

		public static King getInstance() {
			if (king == null) {
				synchronized(LOCK) {
					if (king == null) {
						king = new King();
					}
				}
			}
			return king;
		}
	}

至此，**懒汉式单例模式**已经完全实现！