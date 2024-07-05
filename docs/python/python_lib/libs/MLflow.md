# MLflow-> 模型生命周期管理

## MLFlow概述

### 适用场景

- 产生背景

  每个做过机器学习开发的人都知道机器学的复杂性，除了软件中常见的问题之外，机器学习还存在很多新的挑战。作为一家大数据解决方案公司，Databricks与数百家使用机器学习的公司合作，所以能够清楚地了解他们的痛点，比如工具太过复杂、难以跟踪实验、难以重现结果、难以部署模型。由于这些挑战的存在，机器学习开发必须变得与传统软件开发一样强大、可预测和普及。为此，很多企业已经开始构建内部机器学习平台来管理机器学习生命周期。例如，Facebook、谷歌和优步分别构建了 FBLearner Flow、TFX 和 Michelangelo 来进行数据准备、模型训练和部署。但这些内部平台存在一定的局限性：典型的机器学习平台只支持一小部分内置算法或单个机器学习库，并且被绑定在公司内部的基础设施上。用户无法轻易地使用新的机器学习库，或与社区分享他们的工作成果。Databricks认为应该使用一种**更好的方式来管理机器学习生命周期**，于是他们推出了ML flow，一个开源的机器学习平台。

- 基本情况

  MLflow的灵感来源于现有的机器学习平台，但以开放性作为主要设计目标：

  - **开放接口**：MLflow可与任何机器学习库、算法、部署工具或编程语言一起使用。它基于REST API和简单的数据格式（例如，可将模型视为lambda函数）而构建，可以使用各种工具，而不只是提供一个小部分内置功能。用户可以很容易地将MLflow添加到现有地机器学习代码中，并在组织中共享代码，让其他人也能运行这些代码。
  - **开源**：MLflow是一个开源项目，用户和机器学习库开发人员可以对其进行扩展。此外，利用MLflow地开放格式，可以轻松地跨组织共享工作流步骤和模型。

- 应用场景

  MLflow与库无关。 您可以将其与任何机器学习库以及任何编程语言一起使用，因为所有功能都可以通过REST API和CLI进行访问。 为了方便起见，该项目还包括Python API，R API和Java API。

### 功能特点

- MLflow 是一个用于管理端到端机器学习生命周期的开源平台。它实现了以下四个主要功能：
  - **MLflow Tracking：**跟踪、记录实验过程，交叉比较实验参数和对应的结果。
  - **MLflow Project：**把代码打包成可复用、可复现的格式，可用于成员分享和针对线上部署。
  - **MLflow Models：**管理、部署来自多个不同机器学习框架的模型到大部分模型部署和推理平台。
  - **MLflow Model Registry：**针对模型的全生命周期管理的需求，提供集中式协同管理，包括模型版本管理、模型状态转换、数据标注。

## 安装部署

- 安装python环境和pip，并利用pip安装MLflow：

```shell
pip install mlflow
```

- 在MacOS上使用MLflow
  - 如果使用MacOS默认的系统Python遇到问题，可以通过[Homebrew](https://brew.sh/)包管理器安装Python3，命令如下：

```shell
brew install python
```

- 这种情况下，安装MLflow的命令为：

```shell
pip3 install mlflow
```

- 备注
  - 为了使用特定的MLflow模块和功能（如ML模型持久化/接口，工件存储选项等），需要安装额外的库依赖。例如，mlflow.tensorflow模块需要安装TensorFlow。详细情况见：<https://github.com/mlflow/mlflow/blob/master/EXTRA_DEPENDENCIES.rst>

## API清单

- [MLflow Python API](https://mlflow.org/docs/latest/python_api/index.html)

| module             | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| mlflow             | 该模块为开启和管理一个MLflow run提供高层级的API              |
| mlflow.tracking    | 该模块为MLflow的experiments和runs提供基于python的CRUD接口。这是比mlflow模块封装层级更低的API，能直接转到REST API的调用。若需要管理一个管理“active run”的高层级API，请使用mlflow模块。 |
| mlflow.projects    | 该模块为运行本地或者远程的MLflow project提供API              |
| mlflow.models      | 为机器学习模型的保存提供接口，这些模型用一种下游工具可以理解的”flavors“方式保存。 |
| mlflow.statsmodels | 该模块为记录和加载statsmodels models提供API。                |
| mlflow.pyfunc      | python_function模块flavor充当MLflow Python模型的默认模型接口。任何MLflow Python模型都会被加载为python_function模型。 |
| mlflow.keras       | 该模块为记录和加载Keras模型提供API                           |
| mlflow.pytorch     | 该模块为记录和加载PyTorch模型提供API                         |
| mlflow.sklearn     | 该模块为记录和加载scikit-learn模型提供API                    |
| mlflow.tensorflow  | 该模块为记录和加载TensorFlow模型提供API                      |

### mlflow模块

- 常用函数

| 函数                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| active_run                                                   | 获取当前活跃的Run对象实例，若没有则返回None                  |
| create_experiment(name, artifact_location=None)              | 创建一个MLflow实验                                           |
| delete_experiment(experiment_id)                             | 从后台中删除一个实验                                         |
| delete_run(run_id)                                           | 删除一个run对象实例                                          |
| delete_tag(key)                                              | 删除一个run对象的tag                                         |
| end_run(status='FINISHED')                                   | 结束一个活跃的MLflow run对象                                 |
| get_artifact_uri(artifact_path=None)                         | 获取当前运行的run对象中artifact_path指定的artifact的绝对URI，若artifact_path为None，则返回artifact的根URI |
| get_experiment(experiment_id)                                | 从后台存储中获取experiment_id指定的experiment对象            |
| get_experiment_by_name(name)                                 | 从后台存储中获取name指定的experiment对象                     |
| get_registry_uri                                             | 获取当前注册表的URI，默认返回tracking URI                    |
| get_run(run_id)                                              | 后台存储中获取run对象实例                                    |
| get_tracking_uri                                             | 获取当前的tracking URI                                       |
| list_run_infos(experiment_id, run_view_type=1, max_results=1000, order_by=None) | 返回experiment_id指定实验下的所有run实例的信息               |
| log_artifact(local_path, artifact_path=None)                 | 将本地文件或目录记录为当前活跃run对象的artifact              |
| log_artifacts(local_dir, artifact_path=None)                 | 记录本地目录的所有的内容作为run对象的artifacts               |
| log_metric(key, value, step=None)                            | 记录当前run对象的一个metric                                  |
| log_metrics(metrics, step=None)                              | 记录当前run对象的多个metric                                  |
| log_param(key, value)                                        | 记录当前run对象的一个parameter                               |
| log_params(params)                                           | 记录当前run对象的一组parameters                              |
| log_text(text, artifact_file)                                | 记录text内容作为一个artifact                                 |
| register_model(model_uri, name, await_registration_for=300)  | 在模型注册表中为model_uri指定的模型文件创建新的模型版本      |
| run(uri, entry_point='main', version=None, parameters=None, docker_args=None, experiment_name=None, experiment_id=None, backend='local', backend_config=None, use_conda=True, storage_dir=None, synchronous=True, run_id=None) | 运行一个MLflow项目，这个项目可以在本地或者在GitHub上。       |
| search_runs(experiment_ids=None, filter_string='', run_view_type=1, max_results=100000, order_by=None, output_format='pandas') | 以pandas数据框架类型获取满足查询条件的所有run对象            |
| set_experiment(experiment_name)                              | 设置指定名称的experiment对象                                 |
| set_registry_uri(uri)                                        | 设置注册表服务器的URI                                        |
| set_tag(key, value)                                          | 为当前run对象设置标签                                        |
| set_tags(tags)                                               | 为当前run对象设置一组标签                                    |
| set_tracking_uri(uri)                                        | 设置tracking服务器的URI                                      |
| start_run(run_id=None, experiment_id=None, run_name=None, nested=False, tags=None) | 开启一个新的MLflow run对象实例                               |
       |

### mlflow.tracking模块

该模块为MLflow的experiments和runs提供基于python的CRUD接口。这是比mlflow模块封装层级更低的API，能直接转到REST API的调用。

- **class** mlflow.tracking.MLflowClient(tracking_uri=None, registry_uri=None)

  MLflow Tracking服务器的客户端，可以创建和管理experiments和run，也是MLflow Registry服务器的客户端，可以创建和管理已注册的模型和模型版本。

- MLflowClient的一些方法/函数

  | Functions                                                    | Description                            |
  | ------------------------------------------------------------ | -------------------------------------- |
  | **experiment相关：**                                         |                                        |
  | create_experiment(name, artifact_location=None)              | 创建一个实验                           |
  | delete_experiment(experiment_id)                             | 在后台删除一个实验                     |
  | get_experiment(experiment_id)                                | 通过ID获取实验                         |
  | get_experiment_by_name(name)                                 | 通过名字获取实验                       |
  | rename_experiment(experiment_id, new_name)                   | 更新实验名字                           |
  | restore_experiment(experiment_id)                            | 存储一个已删除实验，除非永久性删除     |
  | set_experiment_tag(experiment_id, key, value)                | 给实验设置标签(字典类型)               |
  |                                                              |                                        |
  | **model_version相关：**                                      |                                        |
  | create_model_version(name, source, run_id=None, tags=None, run_link=None, description=None, await_creation_for=300) | 从给定资源中创建一个新的模型版本       |
  | delete_model_version(name, version)                          | 删除模型版本                           |
  | delete_model_version_tag(name, version, key)                 | 删除模型版本的tag                      |
  | get_model_version(name, version)                             | 返回模型版本对象                       |
  | get_model_version_download_uri(name, version)                | 获取该模型版本存储仓库的下载地址       |
  | get_model_version_stages(name, version)                      | 返回一个合法阶段集合                   |
  | search_model_versions(filter_string)                         | 查找满足过滤条件的模型版本             |
  | set_model_version_tag(name, version, key, value)             | 给模型版本设置标签                     |
  | transition_model_version_stage(name, version, stage, archive_existing_versions=False) | 更新模型版本阶段                       |
  | update_model_version(name, version, description=None)        | 更新模型版本的元数据                   |
  |                                                              |                                        |
  | **registered_model相关：**                                   |                                        |
  | create_registered_model(name, tags=None, description=None)   | 在后台创建一个新的已注册模型           |
  | delete_registered_model(name)                                | 删除已注册模型                         |
  | delete_registered_model_tag(name, key)                       | 删除已注册模型的tag                    |
  | get_registered_model(name)                                   | 返回注册模型对象                       |
  | rename_registered_model(name, new_name)                      | 更新已注册模型名字                     |
  | search_registered_models(filter_string=None, max_results=100, order_by=None, page_token=None) | 查找满足过滤条件的已注册模型           |
  | set_registered_model_tag(name, key, value)                   | 给已注册模型设置标签(字典类型)         |
  | update_registered_model(name, description=None)              | 更新RegisteredModel实体的元数据        |
  |                                                              |                                        |
  | **run相关：**                                                |                                        |
  | create_run(experiment_id, start_time=None, tags=None)        | 创建一个run对象                        |
  | delete_run(run_id)                                           | 删除一个run对象                        |
  | delete_tag(run_id, key)                                      | 删除run对象的标签                      |
  | get_run(run_id)                                              | 获取后台存储的run对象                  |
  | restore_run(run_id)                                          | 存储一个给定ID的已删除的run            |
  | search_runs(experiment_ids, filter_string='', run_view_type=1, max_results=1000, order_by=None, page_token=None) | 查找满足条件的run对象                  |
  | set_tag(run_id, key, value)                                  | 给指定id的run设置标签                  |
  | set_terminated(run_id, status=None, end_time=None)           | 将一个run的状态设置已中止              |
  |                                                              |                                        |
  | **其他函数：**                                               |                                        |
  | download_artifacts(run_id, path, dst_path=None)              | 将run对象的工件或者目录下载到本地      |
  | get_latest_versions(name, stages=None)                       | 获取模型每个请求阶段的最新版本         |
  | get_metric_history(run_id, key)                              | 返回给定metric记录的所有值             |
  | list_artifacts(run_id, path=None)                            | run对象的artifacts集合                 |
  | list_experiments(view_type=None)                             | Experiment对象集合                     |
  | list_registered_models(max_results=100, page_token=None)     | 所以已注册模型的集合                   |
  | list_run_infos(experiment_id, run_view_type=1, max_results=1000, order_by=None, page_token=None) | RunInfo对象集合                        |
  | log_artifact(run_id, local_path, artifact_path=None)         | 在远程artifact_uri中写入本地文件或目录 |
  | log_artifacts(run_id, local_dir, artifact_path=None)         | 在远程artifact_uri中写入文件目录       |
  | log_batch(run_id, metrics=(), params=(), tags=())            | 记录多个metrecs、params、tags          |
  | log_metric(run_id, key, value, timestamp=None, step=None)    | 在给定id的run对象中记录一个metric      |
  | log_param(run_id, key, value)                                | 在给定id的run对象中记录一个param       |
  | log_text(run_id, text, artifact_file)                        | 将text文件记录为一个artifact           |

### moflow.projects模块

该模块为运行本地或者远程的MLflow project提供API。主要函数如下：

```python
mlflow.projects.run(uri, entry_point='main', version=None, parameters=None, docker_args=None, experiment_name=None, experiment_id=None, backend='local', backend_config=None, use_conda=True, storage_dir=None, synchronous=True, run_id=None)
```

- 运行一个MLflow project。这个项目可以在本地也可以存储在一个Git URI中。

- **参数：**

  | Parameters      | Description                                                  |
  | --------------- | ------------------------------------------------------------ |
  | uri             | 所要运行项目的uri。可以是本地或者Git仓库                     |
  | entry_point     | 项目运行的入口点                                             |
  | version         | 基于Git项目的版本                                            |
  | parameters      | 入口点命令的参数（字典类型）                                 |
  | docker_args     | docker命令的参数（字典类型）                                 |
  | experiment_name | 运行项目的实验名字                                           |
  | experiment_id   | 运行项目的实验ID                                             |
  | backend         | 运行后台，MLflow提供内置支持的后台有“local"，”databricks“，”kubernetes“ |
  | backend_config  | JSON文件的路径，这个JSON文件将用于配置后台                   |
  | use_conda       | 若为True，则创建新的conda环境，若为False，则使用当前conda环境。默认为True |
  | storage_dir     | 仅当后台为”local“时可用。artifacts存储位置                   |
  | synchronous     | 其他项目运行完成之前是否阻塞。 默认为True                    |
  | run_id          | 若指定，则MLflow会在指定ID的run下运行项目，而不是创建新的run对象。 |

- **返回值：**

  包含所运行项目相关信息（例如run id）的[mlflow.projects.SubmittedRun](https://mlflow.org/docs/latest/python_api/mlflow.projects.html#mlflow.projects.SubmittedRun)对象。

### moflow.models模块

​	为机器学习模型的保存提供接口，这些模型用一种下游工具可以理解的”flavors“方式保存。

- 内置”flavors“：

  | built-in flavors   |
  | ------------------ |
  | mlflow.pyfunc      |
  | mlflow.keras       |
  | mlflow.lightgbm    |
  | mlflow.pytorch     |
  | mlflow.sklearn     |
  | mlflow.spark       |
  | mlflow.statsmodels |
  | mlflow.tensorflow  |
  | mlflow.xgboost     |
  | mlflow.spacy       |
  | mlflow.fastai      |

- 具体的调用函数在相关依赖中，例如，保存一个sklearn模型，首先引入mlflow.sklearn依赖包，再调用mlflow.sklearn.autolog()函数或者mlflow.sklearn.load_model()函数。具体需参见对应的依赖包。