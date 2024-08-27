---
title: k8s中yaml文件的编写和使用
date: 2021-03-21 20:46:32
tags: 
  - K8S
---
## YAML基本语法
- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格。
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- `#` 表示注释，从这个字符一直到行尾，都会被解析器忽略。

### Maps
- Map 是字典，就是一个`key:value`的键值对，例如：

```yaml
---
apiVersion: v1
kind: Pod

#######PS##########
# 第一行的`---`是分隔符，是可选的，在单一文件中，可用连续三个连字号`---`区分多个文件
# 转为JSON则为：
{
    "apiVersion": "v1",
    "kind": "pod"
}
#######PS##########
```
我们在创建一个相对复杂一点的 YAML 文件，创建一个 KEY 对应的值不是字符串而是一个 Maps：
```
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
```
- 用了两个空格作为缩进，空格的数量并不重要，但是得保持一致，并且至少要求一个空格**（什么意思？就是你别一会缩进两个空格，一会缩进4个空格）。
- 在 YAML 文件中绝对不要使用 tab 键。

同样的将上面的 YAML 文件转换成 JSON 文件：
```JSON
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "kube100-site",
    "labels": {
      "app": "web"
    }
  }
}
```
### Lists
Lists 就是列表，说白了就是数组，在 YAML 文件中可以这样定义：
```YAML
args
  - Cat
  - Dog
  - Fish
```
- 可以有任何数量的项在列表中，每个项的定义以破折号（-）开头的，与父元素直接可以缩进一个空格。对应的 JSON 格式如下：

```JSON
{
    "args": ["Cat", "Dog", "Fish"]
}
```


list 的子项也可以是 Maps，Maps 的子项也可以是list如下所示：

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: jcdemo/flaskapp
      ports:
        - containerPort: 5000
```
我们可以转成如下 JSON 格式文件：
```JSON
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "name": "kube100-site",
        "labels": {
            "app": "web"
        }
    },
    "spec": {
        "containers": [{
            "name": "front-end",
            "image": "nginx",
            "ports": [{
                "containerPort": 80
            }]
        }, {
            "name": "flaskapp-demo",
            "image": "jcdemo/flaskapp",
            "ports": [{
                "containerPort": 5000
            }]
        }]
    }
}
```
### 字面量与折叠样式

对于较长的字符串或者列表，YAML提供了两种不同的表示方式：字面量样式(literal style)和折叠样式(folded style)。

- **字面量样式** (使用 `|` 符号):

  ```YAML
  description: |
    This is a long description that spans multiple lines.
    It can include indents and newlines which will be preserved.
  ```

- **折叠样式** (使用 `>` 符号):

  ```YAML
  description: >
    This is a long description that spans multiple lines.
    It can include indents and newlines which will be folded into a single line.
  ```

### 锚点与别名

锚点(anchor)和别名(alias)可以帮助避免重复的数据。

- **锚点** (使用 `&name` 定义):

  ```YAML
  &baseImage
  image: nginx
  tag: 1.19.2
  ```

- **别名** (使用 `*name` 引用):

  ```YAML
  containers:
    - name: nginx-container
      <<: *baseImage
    - name: another-container
      <<: *baseImage
  ```

### 环境变量

在YAML文件中，可以使用环境变量来动态填充某些字段。这对于需要在部署时根据实际情况变化的值非常有用。

```yaml
image: ${MY_REGISTRY}/nginx:${NGINX_VERSION}
```

### 条件表达式

虽然YAML本身不支持条件逻辑，但可以通过使用模板引擎（如Jinja2）结合YAML来实现复杂的条件逻辑。
