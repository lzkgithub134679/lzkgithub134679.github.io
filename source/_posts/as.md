---
title: 在项目启动后自动执行方法
date: 2019-08-15 14:37:35
tags:
---

# SpringBoot的ApplicationRunner

接口分别为CommandLineRunner和ApplicationRunner。**他们的执行时机为容器启动完成的时候。**

```
@Configuration
public void run(ApplicationArguments args) throws Exception {
		logger.info("正在连接 canal 客户端...");
        //连接 canal 客户端
        CanalClient canalClient = new CanalClient(canalConfig);
        logger.info("正在开启 canal 客户端...");
        //开启 canal 客户端
        canalClient.start();
}
```

