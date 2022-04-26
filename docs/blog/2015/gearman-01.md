# Gearman（一）

:material-clock-time-three-outline:  2015-09-01 15:56

## 简单介绍

Gearman译为“齿轮工”，从字面上看，他的职责就是通过齿轮将不同的组件结合在一起。作为一个任务分发架构，Gearman请求的处理过程涉及三个角色：Client -> Job Server -> Worker。它能够轻松的将前端Client提交的任务通过Job Server分发给后端的空闲的Worker处理。

- Client：请求的发起者，可以是C，PHP，Perl，MySQL UDF等等。
- Job Server：请求的调度者，用来负责协调把Client发出的请求转发给合适的Worker。
- Worker：请求的处理者，可以是C，PHP，Perl等等。

**工作集群示意：**

![cluster.png][1]

详见：[Gearman官网][2]

---

## Server安装

因为项目后台采用Java，Client和Worker无疑是用Java实现，虽然Gearman并没有这样的限制，而且Client和Worker可以采用不同的语言实现。中间的纽带就是Job Server。

目前Job Server（gearmand）的java版本有三个。

1. 官方推荐的[java-gearman-service][3]，目前Google Code上的release版本是0.6.6。项目已迁移至Github的[java-service][4]，没有realease版本，maven上的版本号是0.7.0-SNAPSHOT，不过时间显示已经是两年前了。其中还包括了Client和Worker的客户端API。
2. [gearman-java][5]官方Download中有提及，gearman-java.jar只包括Worker和Client的API，没有Server。
3. [gearman-server][6]，目前的版本是0.6.2，里面包括Web Server：Jetty，持久化框架Redis和Postpresql。

测试时采用了0.6.6版本，直接运行java-gearman-service-0.6.6.jar即可。

但是gearmand作为Job Server除了安装过程中有一些配置之外，在开发过程中可以说是项目无关的。Client和Worker只需关系Job Server的IP和端口。所以在Linux服务器上安装C版本的Gearmand就可以了。

下面开始C版本的gearmand安装。

1. 创建/usr/servers，此后我们把软件安装在此目录
``` bash
mkdir -p /usr/servers
cd /usr/servers
```

2. 下载安装gearmand，最新版在[Launchpad][7]，版本号为1.1.12
``` bash
tar -xzvf gearmand-1.1.12.tar.gz
cd gearmand-1.1.12
./configure --enable-debug
```

执行过程中出现如下错误，原因是本机中缺少依赖包。
```
 checking for cc... no
 checking for gcc... no
 checking for clang... no
 configure: error: in `/usr/servers/gearmand-1.1.12':
 configure: error: no acceptable C compiler found in $PATH
 See `config.log' for more details
```

注： ./configure的options可通过`./configure -help`查看。

3. 安装依赖（本机是centos6.6-DVD1 miniDeskTop版本）
``` bash
yum -y install gcc-c++ boost-devel gperf libevent-devel libuuid-devel uuid-dce uuid-dce-devel uuid-c++ uuid-c++-devel
```

待所有依赖安装完成，执行`./configure --enable-debug`，出现configure结果。
```
---
Configuration summary for gearmand version 1.1.12

* Installation prefix: /usr/local
* System type: unknown-linux-gnu
* Host CPU: x86_64
* C Compiler: cc -std=gnu99 cc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-16)
* C Flags: -H -g -g3 -fno-eliminate-unused-debug-types -fno-omit-frame-pointer -Wno-unknown-pragmas -Wno-pragmas -Wall -Wextra -Wno-attributes -Waddress -Warray-bounds -Wbad-function-cast -Wchar-subscripts -Wcomment -Wfloat-equal -Wformat-security -Wformat=2 -Wformat-y2k -Wlogical-op -Wmissing-field-initializers -Wmissing-noreturn -Wmissing-prototypes -Wnested-externs -Wnormalized=id -Woverride-init -Wpointer-arith -Wpointer-sign -Wredundant-decls -Wshadow -Wsign-compare -Wstrict-overflow=1 -Wswitch-enum -Wundef -funsafe-loop-optimizations -Wclobbered -Wunused -Wunused-variable -Wunused-parameter -Wwrite-strings -fwrapv -pipe -fPIE -pie -Wpacked
* C++ Compiler: c++ c++ (GCC) 4.4.7 20120313 (Red Hat 4.4.7-16)
* C++ Flags: -H -g -g3 -fno-inline -fno-eliminate-unused-debug-types -fno-omit-frame-pointer -Wno-unknown-pragmas -Wno-pragmas -Wall -Wextra -Wno-attributes -Waddress -Warray-bounds -Wchar-subscripts -Wcomment -Wctor-dtor-privacy -Wfloat-equal -Wformat=2 -Wformat-y2k -Wmissing-field-initializers -Wlogical-op -Wnon-virtual-dtor -Wnormalized=id -Woverloaded-virtual -Wpointer-arith -Wredundant-decls -Wshadow -Wsign-compare -Wstrict-overflow=1 -Wswitch-enum -Wundef -funsafe-loop-optimizations -Wclobbered -Wunused -Wunused-variable -Wunused-parameter -Wwrite-strings -Wformat-security -fwrapv -pipe -fPIE -pie -Wpacked
* CPP Flags: -fvisibility=hidden
* LIBS:
* LDFLAGS Flags: -lmcheck
* Assertions enabled: yes
* Debug enabled: yes
* Warnings as failure: no
* Building with libsqlite3 no
* Building with libdrizzle yes
* Building with libmemcached not found
* Building with libpq no
* Building with tokyocabinet no
* Building with libmysql no
* SSL enabled: no
* cyassl found: no
* openssl found: no
* make -j: 2
* VCS checkout: no
* sphinx-build: :

---
```

4. make&install，结果如下。

```
make[2]: Entering directory `/usr/servers/gearmand-1.1.12'
make[2]: warning: -jN forced in submake: disabling jobserver mode.
/bin/mkdir -p '/usr/local/lib'
/bin/sh ./libtool --mode=install /usr/bin/install -c libgearman/libgearman. la '/usr/local/lib'
libtool: install: /usr/bin/install -c libgearman/.libs/libgearman.so.8.0.0 /usr/ local/lib/libgearman.so.8.0.0
libtool: install: (cd /usr/local/lib && { ln -s -f libgearman.so.8.0.0 libgearma n.so.8 || { rm -f libgearman.so.8 && ln -s libgearman.so.8.0.0 libgearman.so.8; }; })
libtool: install: (cd /usr/local/lib && { ln -s -f libgearman.so.8.0.0 libgearma n.so || { rm -f libgearman.so && ln -s libgearman.so.8.0.0 libgearman.so; }; })
libtool: install: /usr/bin/install -c libgearman/.libs/libgearman.lai /usr/local /lib/libgearman.la
libtool: install: /usr/bin/install -c libgearman/.libs/libgearman.a /usr/local/l ib/libgearman.a
libtool: install: chmod 644 /usr/local/lib/libgearman.a
libtool: install: ranlib /usr/local/lib/libgearman.a
libtool: finish: PATH="/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbi n:/bin:/usr/sbin:/usr/bin:/root/bin:/sbin" ldconfig -n /usr/local/lib
----------------------------------------------------------------------
Libraries have been installed in:
/usr/local/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
- add LIBDIR to the `LD_LIBRARY_PATH' environment variable
during execution
- add LIBDIR to the `LD_RUN_PATH' environment variable
during linking
- use the `-Wl,-rpath -Wl,LIBDIR' linker flag
- have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
/bin/mkdir -p '/usr/local/bin'
/bin/sh ./libtool --mode=install /usr/bin/install -c bin/gearman bin/gearadm in '/usr/local/bin'
libtool: install: /usr/bin/install -c bin/.libs/gearman /usr/local/bin/gearman
libtool: install: /usr/bin/install -c bin/gearadmin /usr/local/bin/gearadmin
/bin/mkdir -p '/usr/local/sbin'
/bin/sh ./libtool --mode=install /usr/bin/install -c gearmand/gearmand '/usr /local/sbin'
libtool: install: /usr/bin/install -c gearmand/gearmand /usr/local/sbin/gearmand
/bin/mkdir -p '/usr/local/share/man/man1'
/usr/bin/install -c -m 644 man/gearadmin.1 man/gearman.1 '/usr/local/share/man/ man1'
/bin/mkdir -p '/usr/local/share/man/man3'
/usr/bin/install -c -m 644 man/gearman_allocator_t.3 man/gearman_client_set_mem ory_allocators.3 man/gearman_worker_set_memory_allocators.3 man/gearman_actions_ t.3 man/gearman_argument_make.3 man/gearman_argument_t.3 man/gearman_bugreport.3 man/gearman_client_add_options.3 man/gearman_client_add_server.3 man/gearman_cl ient_add_servers.3 man/gearman_client_add_task.3 man/gearman_client_add_task_bac kground.3 man/gearman_client_add_task_high.3 man/gearman_client_add_task_high_ba ckground.3 man/gearman_client_add_task_low.3 man/gearman_client_add_task_low_bac kground.3 man/gearman_client_add_task_status.3 man/gearman_client_clear_fn.3 man /gearman_client_clone.3 man/gearman_client_context.3 man/gearman_client_create.3 man/gearman_client_do.3 man/gearman_client_do_background.3 man/gearman_client_d o_high.3 man/gearman_client_do_high_background.3 man/gearman_client_do_job_handl e.3 man/gearman_client_do_low.3 man/gearman_client_do_low_background.3 man/gearm an_client_do_status.3 man/gearman_client_echo.3 man/gearman_client_errno.3 man/g earman_client_error.3 man/gearman_client_free.3 man/gearman_client_job_status.3 man/gearman_client_options.3 man/gearman_client_remove_options.3 man/gearman_cli ent_remove_servers.3 man/gearman_client_run_tasks.3 man/gearman_client_set_compl ete_fn.3 man/gearman_client_set_context.3 '/usr/local/share/man/man3'
/usr/bin/install -c -m 644 man/gearman_client_set_created_fn.3 man/gearman_clie nt_set_data_fn.3 man/gearman_client_set_exception_fn.3 man/gearman_client_set_fa il_fn.3 man/gearman_client_set_log_fn.3 man/gearman_client_set_namespace.3 man/g earman_client_set_options.3 man/gearman_client_set_status_fn.3 man/gearman_clien t_set_task_context_free_fn.3 man/gearman_client_set_timeout.3 man/gearman_client _set_warning_fn.3 man/gearman_client_set_workload_fn.3 man/gearman_client_set_wo rkload_free_fn.3 man/gearman_client_set_workload_malloc_fn.3 man/gearman_client_ st.3 man/gearman_client_task_free_all.3 man/gearman_client_timeout.3 man/gearman _client_wait.3 man/gearman_continue.3 man/gearman_execute.3 man/gearman_failed.3 man/gearman_job_free.3 man/gearman_job_free_all.3 man/gearman_job_function_name .3 man/gearman_job_handle.3 man/gearman_job_handle_t.3 man/gearman_job_send_comp lete.3 man/gearman_job_send_data.3 man/gearman_job_send_exception.3 man/gearman_ job_send_fail.3 man/gearman_job_send_status.3 man/gearman_job_send_warning.3 man /gearman_job_st.3 man/gearman_job_take_workload.3 man/gearman_job_unique.3 man/g earman_job_use_client.3 man/gearman_job_workload.3 man/gearman_job_workload_size .3 man/gearman_log_fn.3 man/gearman_parse_servers.3 '/usr/local/share/man/man3'
/usr/bin/install -c -m 644 man/gearman_result_boolean.3 man/gearman_result_inte ger.3 man/gearman_result_is_null.3 man/gearman_result_size.3 man/gearman_result_ store_integer.3 man/gearman_result_store_string.3 man/gearman_result_store_value .3 man/gearman_result_string.3 man/gearman_return_t.3 man/gearman_strerror.3 man /gearman_string_t.3 man/gearman_success.3 man/gearman_task_context.3 man/gearman _task_data.3 man/gearman_task_data_size.3 man/gearman_task_denominator.3 man/gea rman_task_error.3 man/gearman_task_free.3 man/gearman_task_function_name.3 man/g earman_task_give_workload.3 man/gearman_task_is_known.3 man/gearman_task_is_runn ing.3 man/gearman_task_job_handle.3 man/gearman_task_numerator.3 man/gearman_tas k_recv_data.3 man/gearman_task_return.3 man/gearman_task_send_workload.3 man/gea rman_task_set_context.3 man/gearman_task_st.3 man/gearman_task_take_data.3 man/g earman_task_unique.3 man/gearman_verbose_name.3 man/gearman_verbose_t.3 man/gear man_version.3 man/gearman_worker_add_function.3 man/gearman_worker_add_options.3 man/gearman_worker_add_server.3 man/gearman_worker_add_servers.3 man/gearman_wo rker_clone.3 man/gearman_worker_context.3 '/usr/local/share/man/man3'
/usr/bin/install -c -m 644 man/gearman_worker_create.3 man/gearman_worker_defin e_function.3 man/gearman_worker_echo.3 man/gearman_worker_errno.3 man/gearman_wo rker_error.3 man/gearman_worker_free.3 man/gearman_worker_function_exist.3 man/g earman_worker_grab_job.3 man/gearman_worker_options.3 man/gearman_worker_registe r.3 man/gearman_worker_remove_options.3 man/gearman_worker_remove_servers.3 man/ gearman_worker_set_context.3 man/gearman_worker_set_log_fn.3 man/gearman_worker_ set_namespace.3 man/gearman_worker_set_options.3 man/gearman_worker_set_timeout. 3 man/gearman_client_has_option.3 man/gearman_client_options_t.3 man/gearman_tas k_attr_init.3 man/gearman_task_attr_init_background.3 man/gearman_task_attr_init _epoch.3 man/gearman_task_attr_t.3 man/gearman_worker_set_identifier.3 man/gearm an_worker_set_workload_free_fn.3 man/gearman_worker_set_workload_malloc_fn.3 man /gearman_worker_st.3 man/gearman_worker_timeout.3 man/gearman_worker_unregister. 3 man/gearman_worker_unregister_all.3 man/gearman_worker_wait.3 man/gearman_work er_work.3 man/libgearman.3 '/usr/local/share/man/man3'
/bin/mkdir -p '/usr/local/share/man/man8'
/usr/bin/install -c -m 644 man/gearmand.8 '/usr/local/share/man/man8'
/bin/mkdir -p '/usr/local/include'
/bin/mkdir -p '/usr/local/include/libgearman-1.0'
/usr/bin/install -c -m 644 libgearman-1.0/cancel.h libgearman-1.0/status.h lib gearman-1.0/actions.h libgearman-1.0/aggregator.h libgearman-1.0/allocator.h lib gearman-1.0/argument.h libgearman-1.0/client.h libgearman-1.0/client_callbacks.h libgearman-1.0/configure.h libgearman-1.0/connection.h libgearman-1.0/constants .h libgearman-1.0/core.h libgearman-1.0/execute.h libgearman-1.0/function.h libg earman-1.0/gearman.h libgearman-1.0/job.h libgearman-1.0/job_handle.h libgearman -1.0/kill.h libgearman-1.0/limits.h libgearman-1.0/ostream.hpp libgearman-1.0/pa rse.h libgearman-1.0/priority.h libgearman-1.0/protocol.h libgearman-1.0/result. h libgearman-1.0/return.h libgearman-1.0/signal.h libgearman-1.0/strerror.h libg earman-1.0/string.h libgearman-1.0/task.h libgearman-1.0/task_attr.h libgearman- 1.0/util.h libgearman-1.0/version.h libgearman-1.0/visibility.h libgearman-1.0/w orker.h '/usr/local/include/libgearman-1.0'
/bin/mkdir -p '/usr/local/include/libgearman'
/usr/bin/install -c -m 644 libgearman/gearman.h '/usr/local/include/libgearman '
/bin/mkdir -p '/usr/local/include/libgearman-1.0/interface'
/usr/bin/install -c -m 644 libgearman-1.0/interface/client.h libgearman-1.0/in terface/status.h libgearman-1.0/interface/task.h libgearman-1.0/interface/worker .h libgearman-1.0/interface/job.h '/usr/local/include/libgearman-1.0/interface'
/bin/mkdir -p '/usr/local/lib/pkgconfig'
/usr/bin/install -c -m 644 support/gearmand.pc '/usr/local/lib/pkgconfig'
make[2]: Leaving directory `/usr/servers/gearmand-1.1.12'
make[1]: Leaving directory `/usr/servers/gearmand-1.1.12'
```

5. 创建一下log目录
``` bash
mkdir -p /usr/local/var/log/
```

默认就在此目录下

6. 启动
``` bash
gearmand --verbose=INFO -d
```

启动gearmand守护，并打印level为INFO的log。gearmand的options可通过`gearmand -h`查看。


[1]:https://oscimg.oschina.net/oscnet/e0561d57147aae6892acec2100f166029b8.jpg
[2]: http://gearman.org/
[3]: https://code.google.com/p/java-gearman-service/
[4]: https://github.com/gearman/java-service
[5]: https://launchpad.net/gearman-java
[6]: http://code.johnewart.net/maven/org/gearman/gearman-server/
[7]: https://launchpadlibrarian.net/165674261/gearmand-1.1.12.tar.gz