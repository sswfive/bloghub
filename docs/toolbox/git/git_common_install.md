---
title: git常用软件安装
date: 2023-04-02 08:36:58
tags: 
  - Git
  - Tools
---
## lfs安装：

>  支持大文件提交

##### 1、安装

```
yum install git-lfs
apt install git-lfs
git lfs install
```

##### 2、标记大文件

```
git lfs track kernel_resource/4.19.37-rt19.arm64.gz
git lfs track *.gz
```

##### 3、推送

```
git add my.gz
 git commit -m "add gz file"
 git push origin master
```

##### 4、查看

```
git lfs track
git lfs ls-files
vim .gitattributes
```

## gitlab安装与配置

##### 1、拉取gitlab容器镜像

```
docker pull gitlab/gitlab-runner:latest
```

##### 2、运行gitlab

```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
mkdir -p /home/hl/gitlabrunner/DeltaOS/opt /home/hl/gitlabrunner/DeltaOS/builds
docker run -d --name deltaos-gitlab-runner --restart always \
  --privileged=true \
  -v /home/hl/gitlabrunner/DeltaOS/opt:/opt \
  -v /home/hl/gitlabrunner/DeltaOS/builds:/home/gitlab-runner/builds \
  gitlab/gitlab-runner:latest
docker exec deltaos-gitlab-runner gitlab-runner register \
 --non-interactive \
 --name my-runner \
 --url http://192.168.1.1:8000/ \
 --registration-token sDrr7KDi6UXyzwuzPqx- \
 --executor shell \
 --tag-list \
 common-runner
```

##### 3、容器内

```
mknod -m 0660 /dev/loop1 b 7 1
echo "192.168.21.8 gitlab.tmp.com" |tee >> /etc/hosts
#添加root权限
sed -i 's/gitlab-runner:x:999:999/gitlab-runner:x:0:0/g' /etc/passwd
```

##### 4、编辑gitlab-ci

```
vim .gitlab-ci.yml
stages:
  - deploy
deploy:
    stage: deploy
    script:
      - bash /opt/deploy.sh
    only:
      - master
    tags:
      - common-runner
when 可以设置为以下值之一：
on_success - 只有当前一个阶段的所有工作成功时才执行工作。这是默认值。
on_failure - 仅当前一个阶段的至少一个作业发生故障时才执行作业。
always - 无论前一阶段的工作状况如何，执行工作。
manual - 手动执行作业
stages:
 - build
 - test
 - deploy
首先，所有build的jobs都是并行执行的。
所有build的jobs执行成功后，test的jobs才会开始并行执行。
所有test的jobs执行成功，deploy的jobs才会开始并行执行。
所有的deploy的jobs执行成功，commit才会标记为success
任何一个前置的jobs失败了，commit会标记为failed并且下一个stages的jobs都不会执行。
有时，script命令需要用单引号或双引号括起来。例如，包含冒号（:）的命令需要用引号括起来，以便YAML解析器知道将整个事物解释为字符串而不是“key：value”对。使用特殊字符时要小心：:，{，}，[，]，,，&，*，#，?，|，-，<，>，=，!，%，@，`。
例如：
stages:
- build
- cleanup_build
- test
- deploy
- cleanup
build_job:
  stage: build
  script:
  - make build
cleanup_build_job:
  stage: cleanup_build
  script:
  - cleanup build when failed
  when: on_failure
test_job:
  stage: test
  script:
  - make test
deploy_job:
  stage: deploy
  script:
  - make deploy
  when: manual
cleanup_job:
  stage: cleanup
  script:
  - cleanup after jobs
  when: always
以上脚本将：
cleanup_build_job仅在build_job失败时执行。
始终执行cleanup_job作为流水线的最后一步，无论成功或失败。
允许您deploy_job从GitLab的UI 手动执行。
```

##### 5、界面查找url和token

```
Settings —> Pipelines —> Specific Runners
```

##### 6、注册

```
gitlab-runner register --non-interactive --name my-runner --url http://192.168.21.8:8000/ --registration-token Kgu3z28pqfZEZsBmquUh --executor shell --tag-list common-runner
vim /etc/passwd
gitlab-runner:x:0:0:GitLab Runner:/home/gitlab-runner:/bin/bash
#手动注册
docker exec -it gitlab-runner gitlab-ci-multi-runner register
#引导会让你输入gitlab的url，输入自己的url，例如http://gitlab.example.com/
#引导会让你输入token，去相应的项目下找到token，例如ase12c235qazd32
#引导会让你输入tag，一个项目可能有多个runner，是根据tag来区别runner的，输入若干个就好了，比如web,hook,deploy
#引导会让你输入executor，这个是要用什么方式来执行脚本，图方便输入shell就好了。
```

##### 7、查看gitlab-runner

```
gitlab-runner list
```

##### 8、宿主机安装

```
yum 安装
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | bash
yum install -y gitlab-ci-multi-runner
echo '[gitlab-ci-multi-runner]
name=gitlab-ci-multi-runner
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ci-multi-runner/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key' |tee >> /etc/yum.repos.d/gitlab-ci-multi-runner.repo
yum makecache
yum install -y gitlab-ci-multi-runner
#设置gitlab-runner root权限
gitlab-runner:x:0:0:GitLab Runner:/home/gitlab-runner:/bin/bash
```

##### 9、其他

```
jenkins
http://blog.csdn.net/abcdocker/article/details/53840629
https://docs.gitlab.com/ee/ci/yaml/README.html
```

## 