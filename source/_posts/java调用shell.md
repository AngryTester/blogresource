---
title: java调用shell
date: 2017-11-14 16:28:15
tags: [Java]
---

```java
package com.angrytest.utils;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

public class SystemUtil {

	public static void openShell(final String shellPath) {
		new Thread() {
			public void run() {
				// 需要读取输出流和错误流，否则程序执行到一半会挂起，参考：http://www.blogjava.net/alwayscy/archive/2009/05/15/270925.html
				Process process = null;
				try {
					process = new ProcessBuilder(new String[] { "/bin/sh", "-c", shellPath }).start();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				StreamGobbler errorGobbler = new StreamGobbler(process.getErrorStream(), "Error");
				StreamGobbler outputGobbler = new StreamGobbler(process.getInputStream(), "Output");
				errorGobbler.start();
				outputGobbler.start();
			}
		}.run();
	}
}

class StreamGobbler extends Thread {

	InputStream is;
	String type;

	StreamGobbler(InputStream is, String type) {
		this.is = is;
		this.type = type;
	}

	public void run() {
		try {
			InputStreamReader isr = new InputStreamReader(is);
			BufferedReader br = new BufferedReader(isr);
			String line = null;
			while ((line = br.readLine()) != null) {
				if (type.equals("Error"))
					System.out.println(line);
				else
					System.out.println(line);
			}
		} catch (IOException ioe) {
			ioe.printStackTrace();
		}
	}
}

```