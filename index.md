##### 1、云平台的环境搭建
1. 开发工具[vscode](https://code.visualstudio.com/)，脚本命令在终端(Terminal)运行
2. node服务；可参考[node环境配置](https://www.jianshu.com/p/13f45e24b1de)；命令行执行node -v和npm -v不报错标志着node环境搭建成功。
3. npm使用国内淘宝镜像，命令行执行以下代码，将npm服务器地址改为国内镜像，否则有些库需要翻墙下载；可参考[npm国内镜像配置](https://www.cnblogs.com/luyuandatabase/p/12145707.html)
  > npm config set registry https://registry.npm.taobao.org
4. 拉取项目代码，项目根目录为package.json、src、config这一层，大部分脚本命令都在根目录中执行。

![根目录](https://taiyuan-file.oss-cn-shanghai.aliyuncs.com/upload/parklogo/1584673346063.png)
4. 依赖库/npm包的安装：在根目录执行，根据网速快慢大约需要15-30分钟的时间，超过时间卡住就可能是npm国内镜像没配置好；一般不会有红色的报错，如果有，问题可能出现在package.json的配置的依赖库版本不匹配。
```
npm run install
```
##### 2、云平台的启动与打包
###### 脚本命令在根目录的package.json文件中
  1. 在项目根目录下执行以下命令可以启动开发环境前端服务，
```
npm run start-dev
```
  2. 执行编译与打包至/server/build目录，直接提交代码并打tag就可更新至仿真环境
```
npm run build
```
  3. 在本地运行打包与编译后的代码，此处代码就是上传至服务器运行的代码。一般用于本地调试，不常使用。
```
npm run start
```
