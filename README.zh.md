# pyflink环境搭建要点

#### 介绍
flink的Python运行环境搭建和向Yarn提交Pyflink任务的方法

#### Python开发环境搭建说明

1.  在Flink官网的Pyflink文档中，介绍了使用官方SHELL脚本搭建虚拟环境（venv.zip）。但是最新版本（1.15.2）中没有文件的下载链接。
2.  我从简书上找了相应的下载文件，见项目中的setup-pyflink-virtual-env.sh。但是，请不要直接运行该脚本，有能力的可以自己改再运行。或者参照我即将介绍的方法手动搭建环境。
3.  首先，要注意！不能使用Python3自带的virtualenv或者pipenv等虚拟环境搭建，无论是百度、Bing、知乎、阿里云文档等等介绍的方法均是错误的。原因在于，virtualenv、pipenv都不能够将Python3的标准运行库也就是lib、以及Python3运行的环境变量等安装到虚拟环境目录下，后果就是将pyflink作业提交到flink on yarn集群时，会因为计算节点没有Python3环境或者计算节点与开发者的实际开发环境存在差异，造成Python运行时各种报错，最终导致任务运行失败。
4.  官方shell脚本中推荐使用Miniconda3安装环境，请到Conda官网[https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html)或者清华开源镜像站查找下载。
5.  官方选择Miniconda3搭建venv.zip的原因在于，Miniconda3会将完整Python3运行环境——包括了lib、bin等均安装到venv环境中，yarn集群中所有的节点无需安装python3环境直接运行即可。
6.  最后，提醒一点，请严格按照Pyflink官方文档中提到的Python版本选择Miniconda3的版本！

#### Python开发环境搭建步骤

1.  从conda官方下载Miniconda3 Linux 64-bit的.sh安装包（为方便后续说明，下载后我重命名为miniconda.sh）。
2.  sh安装包执行可执行命令
```shell
chmod +x miniconda.sh
```
3.  不要直接运行sh文件，请增加如下参数，这样执行仅仅会搭建一个虚拟环境目录，而不会在linux上安装conda。
```shell
#venv指将创建的目录名称
./miniconda.sh -b -p venv
```
4.  打开运行环境
```shell
source venv/bin/activate ""
```
5.  现在可以通过conda install或者pip3 install安装需要包
```shell
#使用pip3安装时请一定注意增加--target，指向虚拟环境中的site-packages路径
#原因是，如果linux服务器已经安装了相同版本的python3，使用pip3 install时部分本机已有的包会直接调用，而不会安装到venv下。
#--target会将所有的python包复制到venv路径下
pip3 install apache-flink==【flink版本号】 --target venv/lib/python3.8/site-packages
#或者
conda install apache-flink==【flink版本号】
```
6.  python包安装完后即可在此环境下开发。开发完毕后执行下面命令退出环境
```shell
conda deactivate
```
7.  退出环境后，删除掉conda安装python包时，用于缓存的安装包路径
```shell
rm -rf venv/pkgs
```
8.  用zip打包运行环境
```shell
zip -r venv.zip venv
```

#### 关于Pyflink需要jar包说明

1.  关于flink的connecter依赖，请选择使用flink-sql-connector-xxxx.jar，而不要使用flink-connector-xxxx.jar包。因为sql包是将所有依赖打包成了一个jar包，使用flink-connector-xxx.jar，一是需要自己解决依赖包问题，二是即使找全了依赖包、在本地开发环境运行无问题，提交集群时计算节点经常会出现“cannot assign instance of xxxx”的报错。
2.  将所有的jar包放置于例如“javalib”的路径下，然后在pyflink程序下添加如下代码即可。官方文档中已经说明了，使用如下代码，yarn集群会自动将javalib下的jar包全部上传到集群中。
```python
jars = []
lib_path = "/home/xxxx/venv/javalib/"
for file in os.listdir(lib_path):
    if file.endswith('.jar'):
        jars.append(lib_path+file)
str_jars = ';'.join(['file://' + jar for jar in jars])
t_env.get_config().get_configuration().set_string("pipeline.jars", str_jars)
```
3.  下载第三方依赖包的小技巧——自己找java依赖包是很费时费力的事情。推荐使用maven搭建一个空项目，修改pom.xml添加需要下载的依赖包，之后使用如下命令，将jar包下载一个本地目录下：
```shell
#-DoutputDirectory=lib lib代表maven项目路径下的lib文件夹
mvn dependency:copy-dependencies -DoutputDirectory=lib -DincludeScope=compile
```

#### 关于yarn集群提交任务
1.  提交集群任务使用如下命令即可
```shell
bin/flink run  --target yarn-per-job \
-pyarch venv.zip \
-pyexec venv.zip/venv/bin/python3 \
-pyclientexec venv.zip/venv/bin/python3 \
-py pyflink_job.py
```

#### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
