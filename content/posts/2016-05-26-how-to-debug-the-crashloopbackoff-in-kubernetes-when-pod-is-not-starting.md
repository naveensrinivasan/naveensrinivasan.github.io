---
title: How to debug the CrashLoopBackOff in Kubernetes when pod is not starting
author: admin
type: post
date: 2016-05-26T02:01:16+00:00
url: /?p=3163
categories:
  - kubernetes

---
Here is my learning of how I debugged the CrashLoopBackOff in kubernetes when the pod wasn&#8217;t starting.

I wanted to deploy the <a href="https://hub.docker.com/_/jenkins/" target="_blank">jenkins docker</a> image in the cluster. As mentioned in the jenkins docker  repo I wanted to mount an external drive which is an AWS EBS volume.  Here is my deployment yaml.

[gist id = &#8220;98aa6da98ebb9b7e3d7f996c8ef2cb38&#8221;]

After starting the deployment the pod never came up and this the output of  _kubectl get pod_

_NAME                                      READY      STATUS                  RESTARTS  AGE_
  
 _jenkins-3317895845-x84u3  0/1       CrashLoopBackOff      10                 27m_

The next step was to issue the _kubectl describe pod jenkins-3317895845-x84u3_

[gist id = &#8220;7a79af84b53fc923aa609997fdb4fcfd&#8221;]

So from the logs I could make out the container is being pulled correctly but it is failing on **StartContainer**.  Now based on this information the next step was to get the docker logs. But the docker logs aren&#8217;t accessible from my box because it is managed by kubernetes. The only way to get the docker logs was to actually ssh into the box.

But which box should I ssh? The describe output has the node information which is _Node: ip-172-20-0-29.us-west-2.compute.internal/172.20.0.29_ . This_ _is the private ip of the box in aws but with that information you should be able to figure out the public ip to ssh.

Now I know I have to ssh but where is the key for this server. If you have used the <a href="http://releases.k8s.io/release-1.2/cluster/kube-up.sh" target="_blank">kube-up.sh</a> then the keys would be stored  in this location _~/.ssh/kube\_aws\_rsa. ssh -i ~/.ssh/kube\_aws\_rsa admin@public-ip-oftheabovenode._

After_ _sshing into the box I issued the command _sudo docker ps -a | grep naveen _because the container could have been stopped and looked for naveen because that was my container name. This gave me container id which was stopped with exit status as 1.

And this was the output of docker logs command

_admin@ip-172-20-0-29:~$ sudo docker logs c98a338268a1_
  
_touch: cannot touch ‘/var/jenkins\_home/copy\_reference_file.log’: Permission denied_
  
_Can not write to /var/jenkins\_home/copy\_reference_file.log. Wrong volume permissions?_

which identified the _/var/jenkins_home  _which was mounted with aws ebs voulme  didn&#8217;t have permission to write by the jenkins user <a href="https://github.com/kubernetes/kubernetes/issues/2630" target="_blank">https://github.com/kubernetes/kubernetes/issues/2630</a>.

And after doing all of this I realized I could have done _kubectl logs jenkins-3317895845-x84u3_ which would have given the same output without having to ssh into the box. But knowing this handy because when things go wrong we really need to debug the root cause.