### pojo-helper(https://github.com/snowThinker/pojo-helper)

[![Maven Central](https://img.shields.io/maven-central/v/io.github.snowthinker/mybatis-helper.svg?label=Maven%20Central)](https://mvnrepository.com/artifact/io.github.snowthinker/mybatis-helper)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)

## Main Functions

* **Java Bean Mask Support**： Mask or encrypt bean with annotation
* **Java Bean Flexible Convert**： Rich reflection tools, easily to convert between HashMap and Data Transfer Object

Usage:
##### 1. Add maven dependency
	
```xml
<dependency>
	<groupId>io.github.snowthinker</groupId>
	<artifactId>pojo-helper</artifactId>
	<version>0.0.1-RELEASE</version>
</dependency>
```


##### 2. Write mask or encryption bean and rewrite toString() method

```java  
class MaskBean {
	private String name;
	
	@Mask(type=MaskType.MOBILE, format="#")
	private String mobile;

	@Mask(type=MaskType.IDCARD)
	private String idcard;
	
	@Encryption(type=EncryptionType.AES)
	private String cardNumber;

	@Mask(type=MaskType.ADDRESS)
	private String address;
	
	private Date birthday;
	
	@Override
	public String toString() {
		return MaskUtils.toString(this);
	}
	//TODO getter and setters
}
```

##### 3. Run the test case

```java
public class MaskUtilsTest extends TestCase {
	@Test
	public void testMaskBean() {
		MaskBean maskBean = new MaskBean();
		maskBean.setName("Andrew");
		maskBean.setMobile("12318638123");
		maskBean.setIdcard("123212196309222123");
		maskBean.setCardNumber("1238390056241234");
		maskBean.setAddress("2915 Canyon Lake Dr, Rapid City, SD 57702");
		maskBean.setBirthday(new Date());
		maskBean.setAge(17);
		
		AESEncryptor.encryptObject(maskBean);
		
		System.out.printf("after mask and encrypt: %s\r\n", maskBean);
	}
}
```

##### 4.Output: 
```
after mask and encrypt: MaskBean[name=Andrew, mobile=123####8123, idcard=1232121963****2123, cardNumber=jzvCqL2QEMQliI2Pvdx7Chi7uEURzsK8I7iejfobS7Q=, address=2915 Canyon Lake Dr, Rapid City, ****7702, birthday=Sat Apr 29 11:36:35 CST 2017, age=17
```
