---
title: idea插件编写
date: 2020-01-20 15:25:10
tags:
- Show All
- idea
header-img: "Demo.png"
---

为什么开发idea插件?

在工作中，有很多的时候需要对一件事重复做很多次，比较麻烦。我们公司最近就遇到了，领导让同事开发出了一个自动化测试的项目，是根据规则，配置要测试的接口的yml，通过maven命令启动jar扫描配置文件yml，自动对比输入的数据和查出来的是否相同，这就导致每次测试一个接口，都需要输入一个maven命令去指定测试那个yml，每次都很繁琐，然后就想到了怎么才能一键根据当前选中的文件，自动输入文件名，去扫描，这时就想到自定义idea插件。

idea官网文档：<http://www.jetbrains.org/intellij/sdk/docs/welcome.html>

插件分类：

自定义编程语言的支持：包括语法高亮、文件类型识别、代码格式化、代码查看和自动补全等等。这类插件包括.gitignore，.shell这些。

框架继承：其实就是类似基于IntelliJ开发出另一个IDEA，比如AndroidStudio就是通过将Android SDK整合到了IntelliJ IDEA当中。比如还可以将Spring、Struts等框架集成到IDEA中，方便用户在IDEA使用使用特定的框架更加的方便

工具集成：对于IDEA定制一些个性化或者是实用的工具，比如lombok和translation插

附加UI：对于标准的UI界面进行修改，如在编辑框中加入一个背景图片等等。

参考： <https://blog.csdn.net/ExcellentYuXiao/article/details/80273109>

idea创建插件工程

![a](a.png)

刚建好的工程如下：

![b](b.png)

plugin.xml是工程的配置

```
<idea-plugin>
<!-- 插件名称，别人在官方插件库搜索你的插件时使用的名称 -->
<name>MyPlugin</name>

<!-- 插件唯一id，不能和其他插件项目重复，所以推荐使用com.xxx.xxx的格式
插件不同版本之间不能更改，若没有指定，则与插件名称相同 -->
<id>com.example.plugin.myplugin</id>

<!-- 插件的描述 -->
<description>my plugin description</description>

<!-- 插件版本变更信息，支持HTML标签；
将展示在 settings | Plugins 对话框和插件仓库的Web页面 -->
<change-notes>Initial release of the plugin.</change-notes>

<!-- 插件版本 -->
<version>1.0</version>

<!-- 供应商主页和email-->
<vendor url="http://www.jetbrains.com" email="support@jetbrains.com" />

<!-- 插件所依赖的其他插件的id -->
<depends>MyFirstPlugin</depends>

<!-- 插件兼容IDEA的最大和最小 build 号，两个属性可以任选一个或者同时使用
官网详细介绍：http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/build_number_ranges.html-->
<idea-version since-build="3000" until-build="3999"/>

<!-- application components -->
<application-components>
<component>
<!-- 组件接口 -->
<interface-class>com.foo.Component1Interface</interface-class>
<!-- 组件的实现类 -->
<implementation-class>com.foo.impl.Component1Impl</implementation-class>
</component>
</application-components>

<!-- project components -->
<project-components>
<component>
<!-- 接口和实现类相同 -->
<interface-class>com.foo.Component2</interface-class>
</component>
</project-components>

<!-- module components -->
<module-components>
<component>
<interface-class>com.foo.Component3</interface-class>
</component>
</module-components>

<!-- Actions -->
<actions>
...
</actions>

<!-- 插件定义的扩展点，以供其他插件扩展该插件 -->
<extensionPoints>
...
</extensionPoints>

<!-- 声明该插件对IDEA core或其他插件的扩展 -->
<extensions xmlns="com.intellij">
...
</extensions></idea-plugin>
```

这时候就可以开始创建Action：

![c](c.png)

在弹出的对话框中填充下列字段，然后点击 OK：

![d](d.png)

- **Action ID**: action 唯一 id，推荐 format: PluginName.ID

- **Class Name**: 要被创建的 action class 名称

- **Name**: menu item 的文本

- **Description**: action 描述，toolbar 上按钮的提示文本，可选

- **Add to Group**：选择新 action 要被添加到的 action group（**Groups, Actions**）以及相对其他 actions 的位置（**Anchor**）

- **Keyboard Shortcuts**：指定 action 的第一和第二快捷键

  

我选择的是**ProjectViewPopupMenu，这个是将插件放在项目右键的菜单里启动，然后再选择在当前有的哪个插件前或者后**

创建完在yml文件里就会有如下信息对应的是哪个组下的

![e](e.png)

下面是自己编写的小插件

```java
import com.intellij.openapi.actionSystem.*;
import com.intellij.openapi.project.Project;
import com.intellij.openapi.ui.Messages;
import com.intellij.openapi.vfs.VirtualFile;
import org.jetbrains.annotations.SystemIndependent;

import java.io.*;

public class FirstAction extends AnAction {

    private Project mProject;

    public final String PATH = "C:\\case\\case.bat";
    public final String PATHPAGE = "C:\\case";
    public final String YML = "yml";
    public final String MVN = "mvn rj-api-test:run -Dpath=";
    public final String TO_PATH = "/src/test/java";

    @Override
    public void actionPerformed(AnActionEvent e) {
        FileWriter fw = null;
        // 获取当前在操作的工程上下文
        mProject = e.getData(PlatformDataKeys.PROJECT);
        // 项目所在本地路径
        @SystemIndependent String basePath = mProject.getBasePath();
        DataContext dataContext = e.getDataContext();
        if (YML.equals(getFileExtension(dataContext))) {
            try {
                // 获取文件名
                String fileName = getFileName(dataContext);
                // 判断是否在文件夹内
                String appointFilePath = this.getAppointFilePath(basePath + 									TO_PATH,fileName);
                // 创建bat文件
                File file = new File(PATH);
                File file1 = new File(PATHPAGE);
                if(!file1.exists()){
                    file1.mkdir();
                    if (!file.exists()) {
                        file.createNewFile();
                    }
                }
                fw = new FileWriter(file);
                // 写入要执行的命令
                fw.write("cd " + basePath + "\r\n");
                fw.write(MVN + appointFilePath + fileName + "\r\n");
                fw.write("pause");
                fw.flush();
                Runtime.getRuntime().exec("cmd /c start " + PATH);
            } catch (IOException ex) {
                ex.printStackTrace();
            } finally {
                try {
                    fw.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        } else {
            Messages.showMessageDialog(mProject, "请选择yml文件!!! ◕ ں ◕",
                    "温馨提示", Messages.getInformationIcon());
        }
    }
    public String getAppointFilePath(String path,String fileName){
        File fileFoleds = new File(path);
        String[] lists = fileFoleds.list();
        String zhiPath = "";
        for(int i =0;i<lists.length;i++){
            String pa = path+"/"+lists[i];
            File readFile  = new File(pa);
            // 判断是不是文件夹
            if(readFile.isDirectory()){
                String appointFilePath = getAppointFilePath(pa, fileName);
                if(!"".equals(appointFilePath)){
                    return appointFilePath;
                }
            }else{
                //判断是不是文件
                String houzui = 		
                    readFile.getName().substring(readFile.getName().lastIndexOf(".")+1);
                if("yml".equals(houzui) && readFile.isFile() && 
                   readFile.getName().equals(fileName)){
                    String[] split = readFile.getParent().split("\\\\");
                    for(int j = split.length-1;j>=0;j--){
                        if(!"java".equals(split[j])){
                            if(!"".equals(split[j])){
                                zhiPath = split[j]+"/"+zhiPath;
                            }else{
                                zhiPath = split[j];
                            }
                        }else{
                            break;
                        }
                    }
                    return zhiPath;
                }
            }
        }
        return zhiPath;
    }

    @Override
    public void update(AnActionEvent event) {
        //在Action显示之前,根据选中文件扩展名判定是否显示此Action
        String extension = getFileExtension(event.getDataContext());
        this.getTemplatePresentation().setEnabled(extension != null && "yml".equals(extension));
    }

    public static String getFileExtension(DataContext dataContext) {
        VirtualFile file = CommonDataKeys.VIRTUAL_FILE.getData(dataContext);
        return file == null ? null : file.getExtension();
    }
    public static String getFileName(DataContext dataContext) {
        VirtualFile file = CommonDataKeys.VIRTUAL_FILE.getData(dataContext);
        return file == null ? null : file.getName();
    }
}

```

