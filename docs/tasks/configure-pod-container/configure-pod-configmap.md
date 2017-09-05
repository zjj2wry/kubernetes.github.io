<!--
---
title: Use ConfigMap Data in Pods 
---

{% capture overview %}
This page provides a series of usage examples demonstrating how to configure Pods using data stored in ConfigMaps. 
{% endcapture %}

{% capture prerequisites %}
* {% include task-tutorial-prereqs.md %}
* [Create a ConfigMap](/docs/tasks/configure-pod-container/configmap/)
{% endcapture %}

{% capture steps %}
-->

---
标题: 在Pod里使用ConfigMap数据
---

{% capture overview %}
这篇教程提供了一系列的例子，展示如何配置Pod来使用ConfigMaps里面的数据。
{% endcapture %}

{% capture prerequisites %}
* {% include task-tutorial-prereqs.md %}
* [创建一个ConfigMap](/docs/tasks/configure-pod-container/configmap/)
{% endcapture %}

{% capture steps %}

<!--
## Define Pod environment variables using ConfigMap data

### Define a Pod environment variable with data from a single ConfigMap

1. Define an environment variable as a key-value pair in a ConfigMap:

   ```shell
   kubectl create configmap special-config --from-literal=special.how=very 
   ```

1. Assign the `special.how` value defined in the ConfigMap to the `SPECIAL_LEVEL_KEY` environment variable in the Pod specification.  

   ```shell
   kubectl edit pod dapi-test-pod
   ```

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         env:
           # Define the environment variable
           - name: SPECIAL_LEVEL_KEY
             valueFrom:
               configMapKeyRef:
                 # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
                 name: special-config
                 # Specify the key associated with the value
                 key: special.how
     restartPolicy: Never
   ```

1. Save the changes to the Pod specification. Now, the Pod's output includes `SPECIAL_LEVEL_KEY=very`. 
-->

## 使用ConfigMap数据来定义Pod环境变量

### 从单个ConfigMap提取数据定义Pod的环境变量

1. 在ConfigMap里将环境变量定义为键值对：

   ```shell
   kubectl create configmap special-config --from-literal=special.how=very 
   ```

1. 将ConfigMap里的`special.how`值赋给Pod的环境变量`SPECIAL_LEVEL_KEY`.  

   ```shell
   kubectl edit pod dapi-test-pod
   ```

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         env:
           # Define the environment variable
           - name: SPECIAL_LEVEL_KEY
             valueFrom:
               configMapKeyRef:
                 # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
                 name: special-config
                 # Specify the key associated with the value
                 key: special.how
     restartPolicy: Never
   ```

1. 将修改保存到Pod的配置里，现在Pod的输出会包含`SPECIAL_LEVEL_KEY=very`这么一行. 

<!--
### Define Pod environment variables with data from multiple ConfigMaps
 
1. As with the previous example, create the ConfigMaps first.

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: special-config
     namespace: default
   data:
     special.how: very
   ```

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-config
     namespace: default
   data:
     log_level: INFO
   ``` 

1. Define the environment variables in the Pod specification.   

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         env:
           - name: SPECIAL_LEVEL_KEY
             valueFrom:
               configMapKeyRef:
                 name: special-config
                 key: special.how
           - name: LOG_LEVEL
             valueFrom:
               configMapKeyRef:
                 name: env-config
                 key: special.type
     restartPolicy: Never
   ```
 
1. Save the changes to the Pod specification. Now, the Pod's output includes `SPECIAL_LEVEL_KEY=very` and `LOG_LEVEL=info`. 
-->

### 从多个ConfigMap读取数据来定义Pod环境

1. 跟前面的例子一样，我们先创建ConfigMap。

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: special-config
     namespace: default
   data:
     special.how: very
   ```

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-config
     namespace: default
   data:
     log_level: INFO
   ``` 

1. 在Pod配置里定义环境变量.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         env:
           - name: SPECIAL_LEVEL_KEY
             valueFrom:
               configMapKeyRef:
                 name: special-config
                 key: special.how
           - name: LOG_LEVEL
             valueFrom:
               configMapKeyRef:
                 name: env-config
                 key: special.type
     restartPolicy: Never
   ```
 
1. 保存Pod的配置，现在Pod的输出会包括`SPECIAL_LEVEL_KEY=very`和`LOG_LEVEL=info`.

<!--
## Configure all key-value pairs in a ConfigMap as Pod environment variables 

Note: This functionality is available to users running Kubernetes v1.6 and later.

1. Create a ConfigMap containing multiple key-value pairs. 

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: special-config
     namespace: default
   data:
     SPECIAL_LEVEL: very
     SPECIAL_TYPE: charm
   ```

1. Use `env-from` to define all of the ConfigMap's data as Pod environment variables. The key from the ConfigMap becomes the environment variable name in the Pod.
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         envFrom:
         - configMapRef:
             name: special-config
     restartPolicy: Never
   ```

1. Save the changes to the Pod specification. Now, the Pod's output includes `SPECIAL_LEVEL=very` and `SPECIAL_TYPE=charm`. 
-->

## 将所有ConfigMap的键值对都设置为Pod的环境变量

注意: 这个功能只适用于使用Kubernetes 1.6及以上版本的环境.

1. 创建一个包含多个键值对的ConfigMap。

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: special-config
     namespace: default
   data:
     SPECIAL_LEVEL: very
     SPECIAL_TYPE: charm
   ```

1. 使用`env-from`来读取ConfigMap的所有数据作为Pod的环境变量。ConfigMap的键名作为环境变量的变量名。
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: dapi-test-pod
   spec:
     containers:
       - name: test-container
         image: gcr.io/google_containers/busybox
         command: [ "/bin/sh", "-c", "env" ]
         envFrom:
         - configMapRef:
             name: special-config
     restartPolicy: Never
   ```

1. 保存Pod的配置，现在Pod的输出会包含`SPECIAL_LEVEL=very`和`SPECIAL_TYPE=charm`.

<!--
## Use ConfigMap-defined environment variables in Pod commands  

You can use ConfigMap-defined environment variables in the `command` section of the Pod specification using the `$(VAR_NAME)` Kubernetes substitution syntax.

For example:

The following Pod specification

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special_level
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special_type
  restartPolicy: Never
```

produces the following output in the `test-container` container:

```shell
very charm
```
-->

## 在Pod命令里使用ConfigMap定义的环境变量

我们可以利用`$(VAR_NAME)`这个Kubernetes替换变量，在Pod的配置文件的`command`段使用ConfigMap定义的环境变量。

例子如下:

下面的Pod配置

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special_level
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special_type
  restartPolicy: Never
```

在容器`test-container`里产生这样的输出：

```shell
very charm
```

<!--
## Add ConfigMap data to a Volume 

As explained in [Configure Containers Using a ConfigMap](/docs/tasks/configure-pod-container/configmap.html), when you create a ConfigMap using ``--from-file``, the filename becomes a key stored in the `data` section of the ConfigMap. The file contents become the key's value. 

The examples in this section refer to a ConfigMap named special-config, shown below.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.level: very
  special.type: charm
```
-->

## Add ConfigMap data to a Volume 

就像这篇文章[如何使用ConfigMap配置容器](/docs/tasks/configure-pod-container/configmap.html)介绍的,当你使用``--from-file``创建ConfigMap时，
文件名将作为键名保存在ConfigMap的`data`段，文件的内容变成键值。

下面的例子展示了一个名为special-config的ConfigMap的配置：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.level: very
  special.type: charm
```

<!--
### Populate a Volume with data stored in a ConfigMap

Add the ConfigMap name under the `volumes` section of the Pod specification. 
This adds the ConfigMap data to the directory specified as `volumeMounts.mountPath` (in this case, `/etc/config`).
The `command` section references the `special.level` item stored in the ConfigMap.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```

When the pod runs, the command (`"ls /etc/config/"`) produces the output below:

```shell
special.level
special.type
```
-->

### 从ConfigMap里的数据生成一个卷

在Pod的配置文件里的`volumes`段添加ConfigMap的名字。
这会将ConfigMap数据添加到`volumeMounts.mountPath`指定的目录里面（在这个例子里是`/etc/config`）。
`command`段引用了ConfigMap里的`special.level`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```

Pod运行起来后, 执行这个命令(`"ls /etc/config/"`)将产生如下的输出：

```shell
special.level
special.type
```

<!--
### Add ConfigMap data to a specific path in the Volume

Use the `path` field to specify the desired file path for specific ConfigMap items. 
In this case, the `special.level` item will be mounted in the `config-volume` volume at `/etc/config/keys`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.level
          path: keys
  restartPolicy: Never
```

When the pod runs, the command (`"cat /etc/config/keys"`) produces the output below:

```shell
very
```
-->

### 添加ConfigMap数据到卷里指定路径

使用`path`变量定义ConfigMap数据的文件路径。
在我们这个例子里，`special.level` 将会被挂载在`config-volume`的文件`/etc/config/keys`下.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.level
          path: keys
  restartPolicy: Never
```

Pod运行起来后，执行命令(`"cat /etc/config/keys"`)将产生下面的结果：

```shell
very
```

<!--
### Project keys to specific paths and file permissions

You can project keys to specific paths and specific permissions on a per-file
basis. The [Secrets](/docs/concepts/configuration/secret#using-secrets-as-files-from-a-pod) user guide explains the syntax.

### Mounted ConfigMaps are updated automatically

When a ConfigMap already being consumed in a volume is updated, projected keys are eventually updated as well. Kubelet is checking whether the mounted ConfigMap is fresh on every periodic sync. However, it is using its local ttl-based cache for getting the current value of the ConfigMap. As a result, the total delay from the moment when the ConfigMap is updated to the moment when new keys are projected to the pod can be as long as kubelet sync period + ttl of ConfigMaps cache in kubelet.

{% endcapture %}

{% capture discussion %}
-->

### 绑定Key到指定的路径和文件权限

我们可以基于文件来绑定Key到指定的路径和文件权限，详情请查阅[Secrets](/docs/concepts/configuration/secret#using-secrets-as-files-from-a-pod)
这篇文章解释了这个用法。

### 自动更新挂载的ConfigMap

当一个已经被使用的ConfigMap发生了更新时，对应的Key也会被更新。Kubelet会周期性的检查挂载的ConfigMap是否是最新的。
然而，它会使用本地基于ttl的cache来获取ConfigMap的当前内容。因此，从ConfigMap更新到Pod里的新Key更新这个时间，等于Kubelet的同步周期加ConfigMap cache的tll。

{% endcapture %}

{% capture discussion %}

<!--
## Understanding ConfigMaps and Pods

### Restrictions

1. You must create a ConfigMap before referencing it in a Pod specification (unless you mark the ConfigMap as "optional"). If you reference a ConfigMap that doesn't exist, the Pod won't start. Likewise, references to keys that don't exist in the ConfigMap will prevent the pod from starting.

1. If you use `envFrom` to define environment variables from ConfigMaps, keys that are considered invalid will be skipped. The pod will be allowed to start, but the invalid names will be recorded in the event log (`InvalidVariableNames`). The log message lists each skipped key. For example:

   ```shell
   kubectl get events
   LASTSEEN FIRSTSEEN COUNT NAME          KIND  SUBOBJECT  TYPE      REASON                            SOURCE                MESSAGE
   0s       0s        1     dapi-test-pod Pod              Warning   InvalidEnvironmentVariableNames   {kubelet, 127.0.0.1}  Keys [1badkey, 2alsobad] from the EnvFrom configMap default/myconfig were skipped since they are considered invalid environment variable names.
   ```
-->

## 了解ConfigMaps和Pods

### 限制

1. 我们必须在Pod使用ConfigMap之前，创建好ConfigMap（除非你把ConfigMap标志成"optional"）。如果你引用了一个不存在的ConfigMap，
那这个Pod是无法启动的。就像引用了不存在的Key会导致Pod无法启动一样。

1. 如果你使用`envFrom`来从ConfigMap定义环境变量，无效的Key会被忽略。Pod可以启动，但是无效的名字将会被记录在事件日志里(`InvalidVariableNames`). 
日志消息会列出来每个被忽略的Key，比如：

   ```shell
   kubectl get events
   LASTSEEN FIRSTSEEN COUNT NAME          KIND  SUBOBJECT  TYPE      REASON                            SOURCE                MESSAGE
   0s       0s        1     dapi-test-pod Pod              Warning   InvalidEnvironmentVariableNames   {kubelet, 127.0.0.1}  Keys [1badkey, 2alsobad] from the EnvFrom configMap default/myconfig were skipped since they are considered invalid environment variable names.
   ```

<!--
1. ConfigMaps reside in a specific [namespace](/docs/user-guide/namespaces/). A ConfigMap can only be referenced by pods residing in the same namespace.

1. Kubelet doesn't support the use of ConfigMaps for pods not found on the API server. 
   This includes every pod created using kubectl or indirectly via a replication controller. 
   It does not include pods created via the Kubelet's `--manifest-url` flag, `--config` flag, or the Kubelet REST API. (Note: these are not commonly-used ways to create pods.)
   
{% endcapture %}

{% capture whatsnext %}
* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configmap/).
* Follow a real world example of [Configuring Redis using a ConfigMap](/docs/tutorials/configuration/configure-redis-using-configmap/).

{% endcapture %}

{% include templates/task.md %}
-->

1. ConfigMaps存在于指定的[命名空间](/docs/user-guide/namespaces/).则这个ConfigMap只能被同一个命名空间里的Pod所引用。

1. Kubelet不支持在API服务里找不到的Pod使用ConfigMap，这个包括了每个通过Kubectl或者间接通过复制控制器创建的Pod，
不包括通过Kubelet 的 `--manifest-url`标志, `--config`标志, 或者Kubelet的REST API。（注意：这些并不是常规创建Pod的方法）
   
{% endcapture %}

{% capture whatsnext %}
* 想了解更多 [ConfigMaps](/docs/tasks/configure-pod-container/configmap/).
* 想研究一个现实生活的案例 [Configuring Redis using a ConfigMap](/docs/tutorials/configuration/configure-redis-using-configmap/).

{% endcapture %}

{% include templates/task.md %}

