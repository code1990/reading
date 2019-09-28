## jdk windows安装教程

1、创新建环境变量JAVA_HOME的值为JDK的安装目录( 如E:\Program File\Java\jdk1.6.0)
（虚线间的全部要复制过去，一个点都不能少！）

JAVA_HOME
C:\Program Files\Java\jdk1.6.0_24（此为常见目录，jdk的版本号可能不一样）

------

2、创建环境变量CLASSPATH的值为（虚线间的全部要复制过去，一个点都不能少！）

CLASSPATH
.;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\lib\dt.jar;

------

3、在环境变量PATH的值最前面添加如下（虚线间的全部要复制过去，一个点都不能少！）

PATH
%JAVA_HOME%\bin;

------

4、设置完了测试是否正确(按以下步骤)
运行->输入cmd->输入echo %JAVA_HOME%->输入echo %CLASSPTAH%->输入echo %PATH%