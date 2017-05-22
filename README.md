# ueditor
Baidu ueditor对应的java源码做了修改，兼容Spring MVC  

Baidu ueditor源码地址为：https://github.com/fex-team/ueditor. 通过controller.jsp加载config.json与后端交互。
这样的实现无法兼容Spring MVC。因此现在将此源代码修改一下适配Spring MVC。  

新建了两个类com.baidu.ueditor.MyActionEnter和com.baidu.ueditor.MyConfigManager，来分别替换相同包名下的ActionEnter和ConfigManager。

主要的改进点在于将ActionEnter的构造方法 ( HttpServletRequest request, String rootPath )中第二个参数rootPath修改为InputStream，这样在Controller直接传入一个文件输入流即可，不再与config.json文件的路径耦合在一起，相对好一些。

使用Eclipse导入该工程时，需要设置Java Build Path中的Libraries，Java Compiler中的Compiler compliance level，Project Facets中的Java对应的Version等，最好使用JDK 1.7及以上的版本号。

# Gradle

`
compile 'com.github.qiuxiaoj:ueditor:1.1.1'
`

# usage

```java


public class UeditorUploadServlet extends HttpServletBean {
	private static final Logger log = LoggerFactory.getLogger(UeditorUploadServlet.class);
	private static final long serialVersionUID = -115516230072858293L;
	private static final String UE_CONFIG = "classpath:config/ueditor.config.json";
	private ResourceLoader resourceLoader;
	private String configFile = UE_CONFIG;
	private String rootPath;
	
	public UeditorUploadServlet() {

	}

	public UeditorUploadServlet(String rootPath, ResourceLoader resourceLoader) {
		this.rootPath = rootPath;
		this.resourceLoader = resourceLoader;
	}

	public UeditorUploadServlet(String rootPath, String configFile, ResourceLoader resourceLoader) {
		this.rootPath = rootPath;
		this.configFile = configFile;
		this.resourceLoader = resourceLoader;
	}

	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		this.doPost(req, resp);
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		PrintWriter writer = resp.getWriter();
		log.info("ConfigFile: {}, rootPath: {}", this.configFile, this.rootPath);
		writer.write(new MyActionEnter(req, this.resourceLoader.getResource(this.configFile).getInputStream(), this.rootPath).exec());
	}

	public void setRootPath(String rootPath) {
		this.rootPath = rootPath;
	}

	public void setConfigFile(String configFile) {
		this.configFile = configFile;
	}

	public void setResourceLoader(ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}

}

```