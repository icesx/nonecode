Ivscode
========

## Predefined variables[#](https://code.visualstudio.com/docs/editor/variables-reference#_predefined-variables)

The following predefined variables are supported:

- **${workspaceFolder}** - the path of the folder opened in VS Code
- **${workspaceFolderBasename}** - the name of the folder opened in VS Code without any slashes (/)
- **${file}** - the current opened file
- **${fileWorkspaceFolder}** - the current opened file's workspace folder
- **${relativeFile}** - the current opened file relative to `workspaceFolder`
- **${relativeFileDirname}** - the current opened file's dirname relative to `workspaceFolder`
- **${fileBasename}** - the current opened file's basename
- **${fileBasenameNoExtension}** - the current opened file's basename with no file extension
- **${fileDirname}** - the current opened file's dirname
- **${fileExtname}** - the current opened file's extension
- **${cwd}** - the task runner's current working directory on startup
- **${lineNumber}** - the current selected line number in the active file
- **${selectedText}** - the current selected text in the active file
- **${execPath}** - the path to the running VS Code executable
- **${defaultBuildTask}** - the name of the default build task
- **${pathSeparator}** - the character used by the operating system to separate components in file paths

### Predefined variables examples[#](https://code.visualstudio.com/docs/editor/variables-reference#_predefined-variables-examples)

Supposing that you have the following requirements:

1. A file located at `/home/your-username/your-project/folder/file.ext` opened in your editor;
2. The directory `/home/your-username/your-project` opened as your root workspace.

So you will have the following values for each variable:

- **${workspaceFolder}** - `/home/your-username/your-project`
- **${workspaceFolderBasename}** - `your-project`
- **${file}** - `/home/your-username/your-project/folder/file.ext`
- **${fileWorkspaceFolder}** - `/home/your-username/your-project`
- **${relativeFile}** - `folder/file.ext`
- **${relativeFileDirname}** - `folder`
- **${fileBasename}** - `file.ext`
- **${fileBasenameNoExtension}** - `file`
- **${fileDirname}** - `/home/your-username/your-project/folder`
- **${fileExtname}** - `.ext`
- **${lineNumber}** - line number of the cursor
- **${selectedText}** - text selected in your code editor
- **${execPath}** - location of Code.exe
- **${pathSeparator}** - `/` on macOS or linux, `\` on Windows

> **Tip**: Use IntelliSense inside string values for `tasks.json` and `launch.json` to get a full list of predefined variables.



## 基本操作

### 同步

可以登录微软账号，同步本地和服务器保存的配置，放置以后删除的影响。	

### settings.json

F1-



## 基本插件

### IntelliJ IDEA Key Bindings for Visual Studio Code

### markdownlint

### ESP-IDF VS Code Extension

### CMake Tools



### CMake For VisualStudio Code

### VS Code C/C++ Makefile Project

### C/C++ for Visual Studio Code

## 快捷键

1. 命令面板：Ctrl + Shift + P或者F1
2. **注释**：
   1. 单行注释：[ctrl+k,ctrl+c] 或 ctrl+/
   2. 取消单行注释：[ctrl+k,ctrl+u] (按下ctrl不放，再按k + u)
   3. 多行注释：[alt+shift+A]
   4. 多行注释：/
3. **移动行**：alt+up/down
4. 显示/隐藏左侧目录栏 ctrl + b
5. 复制当前行：shift + alt +up/down
6. 删除当前行：shift + ctrl + k
7. 控制台终端显示与隐藏：ctrl + ~
8. 查找文件/安装vs code **插件地址**：ctrl + p
9. **代码格式化**：shift + alt +i
10. 新建一个窗口 : ctrl + shift + n
11. 行增加缩进: ctrl + [
12. 行减少缩进: ctrl + ]
13. 裁剪尾随空格(去掉一行的末尾那些没用的空格) : ctrl + shift + x
14. 字体放大/缩小: ctrl + ( + 或 - )
15. 拆分编辑器 : ctrl + 1/2/3
16. 切换窗口 : ctrl + shift + left/right
17. 关闭编辑器窗口: ctrl + w
18. 关闭所有窗口 :ctrl + k + w
19. 切换全屏 :F11
20. 自动换行: alt + z
21. 显示git: ctrl + shift + g
22. 全局查找文件:ctrl + shift + f
23. 显示相关插件的命令(如：git log)：ctrl + shift + p
24. 选中文字：shift + left / right / up / down
25. 折叠代码： ctrl + k + 0-9 (0是完全折叠)
26. 展开代码： ctrl + k + j (完全展开代码)
27. 删除行 ： ctrl + shift + k
28. 快速切换主题：ctrl + k / ctrl + t
29. 快速回到顶部 ： ctrl + home
30. 快速回到底部 : `ctrl + end`
31. 格式化选定代码：`ctrl + k / ctrl +f`
32. 选中代码 ：shift + 鼠标左键
33. 多行同时添加内容（光标） ：**ctrl + alt + up/down**
34. **全局替换：**ctrl + shift + h
35. 当前文件替换：**ctrl + h**
36. **打开最近打开的文件：**ctrl + r
37. 打开新的命令窗：ctrl + shift + c
38. 重新打开一个关闭的页面：`Ctrl + Shift + T` 
39. 选中文本出现多个：`Ctrl + F2`
40. 复制光标：按`Ctrl + Alt +向上箭头`
41. Explorer：`ctrl+E`
42. Git：`ctrl+shif+G`
43. Debug console:`ctrl+shif+D`
44. Terminal: ctrl+ `
45. 
