# Min
造一个空间，只属于我的空间
二、Docker
  <1>概念
	Docker是一个打包、分发和运行应用程序的平台。
	Docker让开发者可以 打包 自己的应用以及依赖包（或者说应用程序所依赖的整个环境）  到  一个可移植的容器中，然后发布在Linux或Windows中运行。
  <2>Docker有3个主要的组成部分
	1、镜像：Docker镜像里包含了你所打包的应用程序及它的环境
	2、镜像仓库：顾名思义，就是镜像仓库中包含了多个镜像，多个镜像之间共享和征用
	3、容器：Docker容器通常是Linux容器，基于Docker镜像被创建。
			 一个运行中的容器是一个运行在Docker主机上的进程，但是它和主机，以及所有运行在主机上的其他进程都是隔离的。
			这个进程也是受资源限制的，意味着它只能访问和使用分配给它的资源（CPU、内存等）。
  <3>对比虚拟机与Docker容器
	虚拟机
		优：
		       提供完全隔离的环境，每个虚拟机运行在自己的操作系统（Linux内核）上。
		缺：
		      对于应用要求的操作系统不同，需要在虚拟中配置不同的操作系统
	Docker容器
		优：
		       所有的Docker容器运行在同一个操作系统中，容器的资源利用率 > 虚拟机的资源利用率
		       使容器能在不同机器之间移植的系统
		       简化了打包应用的流程
		       简化了打包应用的库和依赖，甚至整个操作系统的文件系统
		缺：
		       虽然同样提供隔离的环境，但是所有的Docker容器运行在同一个操作系统中，如果操作系统发生故障，所有运行在容器内的应用都无法运行
		
     <4>容器的隔离机制：Docker容器有两种隔离机制
	一种是Linux命名空间，它使进程只看到它自己的系统视图（文件、进程、网络接口、主机名等）
		Linux命名空间：存在以下类型的命名空间
					Mount（mmt）
					Process ID（pid）
					Network（net）：一个进程属于什么Network命名空间决定了运行在进程里的应用程序能看见什么网络接口。
								每个网络接口属于一个命名空间，但是可以从一个命名空间转移到另一个。
								每个容器都使用它自己的网络命名空间，因此每个容器仅能看见它自己的一组网络接口。
					Inter-process communication（ipc 进程间通信)
					UTS：决定了运行在命名空间里的进程能看见哪些主机名和域名。
						通过分派两个不同的UTS命名空间给一对进程，能使它们看见不同的本地主机名。换言之，这两个进程就好像正在两个不同的机器上运行一样（至少就主机名而言是这样的）
					User ID(user)
					
	另一种是Linux控制组（cgroups）：它限制了进程能使用的资源量（CPU、内核、网络带宽等）；
		      
一、Kubernetes
  <1>简要了解
     1、什么是Kubernetes?
	   Kubernetes是一种容器技术，用于集群开发和部署应用程序
          2、为什么要使用Kubernetes?
	     Kubernetes的使用，减少了应用对于部署环境的依赖性，或者说让开发和生产在同样的环境下运行。。。。。。
	
	注：Kubernetes不局限于任何一种语言，没有限定任何编程接口
	
   <2>Kubernetes的核心功能
 <3>Kubernetes集群架构
        在硬件级别，一个Kubernetes集群由很多节点组成，这些节点被分成以下两种类型：
	主节点：它承载着Kubernetes控制和管理整个集群系统的控制面板
	工作节点：它们运行用户实际部署的应用



Kubernetes的组件
一、介绍pod
   pod是一组并置的容器，代表了Kubernetes中的基本构建模块。
      实际中我们并不会单独部署容器，更多的针对一组pod的容器进行部署和操作。
      但这并不意味着一个pod总是要包含多个容器——实际上只包含一个单独容器的pod也是非常常见的。

   注意：一个pod的所有容器都运行在同一个工作节点上（也简称节点）；
                一个pod绝不会跨越两个节点；

1.1  为什么需要pod
	为什么多个容器比单个容器中包含多个进程要好
	
	          一个由多个进程组成的应用程序，无论是通过ipc（进程间通信）还是本地存储文件进行通信，都要求它们运行在同一台机器上，在Kubernetes中，我们经常在容器中运行进程，由于每一个容器都非常像一台独立的机器，此时你可能认为在单个容器中运行多个进程是合乎逻辑的，然而在实践中这种做法并不合理。
	         容器被设计为每个容器只运行一个进程（除非进程本身产生子进程）。如果在单个容器中运行多个不相关的进程，那么保持所有进程运行、管理它们的日志等将会是我们的责任。例如，我们需要包含一种在进程崩溃时能够自动重启的机制。同时这些进程都将记录到相同的标准输出中，而此时我们将很难确定每个进程分别记录了什么。
	        综上所述，我们需要让每个进程运行于自己的容器中，而这就是Docker和Kubernetes期望使用的方式。
	
1.2  了解pod
	由于不能将多个进程聚集在一个单独的容器中，我们需要另一种更高级的结构来将容器绑定在一起，并将它们作为一个单元进行管理，这就是pod背后的根本原理。
	  
	同一pod中容器之间的部分隔离
	实现：
	    Kubernetes通过配置Docker来让一个pod内的所有容器共享相同的Linux命名空间，而不是每个容器都有自己的一组命名空间。
	        由于一个pod中的所有容器都在相同的network和UTS命名空间下运行，所以它们都共享相同的主机名和网络接口。同样地，这些容器也都在相同的IPC命名空间下运行，因此能够通过IPC进行通信。在最新的Kubernetes和Docker版本中，它们也能够共享相同的PID命名空间，但是该特性默认是未激活的。
	注意：当同一个pod中的容器使用单独的PID命名空间时，在容器中执行psaux就只会看到容器自己的进程。
	
	但当涉及文件系统时，
	
	容器如何共享相同的IP和端口空间
	         一个pod中的所有容器也都具有相同的loopback网络接口，因此容器可以通过localhos与同一pod中的其他容器进行通信。
	
	平坦pod间网络
	        Kubernetes集群中的所有pod都在同一个共享网络地址空间中，这就意味着每个pod都可以通过其他pod的IP地址来实现相互访问。换言之，这也表示它们之间没有NAT（网络地址转换）网关。
	        当两个pod彼此之间发送网络数据包时，它们都会将对方的实际IP地址看作数据包中的源IP。
	
	总结：Kubernetes集群中，不同node中的pod通过彼此的IP进行访问，同一node中的、pod中的容器通过localhost通信。---------还需要再简化明了
	
1.3  通过pod合理管理容器
	将多层应用分散到多个pod中
	基于扩缩容考虑而分割到多个pod中
	
	为什么不应该将应用程序都放到单一pod中？
	原因：
		<1>对于多层应用程序（例如由前端应用服务器和后端数据库组成），更合理的做法是将pod拆分到连个工作节点上，允许Kubernetes将前端安排到一个节点，将后端安排到另一个节点，从而提高基础架构的利用率。
		<2>扩缩容。pod是扩缩容的基本单位，对于Kubernetes来说，它不能横向扩缩单个容器，只能扩缩整个pod。这意味着如果你的pod由一个前端和一个后端容器组成，那么当你扩大pod的实例数量时，比如扩大为两个，最终会得到两个前端容器和两个后端容器。
	
	何时在pod中使用多个容器
	        将多个容器添加到单个pod的主要原因是应用可能由一个主进程和一个或多个辅助进程组成。
	        例如，pod中的主容器可以是一个仅仅服务于某个目录中的文件的Web服务器，而另一个容器（所谓的sidecar容器）则定期从外部源下载内容并将其存储在Web服务器目录中。
		sidecar容器的其他例子包括日志轮转器和收集器、数据处理器、通信适配器等。
	
	决定何时在pod中使用多个容器
	        回顾一下，容器应该如何分到pod中：当决定是将两个容器放入一个pod还是两个单独的pod时，我们需要问自己一下问题：
			l 它们需要一起运行还是可以在不同的主机上运行？
			l 它们代表的是一个整体还是相互独立的组件？
			l 它们必须一起进行扩缩容还是可以分别进行？
	        基本上，我们总是应该倾向于在单独的pod中运行容器，除非有特定的原因要求它们是同一pod的一部分。 
	
	注：尽管pod可以包含多个容器，但为了保持现有的简单性，容器不应该包含多个进程，pod也不应该包含多个并不需要运行在同一主机上的容器。
	
二、以YAML或JSON描述文件创建pod
	pod和其他Kubernetes资源通常是通过向Kubernetes REST API提供JSON或YAML描述文件来创建的。
	还有一种更简单的创建资源的方法，比如使用kubectl run命令，但这些方法通常只允许配置一组有限的属性。
	另外，通过YAML文件定义所有的Kubernetes对象之后，还可以将它们存储在版本控制系统中，充分利用版本控制所带来的便利性。
	
2.1  检查现有pod的YAML描述文件
——获取pod的整个YAML定义
	$ kubectl  get  po  kubia-zxzij    -o  yaml

        一个YAML描述文件大致由以下部分组成：
	apiVersion:v1——YAML描述文件所使用的Kubernetes API版本
	kind:pod——Kubernetes对象/资源类型
	metadata:——pod元数据（名称、标签和注解等）——包括名称、命名空间、标签和关于该容器的其他信息
	    …
	spec:——pod规格/内容（pod的容器列表、volume等）——包含pod内容的实际说明，例如pod的容器、卷、和其他数据
	    …
	status:——pod及其内部容器的详细状态——包含所运行的pod的当前信息，例如pod所处的条件、每个容器的描述和状态，以及内部IP和其他基本信息
	    …                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       能
	注：在创建新的pod时，永远不需要提供status部分。
	metadata、spec、status三部分展示了Kubernetes API对象的经典结构。、
	
2.2  为pod创建一个简单的YAML描述文件
	例如，一个基本的pod manifest：kubia-manual.yaml
——使用kubectl explain来发现可能的API对象字段
	$ kubectl explain pods
	Kubuctl打印出对象的解释并列出对象可以包含的属性，然后就可以深入了解各个属性的更多信息，
	例如，可以这样
——查看spec属性：
	$ kubectl explain pod.spec

2.3  使用kubectl create来创建pod 
——使用kubectl create命令从YAML文件创建pod
	$ kubectl create -f kubia-manual.yaml
	结果：	pod "kubia-manual" created
	注： kubectl create -f命令用于从YAML或JSON文件创建任何资源（不只是pod）
——查看该pod的完整描述文件
	也可以让kubectl返回JSON格式而不是YAML格式（即使使用YAML创建pod，同样也可以获取JSON格式的描述文件）
	$ kubectl get po kubia-manual -o yaml
	$ kubectl get po kubia-manual -o json
——在pod列表中查看新创建的pod
	$ kubectl get pods
	
2.4 查看应用程序日志 
——容器运行时将这些流重定向到文件，并允许运行以下命令来获取容器的日志：
	$ docker  logs  <container  id>
	使用ssh命令登录到pod正在运行的节点，并使用docker logs命令查看其日志，但Kubernetes提供了一种更为简单的方法
——使用kubectl logs命令获取pod日志
	$ kubectl logs kubia-manual
	结果：Kubia server staring...
	注意：每天或每次日志文件达到10MB大小时，容器日志都会自动轮替kubectl logs命令仅显示最后一次轮替后的日志条目。
——获取多容器pod的日志时指定容器名称
	$ kubectl  logs  kubia-manual  -c  kubia     （ kubia-manual为pod名称， kubia为容器名称）
	结果：	Kubia server staring…
	如果pod包含多个容器，在运行kubectl logs命令时则必须通过包含-c <容器名>选项来显示指定容器名称。
	注意：我们只能获取仍然存在的pod的日志。当一个pod被删除时，它的日志也会被删除。
		如果希望在pod删除之后仍然可以获取其日志，需要设置中心化的、集群范围的日志系统，将所有的日志存储到中心存储中。
		
2.5  向pod发送请求
        连接pod以进行测试和调试的方法，其中之一是通过端口转发。
        将本地网络端口转发到pod中的端口
	如果想要不通过service的情况下与某个特定的pod进行通信（出于调试或其他原因），Kubernetes将允许我们配置端口转发到该pod。
	可以通过kubectl port-forward命令完成上述操作。
——例如，以下命令会将机器的本地端口8888转发到我们的kubia-manual pod的端口8080：
	$ kubectl port-forward kubia-manual 8888:8080
	结果：	…Forwarding from 127.0.0.1:8888 -> 8080
	      	…Forwarding from [::1]:8888 -> 8080
		此时端口转发正在运行，可以通过本地端口连接到pod
    通过端口转发连接到pod
——在另一个终端中，通过运行在localhost：8080上的kubectl port-forward代理，可以使用curl命令向pod发送一个HTTP请求：
	$ curl localhost：8888
	结果：	You'v hit kubia-manual
	
    像这样使用端口转发是一种测试特定pod的有效方法。

三、使用标签组织pod
3.1 介绍标签
        标签是一种简单却功能强大的kubernetes特性，不仅可以组织pod，也可以组织所有其他的Kubernetes资源。
        详细来讲，标签是可以附加到资源的任意键值对，用以选择具有该确切标签的资源（这是通过标签选择器完成的）。只要标签的key在资源内是唯一的，一个资源便可以拥有多个标签。
        通常在创建资源时就会将标签附加到资源上，但之后我们也可以再添加其他标签，或者修改现有标签的值，而无须重新创建资源。
	标签
	• app——指定pod属于哪个应用、组件或微服务
	• rel——显示pod中运行的应用程序版本是stable、beta、还是canary
 
3.2  创建pod时指定标签
——创建带有标签的 kubia-manual-with-labels pod
	$ kubectl create -f kubia-manual-with-labels.yaml
	结果：	pod "kubia-manual-v2" created
——kubectl get pods命令默认不会列出任何标签，但可以使用 --show-labels 选项来查看：
	$ kubectl get po --show-labels
——如果你只对某些标签感兴趣，可以使用 -L 选项指定它们并将它们分别显示在自己的列中，而不是列出所有标签。
	$ kubectl get po -L creation_method,env
	
3.3 修改现有pod的标签
——标签可以在现有pod上进行添加和修改。
	由于pod kubia-manual是手动创建的，所以为其添加creation_method=manual标签：
	$ kubectl label po kubia-manual creation_method=manual
	结果：	pod "kubia-manual" labeled
——现在，将kubia-manual-v2 pod上的env=prod标签更改为env=debug：（更改现有标签需要使用 --overwrite 选项）
	$ kubectl label po kubia-manual-v2 env=debug --overwrite
	
3.4  通过标签选择器列出pod子集
        注意：标签要与标签选择器结合在一起
                     标签选择器允许我们选择标记有特定标签的pod子集，并对这些pod执行操作。
                     可以说标签选择器是一种能够根据是否包含具有特定值的特定标签来过滤资源的准则。
        标签选择器根据资源的一下条件来选择资源：
	l 包含（或不包含）使用特定键的标签
	l 包含具有特定键和值的标签
	l 包含具有特定键的标签，但其值与我们指定的不同

3.4.1  使用标签选择器列出pod
——观察我们手动创建的所有pod（用creation_method=manual标记了它们）：
	$ kubectl get po -l creation_method=manual
——列出包含env标签的所有pod，无论其值如何：
	$  kubectl get po -l env
——列出没有env标签的所有pod：
	$  kubectl get po -l '!env'
	注意：确保使用单引号来圈引  !env ，这样bash shell才不会解释感叹号（译者注：感叹号在bash中有特殊含义，表示事件指示器）
——同理，我们也可以将pod与以下标签选择器进行匹配：
	• creation_method!=manual——选择带有creation_method标签，并且值不等于munual的pod
	• env  in  (prod,devel)——选择带有env标签且值为prod或devel的pod
	• env  notin  (prod,devel)——选择带有env标签，但其值不是prod或devel的pod

3.4.2  在标签选择器中使用多个条件
——例如，只选择product catalog微服务的beta版本pod，可以使用以下选择器：app=pc,rel=beta
	$ kubectl get po product catalog -l app=pc,rel=beta---------自己写的，尚未验证该命令是否正确
	
3.5  使用标签和选择器来约束pod调度
        有时希望将pod调度到特定的某一处，例如，硬件基础设施不同，某些工作节点使用机械硬盘，某些工作节点使用固态硬盘，或者是当需要将执行GPU密集型运算的pod调度到实际提供GPU加速的节点上时，也需要pod调度。如果我们特别说明pod要调度到哪个节点上，这会使应用程序与基础架构强耦合，违背了Kubernetes对运行在其上的应用程序隐藏实际的基础架构的整个构想。
        如果想掌握对一个pod调度的控制，就应该用某种方式描述对节点的需求，使Kubernetes选择一个符合这些需求的节点。
        然而，可以通过节点标签和节点标签选择器来完成。

3.5.1  使用标签分类工作节点
——假设集群中的一个节点刚添加完成，它包含一个用于通用GPU计算的GPU。希望向节点添加标签来展示这个功能特性，可以通过将标签gpu=true添加到一个节点来实现（只需从kubectl get nodes返回的列表中选择一个）
	$  kubectl label node gke-kubia-85f6-node-orrx gpu=true
	结果：	node "gke-kubia-85f6-node-orrx" labeled
——列出包含标签gpu=true的节点：
	$  kubectl get node -l gpu=true
	结果：	NAME	STATUS	AGE
		gke-kubia-85f6-node-orrx	Ready	1d
——列出所有节点，并告知kubectl展示一个现实每个节点的gpu标签值附加列（kubectl get node -L gpu）
	$  kubectl get node -L gpu

3.5.2  将pod调度到特定节点
——使用标签选择器将pod调度到特定节点：kubia-gpu.yaml
	在kubia-gpu.yaml文件中，添加（红色部分）
	apiVersion: v1
	kind: Pod
	metadata:
	  name: kubia-gpu
	spec:
	  nodeSelector:                 节点选择器要求Kubernetes只将pod部署到包含标签gpu=true的节点上
	    gpu: "true"
	  containers:
	  - image: luksa/kubia
	    name: kubia
	然后执行以下命令创建pod
	$  kubectl  create -f kubia-gpu.yaml

3.5.3  调度到一个特定节点
	内容颇多，涉及Replication-Controllers和Service，暂不解释

3.6  注解pod
    除标签外，pod和其他对象还可以包含注解，注解也是键值对，所以它们本质上与标签非常相似。但与标签不同，注解并不是为了保存标识信息而存在的，它们不能像标签一样用于对对象进行分组。当我们可以通过标签选择器选择对象时，就不存在注解选择器这样的东西。
        另一方面，注解容纳更多的信息，并且主要用于工具使用。Kubernetes也会将一些注解自动添加到对象，但其他的注解则需要由用户手动添加。
        大量使用注解可以为每个pod或其他API对象添加说明，以便每个使用该集群的人都可以快速查找有关每个单独对象的信息。例如，指定创建对象的人员姓名的注解可以使在集群中工作的人员支架你的协作更加便利。

3.6.1  查找对象的注解
——可以通过获取pod的完整YAML文件或使用kubectl describe命令，例如
	$  kubectl get po kubia-zxzij -o yaml
	$  kubectl describe  pod  kubia-zxzij
	注意：注解包含数据的总共不超过256KB
	
3.6.2 添加和修改注解
        和标签一样，注解也可以在创建时添加到pod中，也可以在之后再对现有的pod进行添加和修改。
        将注解添加到现有对象的最简单的方法是通过kubectl annotate命令。
——添加注解到kubia-manual pod中
	$  kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"
	结果：	pod "kubia-manual" annotated

3.7  使用命名空间对资源进行分组
        命名空间为资源名称提供了一个作用域。

3.7.1  了解对命名空间的需求

3.7.2  发现其他命名空间及其pod
——列出集群中的所有命名空间
	$  kubectl get ns
	结果：	NAME	LABELS	STATUS	AGE
		default	<none>	Active	1h
		kube-public	<none>	Active	1h
		kube-system	<none>	Active	1h
		test-for-create-aeskey   	<none>	Terminating   	3d23h

——指定命名空间，列出只属于该命名空间的pod：
	$  kubectl  get  po  --namespace  kube-system
	结果：	NAME	READY	STATUS	RESTARTS	AGE
		kube-dns-v9-p8a4t	0/4	Running	0	1h
		kube-ui-v4-kdlai	1/1	Running	92	1h
	注意：也可以使用  -n 来代替 --namespace

3.7.3  创建一个命名空间
        命名空间是一种和其他资源一样的Kubernetes资源，因此可以通过将YAML文件提交到Kubernetes API服务器来创建该资源。

        从YAML文件创建命名空间——选择使用YAML文件创建命名空间的方式，只是为了强化Kubernetes中的所有内容都是一个API对象这一概念。
        例如：
	apiVersion: v1
	kind: Namespace                 这表示我们正在定义一个命名空间
	metadata:
	  name: custom-namespace                  这是命名空间的名称
	 
        使用kubectl create namespace命令创建命名空间
——使用kubectl create namespace命令创建命名空间
	$  kubectl  create namespace custom-namespace
	结果：	namespace "custom-namespace" created
	注意：尽管大多数对象的名称必须符合RFC 1035（域名）中规定的命名规范，这意味着它们可能只包含字母、数字、横杠（-）和点号，但命名空间（和另外几个）不允许包含点号。

3.7.4  管理其他命名空间中的对象
——如果想要在刚创建的命名空间中创建资源，可以选择在metadata字段中添加一个namespace：custom-namespace属性，也可以在使用kubectl create命令创建资源时指定命名空间：
	$  kubectl  create -f kubia-manual.yaml -n custom-namespace
	结果：	pod "kubia-manual" created
	此时有两个同名的pod（kubia-manual），一个在default命名空间中，另一个在custom-namespace中。
	在列出、描述、修改或删除其他命名空间中的对象时，需要给kubectl命令传递 --namespace（或 -n）选项。如果不指定命名空间，kubectl将会在当前上下文配置的默认命名空间中执行操作。而当前上下文的命名空间和当前上下文本身都可以通过kubectl config命令进行更改。
	注意：如果想要切换到不同的命名空间，可以设置一下列名：aliaskcd='kubectl config set-context $（kubectl config current-context）--namespace'。
	             然后，可以使用kcd some-namespace在命名空间之间进行切换。
	
3.7.5  命名空间提供的隔离
        尽管命名空间将对象分隔到不同的组，只允许你对属于特定命名空间的对象进行操作，但实际上命名空间之间并不提供对正在运行的对象的任何隔离。

3.8  停止和移除pod

3.8.1 按名称删除pod
——按名称删除kubia-gpu pod：
	$  kubectl  delete  po kubia-gpu
	结果：	pod "kubia-gpu" deleted
——按名称删除多个pod（通过指定多个空格分隔的名称来删除多个pod）
	$  kubectl  delete  po pod1 pod2
	
3.8.2  使用标签选择器删除pod
——删除kubia-manual和kubia-manual-v2 pod，这两个pod都包含标签creation_method=manual，所以可以通过使用一个标签选择器来删除它们。
	$  kubectl  delete  po -l creation_method=manual
	结果：	pod "kubia-manual" deleted
		pod "kubia-manual-v2" deleted


3.8.3  通过删除整个命名空间来删除pod
——删除命名空间custom-namespace（pod会伴随命名空间自动删除）：
	$  kubectl  delete  ns custom-namespace
	结果：	namespace "custom-namespace" deleted
	
3.8.4  删除命名空间中的所有pod，但保留命名空间
——删除当前命名空间中的所有pod：
	$  kubectl  delete  po --all
	结果：	pod "kubia-zxzij" deleted
	注意：在这里出现了ReplicationController，什么什么不会直接创建pod，而是创建了一个ReplicationController，然后再由ReplicationController创建pod。因此要删除由该ReplicationController
	             创建的pod，它便会立即创建一个新的pod。如果想要删除该pod，还需要删除这个ReplicationController。

3.8.5  删除命名空间中的（几乎）所有资源
——通过使用单个命令删除当前命名空间中的所有资源，可以删除ReplicationController和pod，以及创建的所有service：
	$  kubectl  delete  all  --all
	结果：	pod "kubia-09as0" deleted
		replicationcontroller "kubia" deleted
		service "kubernetes" deleted
		service "kubia-http" deleted
	解释：命令中的第一个all指定正在删除所有资源类型，而--all选项指定将删除所有资源实例，而不是按名称指定它们。
	注意：用all关键字删除所有内容并不是真的完全删除所有内容。一些资源（例如Secret）会被保留下来，并且需要被明确指定删除。
	             kubectl  delete  all  --all命令也会删除名为kubernetes的Service，但它应该会在几分钟后自动重新创建。
	
	
	
	
