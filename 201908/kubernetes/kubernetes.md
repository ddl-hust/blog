Kubernetes Introduction for VMware Users – Part 1: The Theory
=============================================================

``给 VMware 用户的 Kubernetes 简介–-第一部分``：理论

_By Hany Michaels, Senior Staff Solutions Architect NSBU, VMware_

This is the second part of my “Kubernetes in the Enterprise” blog series. As I mentioned in my [last article](http://www.hanymichaels.com/2017/10/04/kubernetes-in-the-enterprise-a-vmware-guide-on-how-to-design-deploy-and-operate-k8saas-with-nsx-t-and-vra/), it is important to get everyone to the same level of understanding about Kubernetes ([K8s](https://kubernetes.io/)) before we can proceed to the design and implementation guides.

```ini
这是我的“ kubernetes 在企业中应用” 博客系列的第二篇文章。正如我在上一篇文章提到的，重要的是要让每个人都相同级别的了解 在我们推进设计与实现指南
```



I am not going to take the traditional approach here to explain the Kubernetes architecture and technologies. I will explain everything through comparisons with the [vSphere platform](https://www.vmware.com/products/vsphere.html) that you, as a VMware user, are already familiar with. You can say that this was the approach I would have liked someone to use to introduce K8s to me. The latter could be very confusing and overwhelming to understand at the beginning. I’d like to add also that I used this approach internally at VMware to introduce Kubernetes to some audiences from different practices, and it has proven to work great and get people up to speed with the core concepts.

```ini
在这里，我不打算采用传统的方法来解释Kubernetes体系结构和技术。我将通过与vSphere平台的比较来解释您作为VMware用户已经熟悉的所有内容。你可以说，这就是我希望别人向我介绍k8的方法。后者在一开始可能会让人感到非常困惑和难以理解。我还想补充一点，我在VMware内部使用这种方法向来自不同实践的一些听众介绍Kubernetes
```



An important note before we kick this off: I am not using this comparison for the sake of it, or to prove any similarities or differences between vSphere and Kubernetes. Both are distributed systems at heart, and they must have similarities like any other similar system out there. What I am trying to achieve here at the end of the day is to introduce an incredible technology like Kubernetes to the broader VMware community.

```

```



![Kubernetes](img/kubernetes-architecture-1024x512.png)

Figure: The Kubernetes overall architecture compared to vSphere

A little bit of history
-----------------------

You should already be familiar with containers before reading this post. I am not going to go through those basics as I am sure there are so many resources out there that talk about this. What I see very often though when I speak with my customers is that they cannot make much sense of why containers have taken our industry by storm and become very popular in record time. To answer this question, and in fact set the context for what is coming, I may have to tell you a little bit about my history as a practical example of how I personally made sense of all the shift that is happening in our industry.

I used to be a web developer back in 2003 before I got introduced to the telecom world, and it was my second paying job after being a network engineer/admin. (I know, I was a jack-of-all-trades back then). I used to code in PHP, and I’ve done all sorts of applications from small internal apps used by my employer, to professional voting apps for TV programs, to telco apps interfacing with VSAT hubs and interacting with satellite systems. Life was great, except for one major hurdle that I am sure every developer can relate to: the dependencies.

I first code my app on my laptop using something like the LAMP stack, and when it works well, I upload the source code to the servers, be it hosted out on the internet (anyone remember RackShack?) or on private servers for our end-customers. As you can imagine, as soon as I do that, my app is broken and it just won’t work on those servers. The reason, of course, is that the dependencies I use (like Apache, PHP, MySQL, etc.) have different releases than what I used on my laptop. So I have to figure out a way to either upgrade those releases on the remote servers (bad idea) or just re-code what I did on my laptop to match the remote stacks (worse idea). It was a nightmare, and sometimes I hated myself and questioned why I’m doing this for a living.

Fast forward 10 years, and along came a company called Docker. I was a VMware consultant in professional services (2013) when I heard about Docker, and let me tell you that I couldn’t make any sense of that technology back in those days. I kept saying things like: “Why would I run containers when I can do that with VMs?” “Why would I give up important features like vSphere HA, DRS or vMotion for those weird benefits of booting up a container instantly or skipping the “hypervisor” layer?” In short, I was looking at this from a pure infrastructure perspective.

But then I started looking closer until it just hit me. Everything Docker is all about relates to developers. Only when I started thinking like one did it click. What if I had this technology back in 2003 and packaged my dependencies? My web apps would work no matter what server they run on. Better yet, I don’t have to keep uploading source code or setting up anything special. I can just “package” my app in an image and tell my customer to download that image and run it. That’s a web developer’s dream!

Docker solved a huge issue for interop and packaging, but now what? As an enterprise customer, how can I operate this app at scale? I still want my HA, my DRS, my vMotion and my DR. It solved my developer problems and it created a whole bunch of new ones for my DevOps team. They need a platform to run those containers the same way they used to run VMs.

But then along came Google to tell the world that it has been actually running containers for years (and in fact invented them – Google: cgroups), and that the proper way to do that is through a platform they called Kubernetes. They then open sourced it, gave it as gift to the community, and that changed everything again.

Understanding Kubernetes by comparing it to vSphere
---------------------------------------------------

So what is Kubernetes? Simply put: it is to containers what vSphere was for VMs to make them data center ready. If you used to run VMware Workstation back in the early 2000s, you know that they were not seriously considered for running inside data centers. Kubernetes brings a way to run and operate containers in a production-ready manner. This is why we will start to compare vSphere side-by-side with Kubernetes in order to explain the details of this distributed system and get you up to speed on its features and technologies.

![Kubernetes](img/p2p3-1024x375.png)

Figure: The VM evolution from Workstation to vSphere compared to the current evolution for containers to Kubernetes

System Overview
---------------

Just like vSphere’s vCenter and ESXi hosts, Kuberentes has the concept of master and nodes. In this context, the K8s master is equivalent to vCenter in that it is the management plane of the distributed system. It is also the APIs’ entry point where you interact with your workloads management. Similarly, the K8s nodes act as the compute resources like ESXi hosts. This is where you run your actual workloads (in K8s’ case we call them pods). The nodes could be virtual machines or physical servers. In vSphere’s case, of course, the ESXi hosts have to be physical always.

![Kubernetes](img/kubernetes-system-1024x624.png)

You can see also that K8s has a key-value store called “etcd.” It is similar to vCenter Server DB in that you store the cluster configuration as the desired state you want to adhere to there.

On the differences side, K8s master can also run workloads, but vCenter cannot. The latter is just a virtual appliance dedicated to management. In K8s master case, it’s still considered a compute resource, but it’s not a good idea to run enterprise apps on it. Only system related apps would be fine.

So, how does this look in the real world? You will mainly use CLI to interact with this system (GUI is also a viable option). In the screenshot below, you can see that I am using a Windows machine to connect to my Kubernetes cluster via command like (I am using cmder in case you are wondering). We see in the screenshot that I have one master and 4 x nodes. They run K8s v1.6.5, and the nodes operating system is Ubuntu 16.04. At the time of this writing, we are mainly living in a Linux world where your master and nodes are always based on Linux distributions.

![Kubernetes](img/clidash-1024x563.png)

Workloads Form-factor
---------------------

In vSphere, a virtual machine is the logical boundary of an operating system. In Kubernetes, pods are the boundaries for containers. Just like an ESXi host that can run multiple VMs, a K8s node can run multiple pods. Each Pod gets a routed IP address just like VMs to communicate with other pods.

In vSphere, applications run inside OS. In Kubernetes, applications run inside containers. A VM can run one single OS, while a Pod can run multiple containers.

![Kubernetes](img/kubernetes-pods-1024x486.png)

This is how you can list the pods inside a K8s cluster using the kubectl tool from the CLI. You can check the health of the pods, the age, the IP addresses and the nodes they are currently running inside.

![Kubernetes](img/cli2-1024x450.png)

Management
----------

So how do we manage our master, nodes and pods? In vSphere, we use the Web Client to manage most (if not all) the components in our virtual infrastructure. This is almost the same with Kubernetes with the use of the Dashboard. It is a nice GUI-based web portal where you can access your browser similarly to  Web Client. We’ve also seen in the previous sections that you can manage your K8s cluster using the kubeclt command from the CLI. It’s always debatable where you will spend most of your time — the CLI or the Dashboard, especially because the latter is becoming more powerful every day (check [this video](https://www.youtube.com/watch?v=3lhf7T9Bp2E) for more details). I personally find the Dashboard very convenient for quickly monitoring the health or showing the details of the various K8s components rather than typing long commands. It’s a preference, and you will find the balance between them naturally.

![Kubernetes](img/kubernetes-management-1024x469.png)

Configurations
--------------

One of the very profound concepts in Kubernetes is the desired state of configurations. You declare what you want for almost any Kubernetes component through a YAML file, and you create that using your kubectl (or through dashboard) as your desired state. Kubernetes will always strive from this moment on to keep that as a running state in your environment. For example, if you want to have four replicas of one pod, K8s will keep monitoring those pods. If one dies or the nodes it’s running have issues, it will self-heal and automatically create that pod somewhere else.

Back to our YAML configuration files — you can think of them like a .VMX file for a VM, or a .OVF descriptor for a virtual appliance that you want to deploy in vSphere. Those files define the configuration of the workload/component you want to run. Unlike VMX/OVF files that are exclusive to VMs/Appliances, the YAML configuration files are used to define any K8s component like ReplicaSets, Services, Deployments, etc. as we will see in the coming sections.

![Kubernetes](img/kubernetes-confiugrations-1024x511.png)

Virtual Clusters
----------------

In vSphere, we have physical ESXi hosts grouped logically to form clusters. We can slice those clusters into other virtual clusters called “Resource Pools.” Those resource pools are mostly used for capping resources. In Kubernetes, we have something very similar. We call them “namespaces,” and they could also be used to ensure resource quotas as we will see in the next section. They are most commonly used, however, as a means of multi-tenancy across applications (or users if you are using shared K8s clusters). This is also one of the ways  we can perform security segmentation with NSX-T across those namespaces as we will see in future posts.

![Kubernetes](img/kubernetes-namespaces-1024x651.png)

Resource Management
-------------------

As I mentioned in the previous section, namespaces in Kubernetes are commonly used as a means of segmentation. The other use case for it is resource allocation, and it is referred to as “Resource Quotas.” As we saw in previous sections, the definition of that is through YAML configuration files where we declare the desirted state. In vSphere, we similarly define this from the Resource Pools settings as you see in the screenshot below.

![Kubernetes](img/kubernetes-resource-quotas.png)

Workloads Identification
------------------------

This is fairly easy and almost identical between vSphere and Kubernetes. In the former, we use the concepts of tags to identify or group similar workloads, while in the latter we use the term “labels” to do this. In Kubernetes’ case, this is mandatory where we use things like “selectors” to identify our containers and apply the different configurations for them.

![Kubernetes](img/kubernetes-labels-1024x331.png)

Redundancy
----------

Now to the real fun. If you were/are a big fan of vSphere FT like me, you will love this feature in Kubernetes despite some differences in the two technologies. In vSphere, this is a VM with a running instance and a shadow one in a lock-step. We record the instructions from the running instance and replay it in the shadow VM. If the running instance goes down, the shadow VM kicks in immediately. vSphere then tries to find another ESXi host to bring another shadow instance to maintain the same redundancy. In Kubernetes, we have something very similar here. The ReplicaSets are a number you specify to run multiple instances of a pod. If one pod goes down, the other instances are available to serve the traffic. In the same time, K8s will try to bring up a substitute for that pod on any available node to maintain the desired state of the configuration. The major difference, as you may have already noticed, is that in the case of K8s, the pod instances are always live and service traffic. They are not shadowed workloads.

![Kubernetes](img/kuberentes-replicasets-1024x546.png)

Load Balancing
--------------

While this might not be a built-in feature in vSphere, it is still a common thing to run load-balancers on that platform. In the vSphere world, we have either virtual or physical load-balancers to distribute the network traffic across multiple VMs. This could be running in many different configuration modes, but let’s assume here that we are referring to the one-armed configuration. In this case, you are load-balancing your network traffic east-west to your VMs.

Similarly in Kubernetes, we have the concepts of “Services.” A K8s Service could also be used in different configuration modes, but let’s pick the “ClusterIP” configuration here to compare to the one-armed LB. In this case, our K8s Service will have a virtual IP (VIP) that is always static and does not change. This VIP will distribute the traffic across multiple pods. This is especially important in the Kubernetes world were pods are ephemeral by nature and where you lose the pod IP address the moment it dies or gets deleted. So you have to always be able to maintain a static VIP.

As I mentioned, the Services have many other configurations like “NodePort,” where you basically assign a port on the node level and then do a port-address-translation down to the pods. There is also the “LoadBalancer,” where you spin up an LB instance from a 3rd-party or a cloud provider.

![Kubernetes](img/kubernetes-services-1024x389.png)

There is another very important load-balancing mechanism in Kuberentes called “Ingress Controller.” You can think of this like an in-line application load-balancer. The core concept behind this is that an ingress-controller (in a form of pod) would be spun up with an externally visible IP address, and that IP could have something like a wild card DNS record. When traffic hits an ingress-controller using the external IP, it will inspect the headers and determine through a set of rules you pre-define to which pod that hostname should belong. Example: _sphinx-v1_.esxcloud.net will be directed to the service “sphinx-svc-1”, while the _sphinx-v2_.esxcloud.net will be directed to the service “sphinx-svc2” and so on and so forth.

![Kubernetes](img/kubernetes-ingress-1024x532.png)

Storage & Networking
--------------------

Storage and networking are rich topics when it comes to Kubernetes. It is almost impossible to talk briefly about these two topics in an introduction blog post, but you can be sure that I will be blogging in details soon about the different concepts and options for each subject. For now, let’s quickly examine how the networking stack works in Kubernetes since we will have a dependency on it in a later section.

Kubernetes has different networking “plugins” that you can use to set up your nodes and pods network. One of the common plugins is “kubenet,” which is currently used on mega-clouds like Google Cloud Provider (GCP) and Amazon Web Services. I am going to talk briefly here about the GCP implementation and then show you a practical example later to examine this yourself on Google Container Engine (GKE).

![Kubernetes](img/gke-kubernetes-networking-1024x747.png)

This might be a bit too much to take in from a first glance, but hopefully you will be able to make sense of all that by the end of this blog post. First, we see that we have two Kubernetes nodes here: node 1 and node (m). Each node has an eth0 interface like any Linux machine, and that interface has an IP address to the external world—in our case here on subnet 10.140.0.0/24. The upstream L3 device is acting as our default gateway to route our traffic. This could be a L3 switch in your data center or a VPC router in a public cloud like GCP as we will see later. So far so good?

Next, we see inside the node that we have a **cbr0** bridge interface. That interface has the default gateway of an IP subnet—10.40.1.0/24 in case of node 1. This subnet gets assigned by Kubernetes to each node. The latter usually get a /24 subnet, but you can control that in case of NSX-T as we will see in future posts. For now, this subnet is the one that we will allocate the pod’s IP addresses from. Any pod inside node 1 will get an IP address from this subnet rage—in our case here, Pod 1 has an IP address of 10.40.1.10. Notice however that this pod has two nested containers within. Remember, a pod can run one or more containers that are tightly coupled together in terms of functionality. This is what we see here. Container 1 is listening to port 80, while container 2 is listening to port 90. Both containers share the same IP address of 10.40.1.10. but they do not own that networking namespace. Alright, so who owns this networking stack then? It’s actually a special container that we call the “Pause Container.” You see it in the diagram as the interface of that pod to the outer world. It owns this networking stack, including the IP address 10.40.1.10 and, of course, it forwards the traffic to container 1 using port 80 and traffic to container 2 using port 90.

Now you should be asking, how is traffic forwarded to the external world? You see that we have a standard Linux IP forwarding enabled here to forward the traffic from cbr0 to eth0. This is great, but then how does the L3 device know how to forward this to the destination? We do not have dynamic routing here to advertise this network in this particular example. And so, this is why we need to have some kind of “static route” on that L3 device to know that in order to reach subnet 10.40.1.0/24, your entry point is the external IP of node 1 (10.140.0.11), and in order to reach subnet 10.40.2.0/24, your next hope is node (m) with the IP address 10.140.0.12.

This is great, but you must be thinking that it’s a very unpractical way to manage your networks. This would be an absolute nightmare for network administrators to maintain all those routes as you scale with your cluster. And you’re right—this is why we need some kind of solution like the CNI (container network plugin) in Kubernetes to use a networking mechanism to manage this for you. NSX-T is one of those solutions with a powerful design for both the networking and security stacks.

Remember, we are examining here the kubernetes plugin, not CNI. The former is what GKE uses, and the way they do this is quite fascinating as it’s completely programmable and automated on their cloud. Those subnet allocations and associated routes are taken care of by GCP for you, as we will see in the next part.

_Stay tuned to the [Cloud-Native Apps blog](https://blogs.vmware.com/cloudnative/) for Part 3 of Hany’s post on designing Kubernetes for the enterprise, and be sure to follow us on Twitter ([@cloudnativeapps](https://twitter.com/cloudnativeapps))._