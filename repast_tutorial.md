# Demo_00：介绍RepastHPC程序的编译运行
这个部分所有样例都只有一个源码文件：Demo_00.cpp
## Step_01 ~ Step_03: Repast环境配置及MPI运行
虚拟机下有两种安装配置RepastHPC的方式：
1. **安装虚拟机+Repast_HPC_VM_Ubuntu_14.04.oav**：这种方式的repastHPC及其依赖环境都不需要自己安装，但是ubunte和RepastHPC的版本比较低，使用起来不是很方便(注意在bios中打开virtual technology)：
2. **Ubuntu_16.04+RepastHPC_2.3**：这种方式需要自己安装repastHPC及其依赖环境（根据repast文件夹中的install.txt一步步安装，也很方便）。**安装时需要注意的问题有：**

(1) `./install.sh netcdf`
出现错误:
	`error: Can't find or link to the z library. Turn off netCDF-4 and      opendap with --disable-netcdf-4 --disable-dap, or see config.log for errors.`
解决方法:netcdf依赖zlib，安装zlib:
`sudo apt-get install zlib*`
(2)`./install.sh rhpc`
出现错误:`error: cannot not find the flags to link with Boost mpi`
解决方法:`sudo apt-get install libboost-all-dev`

**运行zombie和rumor:**
在编译运行之前，需要修改MANUAL_INSTALL中的makefile，将其中的路径改成自己电脑上的安装路径,并把repast的版本改成自己使用的版本：
```
# VARIABLES (Supply values for these; for definitions and examples, see INSTALL)
CXX=mpicxx
CXXLD=mpicxx
BOOST_INCLUDE_DIR=/home/dll/sfw/Boost/Boost_1.61/include
BOOST_LIB_DIR=/home/dll/sfw/Boost/Boost_1.61/lib
BOOST_INFIX=-mt
NETCDF_INCLUDE_DIR=/home/dll/sfw/NetCDF/include
NETCDF_LIB_DIR=/home/dll/sfw/NetCDF/lib
NETCDF_CXX_INCLUDE_DIR=/home/dll/sfw/NetCDF-cxx/include
NETCDF_CXX_LIB_DIR=/home/dll/sfw/NetCDF-cxx/lib
CURL_INCLUDE_DIR=/home/dll/sfw/CURL/include
CURL_LIB_DIR=/home/dll/sfw/CURL/lib
INSTALL_DIR=/home/dll/repast_hpc-2.3.0

REPAST_VERSION=2.3
```
改好之后，`make -f Makefile all`在bin文件夹中生成可执行文件，运行如下：
Zombies:
```
	mpirun -n 4 ./zombie_model config.props model.props
```
Rumor:
```
	mpirun -n 4 ./rumor_model config.props model.props
```
**MPI编译+运行C++：**
  ``` 
  mpicxx -o Demo_00 Demo_00.cpp
  mpirun -n 4 ./Demo_00 //进程数为4
  ```

## Step_04: RepastHPC集成Boost
* 代码，只有一个demo.cpp，与上面的示例相比，不直接使用mpi而是使用boost
  ``` C++{highlight=[2,5,6,9,14]}
  #include <stdio.h>
  #include <boost/mpi.hpp> //#include <mpi.h>

  int main(int argc, char** argv){	
      boost::mpi::environment env(argc, argv); //MPI_Init(&argc, &argv);
	  boost::mpi::communicator world;  //MPI_Comm_rank(MPI_COMM_WORLD, &rank);

	  if(world.rank() == 0){
	      printf("Hello, world! I'm rank %d\n", world.rank()); //省略int rank;
	  }
	  else{
	      printf("Hmm...\n");
	  }	
      //省略MPI_Finalize();
  }
   ```
* 在work文件夹中，通过makefile运行代码，注意修改makefile中的相关内容，修改env文件中的路径为：
```
   MPICXX=/usr/local/bin/mpicxx -std=c++11 

   BOOST_INCLUDE=-I/home/repasthpc/repast_hpc-2.1.0/ext/Boost/Boost_1.58/include/
   BOOST_LIB_DIR=-L/home/repasthpc/repast_hpc-2.1.0/ext/Boost/Boost_1.58/lib/
   BOOST_LIBS=-lboost_mpi-mt -lboost_serialization-mt -lboost_system-mt -lboost_filesystem-mt

   REPAST_HPC_INCLUDE=-I/home/repasthpc/repast_hpc-2.1.0/ext/include/repast_hpc/
   REPAST_HPC_LIB_DIR=-L/home/repasthpc/repast_hpc-2.1.0/lib
   REPAST_HPC_LIB=-lrepast_hpc_2.1
```
   
   或（注意两种不同的安装方式，安装路径的区别）
```
MPICXX=/usr/bin/mpicxx -std=c++11 

BOOST_INCLUDE=-I/home/dll/sfw/Boost/Boost_1.61/include/
BOOST_LIB_DIR=-L/home/dll/sfw/Boost/Boost_1.61/lib/
BOOST_LIBS=-lboost_mpi-mt -lboost_serialization-mt -lboost_system-mt -lboost_filesystem-mt

REPAST_HPC_INCLUDE=-I/home/dll/sfw/repast_hpc-2.3.0/include/
REPAST_HPC_LIB_DIR=-L/home/dll/sfw/repast_hpc-2.3.0/lib/
REPAST_HPC_LIB=-lrepast_hpc-2.3.0
```

* 使用编译+运行C++:
  ```
  make RepastHPC_Demo_xx_Step_xx
  mpirun -n 4 ./Demo.exe
  ```

  
编译成功，运行时出现问题：
**1.**
```
./Demo_00.exe: error while loading shared libraries: librepast_hpc-2.3.0.so: cannot open shared object file: No such file or directory

```
解决过程：
先用
```
sudo gedit /etc/ld.so.conf
include /home/dll/sfw/repast_hpc-2.3.0/lib
sudo ldconfig
```
不成功

参考：https://blog.csdn.net/qing101hua/article/details/53086318
```
sudo gedit ~/.bashrc
添加
export LD_LIBRARY_PATH="/home/dll/sfw/repast_hpc-2.3.0/lib/:$LD_LIBRARY_PATH"

```
**2.**
```
./Demo_00.exe: error while loading shared libraries: libboost_mpi-mt.so.1.61.0: cannot open shared object file: No such file or directory
```

解决如下：
```
sudo gedit ~/.bashrc
添加
export LD_LIBRARY_PATH="/home/dll/sfw/Boost/Boost_1.61/lib/:$LD_LIBRARY_PATH"

```

**3.** 报错如下
```
[ubuntu:03716] *** Process received signal ***
[ubuntu:03716] Signal: Segmentation fault (11)
[ubuntu:03716] Signal code: Address not mapped (1)
[ubuntu:03716] Failing at address: 0x44000098
[ubuntu:03716] [ 0] /lib/x86_64-linux-gnu/libc.so.6(+0x354c0)[0x7fb797bde4c0]
[ubuntu:03716] [ 1] /usr/lib/libmpi.so.12(PMPI_Comm_set_errhandler+0x50)[0x7fb798562470]
[ubuntu:03716] [ 2] /home/dll/sfw/Boost/Boost_1.61/lib/libboost_mpi-mt.so.1.61.0(_ZN5boost3mpi11environmentC2ERiRPPcb+0x51)[0x7fb798a0e6c1]
[ubuntu:03716] [ 3] ./Demo_00.exe[0x40865e]
[ubuntu:03716] [ 4] /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7fb797bc9840]
[ubuntu:03716] [ 5] ./Demo_00.exe[0x408559]
[ubuntu:03716] *** End of error message ***
[ubuntu:03714] *** Process received signal ***
[ubuntu:03714] Signal: Segmentation fault (11)
[ubuntu:03714] Signal code: Address not mapped (1)
[ubuntu:03714] Failing at address: 0x44000098
[ubuntu:03715] *** Process received signal ***
[ubuntu:03715] Signal: Segmentation fault (11)
[ubuntu:03715] Signal code: Address not mapped (1)
[ubuntu:03715] Failing at address: 0x44000098
[ubuntu:03715] [ 0] [ubuntu:03717] *** Process received signal ***
[ubuntu:03717] Signal: Segmentation fault (11)
[ubuntu:03717] Signal code: Address not mapped (1)
[ubuntu:03717] Failing at address: 0x44000098
[ubuntu:03717] [ 0] [ubuntu:03714] [ 0] /lib/x86_64-linux-gnu/libc.so.6(+0x354c0)[0x7ff7f46374c0]
[ubuntu:03714] [ 1] /lib/x86_64-linux-gnu/libc.so.6(+0x354c0)[0x7f9aa42a84c0]
[ubuntu:03715] [ 1] /lib/x86_64-linux-gnu/libc.so.6(+0x354c0)[0x7f127ffd24c0]
/usr/lib/libmpi.so.12(PMPI_Comm_set_errhandler+0x50)[0x7ff7f4fbb470]
[ubuntu:03714] [ 2] /usr/lib/libmpi.so.12(PMPI_Comm_set_errhandler+0x50)[0x7f9aa4c2c470]
[ubuntu:03715] [ 2] /home/dll/sfw/Boost/Boost_1.61/lib/libboost_mpi-mt.so.1.61.0(_ZN5boost3mpi11environmentC2ERiRPPcb+0x51)[0x7ff7f54676c1]
[ubuntu:03714] [ 3] ./Demo_00.exe[0x40865e]
[ubuntu:03714] [ 4] /home/dll/sfw/Boost/Boost_1.61/lib/libboost_mpi-mt.so.1.61.0(_ZN5boost3mpi11environmentC2ERiRPPcb+0x51)[0x7f9aa50d86c1]
[ubuntu:03715] [ 3] ./Demo_00.exe[0x40865e]
[ubuntu:03715] [ 4] /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7ff7f4622840]
[ubuntu:03714] [ 5] ./Demo_00.exe[0x408559]
[ubuntu:03714] *** End of error message ***
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7f9aa4293840]
[ubuntu:03715] [ 5] ./Demo_00.exe[0x408559]
[ubuntu:03715] *** End of error message ***
[ubuntu:03717] [ 1] /usr/lib/libmpi.so.12(PMPI_Comm_set_errhandler+0x50)[0x7f1280956470]
[ubuntu:03717] [ 2] /home/dll/sfw/Boost/Boost_1.61/lib/libboost_mpi-mt.so.1.61.0(_ZN5boost3mpi11environmentC2ERiRPPcb+0x51)[0x7f1280e026c1]
[ubuntu:03717] [ 3] ./Demo_00.exe[0x40865e]
[ubuntu:03717] [ 4] /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7f127ffbd840]
[ubuntu:03717] [ 5] ./Demo_00.exe[0x408559]
[ubuntu:03717] *** End of error message ***

===================================================================================
=   BAD TERMINATION OF ONE OF YOUR APPLICATION PROCESSES
=   PID 3714 RUNNING AT ubuntu
=   EXIT CODE: 139
=   CLEANING UP REMAINING PROCESSES
=   YOU CAN IGNORE THE BELOW CLEANUP MESSAGES
===================================================================================
YOUR APPLICATION TERMINATED WITH THE EXIT STRING: Segmentation fault (signal 11)
This typically refers to a problem with your application.
Please see the FAQ page for debugging suggestions
```
解决：
```
明天试试将整个repastHPC及其依赖全部重装
```
## Step_05: Repast Process（进程）
Repastprocess是最基本的实例(instance)元素。
* 创建及初始化Repastprocess代码如下：
  ``` C++ {highlight=[3,9,14,16]}
  #include <stdio.h>
  #include <boost/mpi.hpp>
  #include "repast_hpc/RepastProcess.h"

  int main(int argc, char** argv){
	
      //Repast HPC需要一个文件，该文件包含有关其配置方面的信息，尤其是有关日志记录（logging）的信息，
      //例如应在何处以何种格式输出有关模拟运行的信息,它作为第一个参数传递给模型。
      std::string configFile = argv[1]; // The name of the configuration file is Arg 1
	
	  boost::mpi::environment env(argc, argv);
	  boost::mpi::communicator world;

	  repast::RepastProcess::init(configFile); //初始化RepastProcess
	
	  repast::RepastProcess::instance()->done();
  }
  ```
* 编译及运行
  ```
  make RepastHPC_Demo_xx_Step_xx
  mpirun -n 4 ./Demo.exe config.props
  ```
## Step_06: Repast Model
创建一个新的类RepastHPCDemoModel,添加代码如下：
  ```C++{highlight=[5-10,18-20]}
  #include <stdio.h>
  #include <boost/mpi.hpp>
  #include "repast_hpc/RepastProcess.h"

  class RepastHPCDemoModel{
  public:
	  RepastHPCDemoModel(){}
	  ~RepastHPCDemoModel(){}
	  void init(){}    //暂时留空，在后续会初始化包含agents和其他元素	
  };

  int main(int argc, char** argv){	
	  std::string configFile = argv[1]; // The name of the configuration file is Arg 1	
	  boost::mpi::environment env(argc, argv);
	  boost::mpi::communicator world;
	  repast::RepastProcess::init(configFile);
	
	  RepastHPCDemoModel* model = new RepastHPCDemoModel();	
	  model->init(); 
	  delete model;
	
	  repast::RepastProcess::instance()->done();	
  }
  ```
## Step_07 初始化schedule
Step_08中将要使Model在repasthpc下运行起来，这一步要为此奠定基础，添加代码如下：
  ```C++{highlight=[11,24,27]}
  #include <stdio.h>
  #include <boost/mpi.hpp>
  #include "repast_hpc/RepastProcess.h"

  class RepastHPCDemoModel{
  public:
 	  RepastHPCDemoModel(){}
	  ~RepastHPCDemoModel(){}
	  void init(){}
      //通过initSchedu通过来初始化schedule
	  void initSchedule(repast::ScheduleRunner& runner){}       
  };

  int main(int argc, char** argv){
	
	  std::string configFile = argv[1]; // The name of the configuration file is Arg 1	
	  boost::mpi::environment env(argc, argv);
	  boost::mpi::communicator world;
	  repast::RepastProcess::init(configFile);	
	  RepastHPCDemoModel* model = new RepastHPCDemoModel();

      //对RepastProcess::instance()->getScheduleRunner()的调用检索到一个对象(object)的句柄，
      //该对象管理事件发生的时间，然后将时间传递给模型，以便模型将事件添加到schedule中。
	  repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();          
	
	  model->init();
	  model->initSchedule(runner);
	  
	  delete model;
	
	  repast::RepastProcess::instance()->done();	
  }
  ```
## Step_08: schedule events
这个步骤中，我们将事件添加到schedule中执行，添加代码如下：
```C++ {highlight=[11-13,17-18,33]}
#include <stdio.h>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"

class RepastHPCDemoModel{
public:
	RepastHPCDemoModel(){}
	~RepastHPCDemoModel(){}
	void init(){}
        //doSomething模块，之后加入schedule中，这里输出正在执行哪个rank
	void doSomething(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){ //runner用于安排模拟中的事件
                //第一个参数表示模拟在第一个时间步时发生，
                //第二个参数repast::Schedule::FunctorPtr指向对象RepastHPCDemoMod的doSomething
		runner.scheduleEvent(1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleStop(2); //模拟在第2个时间步停止
	}
};

int main(int argc, char** argv){
	
	std::string configFile = argv[1]; // The name of the configuration file is Arg 1	
	boost::mpi::environment env(argc, argv);
	boost::mpi::communicator world;
	repast::RepastProcess::init(configFile);	
	RepastHPCDemoModel* model = new RepastHPCDemoModel();
	repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();	
	model->init();
	model->initSchedule(runner);
	
	runner.run(); //按schedule执行事件
	
	delete model;	
	repast::RepastProcess::instance()->done();	
}
```
## Step_09：周期性事件
唯一更改的是class RepastHPCDemoModel中的initSchedule
``` {highlight=2}
//第二个参数为1，表示每隔1个时间步就会调用一次该函数，可以改为任何n
runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
```
这段代码中，事件在第二个时间步重复，同时也在第二个时间步停止，不能确定究竟哪个先执行，因此尽量在写代码过程中避免这种情况。
## Step_10: 复杂schedule
事件在不均匀的时间间隔开始，以不均匀的间隔重复发生，甚至在各个过程中也有所不同。
```C++ {highlight=[8-13,15-24]}

class RepastHPCDemoModel{
public:
	RepastHPCDemoModel(){}
	~RepastHPCDemoModel(){}
	void init(){}
	void doSomething(){//使用instance()->getScheduleRunner().currentTick()获取时间步
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something at " << 
		    repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;
	}
	void doSomethingElse(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something DIFFERENT at " << 
			repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){
		runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleEvent(2.2, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleEvent(2.3, 4, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		if(repast::RepastProcess::instance()->rank() == 0){
			runner.scheduleEvent(3.1, 2, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomethingElse)));
		}
		if(repast::RepastProcess::instance()->rank() == 1){
			runner.scheduleEvent(5.5, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomethingElse)));
		}
		runner.scheduleStop(10.9);
	}
};
```
schedule如下：
* 所有进程从时刻1开始"doSomething",并在每个时刻重复；
* 所有进程从时刻2.2开始"doSomething",并在每个时刻重复（3.2，4.2等）；
* 所有进程从时刻2.3开始"doSomething",并每4个时刻重复（2.3，6.3,10.3等）；
* rank 0从时刻3.1开始"doSomethingElse",并每2个时刻重复（3.1，5.1,7.1等）；
* rank 1从时刻5.5开始"doSomethingElse",不重复；
* 所有进程在10.9时刻停止。

## Step_11: model.props(模型属性文件)
在文件中配置仿真，读入参数，代码如下：
``` c++ {highlight=[4-5, 8,10-13,21,28,33]}
#include <stdio.h>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

class RepastHPCDemoModel{
	int stopAt;
public:
	RepastHPCDemoModel(std::string propsFile){ //属性文件实例，包含文件中指定的所有值
		repast::Properties props(propsFile);
		stopAt = repast::strToInt(props.getProperty("stop.at"));
	}
	~RepastHPCDemoModel(){}
	void init(){}
	void doSomething(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){
		runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleStop(stopAt);
	}
};

int main(int argc, char** argv){
	
	std::string configFile = argv[1]; // The name of the configuration file is Arg 1
	std::string propsFile  = argv[2]; // The name of the properties file is Arg 2
	
	boost::mpi::environment env(argc, argv);
	boost::mpi::communicator world;
	repast::RepastProcess::init(configFile);	
	RepastHPCDemoModel* model = new RepastHPCDemoModel(propsFile); //文件名作为参数
	repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();	
	model->init();
	model->initSchedule(runner);	
	runner.run();	
	delete model;
	repast::RepastProcess::instance()->done();	
}
```
执行时需要model.props文件，内容如下：
```
#Properties file //注释
stop.at = 2 //stop.at赋值
```
运行命令如下:
```
mpirun -n 4 Demo_00.exe config.props model.props
```
## Step_12：广播属性文件内容
上诉代码的问题在于，当进程过多时，由于每个进程都对属性文件执行独立读取，从而导致仿真过程中多次打开、读取、关闭属性文件，造成难以检索属性文件。我们希望通过一个进程读取文件，然后通过MPI属性将文件发送给其他进程，在RepastHPC中通过提供带有mpi通讯的Properties来实现。代码如下：
``` c++ {highlight=[10,11,33]}
#include <stdio.h>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

class RepastHPCDemoModel{
	int stopAt;
public:
	RepastHPCDemoModel(std::string propsFile, boost::mpi::communicator* comm){
		repast::Properties props(propsFile, comm);
		stopAt = repast::strToInt(props.getProperty("stop.at"));
	}
	~RepastHPCDemoModel(){}
	void init(){}
	void doSomething(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){
		runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleStop(stopAt);
	}
};

int main(int argc, char** argv){
	
	std::string configFile = argv[1]; // The name of the configuration file is Arg 1
	std::string propsFile  = argv[2]; // The name of the properties file is Arg 2
	
	boost::mpi::environment env(argc, argv);
	boost::mpi::communicator world;
	repast::RepastProcess::init(configFile);	
	RepastHPCDemoModel* model = new RepastHPCDemoModel(propsFile, &world);
	repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();	
	model->init();
	model->initSchedule(runner);	
	runner.run();	
	delete model;	
	repast::RepastProcess::instance()->done();	
}
```
## Step_13： 命令行传属性
也可通过命令行传参，而不修改属性文件。代码如下：
```c++ {highlight=[10,11,33]}
#include <stdio.h>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

class RepastHPCDemoModel{
	int stopAt;
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm){
		repast::Properties props(propsFile, argc, argv, comm);
		stopAt = repast::strToInt(props.getProperty("stop.at"));
	}
	~RepastHPCDemoModel(){}
	void init(){}
	void doSomething(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){
		runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		runner.scheduleStop(stopAt);
	}
};

int main(int argc, char** argv){
	
	std::string configFile = argv[1]; // The name of the configuration file is Arg 1
	std::string propsFile  = argv[2]; // The name of the properties file is Arg 2
	
	boost::mpi::environment env(argc, argv);
	boost::mpi::communicator world;
	repast::RepastProcess::init(configFile);	
	RepastHPCDemoModel* model = new RepastHPCDemoModel(propsFile, argc, argv, &world);
	repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();	
	model->init();
	model->initSchedule(runner);	
	runner.run();	
	delete model;	
	repast::RepastProcess::instance()->done();	
}
```
将'argc'和'argv'参数传递给Properties构造函数，则在命令行上使用的参数将被传递并解析。任何形式为“属性=值”的命令行参数都将添加到属性文件中，命令行如下：
```
mpirun -n 4 Demo_00.exe config.props model.props stop.at=6
```
## Step_14： 记录每次模拟的属性
properties提供了一种将每次模拟运行的参数记录的方式，代码如下：
```c++ {highlight=[4]}
RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm){
	repast::Properties props(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props.getProperty("stop.at"));
	if(repast::RepastProcess::instance()->rank() == 0) props.writeToSVFile("record.csv");
}
```
运行命令行:
```
mpirun -n 4 Demo_00.exe config.props model.props RunNumber=1 stop.at=6
```
添加RunNumber作为标识并更改每次stop.at值并记录。

## Step_15：事件结束时记录属性
我们将安排一个事件在模拟结束时运行，该事件将利用属性文件，并将模拟结果写入输出文件。 这不是从RepastHPC模拟中获取输出的唯一方法，甚至不是首选方法，但是它是有效且明确的，代码如下：
```c++ {highlight=[2,10,13,17-19,27,30-39]}
#include <stdio.h>
#include <vector>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

class RepastHPCDemoModel{
	int stopAt;
	repast::Properties* props;
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm){
		props = new repast::Properties(propsFile, argc, argv, comm);
		stopAt = repast::strToInt(props->getProperty("stop.at"));
		if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("record.csv");
	}
	~RepastHPCDemoModel(){
		delete props;
	}
	void init(){}
	void doSomething(){
		std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;
	}
	void initSchedule(repast::ScheduleRunner& runner){
		runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
		//事件结束时，调用recordResults
		runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
		runner.scheduleStop(stopAt);
	}
	void recordResults(){ //将属性写入文件
		if(repast::RepastProcess::instance()->rank() == 0){
			props->putProperty("Result","Passed");
			std::vector<std::string> keyOrder;
			keyOrder.push_back("RunNumber");
			keyOrder.push_back("stop.at");
			keyOrder.push_back("Result");
			props->writeToSVFile("results.csv", keyOrder);
	    }
    }
};
```

# Demo_01: 多进程间agents交互（不包含网络和空间）
不同于Demo_00只有一个cpp，在这里我们将头文件、源文件和属性文件分开。
## Step_00: 目录和文件
头文件将在名为“ include”的文件夹中； 源文件（.cpp）将位于名为“ src”的文件夹中； 编译结果将放置在名为“ object”的文件夹中； 链接的结果将放置在名为“ bin”的文件夹中； 属性文件将保存在名为“ props”的文件夹中。

模型代码将分为三个单独的组件。 “ Demo_01_Main.cpp”文件将充当模拟的入口点（int main（argc，argv）函数）。 第二个组件将是“model”代码，现在位于两个单独的文件中（Demo_01_Model.h和Demo_01_Model.cpp）。 第三个组件将是Demo_01_Agent.h和Demo_01_Agent.cpp中的“agent”代码。

makefile的重要性应该开始更加清楚：每个.cpp文件都必须分别编译，然后将它们链接在一起，所有这些操作都将通过基本的“ make”指令自动完成。

在下一个步骤中，我们才会创建agent，因此这里的Demo_01_Agent.h文件只有防卫性声明，
```c++
/* Demo_01_Agent.h */

#ifndef DEMO_01_AGENT
#define DEMO_01_AGENT


#endif
```
Demo_01_Agent.cpp中没有内容。
```c++
/* Demo_01_Agent.cpp */
```
Demo_01_Model.h内容如下：
```c++
/* Demo_01_Model.h */

#ifndef DEMO_01_MODEL
#define DEMO_01_MODEL

#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"

class RepastHPCDemoModel{
	int stopAt;
	repast::Properties* props;
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};

#endif

```
Demo_01_Model.cpp内容如下：
```c++
/* Demo_01_Model.cpp */

#include <stdio.h>
#include <vector>
#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

#include "Demo_01_Model.h"

RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
}

RepastHPCDemoModel::~RepastHPCDemoModel(){
		delete props;
}

void RepastHPCDemoModel::init(){}

void RepastHPCDemoModel::doSomething(){
	std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;

}

void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
}

void RepastHPCDemoModel::recordResults(){
	if(repast::RepastProcess::instance()->rank() == 0){
		props->putProperty("Result","Passed");
		std::vector<std::string> keyOrder;
		keyOrder.push_back("RunNumber");
		keyOrder.push_back("stop.at");
		keyOrder.push_back("Result");
		props->writeToSVFile("./output/results.csv", keyOrder);
    }
}
```
Demo_01_Main.cpp内容如下：
```c++
/* Demo_01_Main.cpp */

#include <boost/mpi.hpp>
#include "repast_hpc/RepastProcess.h"

#include "Demo_01_Model.h"


int main(int argc, char** argv){
	
	std::string configFile = argv[1]; // The name of the configuration file is Arg 1
	std::string propsFile  = argv[2]; // The name of the properties file is Arg 2
	
	boost::mpi::environment env(argc, argv);
	boost::mpi::communicator world;

	repast::RepastProcess::init(configFile);
	
	RepastHPCDemoModel* model = new RepastHPCDemoModel(propsFile, argc, argv, &world);
	repast::ScheduleRunner& runner = repast::RepastProcess::instance()->getScheduleRunner();
	
	model->init();
	model->initSchedule(runner);
	
	runner.run();
	
	delete model;
	
	repast::RepastProcess::instance()->done();
	
}
```
运行命令为：
```
mpirun -n 4 ./bin/Demo_01.exe ./props/config.props ./props/model.props RunNumber=1 stop.at=5
```

## Step_01: agents
agents是AMB(agent based model)的基本组成，在这里我们将构建一个简单的agent。
我们用一个囚徒困境游戏示例来演示：
每个agnet可以问另一个agent：“你愿意和我合作吗？” 询问时，所有agent也可以回答该问题，答案将是“是”或“否”。

但是，进行询问的agent有一个秘密动机：它还计划与另一个agent合作或试图“欺骗”另一个agent。 在交互结束时，进行询问的agent会根据两个agent是否合作，两个agent拒绝合作（“违背”），还是一个叛逃而一个合作而获得回报。收益是：
* 如果我与其他agent合作，我将得到“ 7”
* 如果我合作并且其他agent叛逃，我将得到“ 1”
* 如果我叛逃而其他agent配合，我得到“ 10”
* 如果我叛逃且其他agent叛逃，我得到“ 3”

设置cPayoff累计agent合作时的收益，totalPayoff累计每一轮的收益（只记录发起询问的agent）。

我们开始设置agent:
Demo_01_Agent.h代码如下：
```c++
/* Demo_01_Agent.h */

#ifndef DEMO_01_AGENT
#define DEMO_01_AGENT

#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"


/* Agents */
class RepastHPCDemoAgent{
	
private:
    repast::AgentId   id_;
    double              c;
    double          total;
	
public:
    RepastHPCDemoAgent(repast::AgentId id);
	
    RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal);
	
    ~RepastHPCDemoAgent();
	
    /* Required Getters */
    virtual repast::AgentId& getId(){                   return id_;    }
    virtual const repast::AgentId& getId() const {      return id_;    }
	
    /* Getters specific to this kind of Agent */
    double getC(){                                      return c;      }
    double getTotal(){                                  return total;  }
	
    /* Setter */
    void set(int currentRank, double newC, double newTotal);
	
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context);    // Choose three other agents from the given context and see if they cooperate or not
	
};

#endif
```
其中，引入了很多新元素:
* 第一个是“ AgentId”,Repast HPC使用唯一标识符来指定每个agent,标识符分为四个部分，全部为数字。[4、12、3、5]的AgentId表示该agent是在进程12上创建的，类型为“ 3”，是在该进程上创建的第5个此类agent（数字以0开头，因此agent“ 4”为 第五个），现在正在进程5上。编号，启动进程和类型的三个标识符在整个模拟中唯一地标识一个agent。
本例中，所有agent都必须实现基于AgentId的两个“getter”；通常，这是通过在agent的构造函数中为其提供ID来实现的，此ID记录为实例变量，如此处所述。
* 第二个是“ SharedContext”，这是在模拟运行期间agents将存在的集合。

Demo_01_Agent.cpp代码为：
```c++
/* Demo_01_Agent.cpp */

#include "Demo_01_Agent.h"

RepastHPCDemoAgent::RepastHPCDemoAgent(repast::AgentId id): id_(id), c(100), total(200){ }

RepastHPCDemoAgent::RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal): id_(id), c(newC), total(newTotal){ }

RepastHPCDemoAgent::~RepastHPCDemoAgent(){ }


void RepastHPCDemoAgent::set(int currentRank, double newC, double newTotal){
    id_.currentRank(currentRank);
    c     = newC;
    total = newTotal;
}

bool RepastHPCDemoAgent::cooperate(){
	return true;
}

void RepastHPCDemoAgent::play(repast::SharedContext<RepastHPCDemoAgent>* context){

}

```
如果未提供c和total的值，则构造函数将使用默认值，并且agent的两个关键函数（cooperate（）和play（））暂时留空，我们稍后将详细介绍它们。

**该代码应像先前的演示一样进行编译和运行，但是即使我们已经编译了“ agent”类，也没有任何东西可以将它们连接到“ main”或“ model”类，我们将在下一步中解决。**

## Step_02: context is everything
根据上一节 "context is the collection in which the agent will exist during the simulation runs."
我们通过提供SharedContext来实现，用model存储agents。
Demo_01_Model.h代码如下：
```c++ {highlight=[9,11,16]}
/* Demo_01_Model.h */

#ifndef DEMO_01_MODEL
#define DEMO_01_MODEL

#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"

#include "Demo_01_Agent.h"

class RepastHPCDemoModel{
	int stopAt;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};

#endif
```
需要注意，SharedContext是模板化类，需使用尖括号告诉它它将包含的类-在这种情况下为'RepastHPCDemoAgent'。

## Step_03：初始化和创建agents
现在有了放置agents的位置，我们可以在model中创建它们。这（最终）使我们有机会使用model类的“ init”方法。 Demo_01_Model.h调整后的代码如下：
```c++{highlight=[15]}
/* Demo_01_Model.h */

#ifndef DEMO_01_MODEL
#define DEMO_01_MODEL

#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"

#include "Demo_01_Agent.h"

class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};

#endif
```
Demo_01_Model.cpp代码如下：
```c++ {highlight=[6,16,25-31]}
/* Demo_01_Model.cpp */

#include <stdio.h>
#include <vector>
#include <boost/mpi.hpp>
#include "repast_hpc/AgentId.h"
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"

#include "Demo_01_Model.h"

RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
}

RepastHPCDemoModel::~RepastHPCDemoModel(){
		delete props;
}

void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
		repast::AgentId id(i, rank, 0); //创建agent：先创建agentID，对应rank,类型为0
		id.currentRank(rank);
		RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
		context.addAgent(agent); //将agent添加至context
	}
}

void RepastHPCDemoModel::doSomething(){
	std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something." << std::endl;

}

void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
}

void RepastHPCDemoModel::recordResults(){
	if(repast::RepastProcess::instance()->rank() == 0){
		props->putProperty("Result","Passed");
		std::vector<std::string> keyOrder;
		keyOrder.push_back("RunNumber");
		keyOrder.push_back("stop.at");
		keyOrder.push_back("Result");
		props->writeToSVFile("./output/results.csv", keyOrder);
    }
}
```
model.props文件改变如下：
```
#Properties file
stop.at = 2
count.of.agents = 100
```

## Step_04: 随机性和可复现性
 一方面进行模拟时，其中会产生随机性，并且结果是随机的；另一方面，应该可以完全重新运行给定的模拟，复现之前的操作。

通常，模拟结合随机性的方式是通过随机数发生器，一个关键特性是，可以使用可以重复使用的随机数“seed”来初始化它们，如果使用与以前相同的数字序列产生重复的结果。
Demo_01_Model.cpp代码修改为：
```c++ {highlight=[10,18,37]}
/* Demo_01_Model.cpp */

#include <stdio.h>
#include <vector>
#include <boost/mpi.hpp>
#include "repast_hpc/AgentId.h"
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/initialize_random.h"

#include "Demo_01_Model.h"

RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
}

RepastHPCDemoModel::~RepastHPCDemoModel(){
		delete props;
}

void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
		repast::AgentId id(i, rank, 0);
		id.currentRank(rank);
		RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
		context.addAgent(agent);
	}
}

void RepastHPCDemoModel::doSomething(){
	std::cout << "Rank " << repast::RepastProcess::instance()->rank() << " is doing something: " << repast::Random::instance()->nextDouble() << std::endl;

}

void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
}

void RepastHPCDemoModel::recordResults(){
	if(repast::RepastProcess::instance()->rank() == 0){
		props->putProperty("Result","Passed");
		std::vector<std::string> keyOrder;
		keyOrder.push_back("RunNumber");
		keyOrder.push_back("stop.at");
		keyOrder.push_back("Result");
		props->writeToSVFile("./output/results.csv", keyOrder);
    }
}
```
如果在不提供random.seed属性的情况下运行代码，Repast HPC将使用系统时间来生成随机数种子。没有保留任何记录，因此运行不可重复。重要的是，每个进程都独立执行此操作：如果所有进程都紧密同步(一般不会)，则不同的进程最终可能会获得相同的随机数种子。通常来说，我们需要每个进程有不一样的数字序列。

因此，我们最好自己输入一个random.seed，从技术上讲，Repast HPC会使用提供的random.seed种子，并在进程0上生成用作所有进程种子的随机数；这就是为什么“ initializeRandom”功能需要MPI通信器的原因：进程0将新的种子发送到所有其他进程。

但是，在某些情况下，我们希望所有进程都具有相同的随机数种子。对于这些情况，传递一个名为“ global.random.seed”的属性将允许所有进程使用相同的种子，且结果可复现。

命令行：
```
mpirun -n 4 ./bin/Demo_01.exe ./props/config.props ./props/model.props RunNumber=1 stop.at=5 random.seed=1
```

## Step_05: agent scheduling
让agents做点什么,其中心问题是agents将以什么顺序和时间采取行动。在ABM中，这通常称为“schedule”，有许多策略可以使用，它们可能对模型的行为产生重要影响。我们先忽略复杂性，选择一个案例来说明。

我们将根据一个简单的模式进行安排：在每个时间步中，将对agents列表进行混洗打乱(shuffled)，然后依次要求每个agent与从总体中随机选择的其他三个agent进行“cooporate”游戏。

要了解调度的复杂性，请想象一下第一轮。所有agent的分数相同，因为它们均已初始化为默认值。第一个agent开始与三个随机对手（都具有相同的默认值）进行游戏，完成后将对其分数进行调整。如果，第二个agent随机选择三个agent，并碰巧选择第一个agent。而第一个agent的分数与其他agent不同，因为它已经玩了游戏。在这里也许次序无关紧要，但有时先后次序会成为一个问题。因此，为了“公平”，所有agent都会参加比赛，然后才更新他们的总分。在这些情况下，可以打乱agent；在我们的案例中，我们每轮打乱顺序，以尝试“平衡”由异步更新引起的不均匀性。

Demo_01_Model.h的doSomething代码如下：

```c++{highlight=[2-9,3]}
void RepastHPCDemoModel::doSomething(){
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
		std::cout << (*it)->getId() << " ";
		(*it)->play(&context);
		it++;
    }
	std::cout << std::endl;
}
```
在这段代买中，第三行对agent的选择是最重要的，selectAgents可以从context中选择agent的任何子集，并可用于以任意顺序或数学随机顺序返回那些agent。在这里调用它可确保所有agent都被选择（因为'countOfAgents'是agent的总数）并且返回的顺序是随机的（因为将agent指针放入向量中，这是一个有序列表）。

```
mpirun -n 4 ./bin/Demo_01.exe ./props/config.props ./props/model.props stop.at=4 random.seed=2 count.of.agents=5
```

## Step_06: agent acting(行动)
接下来，我们赋予agent人行动的能力。 这是通过更改agent程序源文件（Demo_01_Agent.cpp），填充我们先前在compare（）和play（）方法中留下的空白来实现的：
```c++ {highlight=[2,6-26]}
bool RepastHPCDemoAgent::cooperate(){
	return repast::Random::instance()->nextDouble() < c/total;
}

void RepastHPCDemoAgent::play(repast::SharedContext<RepastHPCDemoAgent>* context){
    std::set<RepastHPCDemoAgent*> agentsToPlay;
	
    agentsToPlay.insert(this); // Prohibit playing against self
	
    context->selectAgents(3, agentsToPlay, true);
	
    double cPayoff     = 0;
    double totalPayoff = 0;
    std::set<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff;
        totalPayoff             += payoff;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
}
```
注意，cooperate中使用了Repast HPC的'random'类的实例，它将与随机数生成例程集成在一起，因此可以完全重现。还要注意使用“ selectAgents”来获得三个比赛对手，在这种情况下，使用的selectAgents方法的变体意味着：agent以集合形式返回（顺序无关紧要）；该集合最初包含“自我”，这意味着将不会选择“自我”（agent不会与自己对战）； 并且“自我”将从返回集中删除。

## Step_07: Agent Packages
使用Repast HPC的基础是“Agent Packages”的概念。 当agent信息从一个进程转移到另一个进程时，该信息必须打包运输。这可以称为'serializing' or 'archiving'（“序列化”或“归档”），但是关键是必须提供构建原始agent的新副本所需的所有信息。

Repast HPC使用Boost归档archiving框架来实现此目的，代码已添加到Demo_01_Agent.h和Demo_01_Agent.cpp文件中，如下所示：
**Demo_01_Agent.h**
``` c++{highlight=[39-64]}
/* Demo_01_Agent.h */

#ifndef DEMO_01_AGENT
#define DEMO_01_AGENT

#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"

/* Agents */
class RepastHPCDemoAgent{
	
private:
    repast::AgentId   id_;
    double              c;
    double          total;
	
public:
    RepastHPCDemoAgent(repast::AgentId id);	
    RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal);	
    ~RepastHPCDemoAgent();
	
    /* Required Getters */
    virtual repast::AgentId& getId(){                   return id_;    }
    virtual const repast::AgentId& getId() const {      return id_;    }
	
    /* Getters specific to this kind of Agent */
    double getC(){                                      return c;      }
    double getTotal(){                                  return total;  }
	
    /* Setter */
    void set(int currentRank, double newC, double newTotal);
	
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context);    // Choose three other agents from the given context and see if they cooperate or not
	
};

/* Serializable Agent Package */
struct RepastHPCDemoAgentPackage {
	
public:
    int    id;
    int    rank;
    int    type;
    int    currentRank;
    double c;
    double total;
	
    /* Constructors */
    RepastHPCDemoAgentPackage(); // For serialization
    RepastHPCDemoAgentPackage(int _id, int _rank, int _type, int _currentRank, double _c, double _total);
	
    /* For archive packaging */
    template<class Archive>
    void serialize(Archive &ar, const unsigned int version){
        ar & id;
        ar & rank;
        ar & type;
        ar & currentRank;
        ar & c;
        ar & total;
    }	
};
#endif
```
**Demo_01_Agent.cpp**
```c++{highlight=[41-46]}
/* Demo_01_Agent.cpp */
#include "Demo_01_Agent.h"
RepastHPCDemoAgent::RepastHPCDemoAgent(repast::AgentId id): id_(id), c(100), total(200){ }
RepastHPCDemoAgent::RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal): id_(id), c(newC), total(newTotal){ }
RepastHPCDemoAgent::~RepastHPCDemoAgent(){ }

void RepastHPCDemoAgent::set(int currentRank, double newC, double newTotal){
    id_.currentRank(currentRank);
    c     = newC;
    total = newTotal;
}

bool RepastHPCDemoAgent::cooperate(){
	return repast::Random::instance()->nextDouble() < c/total;
}

void RepastHPCDemoAgent::play(repast::SharedContext<RepastHPCDemoAgent>* context){
    std::set<RepastHPCDemoAgent*> agentsToPlay;
	
    agentsToPlay.insert(this); // Prohibit playing against self
	
    context->selectAgents(3, agentsToPlay, true);
	
    double cPayoff     = 0;
    double totalPayoff = 0;
    std::set<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff;
        totalPayoff             += payoff;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
	
}
/* Serializable Agent Package Data */

RepastHPCDemoAgentPackage::RepastHPCDemoAgentPackage(){ }

RepastHPCDemoAgentPackage::RepastHPCDemoAgentPackage(int _id, int _rank, int _type, int _currentRank, double _c, double _total):
id(_id), rank(_rank), type(_type), currentRank(_currentRank), c(_c), total(_total){ }
```
## Step_08: 发送Agent Packages
要将一个Agent Packages从一个进程发送到另一个进程，那么我们需要发送者和接收者角色。从概念上讲，这意味着创建一个或多个提供四个必需功能的类（假设Agent代表正在交换的Agent的类）：
* void providePackage(Agent * agent, std::vector& out);
* void provideContent(repast::AgentRequest req, std::vector& out);
* Agent * createAgent(AgentPackage package);
* void updateAgent(AgentPackage package);

前两个是用于“发送”。 
providePackage功能采用指向agent的指针，在context中找到该agent，创建一个agent Package，然后将该Package添加到agent Package的向量中。
provideContent与第一个非常相似，只是它被赋予了“ AgentRequest”。AgentRequest将包含一个AgentId的集合，并且假定代码将遍历该集合并检索被列出的每个agent的package（可能通过调用“ providePackage”函数实现）。

最后两个函数用于“接​​收”。
createAgent函数必须获取一个agent 'package' 并从中创建一个agent，并返回指向它的指针。当一个agent从一个进程发送到另一个进程时，将使用此方法。agent会在原始进程中销毁，并在新进程中重新创建。
当agent的副本已经在接收进程中时，将使用“ updateAgent”函数，并且仅需要更新从发送进程中接受的信息即可。

任何提供这些功能类都可以被使用。有时模型类采用一种变体来提供这些函数，但是，对于此演示，我们将创建具有唯一函数的类，充当“发送者”和“接收者”角色。
我们将使用的代码已添加到Model.h和Model.cpp文件中：
**Model.h**
```c++{highlight=[10,13-35,43-44]}
/* Demo_01_Model.h */

#ifndef DEMO_01_MODEL
#define DEMO_01_MODEL

#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/AgentRequest.h"
#include "Demo_01_Agent.h"

/* Agent Package Provider */
class RepastHPCDemoAgentPackageProvider {
	
private:
    repast::SharedContext<RepastHPCDemoAgent>* agents;	
public:	
    RepastHPCDemoAgentPackageProvider(repast::SharedContext<RepastHPCDemoAgent>* agentPtr);	
    void providePackage(RepastHPCDemoAgent * agent, std::vector<RepastHPCDemoAgentPackage>& out);	
    void provideContent(repast::AgentRequest req, std::vector<RepastHPCDemoAgentPackage>& out);	
};

/* Agent Package Receiver */
class RepastHPCDemoAgentPackageReceiver {
	
private:
    repast::SharedContext<RepastHPCDemoAgent>* agents;
	
public:
	
    RepastHPCDemoAgentPackageReceiver(repast::SharedContext<RepastHPCDemoAgent>* agentPtr);	
    RepastHPCDemoAgent * createAgent(RepastHPCDemoAgentPackage package);	
    void updateAgent(RepastHPCDemoAgentPackage package);	
};

class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;
	
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};

#endif
```
**Model.cpp**
```c++ {highlight=[14-40,49,48,54,55]}
/* Demo_01_Model.cpp */

#include <stdio.h>
#include <vector>
#include <boost/mpi.hpp>
#include "repast_hpc/AgentId.h"
#include "repast_hpc/RepastProcess.h"
#include "repast_hpc/Utilities.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/initialize_random.h"

#include "Demo_01_Model.h"

RepastHPCDemoAgentPackageProvider::RepastHPCDemoAgentPackageProvider(repast::SharedContext<RepastHPCDemoAgent>* agentPtr): agents(agentPtr){ }

void RepastHPCDemoAgentPackageProvider::providePackage(RepastHPCDemoAgent * agent, std::vector<RepastHPCDemoAgentPackage>& out){
    repast::AgentId id = agent->getId();
    RepastHPCDemoAgentPackage package(id.id(), id.startingRank(), id.agentType(), id.currentRank(), agent->getC(), agent->getTotal());
    out.push_back(package);
}

void RepastHPCDemoAgentPackageProvider::provideContent(repast::AgentRequest req, std::vector<RepastHPCDemoAgentPackage>& out){
    std::vector<repast::AgentId> ids = req.requestedAgents();
    for(size_t i = 0; i < ids.size(); i++){
        providePackage(agents->getAgent(ids[i]), out);
    }
}

RepastHPCDemoAgentPackageReceiver::RepastHPCDemoAgentPackageReceiver(repast::SharedContext<RepastHPCDemoAgent>* agentPtr): agents(agentPtr){}

RepastHPCDemoAgent * RepastHPCDemoAgentPackageReceiver::createAgent(RepastHPCDemoAgentPackage package){
    repast::AgentId id(package.id, package.rank, package.type, package.currentRank);
    return new RepastHPCDemoAgent(id, package.c, package.total);
}

void RepastHPCDemoAgentPackageReceiver::updateAgent(RepastHPCDemoAgentPackage package){
    repast::AgentId id(package.id, package.rank, package.type);
    RepastHPCDemoAgent * agent = agents->getAgent(id);
    agent->set(package.currentRank, package.c, package.total);
}

RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
}

RepastHPCDemoModel::~RepastHPCDemoModel(){
	delete props;
	delete provider;
	delete receiver;
}
void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
		repast::AgentId id(i, rank, 0);
		id.currentRank(rank);
		RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
		context.addAgent(agent);
	}
}

void RepastHPCDemoModel::doSomething(){
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
		(*it)->play(&context);
		it++;
    }
}

void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
}

void RepastHPCDemoModel::recordResults(){
	if(repast::RepastProcess::instance()->rank() == 0){
		props->putProperty("Result","Passed");
		std::vector<std::string> keyOrder;
		keyOrder.push_back("RunNumber");
		keyOrder.push_back("stop.at");
		keyOrder.push_back("Result");
		props->writeToSVFile("./output/results.csv", keyOrder);
    }
}
```
请注意'providePackage'和'provideContent'函数可以协同工作：第一个简单地获取指向agent的指针并制作一个package，第二个获取一个agent列表（来自“ agent request”对象） ，获取指向它们的指针，并使用'providePackage'生成并打包要发送的内容。

还需注意，“ create agent”方法为该agent创建一个新的AgentId实例，匹配其原始ID（“current rank”变量除外，该变量将自动重置为当前rank）。 如果在“ updateAgent”函数中找不到指定的agent，则程序将由于空指针异常而崩溃（我们将在此处省略错误陷阱；永远不会发生。）

最后一点是按顺序进行的：AgentPackage类中的'serialize（）'方法不需要显式调用。 Repast HPC将在后台处理agent的转移，因此普通用户代码将不需要直接调用这些方法中的任何一种--它们被称为Repast HPC内部并行化管理过程的一部分。 当将AgentPackage添加到从一个进程到另一个进程发送的数据集合时，Boost库实际上会调用'serialize（）'方法。我们只需要提供方法即可,调用是自动完成的。
 ## Step 09: 跨进程共享Agent

 Repast HPC旨在实现跨进程共享agent。这是处理ABM并行化时出现的问题的一种方法：跨agent边界的agent交互受到限制。解决方案是将一个进程中实际存在的agent复制到需要它们的其他进程中。

其中一个关键方面是副本是被动的：可以从副本中读取信息，但是对副本所做的更改不会传输回原始文档。

正如我们将在以后的演示中看到的那样，spatial and network contexts会自动处理agent的借用。但是，首先，我们将手动启动和管理借用agent的任务。

为此，需要两个步骤：首先，必须创建一个请求。这基本上是要借用的其他进程上的agent的列表。其次，此请求必须发送给RepastProcess实例。

为此，我们将在演示模拟中创建“模型”类的新方法，并将其安排为模拟中的第一个事件。 （请注意，我们也可以在初始化过程中或在模拟执行过程中的其他时候借用agent；这里使用计划的事件很方便。）首先，将新函数添加到模型类中（在Demo_01_Model.h中）：
```c++{highlight=[14]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;
	
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void requestAgents();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};
```
然后，在Demo_01_Model.cpp文件中，修改schedule以在模拟的第1步（不重复）中调用此方法。 现在，“ doSomething”方法将延迟到步骤2：
```c++{highlight=[2]}
void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::requestAgents)));
	runner.scheduleEvent(2, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
}
```
在Demo_01_Model.cpp中，关键代码如下：
```c++{highlight=[1-20]}
void RepastHPCDemoModel::requestAgents(){
	int rank = repast::RepastProcess::instance()->rank();
	int worldSize= repast::RepastProcess::instance()->worldSize();
	repast::AgentRequest req(rank);
	for(int i = 0; i < worldSize; i++){                              // For each process
		if(i != rank){                                           // ... except this one
			std::vector<RepastHPCDemoAgent*> agents;        
			context.selectAgents(5, agents);                 // Choose 5 local agents randomly
			for(size_t j = 0; j < agents.size(); j++){
				repast::AgentId local = agents[j]->getId();          // Transform each local agent's id into a matching non-local one
				repast::AgentId other(local.id(), i, 0);
				other.currentRank(i);
				req.addRequest(other);                      // Add it to the agent request
			}
		}
	}
	std::cout << " BEFORE: RANK " << rank << " has " << context.size() << " agents." << std::endl;
	repast::RepastProcess::instance()->requestAgents<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, req, *provider, *receiver, *receiver);
	std::cout << " AFTER:  RANK " << rank << " has " << context.size() << " agents." << std::endl;
}
```
前两行仅检索有用的值-进程的rank和世界的大小（MPI进程总数）。

下一行创建一个AgentRequest对象，该对象使用当前的“rank”值初始化。

下一行开始按rank编号循环遍历所有进程，后面的rank不包括在内（无需从“self”借来agent）

接下来的几行创建agent指针的向量，并使用context的“ selectAgent”方法选择五个agent并检索指向它们的指针。

下一行开始通过这五个agent指针的循环。指向的每个agent都将具有一个“ AgentId”，其中包含四个整数：一个id“编号”，创建该agent的rank，该agent的“类型”以及当前该agent的rank。到目前为止，所有agent的类型均为0。

重要说明：程序员负责确保三个值'id, start rank, and type'组合以形成agent的全局唯一标识符。否则可能会导致程序崩溃（因为在Repast HPC内部在这些ID上对agent进行索引并期望它们是唯一的）。

此处的例程获取本地agent的ID值，并将其转换为与另一个进程上的agent的ID相匹配。例如，在进程'0'上，由selectAgents例程选择的第一个agent是Agent [4，0，0，0]-即在rank 0上创建的，type 0和当前rank 0的agent＃4。此ID将转换为代表进程1：[4、1、0、1]上的agent。我们可以保证该agent仅存在于进程1中，因为我们知道所有进程都在创建同样的一组agent并对其进行同等的编号。新的ID被添加到agent request中，并且循环继续进行，直到从其他每个进程请求5个agent为止。

最后三行是我们的主要重点。出于说明目的，第一个打印输出之前在context中的agent数量，最后一个打印输出之后的context中的agent数量。真正的动作在中间：
```c++
repast::RepastProcess::instance()->requestAgents<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, req, *provider, *receiver, *receiver);
```
该行调用“requestAgents”函数，该函数是RepastProcess实例的成员函数。该函数是模板化的：将在充当 'agent', 'package', 'provider', and 'receiver'角色的类中使用的类型必须在括号之间指定。传递的实际参数是： the context, the request, and the provider and receiver objects。

执行此行后，agent request将传输到RepastProcess实例； RepastProcess实例与其他进程进行通信，并将request通知它们，它还接收来自其他进程的request。然后，它发出所请求的agent信息，并接收信息以响应请求。请注意，在传输AgentPackage时，Boost库会调用AgentPackage的'serialize'方法。

运行命令行如下：
```
mpirun -n 4 ./bin/Demo_01.exe ./props/config.props ./props/model.props stop.at=4 random.seed=2 count.of.agents=10
```

# Demo_02：network（网络）
该演示介绍了Repast HPC中的一个新的基本概念：projection(投影)。 projection是一种构造agent关系的方式。在演示01中，我们在 'context' 中收集了agent，并允许他们通过随机选择进行交互。这只是构造agent关系，并确定哪个agent将与哪个其他agent交互的一种方式。

projection提供了一种更仔细地确定agent交互模式的方法。我们将看到有两种不同类型的projection：一种是spatial（空间）的，agent在其中移动，而它们之间的关系则是通过物理近似逻辑来构造的。另一个将是我们在此演示中重点关注：network（网络），在该网络中显式地建立从每个agent到任何其他agent的连接。

这里的演示将展示如何创建网络projection，如何创建和使用agent之间的链接，当agent从一个进程移动到另一个进程时链接的行为以及如何销毁链接。

## Step_00: 初始代码
我们的出发点是来源于Demo 01的代码，所有元素都仍然存在（包括数据收集），只是我们将暂时删除指示agent如何选择与之交互的其他agent的代码部分；我们还将假定schedule也需要修改，并且我们将返回请求agent的一般方式（从每个进程到另一个进程，如HPC：Demo_01,Step_09：跨过程共享agent）。 
对Agent.cpp修改如下：
```c++{highlight=[]}
void RepastHPCDemoAgent::play(repast::SharedContext<lRepastHPCDemoAgent>* context){
    std::set<lRepastHPCDemoAgent*> agentsToPlay;
	
//    agentsToPlay.insert(this); // Prohibit playing against self	<- With these lines removed, 'agentsToPlay' will just be empty
//    context->selectAgents(3, agentsToPlay, true);                         so nothing will happen in the loop below
	
    double cPayoff     = 0;
    double totalPayoff = 0;
    std::set<lRepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff;
        totalPayoff             += payoff;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
}	
```

## Step_01: 创建网络投影(network projection)
创建网络投影我们将使用“ SharedNetwork”中的投影--即一个知道它具有可以跨进程边界共享的链接的网络。
对model.h的文件修改如下：
```c++{highlight=[13]}
/* Demo_02_Model.h */

#ifndef DEMO_02_MODEL
#define DEMO_02_MODEL

#include 
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/AgentRequest.h"
#include "repast_hpc/TDataSource.h"
#include "repast_hpc/SVDataSet.h"
#include "repast_hpc/SharedNetwork.h"

#include "Demo_02_Agent.h"
```
并创建 edge content manager（边缘内容管理）的实例（负责包装边缘内容并在进程之间共享）和作为模型类的私有实例变量的指向网络的指针：
```c++{highlight=[10,13,14]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

        repast::RepastEdgeContentManager<RepastHPCDemoAgent> edgeContentManager;

	repast::SVDataSet* agentValues;
	repast::SharedNetwork<RepastHPCDemoAgent, repast::RepastEdge<RepastHPCDemoAgent>, repast::RepastEdgeContent<RepastHPCDemoAgent>, 
              repast::RepastEdgeContentManager<RepastHPCDemoAgent> >* agentNetwork;
	
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void requestAgents();
	void cancelAgentRequests();
	void removeLocalAgents();
	void moveAgents();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};
```
请注意，为了创建网络，我们必须在模板参数中指定：

* 将用于网络'node'（节点）的类；
* 将用于网络edges（边）的类；
* 用于网络edge content的类；
* 用于管理网络edge content的类；
在这种情况下，node将成为我们的agent；edges将是默认边（由repast提供），并必须告知该edges，它将连接特定类型的元素（因此，它本身就是模板化的）。edge content是指包含在edge中的数据-通常是源顶点和目标顶点，以及边缘可能包含的任何其他数据（例如，“权重”或其他值或一组值）。在这种情况下，边缘内容将是默认，内置于Repast Edge content（包含两个顶点和一个“权重”值），并将由默认的内置Repast Edge Content Manager进行管理。

现在，要创建网络，只需在模型构造函数中添加两行，即可创建网络实例，而将其添加到context中则需要一行：
在demo_02_model.cpp中：
```c++{highlight=[10-12]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
        agentNetwork = new repast::SharedNetwork<RepastHPCDemoAgent, repast::RepastEdge<RepastHPCDemoAgent>,
              repast::RepastEdgeContent<RepastHPCDemoAgent>, repast::RepastEdgeContentManager<RepastHPCDemoAgent> >("agentNetwork", false, &edgeContentManager);
	context.addProjection(agentNetwork);
	
	// Data collection
	// Create the data set builder
	std::string fileOutputName("./output/agent_total_data.csv");
	repast::SVDataSetBuilder builder(fileOutputName.c_str(), ",", repast::RepastProcess::instance()->getScheduleRunner().schedule());
	
	// Create the individual data sets to be added to the builder
	DataSource_AgentTotals* agentTotals_DataSource = new DataSource_AgentTotals(&context);
	builder.addDataSource(createSVDataSource("Total", agentTotals_DataSource, std::plus<int>()));

	DataSource_AgentCTotals* agentCTotals_DataSource = new DataSource_AgentCTotals(&context);
	builder.addDataSource(createSVDataSource("C", agentCTotals_DataSource, std::plus<int>()));

	// Use the builder to create the data set
	agentValues = builder.createDataSet();	
}
```
网络的构造函数中使用的参数有，首先是可用于引用网络的名称（请记住，context中可以有多个投影，因此每个投影都被分配了一个名称）。其次是一个 指示网络是有向（true）还是无向（false）的变量。无向网络意味着两个agent之间的连接是相互的：如果A连接到B，则B连接到A；有向网络允许连接仅在一个方向上进行，例如从A到B，但不从B到A。第三个参数是内容管理器实例（content manager instance）。

最后还有一个非常重要的步骤。 您必须将以下行添加到DemoModel.cpp文件的中，在include之后但在其余代码之前：
```c++{highlight=[6]}
#include "repast_hpc/initialize_random.h"
#include "repast_hpc/SVDataSetBuilder.h"

#include "Demo_02_Model.h"

BOOST_CLASS_EXPORT_GUID(repast::SpecializedProjectionInfoPacket<repast::RepastEdgeContent<RepastHPCDemoAgent> >, "SpecializedProjectionInfoPacket_EDGE");

RepastHPCDemoAgentPackageProvider::RepastHPCDemoAgentPackageProvider(repast::SharedContext* agentPtr): agents(agentPtr){ }
```
原因是为了技术上的方便：Repast HPC使用Boost库，而Boost库提供了一些非常方便的功能，这些功能与跨进程收发信息有关。 此行告诉Boost，将来回发送一个带有特定类型的边内容的类，在本例中为带有RepastHPCDemoAgents的默认RepastEdge。 这样，Boost会执行一些幕后工作，以确保它知道如何打包和发送此内容，而无需进行太多其他准备工作。 但是，如果省略此行，结果将是Boost尝试发送无法识别其结构的信息，并且会发生分段错误。

## Step_02: 构建网络链接
context包含网络投影，将agent加入到context中时，也就添加到了投影中，因此所有的agent都在网络中，但不一定有链接。在此步骤中，我们将在刚刚创建的网络中将agent连接在一起。这将在agent“借用”之后发生，因此网络中将存在一些本地和非本地agent。连接agent的算法为：
* 遍历本地agent
* 从同一进程中的其他agent中随机选择五个agent（包括本地和非本地agent）
* 与五个选定agent中的每个agent建立连接

连接将是无向的，因此一个agent最终可以拥有5个以上的连接（因为它将具有它所建立的5个连接，以及任何其他agent程序所建立的连接。）

我们分三个步骤来完成网络的构建：
**1. 在model.h中声明一个新的函数：**
```c++ {highlight=[18]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

	repast::SVDataSet* agentValues;
	repast::SharedNetwork<RepastHPCDemoAgent, repast::RepastEdge<RepastHPCDemoAgent> >* agentNetwork;
	
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void requestAgents();
	void connectAgentNetwork();
	void cancelAgentRequests();
	void removeLocalAgents();
	void moveAgents();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};
```
**2.在model.cpp中定义该函数**
```c++ {highlight=[1-16]}
void RepastHPCDemoModel::connectAgentNetwork(){
	repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iter    = context.localBegin();
	repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iterEnd = context.localEnd();
	while(iter != iterEnd) {
		RepastHPCDemoAgent* ego = &**iter;
		std::vector<RepastHPCDemoAgent*> agents;
		agents.push_back(ego);                          // Omit self
		context.selectAgents(5, agents, true);          // Choose 5 other agents randomly
		// Make an undirected connection
		for(size_t i = 0; i < agents.size(); i++){
         	    std::cout << "CONNECTING: " << ego->getId() << " to " << agents[i]->getId() << std::endl;
  	  	    agentNetwork->addEdge(ego, agents[i]);	
		}
		iter++;
	}	
}
```
注意：如果要添加到网络的边与现有边具有相同的源节点和目标节点（以相同的角色），则可能不会添加新的边缘；如果有可能添加重复的边，则应首先进行检查以确保没有重复，如果发现重复，则应首先将其删除。

**3.将此函数添加到schedule中**
```c++ {highlight=[3]}
void RepastHPCDemoModel::initSchedule(repast::ScheduleRunner& runner){
	runner.scheduleEvent(1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::requestAgents)));
	runner.scheduleEvent(1.1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::connectAgentNetwork)));
	runner.scheduleEvent(2, 1, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::doSomething)));
	runner.scheduleEvent(3, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::moveAgents)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel> (this, &RepastHPCDemoModel::recordResults)));
	runner.scheduleStop(stopAt);
	
	// Data collection
	runner.scheduleEvent(1.5, 5, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel>(agentValues, &repast::DataSet::record)));
	runner.scheduleEvent(10.6, 10, repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel>(agentValues, &repast::DataSet::write)));
	runner.scheduleEndEvent(repast::Schedule::FunctorPtr(new repast::MethodFunctor<RepastHPCDemoModel>(agentValues, &repast::DataSet::write)));
}
```

## Step_03: 使用网络链接

现在，agent已连接到网络中，接下来我们将允许它们使用网络。我们将使用允许agent使用与其进行连接的agent来替换随机选择竞争伙伴的方法。

主要更改将在agent的'play'方法中，因为我们现在不再从context中选择agent，而是从网络中选择agent，所以我们需要更改此方法的signature以将指针传递给网络。

首先Agent.h引入SharedNetwork
```c++ {highlight=[3]}
#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/SharedNetwork.h"
```

对agent.cpp中的play函数进行修改
```c++ {highlight=[1-6,10]}
void RepastHPCDemoAgent::play(repast::SharedNetwork<RepastHPCDemoAgent,
                              repast::RepastEdge<RepastHPCDemoAgent>,
                              repast::RepastEdgeContent<RepastHPCDemoAgent>,
                              repast::RepastEdgeContentManager<RepastHPCDemoAgent> > *network){
    std::vector<RepastHPCDemoAgent*> agentsToPlay;
    network->successors(this, agentsToPlay);

    double cPayoff     = 0;
    double totalPayoff = 0;
    std::vector<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff;
        totalPayoff             += payoff;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
	
}
```

请注意，要完成play的agent集合（agentsToPlay）现在是向量，而不是集合。对网络对象的“successors”方法的调用将获取与“自身”相连的agent。 （这假设一个无向网络；“successors”方法查找通过从自己到另一agent的有向向量连接的节点；“predecessors”方法找到通过从另一agent到自己的有向向量的连接节点。对于无向网络，“successors”和“predecessors”都返回相同的节点，即所有连接的节点。）

'Model.cpp'中的调用将被修改以调用新方法：
```c++ {highlight=[27]}
void RepastHPCDemoModel::doSomething(){
	int whichRank = 0;
	if(repast::RepastProcess::instance()->rank() == whichRank) std::cout << " TICK " << repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;

	if(repast::RepastProcess::instance()->rank() == whichRank){
		std::cout << "LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() == whichRank)) std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << std::endl;
			}
		}
		std::cout << "NON LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() != whichRank)) std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << std::endl;
			}
		}
	}	
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
		(*it)->play(agentNetwork);
		it++;
    }
	repast::RepastProcess::instance()->synchronizeAgentStates<RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(*provider, *receiver);
}
```
请注意，此示例的效率很低，因为它在每个时间步长为每个agent重新收集了相邻的网络节点。在此示例中，网络是静态的，因此，一次收集邻居agent列表，然后重新使用列表而不进行任何更改将更加有效。此处使用的代码更适合于随时间变化的动态网络。

## Step_04: 权重网络边

在前面的示例中，所有边都被认为是相等的。在此示例中，我们考虑边的属性，称为“边权重（Edge Weight）”。边权重是与边关联的值，加权图在广泛的领域中很有用，使用边权重计算网络结构的分析例程也很常见。

Repast HPC的边实现包括“权重”属性，对于我们的演示，我们仅在连接边时设置边权重。只需在RepastHPCDemoModel::connectAgentNetwork方法中的addEdge方法调用中简单地将权重作为附加参数添加即可完成此操作：
```c++ {highlight=[12]}
void RepastHPCDemoModel::connectAgentNetwork(){
	repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iter    = context.localBegin();
	repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iterEnd = context.localEnd();
	while(iter != iterEnd) {
		RepastHPCDemoAgent* ego = &**iter;
		std::vector<RepastHPCDemoAgent*> agents;
		agents.push_back(ego);                          // Omit self
		context.selectAgents(5, agents, true);          // Choose 5 other agents randomly
		// Make an undirected connection
		for(size_t i = 0; i < agents.size(); i++){
         	    std::cout << "CONNECTING: " << ego->getId() << " to " << agents[i]->getId() << std::endl;
  	  	    agentNetwork->addEdge(ego, agents[i], i + 1);	
		}
		iter++;
	}	
}
```
在上述代码中，我们只是将边权重设置为'i'的值恰好加上1（以避免为零），这意味着我们将获得一条权重为1的边，权重为2的边，依此类推……

要使用此值，我们只需要从网络中检索边并调用它的weight（），我们将在play（）中进行此操作：
```c++　{highlight=[12,17,18]}
void RepastHPCDemoAgent::play(repast::SharedNetwork<RepastHPCDemoAgent,
                              repast::RepastEdge<RepastHPCDemoAgent>,
                              repast::RepastEdgeContent<RepastHPCDemoAgent>,
                              repast::RepastEdgeContentManager<RepastHPCDemoAgent> > *network){
    std::vector<RepastHPCDemoAgent*> agentsToPlay;
    network->successors(this, agentsToPlay);

    double cPayoff     = 0;
    double totalPayoff = 0;
    std::vector<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        double edgeWeight = network->findEdge(this, *agentToPlay)->weight();
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff * edgeWeight * edgeWeight;
        totalPayoff             += payoff * edgeWeight * edgeWeight;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
}
```

## Step_05:自定义边

在前面的步骤中，我们使用了Repast HPC的内置默认边。这仅具有一个属性，即“边权重”。但是，对于某些模拟，你可能希望网络边具有多个属性。例如，你可能希望跟踪网络连接的期限，并在较旧的网络连接过期时将其丢弃，或者您可能希望跟踪使用连接的次数。为此，需要创建一个自定义Edge类。

在RepastHPC中创建自定义边需要几个步骤，因为必须提供扩展功能来替换RepastHPC对于其内置边缘类型不可见执行的一些操作。为此，必须创建三个类：
* 新的自定义边类；通常，这应该扩展默认的RepastEdge类
* 一个新的Custom Edge Content类，将用于跨进程发送Edge信息
* 一个自定义的边管理器（custom Edge Manager），可以打包边内容类的实例，发送有关它们的信息，并在收到此信息后构造边实例
* 与默认的边缘类实现一样，向Boost注册新的边类。

通常，可能选择使用它们自己的.h头文件创建这些类中，而.cpp文件将不会使用，因为该代码是模板代码。 有可能避免使用模板代码，但是使用代码模板会更加简单和典型。）为简单起见，我们将所有代码放在一个名为Demo_02_Network.h的文件中，并将其放置在include目录。

该文件包含所需的三个元素。 第一个是自定义边本身：
```c++ {highlight=[1-20]}
/* Custom Network Components */
template<typename V>
class DemoModelCustomEdge : public repast::RepastEdge<V>{
private:
    int confidence;
    
public:
    DemoModelCustomEdge(){}
    DemoModelCustomEdge(V* source, V* target) : repast::RepastEdge<V>(source, target) {}
    DemoModelCustomEdge(V* source, V* target, double weight) : repast::RepastEdge<V>(source, target, weight) {}
    DemoModelCustomEdge(V* source, V* target, double weight, int confidence) : repast::RepastEdge<V>(source, target, weight), confidence(confidence) {}
    
    DemoModelCustomEdge(boost::shared_ptr<V> source, boost::shared_ptr<V> target) : repast::RepastEdge<V>(source, target) {}
    DemoModelCustomEdge(boost::shared_ptr<V> source, boost::shared_ptr<V> target, double weight) : repast::RepastEdge<V>(source, target, weight) {}
    DemoModelCustomEdge(boost::shared_ptr<V> source, boost::shared_ptr<V> target, double weight, int confidence) : repast::RepastEdge<V>(source, target, weight), confidence(confidence) {}
    
    
    int getConfidence(){ return confidence; }
    void setConfidence(int con){ confidence = con; }    
};
```

这个新的自定义边类仅包含一个变量，我们将其任意命名为“ confidence”。它是一个int，具有基本的getter和setter（类代码的最后两行）。该类的大部分是七个不同的构造函数，首先需要空的构造函数，以便基础的Boost和mpi库将类序列化。仅包含“source”，“target”和“weight”变量的四个构造函数只是对RepastEdge的内置构造函数进行了扩展，其中两个使用标准C++指针，两个使用Boost“shared”指针。还有两个函数包括'confidence'变量，一个函数使用普通指针，另一个函数使用“shared”指针。

类本身被模板化以将任何对象作为顶点，这也反映了Repast的内置边。这意味着该类的名称不是DemoModelCustomEdge，而是DemoModelCustomEdge<ClassName>，这就是引用它的方式。

代码的下一部分定义了与此类相关的Edge Content。在这里，我们将再次扩展Repast的内置类：
```c++{highlight=[1-17]}
/* Custom Edge Content */
template<typename V>
struct DemoModelCustomEdgeContent : public repast::RepastEdgeContent<V>{
    
    friend class boost::serialization::access;
    
public:
    int confidence;
    DemoModelCustomEdgeContent(){}
    DemoModelCustomEdgeContent(DemoModelCustomEdge<V>* edge): repast::RepastEdgeContent<V>(edge), confidence(edge->getConfidence()){}
    
    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        repast::RepastEdgeContent<V>::serialize(ar, version);
        ar & confidence;
    }    
};
```

此部分代码扩展了RepastEdgeContent<V>类。它必须定义一个空的构造函数和一个采用edge类实例的构造函数；它还必须提供serialize，serialize将类打包到实际上跨进程发送的存档（Archive）中。在这种情况下，它使用的过程是调用父类的方法（它将打包source，target和weight属性），然后在子类中添加新的'confidence'属性。

第三部分代码如下：
```c++{highlight=[1-14]}
/* Custome Edge Content Manager */
template<typename V>
class DemoModelCustomEdgeContentManager {
public:
    DemoModelCustomEdgeContentManager(){}
    virtual ~DemoModelCustomEdgeContentManager(){}
    DemoModelCustomEdge<V>* createEdge(DemoModelCustomEdgeContent<V>& content, repast::Context<V>* context){
        return new DemoModelCustomEdge<V>(context->getAgent(content.source), context->getAgent(content.target), content.weight, content.confidence);
    }
    DemoModelCustomEdgeContent<V>* provideEdgeContent(DemoModelCustomEdge<V>* edge){
        return new DemoModelCustomEdgeContent<V>(edge);
    }
};
#endif
```
这部分代码（以标头中“ #ifdef”语句的结尾结尾）定义了自定义的Edge Content Manager。Edge Content Manager必须提供两个功能：“ createEdge”和“ provideEdgeContent”。其中createEdge取得一个Edge Content实例，并从中创建边；provideEdgeContent取得一个边并从中创建一个Edge Content实例。

创建这些类之后，我们必须更改Agent和Model代码以使用它们。

首先，我们必须在Model.h文件中include新的network文件：
```c++{highlight=[15]}
/* Demo_02_Model.h */

#ifndef DEMO_02_MODEL
#define DEMO_02_MODEL

#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/AgentRequest.h"
#include "repast_hpc/TDataSource.h"
#include "repast_hpc/SVDataSet.h"
#include "repast_hpc/SharedNetwork.h"

#include "Demo_02_Network.h"
#include "Demo_02_Agent.h"
```
接下来，我们必须将网络和边内容管理器的实例变量更改为新类型：
```c++{highlight=[10,13]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;
    
        DemoModelCustomEdgeContentManager<RepastHPCDemoAgent> edgeContentManager;

	repast::SVDataSet* agentValues;
	repast::SharedNetwork<RepastHPCDemoAgent, DemoModelCustomEdge<RepastHPCDemoAgent>, DemoModelCustomEdgeContent<RepastHPCDemoAgent>, DemoModelCustomEdgeContentManager<RepastHPCDemoAgent> >* agentNetwork;
```
注意，到了这一点，我们采用了创建的通用类，将其模板化为抽象顶点“ V”，并开始使用特定顶点RepastHPCDemoAgent实例化它们。内容管理器的实例化发生在类声明中，创建网络实例的定义发生在源（.cpp）文件中：
```c++{highlight=[10,11]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
        agentNetwork = new repast::SharedNetwork<RepastHPCDemoAgent, DemoModelCustomEdge<RepastHPCDemoAgent>, DemoModelCustomEdgeContent<RepastHPCDemoAgent>, 
            DemoModelCustomEdgeContentManager<RepastHPCDemoAgent> >("agentNetwork", false, &edgeContentManager);
	context.addProjection(agentNetwork);
	
	// Data collection
	// Create the data set builder
	std::string fileOutputName("./output/agent_total_data.csv");
	repast::SVDataSetBuilder builder(fileOutputName.c_str(), ",", repast::RepastProcess::instance()->getScheduleRunner().schedule());
	
	// Create the individual data sets to be added to the builder
	DataSource_AgentTotals* agentTotals_DataSource = new DataSource_AgentTotals(&context);
	builder.addDataSource(createSVDataSource("Total", agentTotals_DataSource, std::plus<int>()));

	DataSource_AgentCTotals* agentCTotals_DataSource = new DataSource_AgentCTotals(&context);
	builder.addDataSource(createSVDataSource("C", agentCTotals_DataSource, std::plus<int>()));

	// Use the builder to create the data set
	agentValues = builder.createDataSet();	
}
```
接下来，我们需要在connectAgentNetwork方法中更改创建网络时创建边的方式：
```c++{highlight=[13]}
void RepastHPCDemoModel::connectAgentNetwork(){
    repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iter    = context.localBegin();
    repast::SharedContext<RepastHPCDemoAgent>::const_local_iterator iterEnd = context.localEnd();
    while(iter != iterEnd) {
        RepastHPCDemoAgent* ego = &**iter;
        std::vector<RepastHPCDemoAgent*> agents;
        agents.push_back(ego);                          // Omit self
        context.selectAgents(5, agents, true);          // Choose 5 other agents randomly
        // Make an undirected connection
        for(size_t i = 0; i < agents.size(); i++){
            if(ego->getId().id() < agents[i]->getId().id()){
                 std::cout << "CONNECTING: " << ego->getId() << " to " << agents[i]->getId() << std::endl;
                 boost::shared_ptr<DemoModelCustomEdge<RepastHPCDemoAgent> > demoEdge(new DemoModelCustomEdge<RepastHPCDemoAgent>(ego, agents[i], i + 1, i * i));
                agentNetwork->addEdge(demoEdge);
            }
	}
	iter++;
    }
}
```

当调用agent的play方法时，该模型将该网络的实例传递给 doSomething网络中的agent，agent会收到并对此采取行动。因此，还必须更新Agent.h和Agent.cpp代码。

Agent.h必须包含新的网络头文件：
```c++{highlight=[9]}
/* Demo_02_Agent.h */

#ifndef DEMO_02_AGENT
#define DEMO_02_AGENT

#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/SharedNetwork.h"
#include "Demo_02_Network.h"
```
play的更改为：
```c++{highlight=[29-32]}
/* Agents */
class RepastHPCDemoAgent{
	
private:
    repast::AgentId   id_;
    double              c;
    double          total;
	
public:
    RepastHPCDemoAgent(repast::AgentId id);
	RepastHPCDemoAgent(){}
    RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal);
	
    ~RepastHPCDemoAgent();
	
    /* Required Getters */
    virtual repast::AgentId& getId(){                   return id_;    }
    virtual const repast::AgentId& getId() const {      return id_;    }
	
    /* Getters specific to this kind of Agent */
    double getC(){                                      return c;      }
    double getTotal(){                                  return total;  }
	
    /* Setter */
    void set(int currentRank, double newC, double newTotal);
	
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedNetwork<RepastHPCDemoAgent,
              DemoModelCustomEdge<RepastHPCDemoAgent>,
              DemoModelCustomEdgeContent<RepastHPCDemoAgent>,
              DemoModelCustomEdgeContentManager<RepastHPCDemoAgent> > *network);
	
};
```
现在我们就可以在play中检索和使用新的自定义的边的值：
```c++{highlight=[2-4,12,14,20,21]}
void RepastHPCDemoAgent::play(repast::SharedNetwork<RepastHPCDemoAgent,
                              DemoModelCustomEdge<RepastHPCDemoAgent>,
                              DemoModelCustomEdgeContent<RepastHPCDemoAgent>,
                              DemoModelCustomEdgeContentManager<RepastHPCDemoAgent> > *network){
    std::vector<RepastHPCDemoAgent*> agentsToPlay;
    network->successors(this, agentsToPlay);

    double cPayoff     = 0;
    double totalPayoff = 0;
    std::vector<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        boost::shared_ptr<DemoModelCustomEdge<RepastHPCDemoAgent> > edge = network->findEdge(this, *agentToPlay);
        double edgeWeight = edge->weight();
        int confidence = edge->getConfidence();
        
        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff * confidence * confidence * edgeWeight;
        totalPayoff             += payoff * confidence * confidence * edgeWeight;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;	
}
```
最后一步是替换Model.cpp文件顶部的'BOOST_CLASS_EXPORT_GUID'行，以便现在可以导出正确的类：
```c++{highlight=[1]} 
BOOST_CLASS_EXPORT_GUID(repast::SpecializedProjectionInfoPacket<DemoModelCustomEdgeContent<RepastHPCDemoAgent> >, "SpecializedProjectionInfoPacket_CUSTOM_EDGE");
```
请注意引号内的名称是任意的，因此不必更改，只要它对于所有BOOST_CLASS_EXPORT_GUID语句（我们只有一个）是唯一的即可。

# Demo_03 spatial(空间)
在Demo_02中，我们创建了一个网络投影(network projection)并使用网络连接显式地连接了agent。在本演示中，我们将使用另一种类型的投影：空间投影。

显而易见的是，空间投影在空间中定位agent。但是，仅当相对于空间中其他agent的位置考虑时，agent在空间中的位置才相关。因此，从根本上说，空间不是指定agent的位置，而是指定其与其他agent的关系。将空间中的位置视为关系可能是违反直觉的：我们可能将agent视为具有X，Y属性，而此属性是agent的性质。但是实际上，此属性仅具有作为推导其他agent的相对位置的方式的含义，因此，空间仅是构成agent之间关系的另一种方式，空间与网络非常相似。

当然，空间和网络在某些基本方面也有所不同。网络可以具有需要的任何一组关系，但是在一个空间中，只有有限数量的组合是可能的。例如，假设“空间”实际上是一维（一条线），并且在其上有5个元素，这些元素按顺序标记为A，B，C，D和E。标记是任意的，但是在一行中（假设您不能在同一位置拥有两个元素），这些元素必须按顺序排列，每个元素可以具有2个邻居：“ B”是与A和C相邻的邻居； “ D”是C和E的邻居等（如果线实际上是圆，则A和E可以被视为邻居）。如果我们考虑一个网络，我们可能使B与A，C和D连接。但是，在一维空间中，B可以恰好具有两个邻居。而且，这些邻居必须以B作为邻居。在网络中，任何抽象的连接集都是可能的；在空间上，这种连接关系受到约束。

然而，空间和网络都代表关系。这种潜在的相似性是在RepastHPC中将空间和网络一起处理的基础。在本教程的这一部分中，我们将介绍创建和使用空间投影的过程。但是，首先我们将解释RepastHPC如何使用空间以及如何并行化空间模型。

## Space in Repast HPC
RepastHPC中的空间在概念上是二维的,agent在这个二维空间中占据位置。在这一点上RepastHPC中，与大多数其他ABM仿真框架（包括Repast Simphony）相同。但是，当使用该空间并行化仿真时，RepastHPC的表现会大不相同。要了解如何进行操作，请考虑在非并行模拟中的单个统一网格：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/1.png)
我们可能会想象在此网格周围的各个位置都有agent：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/2.png)
为了提高性能，我们希望将仿真分为几部分。要为此使用空间，请想象我们可以在四个进程中运行模拟。我们使用空间划分模拟：每个进程负责一部分空间：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/3.png)
每个过程仅负责-并且“知道”-空间位于其边界之内的一部分。从概念上讲，每个进程仅“看到”其空间的一部分：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/4.png)
这允许并行化，因为每个进程现在仅对一小部分agent进行计算。但是，每个进程都知道其在全局空间中的位置：它知道全局边界，并且知道其空间局部区域的全局坐标。在图示示例中，每个空间中的坐标为：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/5.png)
注意，内角的坐标是共享的。 （注意：实际上，范围包括下限，但不包括上限。）

但是，RepastHPC需要一种方法来管理这些空间边界上正在发生的事情。从理论上讲，空间应该在各个过程之间统一，但是如果我们将空间分成多个部分，那么它真的统一吗？ 考虑红色和蓝色agent：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/6.png)

在统一空间中，它们仅相隔2个单元。但是，当它们处于单独的进程中时：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/7.png)

此处，RepastHPC应用了缓冲区的概念。这是一个由相邻进程管理的空间区域，该区域被复制到本地进程中，缓冲区使相邻进程中的agent可用于本地进程。 关于缓冲区的两个事必须牢记：

* 1.缓冲区的大小可以由用户选择，并由模拟的细节确定：缓冲区的大小应足够大，以容纳两个agent可能跨过程边界进行的任何（基于空间的）交互。例如，如果agent的“vision”范围为3个单位，则缓冲区至少应为3个单位。
* 2.从相邻进程复制的agent（因为它们位于缓冲区中）是非本地agent。 它们只是副本：对非本地agent的更改不会传回原始agent。

示例空间看起来像这样，缓冲区复制到了每个进程，缓冲区大小设置为3。
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/8.png)

尽管此处的示例仅使用四个进程，但是当进程数量较大时，该过程相同。同样，这里的示例使用了一个非环形空间。 在环形空间中，坐标“环绕”，以使最高坐标与最低坐标相邻，并且缓冲区也延伸到所有相邻的进程中。

在RepastHPC的当前实现中，空间始终在各个进程之间平均分配：每个进程的空间在x和y方向上的大小相同。从理论上讲，这不是必需的：每个进程可以负责大小不一的空间，当如果agent在空间中的分布不均匀时，这可能会很有用。如果agent的分布发生变化，也可以即时重新计算空间分配。但是，这些高级技术未在Repast HPC中实现。

## Boundaries
RepastHPC实现两种截然不同的边界条件，适当边界条件的选择取决于仿真：
* WrapAroundBorders
* StrictBorders

WrapAroundBorders也称为环形或周期性的（理论上讲，“周期periodic”边界会创建“环形toroidal”空间。）如果空间从0,0到100,100，且具有“ WrapAroundBorders”，则沿x轴以1单位间隔向上移动的agent将从x = 97传递到x = 98，x = 99，然后x = 0，传递回空间的另一端（从x = 0到x = 99反向移动是一样的）。环形空间在某种意义上是无限的：可以在不停止的情况下继续在一个方向上移动。同样重要的是，距离和查询的计算就好像根本没有边界一样，x = 99处的agent与x = 0处的agent相邻。

StrictBorders，假定空间是有限的，禁止越过指定的边界。 x = 99处的agent与x = 0处的agent相隔99个单位。缓冲区跨WrapAround边界传递，但不跨StrictBorders传递。

从理论上讲，可以一个轴使用一种边界创建空间，而另一个轴使用不同的边界来创建空间，例如：x轴使用“ WrapAround”，y轴为“ Strict”。但RepastHPC不允许这样做：边界是为整个空间指定的，并且适用于所有轴。

上面图片的显示了StrictBorders的情况；而使用WrapAround边界时，图片更加复杂，因为给定进程的所有边上都有缓冲区。在我们的简单演示中，只有四个进程，给定进程中沿一个轴的左右两个缓冲区实际上是同一进程；在更大的仿真中，每个轴上都有更多的过程。该图以中心处的进程1的视图说明了4个进程案例。请注意，进程2在四个角（NW, NE, SE, SW）西北，东北，东南，西南）：
![avatar](C:/Users/Administrator/Desktop/repast/markdown_picture/9.png)

## Step_00
我们返回与Demo_02相同的初始代码，也即Demo_01,Step_09中的代码。
## Step_01 创建空间投影（Creating the spatial projection）
因为空间投影在概念上与网络投影非常相似，并且由于它们是根据RepastHPC中的通用范例实现的，所以使用空间投影与使用网络投影基本上相同。第一步是为Model对象中的投影创建一个实例变量。 在Model.h文件中，这需要两个新的include语句和一个新变量：
```c++{highlight=[1,2]}
#include "repast_hpc/SharedDiscreteSpace.h"
#include "repast_hpc/GridComponents.h"
```
```c++{highlight=[11]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

	repast::SVDataSet* agentValues;
        repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace;
	
public:
	RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm);
	~RepastHPCDemoModel();
	void init();
	void requestAgents();
	void cancelAgentRequests();
	void removeLocalAgents();
	void moveAgents();
	void doSomething();
	void initSchedule(repast::ScheduleRunner& runner);
	void recordResults();
};
```
Model.cpp文件中的更改类似于网络投影所做的更改，添加一个新的include：
```c++{highlight=[]}
#include "repast_hpc/Point.h"
```
投影的实例在DemoModel构造函数中创建：
```c++ {highlight=[10-23]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
    repast::Point<double> origin(-100,-100);
    repast::Point<double> extent(200, 200);
    
    repast::GridDimensions gd(origin, extent);
    
    std::vector<int> processDims;
    processDims.push_back(2);
    processDims.push_back(2);
    
    discreteSpace = new repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >("AgentDiscreteSpace", gd, processDims, 2, comm);
	
    std::cout << "RANK " << repast::RepastProcess::instance()->rank() << " BOUNDS: " << discreteSpace->bounds().origin() << " " << discreteSpace->bounds().extents() << std::endl;
    
   	context.addProjection(discreteSpace);
    
	// Data collection
	// Create the data set builder
	std::string fileOutputName("./output/agent_total_data.csv");
	repast::SVDataSetBuilder builder(fileOutputName.c_str(), ",", repast::RepastProcess::instance()->getScheduleRunner().schedule());
	
	// Create the individual data sets to be added to the builder
	DataSource_AgentTotals* agentTotals_DataSource = new DataSource_AgentTotals(&context);
	builder.addDataSource(createSVDataSource("Total", agentTotals_DataSource, std::plus<int>()));

	DataSource_AgentCTotals* agentCTotals_DataSource = new DataSource_AgentCTotals(&context);
	builder.addDataSource(createSVDataSource("C", agentCTotals_DataSource, std::plus<int>()));

	// Use the builder to create the data set
	agentValues = builder.createDataSet();
}
```
新代码的前两行创建两个“ Point”对象，它们并不用作点，而是用于构造网格的简单数据结构。在这种情况下，网格的最低x和y坐标分别为-100，-100（ 'origin'值），它将在两个方向上分别延伸200个单位。

该空间构造的重要组成部分是倒数第二个参数，这里为'2'。这是缓冲区的宽度，请记住，缓冲区的大小必须谨慎选择，并且大小应足以超过这些agent之间的所有基于空间的交互。

我们还必须更改“ init”方法以将agent放置在空间中的位置，这取代了连接网络。
```c++{highlight=[4,9]}
void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
        repast::Point<int> initialLocation((int)discreteSpace->dimensions().origin().getX() + i,(int)discreteSpace->dimensions().origin().getY() + i);
		repast::AgentId id(i, rank, 0);
		id.currentRank(rank);
		RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
		context.addAgent(agent);
        discreteSpace->moveTo(id, initialLocation);
	}
}
```
第四行只是为agent选择一个位置，在这里，是非常随意的。第九行将agent移动到该位置，请注意，这不是可选的，放置在context中后，必须将agent显式移动到某个位置。

还必须更改'doSomething'：
```c++{highlight=[11-16,25-30]}
void RepastHPCDemoModel::doSomething(){
	int whichRank = 0;
	if(repast::RepastProcess::instance()->rank() == whichRank) std::cout << " TICK " << repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;

	if(repast::RepastProcess::instance()->rank() == whichRank){
		std::cout << "LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() == whichRank)){
                    std::vector<int> agentLoc;
                    discreteSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<int> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
		
		std::cout << "NON LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() != whichRank)){
                    std::vector<int> agentLoc;
                    discreteSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<int> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
	}	
	
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
		(*it)->play(&context);
		it++;
    }
	
    repast::RepastProcess::instance()->synchronizeProjectionInfo<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);
    
	repast::RepastProcess::instance()->synchronizeAgentStates<RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(*provider, *receiver); 
}
```

这些更改主要用于输出：它们说明了获取agent位置的方法：创建具有适当数据类型的向量，从空间请求位置以填充向量，然后（如果需要）将向量转换为Point。

但是请注意，不必更改什么：'synchronizeProjectionInfo'语句与网络情况完全相同。即使空间的字符改变，该语句也不会改变。

运行时，代码应输出空间的大小以及网格中agent的位置。

## Step_02: 在空间中移动的agent
要让agent四处走动，需要进行以下更改：

* 在agent class中创建一个 'move'方法
* 在model class中创建一个方法来调用move方法
* 将model class中的方法进行schedule

首先是对Agent.h的修改：
``` c++ {highlight=[4]}
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context);    // Choose three other agents from the given context and see if they cooperate or not
    void move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);    
```
对Agent.cpp修改如下：
```c++{highlight=[1-9]}
void RepastHPCDemoAgent::move(repast::SharedDiscreteSpace >* space){

    std::vector agentLoc;
    space->getLocation(id_, agentLoc);
    std::vector agentNewLoc;
    agentNewLoc.push_back(agentLoc[0] + (id_.id() < 7 ? -1 : 1));
    agentNewLoc.push_back(agentLoc[1] + (id_.id() > 3 ? -1 : 1));
    space->moveTo(id_,agentNewLoc);    
}
```
请注意，在这里，agent必须询问空间(space)以获取其位置。我们可以为agent提供x，y坐标的记录，但是我们在这里忽略了坐标，为了可以帮助我们清楚地理解，是由投影而不是agent决定了该agent与该空间中其他agent的关系。

我们添加几行以确定新的位置：在这里，我们允许某些agent（基于ID号）通过增加或减少其x和y坐标来移动。然后，我们使用“ moveTo”方法来告知将space将agent移动到新位置。

这要求对Model.cpp代码进行以下更改。我们没有在模型中添加其他方法，而是将其简单地添加到'doSomething'方法中：
```c++{highlight=[43-50,54]}
void RepastHPCDemoModel::doSomething(){
	int whichRank = 0;
	if(repast::RepastProcess::instance()->rank() == whichRank) std::cout << " TICK " << repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;

	if(repast::RepastProcess::instance()->rank() == whichRank){
		std::cout << "LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() == whichRank)){
                    std::vector<int> agentLoc;
                    discreteSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<int> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
		
		std::cout << "NON LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() != whichRank)){
                    std::vector<int> agentLoc;
                    discreteSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<int> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
	}	
	
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
        (*it)->play(&context);
		it++;
    }

    it = agents.begin();
    while(it != agents.end()){
		(*it)->move(discreteSpace);
		it++;
    }

    discreteSpace->balance();
    repast::RepastProcess::instance()->synchronizeAgentStatus<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);
    
    repast::RepastProcess::instance()->synchronizeProjectionInfo<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);

    repast::RepastProcess::instance()->synchronizeAgentStates<RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(*provider, *receiver);    
}
```
该代码通过本地agent创建迭代器，并要求他们'play'。然后，它将迭代器重新设置为（相同）列表的开头，并进行迭代，要求agent 'move'。agent移动后，代码将执行四个任务：

* 在空间上调用 'balance'。 'balance'将已移入缓冲区的agent标记为要移至相邻进程的agent。
* 调用“ synchronizeAgentStatus”，执行移动；已从一个进程的本地边界移出的agent将移至适当的相邻流程。
* 调用“ synchronizeProjectionInfo”，它将在本地边界内并是某些其他进程的缓冲区内复制agent，以便这些agent在其他进程上可见。
* 调用“ synchronizeAgentStates”，对所有非本地agent执行最终更新，以便它们具有其本来agent的当前正确状态
重要说明这四个调用（balance，synchronizeAgentStatus，synchronizeProjectionInfo和synchronizeAgentState）的序列是微妙的。必须在“ synchronizeAgentStatus”之前调用“ balance”；这两个通常称为一个接一个。

但是您可能会发现在不同的位置调用“ synchronizeProjectionInfo”很有用，例如，在调用“ play”方法之前。在此模拟中，可能不需要在“ synchronizeProjectionInfo”之后调用“ synchronizeAgentStates”。但是，在某些情况下，“ synchronizeProjectionInfo”不会使所有非本地agent都是其本来agent状态的最新副本。通常，网络投影容易有这样的问题，而空间投影则一般没有此问题，因此在此可能不必担心。但是，要记住，同步例程不一定会使模拟处于完整和一致的状态。只有同时使用它们，才能保证使仿真保持一致状态。

进行三个调用以进行不同类型的同步似乎比较麻烦，似乎也效率低下。但是，由于RepastHPC管理同步步骤的方式，实际上没有更好的方法。原因是每个步骤将本地和非本地数据分为不同的部分，仅将所需的内容发送给其他进程。合并单独的功能实际上是不可能的，唯一的方法是将它们按顺序链接。例如，如果我们想保证'synchronizeAgentStatuses'可以使模拟处于完全一致的状态，则必须将synchronizeProjectionInfo和syncnizeAgentStates一起调用。当然，可以将两个甚至三个函数包装到一个函数中，但这并不能真正为用户节省很多精力。在实践中，有时可能会遇到对于给定的仿真，一个或多个步骤是不必要的-因此，将这些函数单独进行调用可以使用户有机会针对特定的context进行调整。

## Step_03: 查询邻近的agent：查询空间
通常，您会希望空间中的agent能够检测到自己附近的agent。RepastHPC提供了一种可以在离散的二维空间中完成此操作的简单方法。有两个可以查询agent周围空间的特殊对象：VN2DGridQuery和Moore2DGridQuery。第一个是冯·诺依曼（Von Neumann）查询，它查询紧邻中心单元的N，S，E和W的单元；第二个是摩尔查询，它查询8个周围的小区，包括N，NE，E，SE，S，SW，W和NW邻居。这两个查询不仅可以扩展到直接邻居，还可以扩展到围绕中心点的同心环。

要使用这些，我们将修改Agent.h类以将指针传递给空间，这是'play'方法的一部分：
```c++{highlight=[4]}
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context,
              repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);    // Choose three other agents from the given context and see if they cooperate or not
    void move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);
```
当然，这意味着要修改model.cpp文件中的调用:
```c++{highlight=[5]}
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
        (*it)->play(&context, discreteSpace);
		it++;
    }
```
最后，我们将更改agent的play以使用空间查询选择agent：
```c++{highlight=[1-2]}
#include "repast_hpc/Moore2DGridQuery.h"
#include "repast_hpc/Point.h"
```
```c++{highlight=[2-9,13,15-18]}
void RepastHPCDemoAgent::play(repast::SharedContext<RepastHPCDemoAgent>* context,
                              repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space){
    std::vector<RepastHPCDemoAgent*> agentsToPlay;
    
    std::vector<int> agentLoc;
    space->getLocation(id_, agentLoc);
    repast::Point<int> center(agentLoc);
    repast::Moore2DGridQuery<RepastHPCDemoAgent> moore2DQuery(space);
    moore2DQuery.query(center, 1, false, agentsToPlay);    
    
    double cPayoff     = 0;
    double totalPayoff = 0;
    std::vector<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        std::vector<int> otherLoc;
        space->getLocation((*agentToPlay)->getId(), otherLoc);
        repast::Point<int> otherPoint(otherLoc);
        std::cout << " AGENT " << id_ << " AT " << center << " PLAYING " << ((*agentToPlay)->getId().currentRank() == id_.currentRank() ? "LOCAL" : "NON-LOCAL") << " AGENT " << (*agentToPlay)->getId() << " AT " << otherPoint << std::endl;

        bool iCooperated = cooperate();                          // Do I cooperate?
        double payoff = (iCooperated ?
						 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
        if(iCooperated) cPayoff += payoff;
        totalPayoff             += payoff;
		
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;
}
```
请注意，现在'play'的agent集合是向量，而不是集合。查询的第一部分是获取查询agent所位于的点，这需要一个向量来存储空间中的结果，然后将其用于创建点对象。此后，将创建然后调用Moore2DGridQuery。 query()函数的四个参数是：中心点，范围，是否包括中心单元，以及将被放置在vector中的相邻单元agent。请注意，这里使用的范围是“ 1”。我们可以使用“ 2”，但是如果我们需要使用更大范围的查询，我们还必须扩展空间的缓冲区，以确保本地agent可以“看到”所有可能的邻居。我们也省略了中心单元；从理论上讲，该单元格中可能有相邻的agent，但是我们忽略了这一点，因为这样可以确保进行搜索的agent不包括在结果中。我们还添加了一些有关正在'play'的agent的更多信息，仅用于显示输出。一个有用的练习是查看正在'play'哪些非本地agent。

## Step_04: 严格的边界
在Demo_03中，描述了“ WrapAround”和“strict”边界之间的区别，我们将在这里应用。在step_03中查找靠近的agent：查询使用的是WrapAround边界的空间；我们将对其进行修改以使用严格的边界。

主要变化很简单，在Model.h文件中，更改空间的类型：
```c++ {highlight=[11]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

	repast::SVDataSet* agentValues;
	repast::SharedDiscreteSpace<RepastHPCDemoAgent,  repast::StrictBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace;
```
在model.cpp中对初始化部分进行更改：
```c++{highlight=[19]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
    repast::Point<double> origin(-100,-100);
    repast::Point<double> extent(200, 200);
    
    repast::GridDimensions gd(origin, extent);
    
    std::vector<int> processDims;
    processDims.push_back(2);
    processDims.push_back(2);
    
    discreteSpace = new repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::StrictBorders, repast::SimpleAdder<RepastHPCDemoAgent> >("AgentDiscreteSpace", gd, processDims, 5, comm);
```
在agent.h中进行更改：
```c++{highlight=[29,30]}
class RepastHPCDemoAgent{
	
private:
    repast::AgentId   id_;
    double              c;
    double          total;
	
public:
    RepastHPCDemoAgent(repast::AgentId id);
	RepastHPCDemoAgent(){}
    RepastHPCDemoAgent(repast::AgentId id, double newC, double newTotal);
	
    ~RepastHPCDemoAgent();
	
    /* Required Getters */
    virtual repast::AgentId& getId(){                   return id_;    }
    virtual const repast::AgentId& getId() const {      return id_;    }
	
    /* Getters specific to this kind of Agent */
    double getC(){                                      return c;      }
    double getTotal(){                                  return total;  }
	
    /* Setter */
    void set(int currentRank, double newC, double newTotal);
	
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context,
              repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::StrictBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);    // Choose three other agents from the given context and see if they cooperate or not
    void move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::StrictBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);    
};
```
在agent.cpp中修改如下：
```c++ {highlight=[7-13]}
void RepastHPCDemoAgent::move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::StrictBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space){

    std::vector<int> agentLoc;
    space->getLocation(id_, agentLoc);
    
    std::vector<int> agentNewLoc;
    do{
        agentNewLoc.clear();
        agentNewLoc.push_back(agentLoc[0] + (repast::Random::instance()->nextDouble() < .5 ? 1 : -1));
        agentNewLoc.push_back(agentLoc[1] + (repast::Random::instance()->nextDouble() < .5 ? 1 : -1));
        if(!space->bounds().contains(agentNewLoc)) std::cout << " INVALID: " << agentNewLoc[0] << "," << agentNewLoc[1] << std::endl;
        
    }while(!space->bounds().contains(agentNewLoc));
    
    space->moveTo(id_,agentNewLoc);    
}
```
除了更改'move'外，我们还必须确保agent不会超出范围。使用WrapAround边界，超越全局边界将使之到达空间的另一侧（实际上，移动到边界之外的某个位置将导致agent被放回已知的边界内，就像空间在不断地重复一样。）但是，对于严格边界，在“外部”选择一个位置空间，将在RepastHPC中引发错误。为了解决这个问题，在此演示中，我们允许agent随机行走，如果他们尝试跨出允许的边界，则将其重定向回内部。对此的检查是space->bounds().contains()方法。

运行此命令并检查输出；您应该不会在全局边界边缘（例如-100，-100）附近看到任何agent在与非本地agent 'play'。当然，内部边界处的agent仍然有可能扮演非本地agent。
## Step_05: 连续空间
在前面的步骤中，我们使用了离散空间。从本质上讲，这些是可以放置agent的网格，但只能在特定的规范内：坐标必须全部为整数。RepastHPC还实现了连续的空间，其中agent坐标可以是任何值。

从技术上讲，它可以是C ++'double'表示的任何值； 从严格的数学意义上讲，这并不是真正连续的，但假定它足够接近。

用于实现连续空间的方法几乎与用于实现离散空间的方法相同。我们只需要通过将对“ SharedDiscreteSpace”的所有引用更改为“ SharedContinuousSpace”来修改本演示的步骤2：
在Model.h中：
```c++{highlight=[8]}
#include 
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/AgentRequest.h"
#include "repast_hpc/TDataSource.h"
#include "repast_hpc/SVDataSet.h"
#include "repast_hpc/SharedContinuousSpace.h"
#include "repast_hpc/GridComponents.h"
```
```c++{highlight=[11]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

	repast::SVDataSet* agentValues;
        repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace;
```
在agent.h中：
```c++{highlight=[3]}
#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/SharedContinuousSpace.h"
```
```c++{highlight=[4]}
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context);    // Choose three other agents from the given context and see if they cooperate or not
    void move(repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space);
```
在Model.cpp中：
```c++{highlight=[19,23]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
        repast::Point<double> origin(-100,-100);
        repast::Point<double> extent(200, 200);
    
        repast::GridDimensions gd(origin, extent);
    
        std::vector<int> processDims;
        processDims.push_back(2);
        processDims.push_back(2);
    
        continuousSpace = new repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >("AgentContinuousSpace", gd, processDims, 2, comm);
	
        std::cout << "RANK " << repast::RepastProcess::instance()->rank() << " BOUNDS: " << continuousSpace->bounds().origin() << " " << continuousSpace->bounds().extents() << std::endl;
    
   	context.addProjection(continuousSpace);
```
model.cpp的init:
```c++{highlight=[4,9]}
void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
        repast::Point<double<> initialLocation((double)continuousSpace->dimensions().origin().getX() + i,(double)continuousSpace->dimensions().origin().getY() + i);
		repast::AgentId id(i, rank, 0);
		id.currentRank(rank);
		RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
		context.addAgent(agent);
                continuousSpace->moveTo(id, initialLocation);
	}
}
```

仍然在Model.cpp文件中，我们必须对doSomething进行一些更改：
```c++{highlight=[12,14,26,28,45,49]}
void RepastHPCDemoModel::doSomething(){
	int whichRank = 0;
	if(repast::RepastProcess::instance()->rank() == whichRank) std::cout << " TICK " << repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;

	if(repast::RepastProcess::instance()->rank() == whichRank){
		std::cout << "LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
			    repast::AgentId toDisplay(i, r, 0);
			    RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
			    if((agent != 0) && (agent->getId().currentRank() == whichRank)){
                                std::vector<double> agentLoc;
                                continuousSpace->getLocation(agent->getId(), agentLoc);
                                repast::Point<double> agentLocation(agentLoc);
                                std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                            }
			}
		}
		
		std::cout << "NON LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
			    repast::AgentId toDisplay(i, r, 0);
			    RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
			    if((agent != 0) && (agent->getId().currentRank() != whichRank)){
                                std::vector<double> agentLoc;
                                continuousSpace->getLocation(agent->getId(), agentLoc);
                                repast::Point<double> agentLocation(agentLoc);
                                std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                            }
			}
		}
	}
	
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
        (*it)->play(&context);
		it++;
    }

    it = agents.begin();
    while(it != agents.end()){
		(*it)->move(continuousSpace);
		it++;
    }

    continuousSpace->balance();
    repast::RepastProcess::instance()->synchronizeAgentStatus<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);
    
    repast::RepastProcess::instance()->synchronizeProjectionInfo<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);

    repast::RepastProcess::instance()->synchronizeAgentStates<RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(*provider, *receiver);    
}
```

请注意，在init函数以及此处用于获取和显示agent位置的Vector和Point类中，对“ int”关键字的引用已被替换为“ double”关键字。这是因为连续空间的基础数据类型是double，而不是int，并且数据结构必须反映这一点。这与我们必须在Agent.cpp文件中进行这些更改的原因完全相同：
```c++{highlight=[1,3,5]}
void RepastHPCDemoAgent::move(repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* space){

    std::vector<double> agentLoc;
    space->getLocation(id_, agentLoc);
    std::vector<double> agentNewLoc;
    agentNewLoc.push_back(agentLoc[0] + (repast::Random::instance()->nextDouble() - .5) * 1.5));
    agentNewLoc.push_back(agentLoc[1] + (repast::Random::instance()->nextDouble() - .5) * 1.5));
    space->moveTo(id_,agentNewLoc);    
}
```
注意，我们现在也使agent按分数间隔移动。

## Step_06:多空间的映射：共享边界
之前，我们都单独使用离散空间或连续空间。但是，在某些情况下，同时使用两个空间可能是有利的，其中一个是离散的，另一个是连续的。可能使用的两个空间都代表了模拟的相同方面，在这种情况下，agent在连续空间中的位置可能为（1.7，2.4），而在离散空间中位置必须全部为整数值的离散空间为（1、2）。尽管两个空间投影是完全分开的，但在这种情况下，两个空间中agent的位置都代表相同的事物。尽管在较粗的分辨率下，离散空间坐标始终会更新以匹配连续位置。

我们为什么要这样做？因为agent经常需要在特定距离内找到它周围的agent（如前面step中演示的），并且其中的agent数量与每个进程中agent的总数相比，此半径可能较小。关键是，在离散空间中进行搜索比在连续空间中进行搜索更高效。

高效的原因与离散空间内部存储有关agent位置的数据的方式有关。离散空间将特定位置（例如“ 1、2”）映射到该位置的agent的简单列表。因此，很容易在一个很小的位置集合中（例如，在1,1附近的邻域，包括：0,0 + 0,1 + 0,2 + 1,0 + 1,1）编译所有agent的列表。 + 1,2 + 2,0 + 2,1 + 2,2）。

考虑一个模拟，在该模拟中，agent只能与自己在空间中短距离内的agent进行交互。每个agent必须轮询空间以获取指定距离内的agent列表。在离散的空间中，附近的单元格可以非常快速地返回agent列表。

相反，连续空间仅存储agent列表及其位置。为了确定给定距离内的agent列表，必须计算到所有其他agent的距离。如果给定距离与全局空间中的大小相比较小，并且与该空间之内的大量agent相比，只有少数agent落入此距离内，则此策略将执行许多实际上不需要的计算。

请注意，采用此策略将需要两个步骤：假设您的仿真需要一个非整数值，该值用作圆半径（例如4.5单位）。在这种情况下，最有效的策略是使用离散空间来检索邻域内的所有agent，该邻域要足够大以包括4.5个单位半径（即5个单位）可到达的整个空间。该邻域是矩形，而不是圆形，并且超出了实际需要的半径（5个单位与4.5个单位）。但是，一旦有了该邻域内所有agent的列表，就需要仅计算这些agent的准确距离，而不是所有agent。您可以使用它来确定哪些agent落入仿真实际所需的4.5个单位半径内。

出于这个原因，RepastHPC中的'Relogo'实现使用两个空格。

执行此操作的代码很简单。设置空间的代码由创建两个空间的实例完成，每个'move'操作执行两次：agent在连续空间和离散空间中移动。

可以进行另一项更改以提高效率。连续空间的缓冲区可以设置为零。前面的demo中可知，缓冲区用于标记要在进程之间共享的agent，并用于移动超出本地边界的agent。关于首次操作，一旦被标记，便会将agent与其所有投影信息一起共享。无需对agent进行两次标记；如果离散空间指示应该共享agent，则不需要连续空间也对其进行标记。由于在离散空间中搜索更有效的相同原因，在缓冲区中导出agent的过程也更高效。在大多数情况下，即使模拟所需的实际缓冲区大小为非整数值，在离散空间中使用第二大整数值作为缓冲区大小也要比使用连续模式更有效。跨进程边界移动agent的逻辑相同：'balance' 应仅在离散空间上调用；这将标记已移出进程边界的agent。将缓冲区大小设置为零意味着连续空间将绕过可能需要缓冲区同步的所有内部调用：它将共享或发送的所有agent已经由离散空间共享或发送，因此连续空间上的工作是不必要的。

添加include并在Agent.h的'Agent'类中更改，以在两个空间中move和play:
```c++ {highlight=[3-4]}
#include "repast_hpc/AgentId.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/SharedDiscreteSpace.h"
#include "repast_hpc/SharedContinuousSpace.h"
```
```c++ {highlight=[4-7]}
    /* Actions */
    bool cooperate();                                                 // Will indicate whether the agent cooperates or not; probability determined by = c / total
    void play(repast::SharedContext<RepastHPCDemoAgent>* context,
              repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace,
              repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace);    // Choose three other agents from the given context and see if they cooperate or not
    void move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace,
              repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace);
```
同样，必须将Model.h文件更改为包含两种空间：
```c++ {highlight=[8,9]}
#include <boost/mpi.hpp>
#include "repast_hpc/Schedule.h"
#include "repast_hpc/Properties.h"
#include "repast_hpc/SharedContext.h"
#include "repast_hpc/AgentRequest.h"
#include "repast_hpc/TDataSource.h"
#include "repast_hpc/SVDataSet.h"
#include "repast_hpc/SharedContinuousSpace.h"
#include "repast_hpc/SharedDiscreteSpace.h"
#include "repast_hpc/GridComponents.h"
```
```c++ {highlight=[11,12]}
class RepastHPCDemoModel{
	int stopAt;
	int countOfAgents;
	repast::Properties* props;
	repast::SharedContext<RepastHPCDemoAgent> context;
	
	RepastHPCDemoAgentPackageProvider* provider;
	RepastHPCDemoAgentPackageReceiver* receiver;

	repast::SVDataSet* agentValues;
        repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace;
        repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace;
```
修改模型类以实例化并利用以下两个空间：
```c++ {highlight=[19-20,24-25]}
RepastHPCDemoModel::RepastHPCDemoModel(std::string propsFile, int argc, char** argv, boost::mpi::communicator* comm): context(comm){
	props = new repast::Properties(propsFile, argc, argv, comm);
	stopAt = repast::strToInt(props->getProperty("stop.at"));
	countOfAgents = repast::strToInt(props->getProperty("count.of.agents"));
	initializeRandom(*props, comm);
	if(repast::RepastProcess::instance()->rank() == 0) props->writeToSVFile("./output/record.csv");
	provider = new RepastHPCDemoAgentPackageProvider(&context);
	receiver = new RepastHPCDemoAgentPackageReceiver(&context);
	
    repast::Point<double> origin(-100,-100);
    repast::Point<double> extent(200, 200);
    
    repast::GridDimensions gd(origin, extent);
    
    std::vector<int> processDims;
    processDims.push_back(2);
    processDims.push_back(2);
    
    discreteSpace = new repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >("AgentDiscreteSpace", gd, processDims, 2, comm);
    continuousSpace = new repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >("AgentContinuousSpace", gd, processDims, 0, comm);

    std::cout << "RANK " << repast::RepastProcess::instance()->rank() << " BOUNDS: " << continuousSpace->bounds().origin() << " " << continuousSpace->bounds().extents() << std::endl;
    
    context.addProjection(continuousSpace);
    context.addProjection(discreteSpace);
    
	// Data collection
	// Create the data set builder
	std::string fileOutputName("./output/agent_total_data.csv");
	repast::SVDataSetBuilder builder(fileOutputName.c_str(), ",", repast::RepastProcess::instance()->getScheduleRunner().schedule());
	
	// Create the individual data sets to be added to the builder
	DataSource_AgentTotals* agentTotals_DataSource = new DataSource_AgentTotals(&context);
	builder.addDataSource(createSVDataSource("Total", agentTotals_DataSource, std::plus<int>()));

	DataSource_AgentCTotals* agentCTotals_DataSource = new DataSource_AgentCTotals(&context);
	builder.addDataSource(createSVDataSource("C", agentCTotals_DataSource, std::plus<int>()));

	// Use the builder to create the data set
	agentValues = builder.createDataSet();	
}
```
注意使用0作为连续空间的缓冲区参数。
```c++ {highlight=[4-5,10-11]}
void RepastHPCDemoModel::init(){
	int rank = repast::RepastProcess::instance()->rank();
	for(int i = 0; i < countOfAgents; i++){
            repast::Point<int> initialLocationDiscrete((int)discreteSpace->dimensions().origin().getX() + i,(int)discreteSpace->dimensions().origin().getY() + i);
            repast::Point<double> initialLocationContinuous((double)continuousSpace->dimensions().origin().getX() + i,(double)continuousSpace->dimensions().origin().getY() + i);
	    repast::AgentId id(i, rank, 0);
            id.currentRank(rank);
	    RepastHPCDemoAgent* agent = new RepastHPCDemoAgent(id);
	    context.addAgent(agent);
            discreteSpace->moveTo(id, initialLocationDiscrete);
            continuousSpace->moveTo(id, initialLocationContinuous);
	}
}
```
```c++ {highlight=[39,45,49]}
void RepastHPCDemoModel::doSomething(){
	int whichRank = 0;
	if(repast::RepastProcess::instance()->rank() == whichRank) std::cout << " TICK " << repast::RepastProcess::instance()->getScheduleRunner().currentTick() << std::endl;

	if(repast::RepastProcess::instance()->rank() == whichRank){
		std::cout << "LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() == whichRank)){
                    std::vector<double> agentLoc;
                    continuousSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<double> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
		
		std::cout << "NON LOCAL AGENTS:" << std::endl;
		for(int r = 0; r < 4; r++){
			for(int i = 0; i < 10; i++){
				repast::AgentId toDisplay(i, r, 0);
				RepastHPCDemoAgent* agent = context.getAgent(toDisplay);
				if((agent != 0) && (agent->getId().currentRank() != whichRank)){
                    std::vector<double> agentLoc;
                    continuousSpace->getLocation(agent->getId(), agentLoc);
                    repast::Point<double> agentLocation(agentLoc);
                    std::cout << agent->getId() << " " << agent->getC() << " " << agent->getTotal() << " AT " << agentLocation << std::endl;
                }
			}
		}
	}	
	
	std::vector<RepastHPCDemoAgent*> agents;
	context.selectAgents(repast::SharedContext<RepastHPCDemoAgent>::LOCAL, countOfAgents, agents);
	std::vector<RepastHPCDemoAgent*>::iterator it = agents.begin();
	while(it != agents.end()){
        (*it)->play(&context, discreteSpace, continuousSpace);
		it++;
    }

    it = agents.begin();
    while(it != agents.end()){
		(*it)->move(discreteSpace, continuousSpace);
		it++;
    }

    discreteSpace->balance();
    repast::RepastProcess::instance()->synchronizeAgentStatus<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);
    
    repast::RepastProcess::instance()->synchronizeProjectionInfo<RepastHPCDemoAgent, RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(context, *provider, *receiver, *receiver);

    repast::RepastProcess::instance()->synchronizeAgentStates<RepastHPCDemoAgentPackage, RepastHPCDemoAgentPackageProvider, RepastHPCDemoAgentPackageReceiver>(*provider, *receiver);    
}
```
请注意，所有同步调用都不需要更改，仅更改对agent的move和play调用。
在Agent类中，更改为：
```c++{highlight=[2,3]}
#include "Demo_03_Agent.h"
#include "repast_hpc/Moore2DGridQuery.h"
#include "repast_hpc/Point.h"
```
然后是对move的修改：
```c++{highlight=[1,2,5-13]}
void RepastHPCDemoAgent::move(repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace,
                              repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace){

    std::vector<double> agentLoc;
    continuousSpace->getLocation(id_, agentLoc);
    std::vector<double> agentNewLocContinuous;
    agentNewLocContinuous.push_back(agentLoc[0] + (repast::Random::instance()->nextDouble() - .5) * 1.5);
    agentNewLocContinuous.push_back(agentLoc[1] + (repast::Random::instance()->nextDouble() - .5) * 1.5);
    continuousSpace->moveTo(id_,agentNewLocContinuous);
    std::vector<int> agentNewLocDiscrete;
    agentNewLocDiscrete.push_back((int)(floor(agentNewLocContinuous[0])));
    agentNewLocDiscrete.push_back((int)(floor(agentNewLocContinuous[1])));
    discreteSpace->moveTo(id_, agentNewLocDiscrete);   
}
```
对play修改：
```c++{highlight=[2-3,6-14,21-27,35-38]}
void RepastHPCDemoAgent::play(repast::SharedContext<RepastHPCDemoAgent>* context,
                              repast::SharedDiscreteSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* discreteSpace,
                              repast::SharedContinuousSpace<RepastHPCDemoAgent, repast::WrapAroundBorders, repast::SimpleAdder<RepastHPCDemoAgent> >* continuousSpace){
    std::vector<RepastHPCDemoAgent*> agentsToPlay;
    
    std::vector<int> agentLocDiscrete;
    discreteSpace->getLocation(id_, agentLocDiscrete);
    repast::Point<int> center(agentLocDiscrete);
    repast::Moore2DGridQuery<RepastHPCDemoAgent> moore2DQuery(discreteSpace);
    moore2DQuery.query(center, 2, false, agentsToPlay);
    
    std::vector<double> agentLocContinuous;
    continuousSpace->getLocation(id_, agentLocContinuous);
    repast::Point<double> agentPointContinuous(agentLocContinuous[0], agentLocContinuous[1]);
    
    double cPayoff     = 0;
    double totalPayoff = 0;
    std::vector<RepastHPCDemoAgent*>::iterator agentToPlay = agentsToPlay.begin();
    while(agentToPlay != agentsToPlay.end()){
        
        std::vector<double> otherLocContinuous;
        continuousSpace->getLocation((*agentToPlay)->getId(), otherLocContinuous);
        repast::Point<double> otherPointContinuous(otherLocContinuous[0], otherLocContinuous[1]);
        double distance = continuousSpace->getDistance(agentPointContinuous, otherPointContinuous);
        // Only play if within 1.5
        if(distance < 1.5){
            std::cout << " AGENT " << id_ << " AT " << agentPointContinuous << " PLAYING " << (*agentToPlay)->getId() << " at " << otherPointContinuous <<  " (distance = " << distance << " )" << std::endl;

            bool iCooperated = cooperate();                          // Do I cooperate?
            double payoff = (iCooperated ?
		    				 ((*agentToPlay)->cooperate() ?  7 : 1) :     // If I cooperated, did my opponent?
						 ((*agentToPlay)->cooperate() ? 10 : 3));     // If I didn't cooperate, did my opponent?
            if(iCooperated) cPayoff += payoff;
            totalPayoff             += payoff;
	}
        else{
            std::cout << " AGENT " << id_ << " AT " << agentPointContinuous << " NOT PLAYING " << (*agentToPlay)->getId() << " at " << otherPointContinuous <<  " (distance = " << distance << " )" << std::endl;
        }
        agentToPlay++;
    }
    c      += cPayoff;
    total  += totalPayoff;	
}
```