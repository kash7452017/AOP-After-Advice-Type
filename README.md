## AOP：@After Advice Type
>參考網址：https://blog.csdn.net/qq_22076345/article/details/115742692
>
>是一種通知類，可確保在方法執行後運行通知
>
>從springframework5.2.7開始，在同一@Aspect 類中定義的需要在同一連接點上運行的通知方法根據其通知類型按以下順序分配優先級：從最高到最低優先級：@Around、@Before、@After、@AfterReturning、@AfterThrowing。但是，請注意，@After 通知方法將在同一切面的任何@AfterReturning 或@AfterThrowing 通知方法之後有效地調用，遵循AspectJ 對@After 的“After finally advice”語義

**添加@After建議方法，同時保留@AfterThrowing與@AfterReturning建議方法，我們將用來驗證成功與失敗情形**
```
@After("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
	public void afterFinallyFindAccountsAdvice(JoinPoint theJoinPoint) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @After (finally) on method: " + method);
}
```
**MyDemoLoggingAspect.java完整程式碼**
```
@Aspect
@Component
@Order(2)
public class MyDemoLoggingAspect {
	
	@After("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
	public void afterFinallyFindAccountsAdvice(JoinPoint theJoinPoint) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @After (finally) on method: " + method);
	}

	@AfterThrowing(
	pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
	throwing="theExc")
	public void afterThrowingFindAccountingAccountAdvice(
					JoinPoint theJoinPoint, Throwable theExc) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @AfterThrowing on method: " + method);
		
		// log the exception
		System.out.println("\n=====>>> The exception is: " + theExc);
	}
	
	@AfterReturning(
			pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
			returning="result")
	public void afterReturningAccountsAdvice(
					JoinPoint theJoinPoint, List<Account> result) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @AfterReturning on method: " + method);
		
		// print out the results of the method call
		System.out.println("\n=====>>> result is: " + result);
		
		// let's post-process the data ... let's modify it
		// convert the account names to uppercase
		convertAccountNamesToUpperCase(result);
		
		System.out.println("\n=====>>> result is: " + result);
	}
	
	private void convertAccountNamesToUpperCase(List<Account> result) {
		
		// loop through accounts
		for (Account tempAccount : result) {
		
			// get uppercase version of name
			String theUpperName = tempAccount.getName().toUpperCase();
		
			// update the name on the account
			tempAccount.setName(theUpperName);
		}
	}

	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void beforeAddAccountAdvice(JoinPoint theJoinPoint) {
		
		System.out.println("\n=====>>> Excuting @Before advice on method()");
		
		// display the method signature
		MethodSignature methodSig = (MethodSignature) theJoinPoint.getSignature();
		
		System.out.println("Method: " + methodSig);
		
		// display method arguments
		// get args
		Object[] args = theJoinPoint.getArgs();
		
		// loop thru args
		for (Object tempArg : args) {
			System.out.println(tempArg);
			
			if (tempArg instanceof Account) {
				
				// downcast and print Account specific stuff
				Account theAccount = (Account) tempArg;
				
				System.out.println("account name: " + theAccount.getName());
				System.out.println("account level: " + theAccount.getLevel());
			}
		}
	}
}
```

**創建AfterFinallyDemoApp演示主應用程序，`boolean tripWire = true;`當此值為True時，程序會強制拋出例外錯誤，用以驗證方法失敗時之情形；反之當此值為Fasle時，方法將正常執行，此時驗證方法成功執行時之情形**
```
public class AfterFinallyDemoApp {

	public static void main(String[] args) {
		
		// read spring config java class
		AnnotationConfigApplicationContext context = 
				new AnnotationConfigApplicationContext(DemoConfig.class);
		
		// get the bean from spring container
		AccountDAO theAccountDAO = context.getBean("accountDAO", AccountDAO.class);
		
		// call method to find the accounts
		List<Account> theAccounts = null;
		
		try {
			// add a boolean flag to simulate exceptions
			boolean tripWire = true;
			theAccounts = theAccountDAO.findAccounts(tripWire);
		} 
		catch (Exception exc) {
			System.out.println("\n\nMain Program ... caught exception: " + exc);
		}

		// diaplay the accounts
		System.out.println("\n\nMain Program: AfterThrowingDemoApp");
		System.out.println("----");
		
		System.out.println(theAccounts);
		
		System.out.println("\n");		
		
		// close the context
		context.close();
	}
}
```
## **當`boolean tripWire = true;`**

![image](https://user-images.githubusercontent.com/101872264/217284048-44e8b895-53dd-4829-ac0b-a2d52a7f65a8.png)

## **當`boolean tripWire = fasle;`**
![image](https://user-images.githubusercontent.com/101872264/217284391-7a4e4eef-b4f3-41dd-ab8b-c5124cae7ce5.png)

## 結論：
>* @After建議，如同try-catch-finally，無論方法成功與否，皆會一直運作下去
>* 情況一：出現例外，@After會在@AfterThrowing之前執行，@After同樣繼續執行
>* 情況二：沒有例外，@After會在@AfterReturning之前執行，@After同樣繼續執行
>
>再次強調!!!!!**`從springframework5.2.7開始，在同一@Aspect類中定義的需要在同一連接點上運行的通知方法根據其通知類型按以下順序分配優先級：從最高到最低優先級：@Around、@Before、@After、@AfterReturning、@AfterThrowing。但是，請注意，@After通知方法將在同一切面的任何@AfterReturning或@AfterThrowing通知方法之後有效地調用，遵循AspectJ 對@After 的“After finally advice”語義`**
