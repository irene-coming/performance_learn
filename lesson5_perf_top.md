cpu统计信息的来源 /proc/stat or /proc/[pid]/stat
cpu统计信息各列的含义
cpu统计信息代表【某段时间】的使用率

遇到cpu瓶颈时诊断的主要思路：
top 找出占用高的进程，1 查看各个cpu的占用情况
perf top -g -p pid 找出占比高的进程，箭头选中后enter查看使用的具体函数

tips： 如果系统是centos7，用 perf top -g -p pid没有看到函数名称，只能看到一堆十六进制的东西，解决方法：
分析：当没有看到函数名称，只看到了十六进制符号，下面有Failed to open /usr/lib/x86_64-linux-gnu/libxml2.so.2.9.4, continuing without symbols 这说明perf无法找到待分析进程所依赖的库。这里只显示了一个，但其实依赖的库还有很多。这个问题其实是在分析Docker容器应用时经常会碰到的一个问题，因为容器应用所依赖的库都在镜像里面。

两个解决思路：
（1）在容器外面构建相同路径的依赖库。这种方法不推荐，一是因为找出这些依赖比较麻烦，更重要的是构建这些路径会污染虚拟机的环境。
（2）在容器外面把分析纪录保存下来，到容器里面再去查看结果，这样库和符号的路径就都是对的了。

操作：
（1）在Centos系统上运行 perf record -g -p <pid>，执行一会儿（比如15秒）按ctrl+c停止
（2）把生成的 perf.data文件拷贝到容器里面分析:
docker cp perf.data target-contianer:/tmp
docker exec -it target-contianer bash
$ cd /tmp/
$ apt-get update && apt-get install -y linux-perf linux-tools procps
$ perf_4.9 report

注意：最后运行的工具名字是容器内部安装的版本 perf_4.9，而不是 perf 命令，这是因为 perf 会去跟内核的版本进行匹配，但镜像里面安装的perf版本有可能跟虚拟机的内核版本不一致。
注意：上面的问题只是在centos系统中有问题，ubuntu上没有这个问题
