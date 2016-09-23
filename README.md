# tuziGG
My own code And The crawler projects demonstrate
#通过手机号码获取用户京东用户名

package com.lyitong.util;
import java.awt.image.BufferedImage;
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

import org.apache.commons.lang3.StringUtils;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.openqa.selenium.By;
import org.openqa.selenium.Dimension;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.Point;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.phantomjs.PhantomJSDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriverService;
import org.openqa.selenium.remote.Augmenter;
import org.openqa.selenium.remote.DesiredCapabilities;
import cn.edu.hfut.dmic.webcollector.util.FileUtils;


/** 
* @ClassName: UserNameCrawler 
* @Description: 通过手机号码获取用户京东用户名 
* @author John_hawkings
* @date 2016年9月22日 下午1:35:26 
*  
*/
public class UserNameCrawler {
	
	//添加日志支持
  	public static Log log = LogFactory.getLog(UserNameCrawler.class);
	
	/** 
	* @Title: getUserNameFromMJDByTel 
	* @Description: 通过手机号码查找京东用户名 
	* @param @param phoneNum
	* @param @return
	* @param @throws Exception 
	* @return String
	* @throws 
	* @date 2016年9月23日上午9:06:15
	*/
	@SuppressWarnings("null")
	public String getUserNameFromMJDByTel(String phoneNum) throws Exception{
		         
		
		        //设置必要参数
		        DesiredCapabilities dcaps = new DesiredCapabilities();
		        dcaps.setCapability("takesScreenshot", true);
		        dcaps.setCapability(PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY,"C:\\Python34\\Scripts\\phantomjs.exe");
 		        //创建无界面浏览器对象
		        PhantomJSDriver driver = new PhantomJSDriver(dcaps);

				// 让浏览器访问 手机京东-
				driver.get("http://m.jd.com");
				//获取sid
				String sid = driver.manage().getCookieNamed("sid").getValue();
				if(StringUtils.isBlank(sid)){
					log.error("sid读取失败,sid="+sid);
				}
				//System.out.println(sid);
				//拼接sid进入相应界面
				driver.get("https://plogin.m.jd.com/user/login.action?appid=100&returnurl=http%3A%2F%2Fm.jd.com%3Findexloc%3D1%26sid%"+sid);
				//System.out.println(lsid);
				//进入下级页面
				driver.get("https://plogin.m.jd.com/cgi-bin/m/mfindpwd");
				//获取文本框元素节点
				WebElement usernmaeEle = driver.findElement(By.id("username"));
				WebElement authCodeEle = driver.findElement(By.id("imgVerify"));
				if(usernmaeEle==null||authCodeEle==null){
					log.error("文本框节点获取失败,usernmaeEle="+usernmaeEle.toString()+" ,authCodeEle="+authCodeEle.toString());
				}
				
				//获取验证码的图片地址并获取webelement的位置和大小
				WebElement element = driver.findElement(By.id("imgCode"));
				if(element==null){
					log.error("验证码图片元素获取失败,element="+element.toString());
				}

                String filePath = "D:\\image\\codingpys.png"; 
                String authCodeImgUrl = "D:\\image\\authCode.png";
                snapshot2(driver, filePath);
                BufferedImage inputImg = createElementImage(driver, element);
                ImageIO.write(inputImg, "png", new File(authCodeImgUrl));
                //解析验证码并填充文本框
                String code = JDVerificationCodeController.getVerificationCode(authCodeImgUrl);
				usernmaeEle.sendKeys(phoneNum);
				authCodeEle.sendKeys(code);
				driver.findElement(By.id("sureBtn")).click();
				
				//获取最后一级页面的句柄并读取需要的数据
				String windowHandle = driver.getWindowHandle();
				driver.switchTo().window(windowHandle);
				
				String userInfo = driver.findElement(By.className("page-txt")).getText();
				if(StringUtils.isBlank(userInfo)){
					log.error("用户名获取失败,userInfo="+userInfo);
				}
				//System.out.println(userInfo.split("：")[1].trim());
				//关闭浏览器和驱动
				driver.close();
				driver.quit();
				
				return userInfo.split("：")[1].trim();

}
	
	 public static void snapshot(TakesScreenshot driver, String filename)  
	  {  
	      // this method will take screen shot ,require two parameters ,one is driver name, another is file name  
	        
	        
	        File scrFile = driver.getScreenshotAs(OutputType.FILE);  
	        // Now you can do whatever you need to do with it, for example copy somewhere  
	        try {  
	            System.out.println("save snapshot path is:"+filename);  
	            FileUtils.copy(scrFile, new File(filename));  
	        } catch (IOException e) {  
	            System.out.println("Can't save screenshot");  
	            e.printStackTrace();  
	        }   
	        finally  
	        {  
	            System.out.println("screen shot finished");  
	        }  
	  }  
	    
	 public static void snapshot2(WebDriver driver, String filename)  
	  {  
	      // this method will take screen shot ,require two parameters ,one is driver name, another is file name  
	        
	        
	      //  File scrFile = drivername.getScreenshotAs(OutputType.FILE);  
	        // Now you can do whatever you need to do with it, for example copy somewhere  
	        try {  
	              WebDriver augmentedDriver = new Augmenter().augment(driver);  
	              File screenshot = ((TakesScreenshot) augmentedDriver).getScreenshotAs(OutputType.FILE);  
	              File file = new File(filename);  
	              FileUtils.copy(screenshot, file);  
	        } catch (IOException e) {  
	            System.out.println("Can't save screenshot");  
	            e.printStackTrace();  
	        }   
	        finally  
	        {  
	            System.out.println("screen shot finished");  
	        }  
	  }  
	    
	 public static byte[] takeScreenshot(WebDriver driver) throws IOException {  
	        WebDriver augmentedDriver = new Augmenter().augment(driver);  
	      return ((TakesScreenshot) augmentedDriver).getScreenshotAs(OutputType.BYTES);  
	        //TakesScreenshot takesScreenshot = (TakesScreenshot) driver;  
	        //return takesScreenshot.getScreenshotAs(OutputType.BYTES);  
	        }  
	  
	 public static BufferedImage createElementImage(WebDriver driver,WebElement webElement)  
	                throws IOException {  
	                // 获得webElement的位置和大小。  
	                Point location = webElement.getLocation();  
	                Dimension size = webElement.getSize();  
	                // 创建全屏截图。  
	                BufferedImage originalImage =  
	                ImageIO.read(new ByteArrayInputStream(takeScreenshot(driver)));  
	                // 截取webElement所在位置的子图。  
	                BufferedImage croppedImage = originalImage.getSubimage(  
	                location.getX(),  
	                location.getY(),  
	                size.getWidth(),  
	                size.getHeight());  
	                return croppedImage;  
	                }  

}

