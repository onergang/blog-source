# blogSource
博客源码

 ![hexo](https://hexo.io/icon/favicon-32x32.png)

基于 hexo 构建。

构建步骤 	[hexo setup](https://hexo.io/zh-cn/docs/setup.html)

使用主题	 	[jacman](http://wuchong.me/jacman/how-to-use-jacman/)



执行步骤(根目录下执行)：

1. 安装依赖	 

   ```
   npm install
   npm install hexo -g
   ```

2. 下载主题

   ```shell
   git clone https://github.com/wuchong/jacman.git themes/jacman
   ```


3. 替换文件

   ```shell
   ./replace.sh
   ```


4. 页面生成

   ```shell
   hexo clean
   hexo g 或 hexo generate
   ```
   本地启动后 http://localhost:4000/ 即可看到效果。