# Tigase运行流程分析


![2017031685814Screen Shot 2017-03-16 at 10.52.05 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017031685814Screen Shot 2017-03-16 at 10.52.05 PM.png)

IM项目已经搭建起来，第一版本需要将原生的XMPP接口开发成为HTTP接口，比如，注册，修改个人信息等等。每次看Tigase源码，总是很累啊，人家从不用主流框架，一言不合都是自己写框架。上一篇文章记录Tigase整个开发环境的搭建，这篇文章我们来分析分析Tigase的运行流程，深入的研究一下Tigase的运行机制。为后期接口更新打下基础。这个星期实在累的不行。想休息休息了。周末好好吃一顿，或者好好休息一天。

<!--more-->

## 加载配置文件

我们在上一篇文章中讲过，运行的时候选择的主类是`XMPPServer`。打开`XMPPServer`的`main`函数入口，直接上源码进行分析:

```java
/**
     *
     * @param args 表示program argument
     */
	@SuppressWarnings("PMD")
	public static void main( final String[] args ) {
		// 开始运行
		System.out.println("开始执行");
		parseParams( args );
		System.out.println("开始加载组建");
		System.out.println( ( new ComponentInfo( XMLUtils.class ) ).toString() );
		System.out.println( ( new ComponentInfo( ClassUtil.class ) ).toString() );
		System.out.println( ( new ComponentInfo( XMPPServer.class ) ).toString() );
		start( args );
	}
```

首先看到的是`parseParams( args );`解析参数函数，判断`args`是否合法，进入函数:

```java
/**
	 * Method description
	 * 传入init.property文件,全部使用这些文件，判断全部不会执行
	 * 	
	 * @param args
	 */
	@SuppressWarnings("PMD")
	public static void parseParams( final String[] args ) {
      // 判断args
		if ( ( args != null ) && ( args.length > 0 ) ){
			for ( int i = 0 ; i < args.length ; i++ ) {
				if ( args[i].equals( "-h" ) ){
					System.out.print( help() );
					System.exit( 0 );
				}

				if ( args[i].equals( "-v" ) ){
					System.out.print( version() );
					System.exit( 0 );
				}

				if ( args[i].equals( "-n" ) ){
					if ( i + 1 == args.length ){
						System.out.print( help() );
						System.exit( 1 );
					}
					else {
						serverName = args[++i];
					}
				}

			}
		}
	}
```

咱们的`args`其实是这个

![201703179088Screen Shot 2017-03-17 at 10.03.27 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/201703179088Screen Shot 2017-03-17 at 10.03.27 PM.png)

所以，没有一个是合法的。所以直接往下走。然后就是一大堆的输出:

```java
		System.out.println("开始加载组建");
		System.out.println( ( new ComponentInfo( XMLUtils.class ) ).toString() );
		System.out.println( ( new ComponentInfo( ClassUtil.class ) ).toString() );
		System.out.println( ( new ComponentInfo( XMPPServer.class ) ).toString() );
```

并没有什么实际意义。再往下就是`start( args );`进入函数看看.

```java
public static void start( String[] args ) {
		Thread.setDefaultUncaughtExceptionHandler( new ThreadExceptionHandler() );
		if ( !isOSGi() ){
			String initial_config
						 = "tigase.level=ALL\n" + "tigase.xml.level=INFO\n"
							 + "handlers=java.util.logging.ConsoleHandler\n"
							 + "java.util.logging.ConsoleHandler.level=ALL\n"
							 + "java.util.logging.ConsoleHandler.formatter=tigase.util.LogFormatter\n";

			ConfiguratorAbstract.loadLogManagerConfig( initial_config );
		}

		try {
			// 解析init.properties
			String config_class_name = System.getProperty( CONFIGURATOR_PROP_KEY,DEF_CONFIGURATOR );
			// 根据拿到的ConfiguratorAbstract配置进行解析
			config = (ConfiguratorAbstract) Class.forName( config_class_name ).newInstance();
            config.init( args );
			config.setName( "basic-conf" );
			String mr_class_name = config.getMessageRouterClassName();
            System.out.println("mr_class_name的值为: "+mr_class_name);
            // tigase.server.MessageRouter,配置消息路由
            router = (MessageRouterIfc) Class.forName( mr_class_name ).newInstance();
			router.setName( serverName );
			router.setConfig( config );
			router.start();
		} catch ( ConfigurationException e ) {
			System.err.println( "" );
			System.err.println( "  --------------------------------------" );
			System.err.println( "  ERROR! Terminating the server process." );
			System.err.println( "  Invalid configuration data: " + e );
			System.err.println( "  Please fix the problem and start the server again." );
			System.exit( 1 );
		} catch ( Exception e ) {
			System.err.println( "" );
			System.err.println( "  --------------------------------------" );
			System.err.println( "  ERROR! Terminating the server process." );
			System.err.println( "  Problem initializing the server: " + e );
			System.err.println( "  Please fix the problem and start the server again." );
			e.printStackTrace();
			System.exit( 1 );
		}
	}
```

进入函数后，先判断`isOSGi`模式，默认就是`false`,并没有进行变动。然后通过系统初始化一个`ConfiguratorAbstract`,这是一个全局配置类，通过反射生成。通过传入`args`进行配置。

```java
	/**
     * 初始化配置核心类
	 * @throws ConfigurationException
	 * @throws TigaseDBException
	 */
	public void init(String[] args) throws ConfigurationException, TigaseDBException {
		parseArgs(args);

		String stringprep = (String) initProperties.get(STRINGPREP_PROCESSOR);
		if (stringprep != null) {
			BareJID.useStringprepProcessor(stringprep);
		}
		String cnf_class_name = System.getProperty(CONFIG_REPO_CLASS_PROP_KEY);
        if (cnf_class_name != null) {
			initProperties.put(CONFIG_REPO_CLASS_INIT_KEY, cnf_class_name);
		}
		cnf_class_name = (String) initProperties.get(CONFIG_REPO_CLASS_INIT_KEY);
		if (cnf_class_name != null) {
            System.out.println("cnf_class_name: "+cnf_class_name);
			try {
				configRepo = (ConfigRepositoryIfc) Class.forName(cnf_class_name).newInstance();
			} catch (Exception e) {
				log.log(Level.SEVERE, "Problem initializing configuration system: ", e);
				log.log(Level.SEVERE, "Please check settings, and rerun the server.");
				log.log(Level.SEVERE, "Server is stopping now.");
				System.err.println("Problem initializing configuration system: " + e);
				System.err.println("Please check settings, and rerun the server.");
				System.err.println("Server is stopping now.");
				System.exit(1);
			}
		}
		configRepo.addRepoChangeListener(this);
		configRepo.setDefHostname(getDefHostName().getDomain());
		try {
            // idea就是在这儿出问题
			configRepo.initRepository(null, (Map) initProperties);
		} catch (DBInitException ex) {
			throw new ConfigurationException(ex.getMessage(), ex);
		}

		for (String prop : initSettings) {
			ConfigItem item = configRepo.getItemInstance();
			item.initFromPropertyString(prop);
            configRepo.addItem(item);
		}

		Map<String, Object> repoInitProps = configRepo.getInitProperties();

		if (repoInitProps != null) {
			initProperties.putAll(repoInitProps);
		}
		String property_filenames = (String) initProperties.get(PROPERTY_FILENAME_PROP_KEY);
		if (property_filenames != null) {
			String[] prop_files = property_filenames.split(",");
            // 配置监视器
			initMonitoring((String) initProperties.get(MONITORING), new File(prop_files[0])
					.getParent());
		}
	}
```

第一步进行配置文件参数读取工作

```java
/**
     * 解析 args,默认配置是/etc/init.properties
     * 将文件中的属性加入 initProperties map中
     * @param args
     */
	public void parseArgs(String[] args) {
        // 配置'--test'
		initProperties.put(GEN_TEST, Boolean.FALSE);
        // 配置'config-type'
		initProperties.put("config-type", GEN_CONFIG_DEF);
        // initProperties加入 --property-file=etc/init.properties
		if ((args != null) && (args.length > 0)) {
			for (int i = 0; i < args.length; i++) {
				String key = null;
				Object val = null;

				if (args[i].startsWith(GEN_CONFIG)) {
					key = "config-type";
					val = args[i];
				}
				if (args[i].startsWith(GEN_TEST)) {
					key = args[i];
					val = Boolean.TRUE;
				}
				if ((key == null) && args[i].startsWith("-") &&!args[i].startsWith(GEN_CONFIG)) {
					key = args[i];
					val = args[++i];
				}
				if ((key != null) && (val != null)) {
                    // Setting defaults: --property-file=etc/init.properties
					initProperties.put(key, val);
					log.log(Level.CONFIG, "Setting defaults: {0} = {1}", new Object[] { key,
							val.toString() });
				}
			}
		}
        // init.properties
		String property_filenames = (String) initProperties.get(PROPERTY_FILENAME_PROP_KEY);
		if (property_filenames == null) {
			property_filenames = PROPERTY_FILENAME_PROP_DEF;
				log.log(Level.WARNING, "No property file not specified! Using default one {0}",
						property_filenames);
		}

		// always true
		if (property_filenames != null) {
			String[] prop_files = property_filenames.split(",");
			if ( prop_files.length == 1 ){
				File f = new File( prop_files[0] );
				if ( !f.exists() ){
					log.log( Level.WARNING, "Provided property file {0} does NOT EXISTS! Using default one {1}",
									 new String[] { f.getAbsolutePath(), PROPERTY_FILENAME_PROP_DEF } );
					prop_files[0] = PROPERTY_FILENAME_PROP_DEF;
				}
			}
			for (String property_filename : prop_files) {
				log.log(Level.CONFIG, "Loading initial properties from property file: {0}",
						property_filename);
				try {
					Properties defProps = new Properties();

					defProps.load(new FileReader(property_filename));

					Set<String> prop_keys = defProps.stringPropertyNames();

                    // 获取到配置文件并且进行遍历
					for (String key : prop_keys) {
                        String value = defProps.getProperty(key).trim();
                        if (key.startsWith("-") || key.equals("config-type")) {
							if (GEN_TEST.equalsIgnoreCase(key)) {
								initProperties.put(key.trim().substring(2), DataTypes.parseBool(value));
								initProperties.put(key.trim(), DataTypes.parseBool(value));
							} else {
								initProperties.put(key.trim(), value);
							}
							log.log(Level.CONFIG, "Added default config parameter: ({0}={1})",
									new Object[] { key,
									value });
						} else {
							initSettings.add(key + "=" + value);
						}
					}
				} catch (FileNotFoundException e) {
					log.log(Level.WARNING, "Given property file was not found: {0}",
							property_filename);
				} catch (IOException e) {
					log.log(Level.WARNING, "Can not read property file: " + property_filename, e);
				}
			}
		}
		for (Map.Entry<String, Object> entry : initProperties.entrySet()) {
			if (entry.getKey().startsWith("--")) {
				System.setProperty(entry.getKey().substring(2), ((entry.getValue() == null)
						? null
						: entry.getValue().toString()));

				// In cluster mode we switch DB cache off as this does not play well.
				if (CLUSTER_MODE.equals(entry.getKey())) {
					if ("true".equalsIgnoreCase(entry.getValue().toString())) {
						System.setProperty("tigase.cache", "false");
					}
				}
			}
		}
	}
```

先将`GEN_TEST`加入到设置为false放入map中

- ```java
  GEN_TEST: false
  ```

- ```java
  config-type=--gen-config-def
  ```

然后对通过查询args中的值对key和val进行初始化。得到相应的值为
```java
--property-file=etc/init.properties
```
往下走，拿到`property_filenames`,其实就是`etc/init.properties`。如果你没有设置默认的配置文件，其实加载的还是他。这一句就是:

```java
if (property_filenames == null) {
			property_filenames = PROPERTY_FILENAME_PROP_DEF;
				log.log(Level.WARNING, "No property file not specified! Using default one {0}",
						property_filenames);
		}
```

然后我们看，直接对配置文件进行检查，判断是否设置有多个配置文件。这里我们没有多个配置文件。拿到文件之后，对文件进行遍历，将`init.properties`文件中的属性放入`initProperties`,`initSettings`文件中。配置完成后，启动各种应该启动的组建和插件。

## 启动服务

配置完成，启动插件和组件之后，就开始初始化消息路由:

```java
			String mr_class_name = config.getMessageRouterClassName();
            System.out.println("mr_class_name的值为: "+mr_class_name);
            // tigase.server.MessageRouter,配置消息路由
            router = (MessageRouterIfc) Class.forName( mr_class_name ).newInstance();
			router.setName( serverName );
			router.setConfig( config );
			router.start();
```

`mr_class_name`是`tigase.server.MessageRouter`，`MessageRouterIfc` 是`MessageRouter`的代理。我们可以看看

```java
			router.setConfig( config );
			router.start();
```

这两个方法。

`router.setConfig( config );`这个方法是通过全局配置来加载组件。

```java
@Override
	public void setConfig(ConfiguratorAbstract config) throws ConfigurationException {
		components.put(getName(), this);
		this.config = config;
		addRegistrator(config);
	}
```

`router.start();`方法是启动线程。这个是最重要的。来看看:

```java
/**
	 * 线程启动
	 */
	@Override
	public void start() {
		if (log.isLoggable(Level.FINER)) {
			log.log(Level.INFO, "{0}: starting queue management threads ...", getName());
		}
		startThreads();
	}
```

来贴一波启动线程的代码，下一篇博文就写启动县城的代码。

```java
/**
     * 重点
     */
	private void startThreads() {
		if (threadsQueue == null) {
			threadsQueue = new ArrayDeque<QueueListener>(8);
			for (int i = 0; i < in_queues_size; i++) {
				QueueListener in_thread = new QueueListener(in_queues.get(i), QueueType.IN_QUEUE);

				in_thread.setName("in_" + i + "-" + getName());
				in_thread.start();
				threadsQueue.add(in_thread);
			}
			for (int i = 0; i < out_queues_size; i++) {
				QueueListener out_thread = new QueueListener(out_queues.get(i), QueueType
						.OUT_QUEUE);

				out_thread.setName("out_" + i + "-" + getName());
				out_thread.start();
				threadsQueue.add(out_thread);
			}
		}
		receiverScheduler = Executors.newScheduledThreadPool(schedulerThreads_size, threadFactory);
		receiverTasks     = new Timer(getName() + " tasks", true);
		receiverTasks.scheduleAtFixedRate(new TimerTask() {
			@Override
			public void run() {
				everySecond();
			}
		}, SECOND, SECOND);
		receiverTasks.scheduleAtFixedRate(new TimerTask() {
			@Override
			public void run() {
				everyMinute();
			}
		}, MINUTE, MINUTE);
		receiverTasks.scheduleAtFixedRate(new TimerTask() {
			@Override
			public void run() {
				everyHour();
			}
		}, HOUR, HOUR);
	}
```

好了，不想写了。这种博客全是代码。下次还是画图得了。