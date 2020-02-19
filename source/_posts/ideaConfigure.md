---
title: idea的注释自定义模板
date: 2019-10-15 15:46:18
tags:
- Show All
- idea
header-img: "Demo.png"

---

## idea的注释自定义模板

file->settings->live Templates->点击加号创建一个Template Group->然后在这个组下创建live Templates->

我经常用的注释模板:

```
*
 * @Description : TODO
 * @Author : lizhikang@youngyedu.com, $date$ $time$
 * @Modified : lizhikang@youngyedu.com, $date$
 * $params$
 * @return $return$
 */
```

参数的配置：

date: 

```
 date("yyyy年MM月dd日")
```

time: 

```
 time("HH:mm:ss")
```

params:  

```
groovyScript(" 	def result='';   	def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList();    	for(i = 0; i < params.size(); i++) {   	 		if(i!=0)result+= ' * ';    	 		result+='@param ' + params[i] + ((i < (params.size() - 1)) ? '\\n' : '');    	};     	return result", methodParameters())
```

return:

```
methodReturnType()
```

