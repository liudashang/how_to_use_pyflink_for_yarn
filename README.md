# How To Use PyFlink For Flink ON Yarn Cluster
The way about sumit flink-python job to flink on yarn cluster

#### Python Development environment Building

1.  Flink Document About the Pyflink, Introducing the shell which can create a python virtual environment. But, there not have the download url.
2.  I search Internet, find a older shell, please see my github file -- setup-pyflink-virtual-env.sh. Do not run the shell directly. You need edit it or reading the following way about creating venv.zip, step by step.
3.  If your yarn cluster's servers, ALL CLUSTER NODES, doesn't have same python3 environment. don't use the pipenv/virtualenv to create venv.zip.
4.  You should to use Miniconda3 [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html), creating a python3 virtual enviroment.
5.  Because the Miniconda3, installing the python3 to venv.zip completely.
6.  Notice! the Miniconda3's version, must satisfied to the PyFlink Document.

#### Python Environment Building STEP

1.  download the Miniconda3 Linux 64-bit shell file. and rename the shell to miniconda.sh.
2.  add x (runable) to the shell.
```shell
chmod +x miniconda.sh
```
3.  when runing the shell, Please add following parameters in the command, the way only install the Python3 to venv, Not the Linux system.
```shell
./miniconda.sh -b -p venv
```
4.  activate the environment:
```shell
source venv/bin/activate ""
```
5.  Now, you can install the python3 packages, by using 'conda install' or 'pip3 install'
```shell
#Notice when using pip3, please add the parameter '--target' in command
#For ensure the all package installed to venv.zip lib path, Not the Linux system's lib path.
pip3 install apache-flink==【flink vesion number】 --target venv/lib/python3.8/site-packages
#OR
conda install apache-flink==【flink vesion number】
```
6.  exit the vitual environment
```shell
conda deactivate
```
7.  delete the conda install cache:
```shell
rm -rf venv/pkgs
```
8.  zip it.
```shell
zip -r venv.zip venv
```

#### About Pyflink Java Dependency

1.  Pleas using flink-sql-connector-xxxx.jar, Not the flink-connector-xxxx.jar. Because the flink-sql-connector-xxxx.jar is ALL-IN-ONE jar package. if using the flink-connector-xxxx.jar, you need find the all dependency package. and submiting job to the cluster maybe cause Exception like "cannot assign instance of xxxx".
2.  Moving all jar files in the direcotry, like "javalib".You can add the following python code to the pyflink program. the code will upload all jars to yarn cluster which in javalib
```python
jars = []
lib_path = "/home/xxxx/venv/javalib/"
for file in os.listdir(lib_path):
    if file.endswith('.jar'):
        jars.append(lib_path+file)
str_jars = ';'.join(['file://' + jar for jar in jars])
t_env.get_config().get_configuration().set_string("pipeline.jars", str_jars)
```
3.  Also, You can using the maven to build a empty project, and editing the pom.xml to add dependency jar files. then runing the command to get all jar files in a lib diretory：
```shell
#-DoutputDirectory=lib lib is maven project's lib diretory.
mvn dependency:copy-dependencies -DoutputDirectory=lib -DincludeScope=compile
```

#### about the submit to yarn cluster:

```shell
bin/flink run  --target yarn-per-job \
-pyarch venv.zip \
-pyexec venv.zip/venv/bin/python3 \
-pyclientexec venv.zip/venv/bin/python3 \
-py pyflink_job.py
```
