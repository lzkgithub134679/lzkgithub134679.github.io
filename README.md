# hexo
这是搭建博客用的

克隆后，安装nodejs
sudo apt-get install nodejs
sudo apt-get install npm
在安装 hexo
npm install -g hexo-cli



更换电脑操作
一样的，跟之前的环境搭建一样，

安装git
sudo apt-get install git
1
设置git全局邮箱和用户名
git config --global user.name "yourgithubname"
git config --global user.email "yourgithubemail"


设置ssh key
ssh-keygen -t rsa -C "youremail"
#生成后填到github和coding上（有coding平台的话）
#验证是否成功
ssh -T git@github.com
ssh -T git@git.coding.net #(有coding平台的话)

安装nodejs
sudo apt-get install nodejs
sudo apt-get install npm

安装hexo
sudo npm install hexo-cli -g
1
但是已经不需要初始化了，

直接在任意文件夹下，

git clone git@………………
1
然后进入克隆到的文件夹：

cd xxx.github.io
npm install
npm install hexo-deployer-git --save

生成，部署：

hexo g
hexo d

然后就可以开始写你的新博客了

hexo new newpage

git add .
git commit –m "xxxx"
git push
