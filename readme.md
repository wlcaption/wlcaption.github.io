部署到github
hexo clean
hexo generate
hexo deploy
一些常用的命令:
hexo new "postName" #新建文章
hexo new page "postName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口
hexo help #查看帮助
hexo version #查看当前Hexo版本
hexo s -p 5000 #运行在5000端口上

报错总结
ERROR Deployer not found: git 或者 ERROR Deployer not found: github
解决方法： npm install hexo-deployer-git --save