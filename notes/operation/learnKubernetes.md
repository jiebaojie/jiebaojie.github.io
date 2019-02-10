---
layout: post
notes: true
subtitle: "【技术博客】Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers"
comments: false
author: "Rinor Maloku（梁晓勇 译）"
date: 2019-01-19 00:00:00

---

原文：[https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882)

译文：[http://dockone.io/article/5132](http://dockone.io/article/5132)

# 应用程序演示

该应用程序只有一个功能：它以一个句子作为输入，使用文本分析计算出句子的情感值。

![](/img/notes/operation/learnKubernetes/sentiment_analyser.gif)

从技术角度来看，这个应用程序由三个微服务组成。每个微服务都具有一个特定功能：

*	SA-Frontend：提供ReactJS静态文件访问的Nginx Web服务器。
*	SA-WebApp：处理来自前端请求的Java Web应用程序。
*	SA-Logic：执行情绪分析的Python应用程序。

微服务不是孤立存在的，它们可以实现“关注点分离”，但还是需要互相交互，理解这一点非常重要。

![](/img/notes/operation/learnKubernetes/micro_service.gif)

通过展示数据在它们之间的流动方式是对这种交互最好的说明：

1.	客户端应用程序请求index.html（继而请求ReactJS应用程序的捆绑脚本）
2.	用户与应用程序交互会触发对Spring WebApp的请求。
3.	Spring WebApp将情绪分析请求转发给Python应用程序。
4.	Python应用程序计算情感值并将结果作为响应返回。
5.	Spring WebApp将响应返回给React应用程序（然后将信息展示给用户）。

# 1. 在你的计算机上运行基于微服务的应用程序

## 为本地开发设置React

## 发布我们的React应用

## 用Nginx提供静态文件访问

## 设置Spring Web应用程序

## 设置Python应用程序

# 2. 为每个服务构建容器镜像

Kubernetes是一个容器编排器。很显然，我们需要容器以便对其进行编排。那么什么是容器？这从Docker的文档中可以得到最好的回答。

	容器镜像是一个轻量级、独立的可执行软件包，它包含了软件运行所需的全部内容：代码、运行时、系统工具、系统库、设置等等。不论是基于Linux或是Windows的应用，容器化的软件在任何环境中都能同样运行。
	
这意味着容器可以在包括生产服务器的任何计算机上无差别地运行。

## 通过VM来提供React静态文件访问

使用虚拟机的缺点：

1.	资源效率低下，每个虚拟机都有一个完整的操作系统的额外开销。
2.	它依赖于平台。在你的计算机上正常工作的东西，在生产服务器上有可能无法正常工作。
3.	与容器相比，属于重量级产品，且扩展较慢。

![](/img/notes/operation/learnKubernetes/vm.png)

## 通过容器来提供React静态文件访问

1.	在Docker的协助下使用宿主机操作系统，资源利用率较高。
2.	平台无关。在你的计算机上正常运行的容器在任何地方也都可以正常工作。
3.	使用镜像层实现轻量化。

![](/img/notes/operation/learnKubernetes/docker.png)

## 构建React应用的容器镜像（Docker简介）

Docker容器的基础构建块是Dockerfile。Dockerfile开头是一个基础容器镜像，后面是有关构建满足应用程序需求的新容器镜像的一系列指令。

在开始定义Dockerfile之前，回忆一下使用Nginx提供静态文件访问的步骤：

1.	构建静态文件（npm run build）
2.	启动Nginx服务器
3.	将sa-frontend项目中build目录的内容复制到nginx/html中。

### 定义SA-Frontend的Dockerfile

SA-Frontend的Dockerfile是执行两个步骤的指令。Nginx团队已经提供了一个Nginx基础镜像，我们只要在它的基础上进行构建即可。这两个步骤是：

1.	从基础的Nginx镜像开始。
2.	将sa-frontend/build目录复制到容器的nginx/html目录。

转换成Dockerfile看起来是这样的：

	FROM nginx
	COPY build /usr/share/nginx/html
	
是不是很酷？这个文件具有很好的可读性，复述起来就是：

从Nginx镜像开始（不管那些家伙在里面做了什么），将build目录复制到镜像中的nginx/html目录。完成！

你可能会问，怎么知道build文件要复制到哪里？也就是/usr/share/nginx/html。很简单，查看Docker Hub上Nginx镜像的文档。

### 构建并推送容器

在推送镜像之前，我们需要一个容器Registry来托管容器。Docker Hub是一个免费的云容器服务，我们将用其进行演示。在继续之前，你有三项任务：

1.	安装Docker CE。
2.	在Docker Hub上注册。
3.	在终端中执行以下命令进行登录：

命令：

	docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
	
完成上述任务后，定位到sa-frontend目录。然后执行以下命令（请把$DOCKER_ID_USER替换成你的Docker Hub用户名，比如rinormaloku/sentiment-analysis-frontend）：

	docker build -f Dockerfile -t $DOCKER_ID_USER/sentiment-analysis-frontend .
	
-f Dockerfile可以省略，因为我们正处在包含Dockerfile的目录中。

要推送镜像，请使用docker push命令：

	docker push $DOCKER_USER_ID/sentiment-analysis-frontend
	
在你的Docker Hub仓库中验证一下镜像是否已推送成功。

### 运行容器

现在，任何人都可以拉取并运行$DOCKER_USER_ID/sentiment-analysis-frontend中的镜像：

	docker pull $DOCKER_USER_ID/sentiment-analysis-frontend
	docker run -d -p 80:80 $DOCKER_ID_USER/sentiment-analysis-frontend
	
我们的Docker容器运行起来了！

在继续之前，需要说明一下易发生混淆的80:80：

*	第一个80是宿主机的端口（即我的电脑）。
*	第二个80表示请求所要转发的容器端口。

![](/img/notes/operation/learnKubernetes/port.png)

它将<宿主机端口>映射到<容器端口>上。这意味着对宿主机80端口的请求将被映射到容器的80端口中。

由于这个端口运行在宿主机（你的计算机）的80端口上，因此可以通过localhost:80进行访问。如果没有原生的Docker支持，你可以通过<docker-machine ip>:80打开该应用程序。执行docker-machine ip可获得docker-machine的IP地址。

### Dockerignore文件

我们需要的唯一数据都在build文件夹中，上传其他内容是在浪费时间。我们可以通过忽略其他目录来减少构建时间。这就是.dockerignore的作用。对你而言这个文件应该不陌生，因为它与.gitignore类似，只要将所有想忽略的目录都添加到.dockerignore文件中即可。

## 为Java应用程序构建容器镜像

Dockerfile如下：

	ENV SA_LOGIC_API_URL http://localhost:5000
	…
	EXPOSE 8080
	
ENV关键字在Docker容器中声明了一个环境变量。这让我们可以在启动容器时指定情绪分析API的URL。

其次，EXPOSE关键字暴露了我们稍后要访问的端口。但是，我们在SA-Frontend的Dockerfile中并没有这么做。好问题！原因是它仅用于文档目的，换句话说，它仅仅是为了给阅读Dockerfile的人提供相关信息用的。

## 为Python应用程序构建容器镜像

## 测试容器化的应用程序

运行sa-logic容器并配置其监听5050端口：

	docker run -d -p 5050:5000 $DOCKER_ID_USER/sentiment-analysis-logic
	
运行sa-webapp容器并配置其监听8080端口（因为我们修改了Python应用监听的端口，我们需要覆盖SA_LOGIC_API_URL环境变量）：

	docker run -d -p 8080:8080 $DOCKER_USER_ID/sentiment-analysis-web-app -e SA_LOGIC_API_URL='http://<container_ip or docker machine ip>:5000' $DOCKER_USER_ID/sentiment-analysis-web-app
	
运行sa-frontend容器：

	docker run -d -p 80:80 $DOCKER_ID_USER/sentiment-analysis-frontend
	
# Kubernetes介绍

## Kubernetes是什么

在从容器启动我们的微服务之后，我们碰到了一个问题，下面我们以问答形式进一步阐述它：

问：如何进行容器扩展？

答：再启动一个。

问：如何在它们之间分担负载？如果服务器已经使用到极限，并且我们的容器需要另一台服务器，该怎么办？如何计算最佳的硬件利用率？

答：啊……呃……（我Google一下）。

问：如何在不影响任何内容的情况下进行更新？如果需要，该如何回退到可工作的版本？

Kubernetes解决了所有这些问题（以及其他更多问题！）。我可以用一句话来介绍Kubernetes：“Kubernetes是一个容器编排器，它对底层基础设施（容器运行的地方）进行了抽象。”

## 对底层基础设施进行抽象

Kubernetes提供了一套简单的用于发送请求的API，对底层基础设施进行抽象。Kubernetes会尽其最大能力来满足这些请求。比如说，可以简单地请求“Kubernetes启动4个x镜像的容器”，然后Kubernetes会找出利用率不足的节点，在这些节点中启动新的容器。

![](/img/notes/operation/learnKubernetes/kubernetes.png)

对开发者这意味着什么？意味着他无须关心节点的数量、容器在哪启动以及它们之间如何通信。他无须处理硬件优化，也无须担心节点会宕机（根据墨菲定律，节点一定会宕机），因为他可以将新节点添加到Kubernetes集群中。Kubernetes会在正常运转的节点上启动容器。它会尽其最大能力来做到这一点。

## 云服务提供商标准化

# Kubernetes实践——Pod

把微服务运行在容器中，虽然行得通，但是设置过程相当繁琐。我们还提到这个解决方案不具有可扩展性和弹性，而Kubernetes则可解决这些问题。

![](/img/notes/operation/learnKubernetes/kubernates_microservices.png)

## 安装并启动Minikube

Minikube提供了一个只有一个节点的Kubernetes集群，但别忘了我们并不关心节点的数量，Kubernetes已经将其抽象掉了，并且这对学习Kubernetes并不重要。

## Pod

Pod可以由一个甚至是一组共享相同运行环境的容器组成。

![](/img/notes/operation/learnKubernetes/pod.png)

总结来说，Pod的主要属性是：

1.	每个Pod在Kubernetes集群中都有一个唯一的IP地址。
2.	Pod可以包含多个容器。这些容器共享相同的端口空间，因此它们可以通过localhost进行通信（由此可见，它们不能使用相同的端口），与其他Pod的容器进行通信必须使用其Pod的IP地址。
3.	Pod中的容器共享相同的数据卷、相同的IP地址、端口空间、IPC命名空间。

容器拥有独立的文件系统，不过它们可以使用Kubernetes资源卷来共享数据。

## Pod定义

以下是第一个Pod sa-front的清单（manifest）文件。后面是对各个要点的解释。

	apiVersion: v1
	kind: Pod                                            # 1
	metadata:
	name: sa-frontend                                  # 2
	spec:                                                # 3
	containers:
	- image: rinormaloku/sentiment-analysis-frontend # 4
	  name: sa-frontend                              # 5
	  ports:
		- containerPort: 80                          # 6
		
对各个要点的解释：

1.	Kind指定我们想要创建的Kubernetes资源的种类。在这个例子中是Pod。
2.	Name：定义资源的名称。我们将它命名为sa-front。
3.	Spec是用于定义资源的期望状态的对象。Pod Spec最重要的参数是容器的数组。
4.	Image是要在此Pod中启动的容器镜像。
5.	Name是Pod中容器的唯一名称。
6.	ContainerPort：容器所要监听的端口。这只是面向读者的一个指示信息（删除该端口并不会限制访问）。

## 创建SA前端Pod

上述Pod定义在resource-manifests/sa-frontend-pod.yaml文件中。你可以在终端中定位其所在目录​​，或者在命令行中提供完整路径。然后执行以下命令：

	kubectl create pod -f sa-frontned-pod.yaml
	pod "sa-frontend" created
	
要检查Pod是否正在运行，请执行以下命令：

	kubectl get pods
	NAME                          READY     STATUS    RESTARTS   AGE
	sa-frontend                   1/1       Running   0          7s
	
如果其状态仍是ContainerCreating，则可以使用--watch参数执行上述命令，以便在Pod处于Running状态时获得更新信息。

### 从外部访问应用程序

为了从外部访问应用程序，正常来说，我们需要创建一个Service类型的Kubernetes资源，但为了快速调试，我们使用另一个方法，也就是端口转发：

	kubectl port-forward sa-frontend-pod 88:80
	Forwarding from 127.0.0.1:88 -> 80
	
在浏览器中打开127.0.0.1:88即可访问我们的React应用程序。

### 向上扩展的错误方法

我们说过Kubernetes的主要功能之一是可扩展性，为了说明这一点，我们来运行另一个Pod。为此，创建另一个Pod资源，其定义如下：

	apiVersion: v1
	kind: Pod                                            
	metadata:
	name: sa-frontend2      # The only change
	spec:                                                
	containers:
	- image: rinormaloku/sentiment-analysis-frontend 
	  name: sa-frontend                              
	  ports:
		- containerPort: 80
		
通过执行以下命令来创建新的Pod：

	kubectl create pod -f sa-frontned-pod2.yaml
	pod "sa-frontend2" created
	
通过执行以下命令验证第二个Pod是否正在运行：

	kubectl get pods
	NAME                          READY     STATUS    RESTARTS   AGE
	sa-frontend                   1/1       Running   0          7s
	sa-frontend2                  1/1       Running   0          7s
	
现在有两个Pod在运行了！

**注意：**这不是最终的解决方案，它的问题很多。我们将在Kubernetes的**Deployment**资源环节对此进行改进。

## Pod总结

提供静态文件服务的Nginx Web服务器运行在两个不同的Pod中。现在有两个问题：

*	如何将其暴露给外界，以便通过URL进行访问？
*	如何在它们之间进行负载平衡？

![](/img/notes/operation/learnKubernetes/lb.png)

Kubernetes为此提供了Service资源。

# Kubernetes实践——Service

Kubernetes的Service资源为提供相同功能服务的一组Pod充当入口。此类资源负责发现服务和负载平衡，任务繁重。

![](/img/notes/operation/learnKubernetes/kubernetes_service.png)

我们的Kubernetes集群是由多个具有不同功能性服务的Pod组成的（前端、Spring WebApp和Flask Python应用程序）。那么问题来了，Service如何知道要定位到哪些Pod？也就是说，它如何生成Pod的端点列表？

答案是，通过标签（Labels）来实现的，这是一个两步过程：

1.	将标签应用于所有我们希望Service定位的Pod上
2.	为我们的Service应用一个“筛选器（selector）”，以定义要定位哪个标记的Pod。

![](/img/notes/operation/learnKubernetes/labels.png)

可以看到，Pod被打上了“app:sa-frontend”标签，而Service也使用同一标签来定位Pod。

## 标签

标签为组织Kubernetes资源提供了一种简单的方法。它们的表现形式为键值对，可以应用于所有资源。修改Pod的清单文件以匹配此前上图中所示的示例。

完成修改后保存文件，并使用以下命令来应用：

	kubectl apply -f sa-frontend-pod.yaml
	Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
	pod "sa-frontend" configured
	kubectl apply -f sa-frontend-pod2.yaml 
	Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
	pod "sa-frontend2" configured
	
这里出现了一个警告（apply与create的不同）。下一行可以看到“sa-frontend”和“sa-frontend2”Pod配置好了。我们可以通过过滤要显示的Pod来验证Pod是否已打上标签：

	kubectl get pod -l app=sa-frontend
	NAME           READY     STATUS    RESTARTS   AGE
	sa-frontend    1/1       Running   0          2h
	sa-frontend2   1/1       Running   0          2h

验证Pod被打上标签的另一种方法是在上述命令中附加--show-labels参数。这将显示每个Pod的所有标签。

![](/img/notes/operation/learnKubernetes/lb_service.gif)

## Service定义

Loadbalancer Service的YAML定义如下所示：

	apiVersion: v1
	kind: Service              # 1
	metadata:
	name: sa-frontend-lb
	spec:
	type: LoadBalancer       # 2
	ports:
	- port: 80               # 3
	protocol: TCP          # 4
	targetPort: 80         # 5
	selector:                # 6
	app: sa-frontend       # 7
	
解释：

1.	Kind：一个Service。
2.	Type：规格类型，我们选择LoadBalancer是因为我们要实现Pod之间的负载均衡。
3.	Port：指定Service接收请求的端口。
4.	Protocol：定义通信协议。
5.	TargetPort：请求转发的端口。
6.	Selector：包含选择Pod的参数的对象。
7.	App：sa-frontend定义了要定位的是打了“app:sa-frontend”标签的Pod。

请执行以下命令创建该Service：

	kubectl create -f service-sa-frontend-lb.yaml
	service "sa-frontend-lb" created
	
可通过执行以下命令来检查Service的状态：

	kubectl get svc
	NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
	sa-frontend-lb   LoadBalancer   10.101.244.40   <pending>     80:30708/TCP   7m

外部IP处于pending状态（不用等了，它不会变的）。这是因为我们使用的是Minikube。如果我们在Azure或GCP这样的云提供商中执行该操作，将获得一个公网IP，以便在全球范围内访问我们的服务。

尽管如此，我们不会因此受阻，Minikube为本地调试提供了一个有用的命令，执行以下命令：

	minikube service sa-frontend-lb
	Opening kubernetes service default/sa-frontend-lb in default browser...
	
这会在你的浏览器中打开Service的IP地址。在Service收到请求后，它会将其转发给其中一个Pod（哪一个无关紧要）。这种抽象让我们可以使用Service作为入口将多个Pod作为一个单元来看待和交互。

## Service总结

本节我们介绍了标签资源，将它们用于Service的筛选器，同时我们定义并创建了一个LoadBalancer Service。这满足了我们扩展应用程序的需求（只需添加新的打上标签的Pod）并使用Service作用入口实现Pod之间的负载均衡。

# Kubernetes实践——Deployment

应用程序是在不断变化的，Kubernetes的Deployment负责保证这些应用的一致。唯有那些死掉的应用程序才不会改变，否则新的需求会出现，更多的代码会被发布、打包并部署。在这个过程的每一步中，都可能犯错误。

Deployment资源可以自动迁移应用程序版本，实现零停机，并且可以在失败时快速回滚到前一版本。

## Deployment实践

目前，我们有两个Pod和一个用于暴露这两个Pod并在它们之间做负载均衡的Service。单独部署这些Pod并非理想的方案。它要求每个Pod进行单独管理（创建、更新、删除及监控其健康状况）。快速更新和回滚更是不可能！这是无法接受的，而Kubernetes的Deployment资源则解决了这些问题。

![](/img/notes/operation/learnKubernetes/current_state.png)

在继续之前，我说明我们想要实现的目标，因为这将为我们提供一个总览，让我们能够理解Deployment资源的清单定义。我们想要的是：

1.	运行rinormaloku/sentiment-analysis-frontend镜像的两个Pod
2.	零停机时间部署
3.	为Pod打上app: sa-frontend标签，以便sa-frontend-lb Service能发现这些服务

## Deployment定义

实现上述所有需求的YAML资源定义：

	apiVersion: extensions/v1beta1
	kind: Deployment                                          # 1
	metadata:
	name: sa-frontend
	spec:
	replicas: 2                                             # 2
	minReadySeconds: 15
	strategy:
	type: RollingUpdate                                   # 3
	rollingUpdate: 
	  maxUnavailable: 1                                   # 4
	  maxSurge: 1                                         # 5
	template:                                               # 6
	metadata:
	  labels:
		app: sa-frontend                                  # 7
	spec:
	  containers:
		- image: rinormaloku/sentiment-analysis-frontend
		  imagePullPolicy: Always                         # 8
		  name: sa-frontend
		  ports:
			- containerPort: 80
			
解释如下：

1.	Kind：一个Deployment。
2.	Replicas是Deployment Spec对象的一个属性，用于定义我们需要运行几个Pod。这里是2个。
3.	Type指定了这个Deployment在迁移版本时使用的策略。RollingUpdate策略将保证实现零当机时间部署。
4.	MaxUnavailable是RollingUpdate对象的一个​​属性，用于指定执行滚动更新时不可用的Pod的最大数（与预期状态相比）。我们的部署具有2个副本，这意味着在终止一个Pod后，仍然有一个Pod在运行，这样可以保持应用程序的可访问性。
5.	MaxSurge是RollingUpdate对象的另一个属性，用于定义可以添加到部署中的最大Pod数量（与预期状态相比）。在我们的部署是，这意味着在迁移到新版本时，我们可以添加一个Pod，也就是同时有3个Pod。
6.	Template：指定Deployment创建新Pod所用的Pod模板。你马上就发现它与Pod的定义相似。
7.	app: sa-frontend是使用该模板创建出来的Pod所使用的标签。
8.	ImagePullPolicy设置为Always表示，每次重新部署时都会拉取容器镜像。

老实说，即便是我也会被这堆文字弄糊涂，我们直接用这个示例入手：

	kubectl apply -f sa-frontend-deployment.yaml
	deployment "sa-frontend" created
	
跟之前一样，我们来验证一下一切是否正常：

	kubectl get pods
	NAME                           READY     STATUS    RESTARTS   AGE
	sa-frontend                    1/1       Running   0          2d
	sa-frontend-5d5987746c-ml6m4   1/1       Running   0          1m
	sa-frontend-5d5987746c-mzsgg   1/1       Running   0          1m
	sa-frontend2                   1/1       Running   0          2d
	
现在运行了4个Pod，有两个是由Deployment部署的，另外两个是我们手动创建的。可使用命令kubectl delete pod &lt;Pod名>来删除手动创建的那两个。

删除Deployment部署的一个Pod，看看会发生什么。

**说明：**删除一个Pod后Deployment将发现当前状态（运行着1个Pod）与预期状态不同（运行着2个Pod），因此它会再启动一个Pod。

除了保证预期状态之外，Deployment还有什么好处？下面我们一一来看下它的优点：

### 优点1：零停机时间滚动部署

"滚动更新"的幕后：

在应用新的Deployment后，Kubernetes会对新旧状态进行比较。在我们的示例中，新状态请求两个使用rinormaloku/sentiment-analysis-frontend:green的Pod。这与当前运行状态不同，因此它会启用**RollingUpdate**。

![](/img/notes/operation/learnKubernetes/rolling_update.png)

RollingUpdate会根据我们指定的规则进行操作，即“maxUnavailable: 1”和“maxSurge: 1”。这意味着部署时只能终止一个Pod，并且只能启动一个新的Pod。该过程会不断重复直到所有的Pod都被更换。

### 优点2：回滚到之前的状态

查看历史版本：

	kubectl rollout history deployment sa-frontend
	deployments "sa-frontend"
	REVISION  CHANGE-CAUSE
	1         <none>         
	2         kubectl.exe apply --filename=sa-frontend-deployment-green.yaml --record=true
	
回滚到上一个版本：

	kubectl rollout undo deployment sa-frontend --to-revision=1
	deployment "sa-frontend" rolled back

# Kubernetes和其他实践

我们将所有仍然需要做的事情灰化了。让我们从最下面开始：部署sa-logic Deployment。

![](/img/notes/operation/learnKubernetes/sa_logic_deployment.png)

## 部署SA-Logic

在终端中定位到resource-manifests目录并执行以下命令：

	kubectl apply -f sa-logic-deployment.yaml --record
	deployment "sa-logic" created
	
SA-Logic Deployment创建了三个Pod（运行着Python应用程序容器），并给它们打上了app: sa-logic标签。该标签让我们能够使用SA-Logic Service中的筛选器来定位它们。

## Service SA-Logic

我们的Java应用程序（运行在SA-WebApp Deployment的Pod中）依赖于Python应用程序完成的情绪分析。但是，与之前全部在本地运行不同，现在我们不再是使用单一一个Python应用程序监听一个端口，而是两个甚至更多。

这就是为什么我们需要一个Service“作为提供相同功能服务的一组Pod的入口”。这意味着我们可以使用Service SA-Logic作为所有SA-Logic Pod的入口。

执行以下命令：

	kubectl apply -f service-sa-logic.yaml --record
	service "sa-logic" created
	
**更新后的应用程序状态：**我们运行了2个Pod（包含Python应用程序），并且有一个SA-Logic Service作为即将在SA-WebApp Pod中使用的入口。

现在我们需要使用Deployment资源来部署SA-WebApp Pod。

## 部署SA-WebApp

我们对Deployment已经非常熟悉，不过此处还是有一个新功能。如果你打开sa-web-app-deployment.yaml文件，你会发现这部分是新的：

	- image: rinormaloku/sentiment-analysis-web-app
	imagePullPolicy: Always
	name: sa-web-app
	env:
	- name: SA_LOGIC_API_URL
	  value: "http://sa-logic"
	ports:
	- containerPort: 8080
	
我们感兴趣的是env属性是做什么的？我们推测它是在Pod中声明环境变量SA_LOGIC_API_URL的值为“http://sa-logic”。但为什么我们将它初始化为http://sa-logic，什么是sa-logic？

这里需要介绍一下kube-dns。

### KUBE-DNS

Kubernetes有一个特殊的Pod kube-dns。默认情况下，所有Pod都会将其作为DNS服务器。kube-dns一个重要特性是它会为每个新建的Service创建一条DNS记录。

这意味着当我们创建Service sa-logic时，它获得了一个IP地址。它的名字（与IP一起）会被添加到kube-dns记录中。这使得所有的Pod能够将sa-logic转换为SA-Logic Service的IP地址。

## 部署SA-WebApp（续）

执行以下命令：

	kubectl apply -f sa-web-app-deployment.yaml --record
	deployment "sa-web-app" created
	
完成。剩下的是使用LoadBalancer Service对外暴露SA-WebApp Pod。以便让我们的React应用程序可以向作为SA-WebApp Pod入口的Service发送HTTP请求。

## Service SA-WebApp

执行一下命令：

	kubectl apply -f sa-web-app-deployment.yaml
	deployment "sa-web-app" created
	
整个架构完成了。不过还有一点没完善。在部署SA-Frontend Pod时，容器镜像将SA-WebApp指向了http://localhost:8080/sentiment。但是现在我们需要将其更新为指向SA-WebApp LoadBalancer（充当SA-WebApp Pod的入口）的IP地址。

# 文章总结

Kubernetes对团队和项目都非常有益，它简化了部署、可扩展性和弹性，让我们能够使用任意的底层基础设施。

