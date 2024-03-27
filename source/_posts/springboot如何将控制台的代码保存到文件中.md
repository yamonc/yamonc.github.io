---
title: springboot如何将控制台的输出保存到文件中
date: 2023-06-28 17:27:58
tags: [Java, SpringBoot]
categories: [开发问题]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306281727933.jpeg
comment: 是
keywords: 
---
# springboot如何将控制台的输出保存到文件中

![a painting of a blue and yellow flower](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306281727933.jpeg)

```java
/**
 * 控制台日志写入文件
 * @author Mr peng
 *
 */
@Component
public class ConsoleLogWrite extends OutputStream{
	
	//window输出文件路径
	@Value("${consoleLogWrite.windowsUrl}")
	private String consoleLogWriteWindowsUrl;
	//linux输出文件路径
	@Value("${consoleLogWrite.linuxUrl}")
	private String consoleLogWriteLinuxUrl;
	
	private OutputStream oldOutputStream, newOutputStream;
	
	public ConsoleLogWrite() {
		
	}
	
	public ConsoleLogWrite(OutputStream oldOutputStream, OutputStream newOutputStream) {
        this.oldOutputStream = oldOutputStream;
        this.newOutputStream = newOutputStream;
    }
	
	//重写输出流的方式，改为两种，一种控制台输出，一种写入指定文件
	@Override
	public void write(int b) throws IOException {
		oldOutputStream.write(b);
		newOutputStream.write(b);
	}

	//当前bean初始化前调用
	@PostConstruct
	public void writeLogToFile() throws Exception {
		File tmplLogFile = new File(getUploadPath(consoleLogWriteLinuxUrl, consoleLogWriteWindowsUrl));
		//启一个定时线程延迟15分钟后每过30分钟检查文件大小是否超过100M，如果超过则删除重新创建
		ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
		executorService.scheduleWithFixedDelay(new Runnable() {
			@Override
			public void run() {
				try {
					//文件不存在就创建
			        if (!tmplLogFile.exists()) {
			            try {
							tmplLogFile.createNewFile();
						} catch (IOException e) {
							e.printStackTrace();
						}
			        }
					//文件大于100M就删除，重新创建
			        double KB = 1024 * 1024;
			        double MB = KB * 1024;
			        if(tmplLogFile.length() > MB * 100){
			            tmplLogFile.delete();
			        }
				}catch (Exception e) {
					e.printStackTrace();
				}
			}
		}, 15, 30, TimeUnit.MINUTES);
		//设置输出模式
        PrintStream oldOutputStream = System.out;
        OutputStream newOutputStream = new FileOutputStream(tmplLogFile);
        ConsoleLogWrite multiOutputStream = new ConsoleLogWrite(oldOutputStream, new PrintStream(newOutputStream));
        System.setOut(new PrintStream(multiOutputStream));
        System.setErr(new PrintStream(multiOutputStream));
	}
	
	/**
     * 根据当前系统返回对应的路径 
     * @param linuxPath
     * @param windowsPath
     * @return
     */
    public static String getUploadPath(String linuxPath, String windowsPath) {
    	if(System.getProperty("os.name").toLowerCase().indexOf("linux") > 0) {
    		return linuxPath;
    	}
    	return windowsPath;
    }
	
}

```

