---
title: Welcome home...
date: 2020-04-12 10:20:00
---

<center><font  color= #518FC1 size=6 class="ml3">AstonWang‘s Blog</font></center>
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/2.0.2/anime.min.js"></script>

<!-- 可选一言 -->

<center>
<font  color= #608DBD size=5>
<p id="hitokoto">
  <a href="#" id="hitokoto_text" target="_blank"></a>
</p>
<script>
  fetch('https://v1.hitokoto.cn')
    .then(response => response.json())
    .then(data => {
      const hitokoto = document.querySelector('#hitokoto_text')
      hitokoto.href = `https://hitokoto.cn/?uuid=${data.uuid}`
      hitokoto.innerText = data.hitokoto
    })
    .catch(console.error)
</script>
</font>
</center>


<div id="rcorners2" >
  <div id="rcorners1">
    <i class="fa fa-calendar" style="font-size:100"></i>
    <body>
      <font color="#4351AF">
      编程是一种技艺，一种需要用心学习的技艺
        <p class="p1"></p>
<script defer>
    //格式：2020年04月12日 10:20:00 星期二
    function format(newDate) {
        var day = newDate.getDay();
        var y = newDate.getFullYear();
        var m =
            newDate.getMonth() + 1 < 10
                ? "0" + (newDate.getMonth() + 1)
                : newDate.getMonth() + 1;
        var d =
            newDate.getDate() < 10 ? "0" + newDate.getDate() : newDate.getDate();
        var h =
            newDate.getHours() < 10 ? "0" + newDate.getHours() : newDate.getHours();
        var min =
            newDate.getMinutes() < 10
                ? "0" + newDate.getMinutes()
                : newDate.getMinutes();
        var s =
            newDate.getSeconds() < 10
                ? "0" + newDate.getSeconds()
                : newDate.getSeconds();
        var dict = {
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
            0: "天",
        };
        //var week=["日","一","二","三","四","五","六"]
        return (
            y +
            "年" +
            m +
            "月" +
            d +
            "日" +
            " " +
            h +
            ":" +
            min +
            ":" +
            s +
            " 星期" +
            dict[day]
        );
    }
    var timerId = setInterval(function () {
        var newDate = new Date();
        var p1 = document.querySelector(".p1");
        if (p1) {
            p1.textContent = format(newDate);
        }
    }, 1000);
</script>
      </font>
    </body>
  </div>
</div> 

<!-- *** -->
<!-- - 一切看似逝去的，都不曾离开，你所给予的爱与温暖，让我执着地守护着这里 ！     -->
<!-- - All problems in computer science can be solved by another level of indirection ! -->
<!-- *** -->

<!-- 
!!! info
    有人说：种一颗树，最好的时间是十年前，其次是现在... -->

=== "2024"
    - 2024-09-24 [常用软件工具集合](./toolbox/tools_software_dev_collection.md)
    - 2024-09-03 [ Kserve安装部署](./notehub/mlops/kserve_deploy.md)
    - 2024-08-23 [ Longhorn部署与使用](./notehub/cloud_native/k8s/longhorn_deploy.md)
    - 2024-08-21 [Docker镜像无法拉取问题的解决方案](./notehub/cloud_native/docker/docker_image_pull_problem_for_solution.md)
    - 2024-08-18 [k8s包管理工具Helm的使用](./notehub/cloud_native/k8s/helm_usage.md)
    - 2024-08-16 [K8S集群部署指南(v1.27.9)](./notehub/cloud_native/k8s/kubeadm_deploy_k8s_cluster_v1279.md)
    - 2024-07-31 [Python中不同场景中使用日志的方式](./python/pybestpractices/py_logger_usage.md)
    - 2024-07-30 [甄选GPT-AI工具](./toolbox/tools_ai.md)
    - 2024-07-17 [k8s实用工具之KtConnect](./notehub/cloud_native/k8s/ktconnect_usage.md) 
    - 2024-06-18 [常用插件集](./toolbox/tools_plugin.md)
    - 2024-06-15 [常用软件命令速查](./toolbox/tools_cheatsheet.md)

=== "2023"
    - 2023-11-01 [MLflow之模型生命周期管理](./python/pylibs/mlflow.md)
    - 2023-08-19 [学习应该自底向上，还是自顶向下](./thinking/posts/06_learn_top2down_or_bottom2up.md) 
    - 2023-08-15 [Flink(On YARN)安装部署](./toolbox/dev_software/flink_on_yarn.md)
    - 2023-08-15 [Spark(ON YARN）安装部署](./toolbox/dev_software/spark_on_yarn.md)
    - 2023-08-15 [YARN集群安装部署](./toolbox/dev_software/YARN.md)
    - 2023-08-15 [Hadoop安装部署](./toolbox/dev_software/hadoop.md)
    - 2023-08-14 [kafka安装部署](./toolbox/dev_software/kafka.md)
    - 2023-08-14 [Zookeeper安装部署](./toolbox/dev_software/zookeeper.md)
    - 2023-08-13 [Java环境安装](./toolbox/dev_software/java.md)
    - 2023-08-09 [PyFlink之分布式计算](./python/pylibs/pyflink.md)
    - 2023-08-02 [dockerignore文件](./notehub/cloud_native/docker/dockerignore_template.md)
    - 2023-08-01 [Go安全指南(摘录)](./golang/gobestpractices/go_security_guide.md)
    - 2022-07-28 [Mongodb的安装指南](./toolbox/dev_software/mongodb.md)
    - 2023-07-20 [PostgreSQL主从部署](./toolbox/dev_software/postgresql.md)
    - 2023-07-20 [Etcd集群部署](./toolbox/dev_software/etcd.md)
    - 2023-07-20 [Redis主从-哨兵部署](./toolbox/dev_software/redis.md)
    - 2023-07-20 [Nginx部署](./toolbox/dev_software/nginx.md)
    - 2023-07-19 [查看Docker容器启动的详细命令及参数](./notehub/cloud_native/docker/docker_detail_commands.md)
    - 2023-06-18 [Docker构建Python镜像实践指南](./python/pybestpractices/docker_build_python_image.md)
    - 2023-06-15 [个人战略上的聚焦和放弃](./thinking/posts/05_personal_focus_and_abandon.md) 
    - 2023-05-06 [Mac使用brew services报错](./notehub/solution/opshub/mac_brew_service_error_solution.md)
    - 2023-05-01 [Python版本及包管理工具选型](./python/pybestpractices/env_package_selection.md)
    - 2023-04-15 [gitflow的使用](./toolbox/git/gitflow_usage.md)
    - 2023-04-06 [ win10的cmd中操作git中文乱码解决方法](./notehub/solution/opshub/win10_git_chinese_solution.md)
    - 2023-04-02 [git常用软件安装](./toolbox/git/git_common_install.md)
    - 2023-03-31 [Go学习资料库](./golang/go_learn_docs.md)
    - 2023-03-28 [使用pypi进行Python包的发布](./python/pybestpractices/pypi_python_pkg_publish.md)
    - 2023-03-22 [pathlib之使用面向对象的方式操作路径](./python/pylibs/pathlib.md)
    - 2023-03-08 [git实用配置](./toolbox/git/git_config.md)
    - 2023-03-05 [git使用总结](./toolbox/git/git_usage_summary.md)
    - 2023-02-11 [Mac常用快捷键](./toolbox/macos_quickly_key.md)
    - 2023-02-08 [ Go在不同平台的安装及配置指南](./golang/gobestpractices/go_install_guide.md)
    

=== "2022"
    - 2022-12-12 [Python中的数据类型用法剖析](./python/pybestpractices/py_data_struct_summary.md)
    - 2022-11-10 [提问的智慧](./thinking/posts/04_wisdom_of_asking_questions.md)
    - 2022-10-18 [如何高效学习开源项目](./thinking/posts/03_learn_open_source_project.md) 
    - 2022-10-05 [Pandas之数据分析工具](./python/pylibs/pandas.md)
    - 2022-10-02 [Numpy之数据计算工具](./python/pylibs/numpy.md)
    - 2022-09-26 [Jupyter matplotlib画图中文字体缺失](./notehub/solution/bughub/jupyter_matplotlib_font_bug.md)
    - 2022-08-30 [Jupyter运行asyncio run报错](./notehub/solution/bughub/jupyter_asyncio_run_bug.md)
    - 2022-07-31 [甄选实用软件](./toolbox/tools_common.md)
    - 2022-07-28 [多语言之间的RSA加解密问题](./notehub/solution/bughub/rsa_encryption_bug.md)
    - 2022-07-18 [APScheduler之定时任务工具](./python/pylibs/apscheduler.md)
    - 2022-07-08 [Conda修改虚拟环境名导致异常](./notehub/solution/bughub/conda_rename_env_bug.md)
    - 2022-07-06 [Centos的yum源误删解决方法](./notehub/solution/opshub/centos_yum_source_delete_solution.md)
    - 2022-06-19 [Jupyter系列工具的使用总结](./python/pybestpractices/jupyter_tool_usage.md)
    - 2022-05-29 [Supervisor之进程管理工具](./python/pylibs/supervisor.md)
    - 2022-05-01 [技术一言](./thinking/posts/02_tech_yiyan.md)
    - 2022-04-25 [docker-compose常用软件安装汇总](./notehub/cloud_native/dockercompose/docker_compose_software_usage.md)
    - 2022-04-21 [docker-compose使用总结](./notehub/cloud_native/dockercompose/docker_compose_summary.md)
    - 2022-04-15 [Dockerfile常用命令与编写指南](./notehub/cloud_native/docker/dockerifle_guide.md)
    - 2022-04-12 [subprocess之子进程管理](./python/pylibs/subprocess.md)
    - 2022-04-10 [docker-registry搭建与使用](./notehub/cloud_native/docker/docker_registry_usage.md)
    - 2022-04-04 [Docker常用命令速查](./notehub/cloud_native/docker/docker_command.md)
    - 2022-04-02 [Docker安装常用软件汇总](./notehub/cloud_native/docker/docker_common_software_install.md)
    - 2022-04-01 [Docker实用技巧](./notehub/cloud_native/docker/docker_practical_skills.md)
    - 2022-04-01 [Docker安装卸载指南](./notehub/cloud_native/docker/docker_install_guide.md)
    - 2022-03-27 [Docker架构和原理学习](./notehub/cloud_native/docker/docker_arch_and_principle.md)
    - 2022-03-22 [Conda的实践应用](./python/pybestpractices/conda_usage.md)
    - 2022-02-06 [服务器磁盘空间满的解决办法](./notehub/solution/opshub/server_disk_full_solution.md)
    - 2022-02-04 [Go的指针和结构体](./golang/gosyntax/go_point.md)
    - 2022-02-03 [如何更聪明的做事情](./thinking/posts/01_smart_to_do_things.md) 
    - 2022-02-03 [Go的容器数据类型](./golang/gosyntax/go_data_type.md)
    - 2022-02-02 [Go的基本数据类型](./golang/gosyntax/go_data_type.md)
    - 2022-02-01 [Go的基本知识点](./golang/gosyntax/go_basics.md)
    - 2022-01-27 [Go环境安装和配置](./golang/gobestpractices/go_install_guide_2.md)
    - 2022-01-26 [Go编码规范](./golang/gobestpractices/go_code_standard.md)
    - 2022-01-13 [常用pypi源汇总及使用](./python/pybestpractices/pypi_source_usage.md)
    - 2022-01-05 [ window环境中pip安装报错](./notehub/solution/bughub/window_install_pip_bug.md)
    - 2022-01-02 [Jupyterhub离线部署踩坑总结](./notehub/solution/bughub/jupyterhub_deploy_bug.md)

=== "2021"
    - 2021-12-21 [Python安全指南(摘录)](./python/pybestpractices/py_security_guide.md)
    - 2021-11-06 [Pickle之数据序列化工具的使用](./python/pylibs/pickle.md)
    - 2021-10-06 [RabbitMQ介绍和使用](./toolbox/dev_software/rabbitmq.md)
    - 2021-08-09 [Zabbix常用配置](./toolbox/dev_software/zabbix.md)
    - 2021-07-17 [Python常用开发工具](./python/pybestpractices/py_dev_tool_selection.md)
    - 2021-07-12 [k8s中etcd挂载内存卷步骤](./notehub/cloud_native/k8s/k8s_ectd_mount_memo.md)
    - 2021-07-10 [k8s的storageclass-nfs安装和使用](./notehub/cloud_native/k8s/k8s_storageclass_nfs.md)
    - 2021-07-08 [k8s的Dashboard安装](./notehub/cloud_native/k8s/k8s_dashboard.md)
    - 2021-03-22 [k8s部署常见问题汇总](./notehub/cloud_native/k8s/k8s_common_qa.md)
    - 2021-03-21 [k8s中yaml文件的编写和使用](./notehub/cloud_native/k8s/yaml_usage.md)
    - 2021-03-18 [k8s实用小技巧](./notehub/cloud_native/k8s/k8s_usage_skills.md)
    - 2021-03-18 [k8s核心概念](./notehub/cloud_native/k8s/k8s_core_name.md)
    - 2021-03-09 [k8s常用命令总结](./notehub/cloud_native/k8s/k8s_common_cmd.md)
    - 2021-03-06 [django源码学习之LazyObject](./python/django/django_lazyobject_analysis.md)
    - 2021-03-02 [k8s集群部署指南(v1.19.4)](./notehub/cloud_native/k8s/kubeadm_deploy_k8s_cluster_v1194.md) 

=== "2020"
    - 2020-12-22 [Python2到Python3的变化](./python/pybestpractices/py2_py3_update.md)
    - 2020-11-23 [Python编码规范](./python/pybestpractices/coding_standard.md)
    - 2020-10-13 [django中使用websocket](./python/django/django_websocket_usage.md)
    - 2020-10-05 [django中ORM查询方法总结](./python/django/django_orm_query.md)
    - 2020-10-04 [django中ORM字段总结](./python/django/django_orm_fields.md)
    - 2020-09-02 [django中类视图的使用](./python/django/django_class_view_usage.md)
    - 2020-09-01 [django中间件的使用](./python/django/django_middleware_usage.md)
    - 2020-08-23 [django批量导入数据](./python/django/django_bulk_create.md)
    - 2020-08-21 [django1.7以前的版本执行数据库迁移](./python/django/django_17_db_migrate.md)
    - 2020-04-05 [Python之collections的用法](./python/pylibs/collections.md)

=== "2019"
    - 2019-06-09 [python中的GC](./python/pysyntax/py_gc.md)
    - 2019-06-07 [python中的内存管理](./python/pysyntax/py_mem_manage.md)
    - 2019-06-06 [python中的GIL](./python/pysyntax/py_gc.md)
    - 2019-05-26 [python中的Futures](./python/pysyntax/futures.md)
    - 2019-05-20 [Python中的Asyncio](./python/pysyntax/asyncio.md)
    - 2019-05-18 [python中is和equals的区别](./python/pysyntax/is_and_equals.md)
    - 2019-05-15 [python中的面向对象编程OOP](./python/pysyntax/oop.md)
    - 2019-05-10 [def函数与lambda函数](./python/pysyntax/def_and_lambda.md)
    - 2019-05-10 [python中的property](./python/pysyntax/property.md)
    - 2019-05-09 [迭代器iterator用法](./python/pysyntax/iterator.md)
    - 2019-05-09 [生成器generator用法](./python/pysyntax/generator.md)
    - 2019-05-07 [装饰器(decorator)用法](./python/pysyntax/decorator.md)
    - 2019-05-06 [闭包(closure)用法](./python/pysyntax/closure.md)
    - 2019-05-03 [浅拷贝与深拷贝](./python/pysyntax/deep_copy.md)
    - 2019-05-03 [python异常处理](./python/pysyntax/exception.md)
    - 2019-05-02 [python之容器类型](./python/pysyntax/list_tuple_dict.md)
    - 2019-05-01 [python变量与注释](./python/pysyntax/variable_comment.md)
    - 2019-05-01 [Python基础知识梳理](./python/pysyntax/py_basics.md)
   