### MD5加密操作

> MD5加密算法可以对任意类型的数据形式进行加密，Java的基本类型，字符串，对象等，包括文件也能进过MD5加密后，获得文件的MD5值，主要是因为digest(byte[]) 接收的是一个数组，所以只要将不同类型的数据，转换成数组，经过MD5加密后，就能获取其相应的MD5值。

##### 字符串加密(明文密码加密)

> 一般用户的密码都是要进行加密后在存储的，这样可以提高安全性，但是现在的MD5加密已经就人专门进行破解，可以直接在网上输入加密后的MD5值，对于简单的加密可以直接获得对应的明文密码，所以下面在进行加密时，故意在字符串之后添加一个称之为“盐”的字符串，这是一个杂乱的字符串，使用这个杂乱的字符串和明文密码进行混合后，在进行MD5加密，被破解的几率就很小了。

	/**
	 * 设置一个杂乱的字符串，在进行加密时和密码一起进行加密，这样确保了加密的安全性
	 */
	private static final String OTHER_PWD = "ashdfjkahsdfkj32423k4hj234j233jn4kj";

	/**
	 * 将密码进行MD5加密，然后再保存
	 * @param pwd
	 * @return
	 */
	public static String pwdToMD5(String pwd)
	{

		try {
			MessageDigest md = MessageDigest.getInstance("md5");
			pwd +=  OTHER_PWD;
			byte[] pwdByte = md.digest(pwd.getBytes());
			
			StringBuffer sb = new StringBuffer();
			for(int i = 0 ;i < pwdByte.length ; i++)
			{
				//先转换成整型，将md5加密后的字节中，负数的变成正数
				int num = pwdByte[i] & 0xff;
				//将转换后的整形转换成16进制的字符串类型
				String hexNum = Integer.toHexString(num);
				if(hexNum.length() == 1)
				{
					sb.append("0"+hexNum);
				}
				else
					sb.append(hexNum);
			}
			return sb.toString();
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		}
		
		return null;	
	}


#### 文件加密

> 指定一个文件路径，使用IO流获取文件的字节数组，然后使用MessageDigest类中的update()方法，将遍历的到的字节传入，最后通过digest()方法，获得文件的MD5加密字节，最终转换成MD5加密的格式。



	/**
	 * 获取一个指定文件的MD5值
	 * @param fileName
	 * @return
	 */
	public static String fileToMD5(String fileName)
	{
		try {
			MessageDigest md = MessageDigest.getInstance("md5");
			FileInputStream fis = new FileInputStream(fileName);
			int len = 0;
			byte[] buffer = new byte[1024];
			while((len = fis.read(buffer))!=-1)
			{
				md.update(buffer, 0, len);				
			}
			byte[] fileByte = md.digest();
			StringBuffer sb = new StringBuffer();
			for(int i = 0 ;i < fileByte.length ; i++)
			{
				//先转换成整型，将md5加密后的字节中，负数的变成正数
				int num = fileByte[i] & 0xff;
				//将转换后的整形转换成16进制的字符串类型
				String hexNum = Integer.toHexString(num);
				if(hexNum.length() == 1)
				{
					sb.append("0"+hexNum);
				}
				else
					sb.append(hexNum);
			}
			fis.close();
			return sb.toString();
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}