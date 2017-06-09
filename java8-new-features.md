#java8 新特性介绍

##一. 介绍

　　毫无疑问，Java 8是自Java  5（2004年）发布以来Java语言最大的一次版本升级，Java 8带来了很多的新特性，比如编译器、类库、开发工具和JVM（Java虚拟机）


##二. Lambda表达式

　　Lambda表达式允许我们将一个函数当作方法的参数（传递函数），或者说把代码当作数据;

　　**Lambda语法**  

包含三个部分:   
1. 一个括号内用逗号分隔的形式参数，参数是函数式接口里面方法的参数，编译器会根据上下文来推测参数的类型，也可以显示地指定参数类型，只需要将类型包在括号里；    
2. 一个箭头符号：->  
3. 方法体，可以是表达式和代码块，方法体函数式接口里面方法的实现，如果是代码块，则必须用{}来包裹起来，且需要一个 return 返回值

总体看起来像这样  

	(parameters) -> expression 或者 (parameters) -> { statements; }

如：

	Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> {
		    int result = e1.compareTo( e2 );
		    return result;
		} );

	Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) );


了解更多：   
http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html

##三. 函数式接口

　　简单来说，函数式接口是只包含一个方法（默认的方法和静态方法除外）的接口。比如Java标准库中的java.lang.Runnable和java.util.Comparator都是典型的函数式接口。 
 
　　java 8提供 @FunctionalInterface作为注解,这个注解是非必须的，只要接口符合函数式接口的标准（即只包含一个方法的接口），虚拟机会自动判断，但最好在接口上使用注解@FunctionalInterface进行声明，以免团队的其他人员错误地往接口中添加新的方法。

	@FunctionalInterface
	public interface FunctionalDefaultMethods {
		void method();
		default void defaultMethod() {
		}
	}

　　Java中的lambda无法单独出现，它需要一个函数式接口来盛放，lambda表达式方法体其实就是函数接口的实现，看一个完整的例子，方便理解

	/**
	 * 测试lambda表达式
	 *
	 */
	public class TestLambda {
		public static void runThreadUseLambda() {
			// Runnable是一个函数接口，只包含了有个无参数的，返回void的run方法；
			// 所以lambda表达式左边没有参数，右边也没有return，只是单纯的打印一句话
			new Thread(() -> System.out.println("lambda实现的线程")).start();
		}
	
		public static void runThreadUseInnerClass() {
			// 这种方式就不多讲了，以前旧版本比较常见的做法
			new Thread(new Runnable() {
				@Override
				public void run() {
					System.out.println("内部类实现的线程");
				}
			}).start();
		}
	
		public static void main(String[] args) {
			TestLambda.runThreadUseLambda();
			TestLambda.runThreadUseInnerClass();
		}
	}

　　JDK 1.8 API包含了很多内建的函数式接口，除了在之前版本中的Java中常用到的比如Comparator或者Runnable接口，Java 8 API还提供了很多全新的函数式接口来让工作更加方便,

**Predicate接口**  
　　Predicate 接口只有一个参数，返回boolean类型。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）;

**Function接口**  
　　Function 接口有一个参数并且返回一个结果，并附带了一些可以和其他函数组合的默认方法（compose, andThen）;

**Supplier接口**  
　　Supplier 接口返回一个任意范型的值，和Function接口不同的是该接口没有任何参数;  

**Consumer接口**  
　　Consumer 接口表示执行在单个参数上的操作;

**Comparator接口**  
　　Comparator 是老Java中的经典接口， Java 8在此之上添加了多种默认方法;



##四. 方法引用

　　方法引用其实是lambda表达式的一个简化写法，所引用的方法其实是lambda表达式的方法体实现，语法也很简单，左边是容器（可以是类名，实例名），中间是”::”，右边是相应的方法名。如下所示：

	ObjectReference::methodName

一般方法的引用格式是：  
1. 如果是静态方法，则是 ClassName::methodName。如 `Object ::equals`  
2. 如果是实例方法，则是 Instance::methodName。 如 `Object obj=new Object(); obj::equals`   
3. 构造函数.则是 ClassName::new  


看一个完整的例子：

	import java.util.Comparator;
	import java.util.Objects;
	import java.util.function.Consumer;
	import java.util.function.Function;
	import java.util.function.Predicate;
	import java.util.function.Supplier;
	
	public class FunctionalInterfaceTest {
	
		public static void main(String[] args) {
			// Predicate接口
			Predicate<String> predicate = (s) -> s.length() > 0;
			predicate.test("foo"); // true
			predicate.negate().test("foo"); // false
			
			Predicate<Boolean> nonNull = Objects::nonNull;
			Predicate<Boolean> isNull = Objects::isNull;
			Predicate<String> isEmpty = String::isEmpty;
			Predicate<String> isNotEmpty = isEmpty.negate();
	
			// Function 接口
			Function<String, Integer> toInteger = Integer::valueOf;
			Function<String, String> backToString = toInteger.andThen(String::valueOf);
			backToString.apply("123"); // "123"
	
			// Supplier 接口
			Supplier<Person> personSupplier = Person::new;
			personSupplier.get(); // new Person
	
			// Consumer 接口
			Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
			greeter.accept(new Person("Luke", "Skywalker"));
	
			// Comparator 接口
			Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
			Person p1 = new Person("John", "Doe");
			Person p2 = new Person("Alice", "Wonderland");
			comparator.compare(p1, p2); // > 0
			comparator.reversed().compare(p1, p2); // < 0
	
		}
	}
	
	class Person {
		String firstName;
		String lastName;
		Person() {
		}
		Person(String firstName, String lastName) {
			this.firstName = firstName;
			this.lastName = lastName;
		}
	}



了解更多：  
http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html


##五. 接口的默认方法和静态方法

　**1. 默认方法：**

　　就是接口可以有实现方法，而且不需要实现类去实现其方法。只需在方法名前面加个default关键字即可。引进的默认方法的目的是为了解决接口的修改与现有的实现不兼容的问题

	public interface A {
		default void foo() {
			System.out.println("Calling A.foo()");
		}
	}
	
	public class Clazz implements A {
		public static void main(String[] args) {
			Clazz clazz = new Clazz();
			clazz.foo();// 调用A.foo()
		}
	}


**相同点：**  

1. 都是抽象类型；   
2. 都可以有实现方法（以前接
口不行）；  
3. 都可以不需要实现类或者继
承者去实现所有方法，（以前
不行，现在接口中默认方法不
需要实现者实现）

**不同点：**   

1. 抽象类不可以多重继承，接口可以（无论是多重类型
继承还是多重行为继承）；  
2. 抽象类和接口所反映出的设计理念不同。其实抽象类
表示的是”is-a”关系，接口表示的是”like-a”关系；   
3. 接口中定义的变量默认是public static final 型，且必
须给其初值，所以实现类中不能改变其值；抽象类中
的变量默认是 friendly 型，其值可以在子类中重新定
义，也可以重新赋值。



**多重继承的冲突说明：**  

由于同一个方法可以从不同接口引入，自然而然的会有冲突的现象，默认方法判断冲突的规
则如下：  

1. 一个声明在类里面的方法优先于任何默认方法（classes always win）  
2. 否则，则会优先选取最具体的实现。



 **2. 静态方法：**  

　　Java 8 的另外一个有意思的新特性是接口里可以声明静态方法，并且可以实现；

整合例子：

	import java.util.function.Supplier;
	
	public class DefaulAndStaticMethodsTest {
	
		private interface Defaulable {
			
			// Interfaces now allow default methods, the implementer may or may not implement (override) them.
			default String notRequired() {
				return "Default implementation";
			}
		}
	
		private static class DefaultableImpl implements Defaulable {
		}
	
		private static class OverridableImpl implements Defaulable {
			@Override
			public String notRequired() {
				return "Overridden implementation";
			}
		}
	
		private interface DefaulableFactory {
	
			// Interfaces now allow static methods
			static Defaulable create(Supplier<Defaulable> supplier) {
				return supplier.get();
			}
	
		}
	
		public static void main(String[] args) {
			Defaulable defaulable = DefaulableFactory.create(DefaultableImpl::new);
			System.out.println(defaulable.notRequired());
	
			defaulable = DefaulableFactory.create(OverridableImpl::new);
			System.out.println(defaulable.notRequired());
		}
	
	}



##六. Stream API

　　流（Stream）仅仅代表着数据流，并没有数据结构，所以他遍历完一次之后便再也无法遍历
（这点在编程时候需要注意，不像Collection，遍历多少次里面都还有数据），它的来源可以
是Collection、array、io等等。

**中间与终点方法**  
　　流作用是提供了一种操作大数据接口，让数据操作更容易和更快。它具有过滤、映射以及遍历数等方法，这些方法分两种：中间方法和终端方法，“流”抽象天生就该是持续的，中间
方法永远返回的是Stream，因此如果我们要获取最终结果的话，必须使用终点操作才能收集
流产生的最终结果。区分这两个方法是看他的返回值，如果是Stream则是中间方法，否则是终点方法。具体请参照Stream的api。

**1.创建**  
	Java 8扩展了集合类，可以通过 Collection.stream() 或者 Collection.parallelStream() 来创建一个Stream

**2.Filter过滤**  
	过滤通过一个predicate接口来过滤并只保留符合条件的元素，该操作属于中间操作，所以我们可以在过滤后的结果来应用其他Stream操作（比如forEach）。forEach需要一个函数来对过滤后的元素依次执行。forEach是一个最终操作，所以我们不能在forEach之后来执行其他Stream操作。

**3.Sort排序**  
	排序是一个中间操作，返回的是排序好后的Stream。如果你不指定一个自定义的Comparator则会使用默认排序。

**4.Map映射**  
	中间操作map会将元素根据指定的Function接口来依次将元素转成另外的对象，下面的示例展示了将字符串转换为大写字符串。你也可以通过map来讲对象转换成其他类型，map返回的Stream类型是根据你map传递进去的函数的返回值决定的。

**5.Match匹配**  
	Stream提供了多种匹配操作，允许检测指定的Predicate是否匹配整个Stream。所有的匹配操作都是最终操作，并返回一个boolean类型的值。

**6.Count计数**  
	计数是一个最终操作，返回Stream中元素的个数，返回值类型是long。

**7.Reduce规约**  
	这是一个最终操作，允许通过指定的函数来将stream中的多个元素规约为一个元素，规约后的结果是通过Optional接口表示的：

**8.例子整合**  
	
	import java.util.ArrayList;
	import java.util.List;
	import java.util.Optional;
	
	public class StreamTest {
		public static void main(String[] args) {
			
			//创建实例代码的用到的数据List
			List<String> stringCollection = new ArrayList<>();
			stringCollection.add("ddd2");
			stringCollection.add("aaa2");
			stringCollection.add("bbb1");
			stringCollection.add("aaa1");
			stringCollection.add("bbb3");
			stringCollection.add("ccc");
			stringCollection.add("bbb2");
			stringCollection.add("ddd1");
			
			//Filter过滤
			stringCollection
		    .stream()
		    .filter((s) -> s.startsWith("a"))
		    .forEach(System.out::println);
			System.out.println("-------------------------------");
			
			//Sort排序
			stringCollection
		    .stream()
		    .sorted()
		    .filter((s) -> s.startsWith("a"))
		    .forEach(System.out::println);
			System.out.println("-------------------------------");
			
			//Map映射
			stringCollection
		    .stream()
		    .map(String::toUpperCase)
		    .sorted((a, b) -> b.compareTo(a))
		    .forEach(System.out::println);
			System.out.println("-------------------------------");
			
			//Match匹配
			boolean anyStartsWithA = 
			    stringCollection
			        .stream()
			        .anyMatch((s) -> s.startsWith("a"));
			System.out.println(anyStartsWithA);      // true
			boolean allStartsWithA = 
			    stringCollection
			        .stream()
			        .allMatch((s) -> s.startsWith("a"));
			System.out.println(allStartsWithA);      // false
			boolean noneStartsWithZ = 
			    stringCollection
			        .stream()
			        .noneMatch((s) -> s.startsWith("z"));
			System.out.println(noneStartsWithZ);      // true
			System.out.println("-------------------------------");
			
			//Count计数
			long startsWithB = 
				    stringCollection
				        .stream()
				        .filter((s) -> s.startsWith("b"))
				        .count();
			System.out.println(startsWithB);    // 3
			System.out.println("-------------------------------");
			
			//Reduce规约
			Optional<String> reduced =
				    stringCollection
				        .stream()
				        .sorted()
				        .reduce((s1, s2) -> s1 + "#" + s2);
			reduced.ifPresent(System.out::println);
			System.out.println("-------------------------------");
			
		}
	}

**9.并行stream**  
	当使用顺序串行方式去遍历时，每个item读完后再读下一个item。而使用并行去遍历时，数组会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。
	
	import java.util.ArrayList;
	import java.util.List;
	import java.util.UUID;
	import java.util.concurrent.TimeUnit;
	
	public class ParallelStreamTest2 {
	
		public static void test1(List<String> values) {
	
			long t0 = System.nanoTime();
			long count = values
					.stream()
					.sorted()
					.count();
			System.out.println(count);
			long t1 = System.nanoTime();
			long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
			System.out.println(String.format("sequential sort took: %d ms", millis));
		}
	
		public static void test2(List<String> values) {
			
			long t0 = System.nanoTime();
			long count = values
					.parallelStream()
					.sorted()
					.count();
			System.out.println(count);
			long t1 = System.nanoTime();
			long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
			System.out.println(String.format("parallel sort took: %d ms", millis));
		}
	
		public static void main(String[] args) {
			int max = 1000000;
			List<String> values = new ArrayList<>(max);
			for (int i = 0; i < max; i++) {
				UUID uuid = UUID.randomUUID();
				values.add(uuid.toString());
			}
	
			test1(values);
			test2(values);
		}
	
	}



　　对hadoop有稍微了解就知道，里面的 MapReduce 本身就是用于并行处理大数据集的软件框架，其 处理大数据的核心思想就是大而化小，分配到不同机器去运行map，最终通过reduce将所有机器的结果结合起来得到一个最终结果，与MapReduce不同，Stream则是利用多核技术可将大数据通过多核并行处理，而MapReduce则可以分布式的。


##七. 集合中的方法

1. Map类型不支持stream，不过Map提供了一些新的有用的方法来处理一些日常任务。

	例子
		
		import java.util.HashMap;
		import java.util.Map;
		
		public class MapTest {
		
			public static void main(String[] args) {
				Map<Integer, String> map = new HashMap<>();
				for (int i = 0; i < 10; i++) {
					map.putIfAbsent(i, "val" + i);
				}
				map.forEach((id, val) -> System.out.println(val));
				//putIfAbsent 不需要我们做额外的存在性检查，
				//forEach则接收一个Consumer接口来对map里的每一个键值对进行操作。
				
				System.out.println("-------------------------------------------");
				map.computeIfPresent(3, (num, val) -> val + num);
				System.out.println(map.get(3));             // val33
				map.computeIfPresent(9, (num, val) -> null);
				System.out.println(map.containsKey(9));     // false
				map.computeIfAbsent(23, num -> "val" + num);
				System.out.println(map.containsKey(23));    // true 
				map.computeIfAbsent(3, num -> "bam");
				System.out.println(map.get(3));             // val33
				System.out.println("-------------------------------------------");
				//删除一个键值全都匹配的项
				map.remove(3, "val3");
				System.out.println(map.get(3));             // val33
				map.remove(3, "val33");
				System.out.println(map.get(3));             // null
				System.out.println(map.getOrDefault(42, "not found"));  // not found
				
				System.out.println("-------------------------------------------");
				System.out.println(map.get(9));  
				map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
				System.out.println(map.get(9));             // val9
				map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
				System.out.println(map.get(9));             // val9concat
				//如果键名不存在则插入，否则则对原键对应的值做合并操作并重新插入到map中
			}
		}
		 

2. 以前Java集合是不能够表达内部迭代的，而只提供了一种外部迭代的方式，也就是for或者while循环；

		List persons = asList(new Person("Joe"), new Person("Jim"), new Person("John"));
		for (Person p : persons) {
			p.setLastName("Doe");
		}

	下面利用lambda和Collection.forEach重写上面的循环

		persons.forEach(p->p.setLastName("Doe"));

	现在是由jdk 库来控制循环了，我们不需要关心last name是怎么被设置到每一个person对象里面去的，库可以根据运行环境来决定怎么做，并行，乱序或者懒加载方式。这就是内部迭代，客户端将行为p.setLastName当做数据传入api里面。


##八. 重复注解

　　Java 8开始允许相同注释在声明使用的时候重复使用超过一次。 重复注释本身需要被@Repeatable注释。实际上，他不是一个语言上的改变，只是编译器层面的改动，技术层面仍然是一样的。
例子
		
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Repeatable;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	
	public class RepeatingAnnotationsTest {
	
		@Target(ElementType.TYPE)
		@Retention(RetentionPolicy.RUNTIME)
		public @interface Filters {
			Filter[] value();
		}
	
		@Target(ElementType.TYPE)
		@Retention(RetentionPolicy.RUNTIME)
		@Repeatable(Filters.class)
		public @interface Filter {
			String value();
		}
	
		@Filter("filter1")
		@Filter("filter2")
		public interface Filterable {
		}
	
		public static void main(String[] args) {
			for (Filter filter : Filterable.class.getAnnotationsByType(Filter.class)) {
				System.out.println(filter.value());
			}
			System.out.println("--------------------");
			for (Filter filter : Filterable.class.getAnnotation(Filters.class).value()) {
				System.out.println(filter.value());
			}
	
		}
	}

　　我们可以看到，注释Filter被@Repeatable( Filters.class )注释。Filters 只是一个容器，它持有Filter；  
　　另外，反射的API提供一个新方法getAnnotationsByType() 来返回重复注释的类型（Filterable.class.getAnnotation( Filters.class )将会返回编译器注入的Filters实例）。

##九. 注解的扩展

　　Java 8 新增加了两个注解的程序元素类型 `ElementType.TYPE_USE` 和 `ElementType.TYPE_PARAMETER` ，现在我们几乎可以在所有的地方：局部变量、泛型、超类和接口实现、甚至是方法的Exception声明  
例子：
	
	public class AnnotationsTest {
	
		@Retention(RetentionPolicy.RUNTIME)
		@Target({ ElementType.TYPE_USE })
		public @interface TypeUse {
		}
	
		@Retention(RetentionPolicy.RUNTIME)
		@Target({ ElementType.TYPE_PARAMETER })
		public @interface TypeParameter {
		}
	
		public static class Holder<@TypeParameter T> extends @TypeUse Object {
			public void method() throws @TypeUse Exception {
			}
		}
	
		@SuppressWarnings("unused")
		public static void main(String[] args) {
			final Holder<String> holder = new @TypeUse Holder<String>();
			@TypeUse
			Collection<@TypeUse String> strings = new ArrayList<>();
		}
	}



##十. Optional容器

　　Optional是个用来防止NullPointerException异常的辅助类型容器，它可以保存一些类型的值或者null。在Java 8之前一般某个函数应该返回非空对象但是偶尔却可能返回了null，而在Java 8中，不推荐你返回null而是返回Optional。  

例子  

	import java.util.Optional;
	
	public class OptionalTest {
	
		public static void main(String[] args) {
			
			Optional<String> fullName = Optional.ofNullable(null);
			
			System.out.println("Full Name is set? " + fullName.isPresent());
			System.out.println("Full Name: " + fullName.orElseGet(() -> "[none]"));
			System.out.println(fullName.map(s -> "Hey " + s + "!").orElse("Hey Stranger !"));
	
			//如果Optional实例有非空的值，方法 isPresent() 返回true否则返回false。
			//方法orElseGet提供了回退机制，当Optional的值为空时接受一个方法返回默认值。
			//map()方法转化Optional当前的值并且返回一个新的Optional实例。
			//orElse方法和orElseGet类似，但是它不接受一个方法，而是接受一个默认值。上面代码运行结果如下：
			System.out.println("--------------------------------------------------------");
			
			Optional< String > firstName = Optional.of( "Tom" );
			System.out.println( "First Name is set? " + firstName.isPresent() );        
			System.out.println( "First Name: " + firstName.orElseGet( () -> "[none]" ) ); 
			System.out.println( firstName.map( s -> "Hey " + s + "!" ).orElse( "Hey Stranger!" ) );
			System.out.println();
		}
	}

更多了解：  
http://docs.oracle.com/javase/8/docs/api/java/util/Optional.html


##十一. Date API

　　Java 8 在包java.time下包含了一组全新的时间日期API，包括instants, durations, dates, times, time-zones and periods。这些都是基于ISO日历系统，它又是遵循 Gregorian规则的。最重要的一点是值不可变，且线程安全。新的日期API和开源的Joda-Time库差不多，但又不完全一样，其实JSR310的规范领导者Stephen Colebourne，同时也是Joda-Time的创建者，JSR310是 在Joda-Time的基础上建立的，参考了绝大部分的API，但并不是说JSR310=JODA-Time。下面的例子展示了这组新API里最重要的一些部分：


**Clock时钟**  

　　Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis() 来获取当前的微秒数。某一个特定的时间点也可以使用Instant类来表示，Instant类也可以用来创建老的java.util.Date对象。

**Timezones时区**
	
　　在新API中时区使用ZoneId来表示。时区可以很方便的使用静态方法of来获取到。 时区定义了到UTS时间的时间差，在Instant时间点对象到本地日期对象之间转换的时候是极其重要的。

**LocalTime本地时间**

　　LocalTime 定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位计算两个时间的时间差：

**LocalDate本地日期**

　　LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。

**LocalDateTime 本地日期时间**

例子整合

	import java.time.Clock;
	import java.time.DayOfWeek;
	import java.time.Instant;
	import java.time.LocalDate;
	import java.time.LocalDateTime;
	import java.time.LocalTime;
	import java.time.Month;
	import java.time.ZoneId;
	import java.time.format.DateTimeFormatter;
	import java.time.format.FormatStyle;
	import java.time.temporal.ChronoField;
	import java.time.temporal.ChronoUnit;
	import java.util.Date;
	import java.util.Locale;
	
	public class DateTest {
		public static void main(String[] args) {
	
			// Clock 时钟
			Clock clock = Clock.systemDefaultZone();
			long millis = clock.millis();
			Instant instant = clock.instant();
			Date legacyDate = Date.from(instant); // legacy java.util.Date
			System.out.println("-----------------------------------------");
	
			// Timezones 时区
			System.out.println(ZoneId.getAvailableZoneIds());
			// prints all available timezone ids
			ZoneId zone1 = ZoneId.of("Europe/Berlin");
			ZoneId zone2 = ZoneId.of("Brazil/East");
			System.out.println(zone1.getRules());
			System.out.println(zone2.getRules());
			// ZoneRules[currentStandardOffset=+01:00]
			// ZoneRules[currentStandardOffset=-03:00]
	
			// LocalTime 本地时间
			LocalTime now1 = LocalTime.now(zone1);
			LocalTime now2 = LocalTime.now(zone2);
			System.out.println(now1.isBefore(now2)); // false
			long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
			long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);
			System.out.println(hoursBetween); // -4
			System.out.println(minutesBetween); // -299
	
			// LocalTime 提供了多种工厂方法来简化对象的创建，包括解析时间字符串。
			LocalTime late = LocalTime.of(23, 59, 59);
			System.out.println(late); // 23:59:59
			DateTimeFormatter germanFormatter = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT).withLocale(Locale.GERMAN);
			LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
			System.out.println(leetTime); // 13:37
	
			// LocalDate 本地日期
			LocalDate today = LocalDate.now();
			LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
			LocalDate yesterday = tomorrow.minusDays(2);
			LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
			DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
	
			System.out.println(dayOfWeek); // FRIDAY
			// 从字符串解析一个LocalDate类型和解析LocalTime一样简单：
	
			DateTimeFormatter germanFormatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).withLocale(Locale.GERMAN);
			LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter2);
			System.out.println(xmas); // 2014-12-24
	
			// LocalDateTime 本地日期时间
			LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);
			DayOfWeek dayOfWeek2 = sylvester.getDayOfWeek();
			System.out.println(dayOfWeek2); // WEDNESDAY
			Month month = sylvester.getMonth();
			System.out.println(month); // DECEMBER
			long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
			System.out.println(minuteOfDay); // 1439
	
			// 只要附加上时区信息，就可以将其转换为一个时间点Instant对象，Instant时间点对象可以很容易的转换为老式的java.util.Date。
			Instant instant2 = sylvester.atZone(ZoneId.systemDefault()).toInstant();
			Date legacyDate2 = Date.from(instant2);
			System.out.println(legacyDate2); // Wed Dec 31 23:59:59 CET 2014
	
			// 格式化LocalDateTime和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式：
			DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMM dd, yyyy - HH:mm");
			LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
			String string = formatter.format(parsed);
			System.out.println(string); // Nov 03, 2014 - 07:13
	
			// 和java.text.NumberFormat不一样的是新版的DateTimeFormatter是不可变的，所以它是线程安全的。
		}
	}

更多了解：  
http://download.java.net/jdk8/docs/api/java/time/format/DateTimeFormatter.html


##十二. Base64 API

　　BASE64 编码是一种常用的字符编码，在很多地方都会用到。但base64不是安全领域下的加密解密算法。能起到安全作用的效果很差，而且很容易破解，他核心作用应该是传输数据的正确性，有些网关或系统只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法，而且base64特别适合在http，mime协议下快速传输数据。（传输文件等场景）

java.util.Base64工具类提供了一套静态方法获取下面三种BASE64编解码器：  

1. Basic编码  
2. URL编码  
3. MIME编码  

Basic编码是标准的BASE64编码，用于处理常规的需求：输出的内容不添加换行符，而且输出的内容由字母加数字组成;

URL编码也是我们经常会面对的需求，但由于URL对反斜线“/”有特殊的意义，因此URL编码需要替换掉它，使用下划线替换;

MIME编码器会使用基本的字母数字产生BASE64输出，而且对MIME格式友好：每一行输出不超过76个字符，而且每行以“\r\n”符结束;
	
	import java.io.UnsupportedEncodingException;
	import java.util.Base64;
	import java.util.UUID;
	
	public class Test {
		public static void main(String[] args) throws UnsupportedEncodingException {

			//基本的编码器
			// 编码
			String asB64 = Base64.getEncoder().encodeToString("some string".getBytes("utf-8"));
			System.out.println(asB64); // 输出为: c29tZSBzdHJpbmc=
	
			// 解码
			byte[] asBytes = Base64.getDecoder().decode("c29tZSBzdHJpbmc=");
			System.out.println(new String(asBytes, "utf-8")); // 输出为: some string
	
			System.out.println("-----------------------------------------------");
			
			//URL编码器
			String basicEncoded = Base64.getEncoder().encodeToString("subjects?abcd".getBytes("utf-8"));
			System.out.println("Using Basic Alphabet: " + basicEncoded);
	
			String urlEncoded = Base64.getUrlEncoder().encodeToString("subjects?abcd".getBytes("utf-8"));
			System.out.println("Using URL Alphabet: " + urlEncoded);
			// 输出为:
			// Using Basic Alphabet: c3ViamVjdHM/YWJjZA==
			// Using URL Alphabet: c3ViamVjdHM_YWJjZA==
			System.out.println("-----------------------------------------------");
	
			//MIME编码器
			StringBuilder sb = new StringBuilder();
			for (int t = 0; t < 10; ++t) {
				sb.append(UUID.randomUUID().toString());
			}
	
			byte[] toEncode = sb.toString().getBytes("utf-8");
			String mimeEncoded = Base64.getMimeEncoder().encodeToString(toEncode);
			System.out.println(mimeEncoded);
	
			System.out.println("-----------------------------------------------");
		}
	}
	




##十三. 参考资料

https://www.gitbook.com/book/wizardforcel/java8-new-features/details

http://www.jb51.net/article/48304.htm

http://www.importnew.com/19345.html













