---
title: phantomJSDriver无法访问https的解决办法
date: 2017-07-14 08:50:02
tags: [phantomjs]
---


		DesiredCapabilities dCaps = DesiredCapabilities.phantomjs();
		String[] phantomArgs = new String[] { "--webdriver-loglevel=NONE"," --ignore-ssl-errors=true" };
		dCaps.setCapability(PhantomJSDriverService.PHANTOMJS_CLI_ARGS, phantomArgs);
		dCaps.setCapability(PhantomJSDriverService.PHANTOMJS_PAGE_SETTINGS_PREFIX + "userAgent",
		"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_2) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.97 Safari/537.11");
		dCaps.setJavascriptEnabled(true);
		dCaps.setCapability("takesScreenshot", true);

> 重点是`--ignore-ssl-errors=true`
