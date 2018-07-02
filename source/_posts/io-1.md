---
title: java二进制流
date: 2016-12-19 17:03:18
tags: java
categories:
  - NIO
---
   通过这个测试用例，明确Java中的io流是怎么一回事，以及在项目中如何通过二进制流传输我们需要的文件。以下代码仅供测试使用。
  
- 将文件转化成二进制流，然后通过流接收。

```
package com.caoqiwen.io;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Random;
import java.util.zip.DeflaterOutputStream;

/***
 * 字节流的做法
 * @author caoqiwen
 *
 */

public class Demo5 {

	public static void main(String[] args) throws IOException {
		byte[] arr = getBytesFromFile("F:/Android/Java/JavaDemo/JavaDemo2/2.txt");
		int random = new Random().nextInt(100);
		String name = String.valueOf(random);
		String destPath = "F:/Android/Java/JavaDemo/JavaDemo2/" + name + ".txt";
		toFileFromByteArray(arr, destPath);
	}


//转换成byte数组
	public static byte[] getBytesFromFile(String srcPath) throws IOException {
		File file = new File(srcPath);
		byte[] dest = null;
		InputStream is = new BufferedInputStream(new FileInputStream(srcPath));
		// 字节数组输出流
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();

		byte[] flush = new byte[1024];   //缓冲空间为1024个字节。
		int len = 0;
		while (-1 != (len = is.read(flush))) {
			byteArrayOutputStream.write(flush, 0, len);
		}
		byteArrayOutputStream.flush();
		dest = byteArrayOutputStream.toByteArray(); // 转换成byte[]数组
		byteArrayOutputStream.close();
		is.close();
		return dest;
	}

	public static void toFileFromByteArray(byte[] src, String destPath) throws IOException {
		File dest = new File(destPath);
		InputStream iStream = new BufferedInputStream(new ByteArrayInputStream(src));
		OutputStream oStream = new BufferedOutputStream(new FileOutputStream(dest));
		byte[] flush = new byte[1024];
		int len = 0;
		while (-1 != (len = iStream.read(flush))) {
			oStream.write(flush, 0, len);
		}
		oStream.flush();   
		oStream.close();   //后面打开的先关闭
		iStream.close();
	}
}

```

