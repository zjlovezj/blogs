# Electron的必要性

从对硬件的利用上讲，web < PWA < native。用到Electron的地方就是需要更加充分利用硬件，或者已有的浏览器接口无法实现需求。  
相比依赖于具体OS的编程平台，Electron的跨平台移植成本较低。  
相对而言，JavaScript社区有众人拾柴火焰高的效果，很多开源工具可以使用，也更容易得到贡献和帮助。

# Electron应用的架构

Electron应用的核心场景可能是本地，比如VS Code。或者Web。  
基于此，程序员要面对2个世界，一个世界是NodeJs和Electron，另一个世界是Web。  
这两者可以通过往Web世界里面注入JS来沟通？(ipcRenderer/ipcMain/remote待学习)，不要在Web里面直接访问NodeJs，这是很大的安全隐患。

# Electron的文件目录结构

1. NodeJs在哪里？
2. npm包在哪里？
3. source code在哪里？ electron/Electron.app/Contents/Resources/app/ 或 electron/resources/app。  
   可以将package.json等文件放到这里，即可执行。
   或者打包为asar。Electron对fs做了patch，可以把此包当成一个文件夹。
