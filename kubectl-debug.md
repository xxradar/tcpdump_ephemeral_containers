```
kubectl create deploy www --image nginx --replicas 3

kubectl expose deploy www  --name www-service --port 80 --target-port 80

kubectl run -it --rm --image xxradar/hackon debug -- curl http://www-service

kubectl run -it --rm --image xxradar/hackon debug3
                curl -kv  http://www-service



kubectl get po -o wide
export PODNAME=www-6d49b97f5-4s7q9


------------------------------------------------------------------------------------------------------------------------
kubectl debug -it $PODNAME  --image=xxradar/hackon  -c debug -- bash
 ... ps aux

At this point, the debug container only shares the network namespace of the pod.
The pod is not restarted or terminated after we exist the the debug container.
 .... check kubectl describe po  .... see the section Ephemeral Containers ...ou can see their state 

------------------------------------------------------------------------------------------------------------------------
kubectl debug -it $PODNAME --image=xxradar/hackon --target nginx -c debug -- bash
At this point, we are capable of seeing the processes in the target container --target nginx container (of the pod)
This is till an ephemeral container 
... ps aux 
... cat /proc/1/root/etc/hostname
... ls -la /proc/1/root/var/log/nginx/access.log


------------------------------------------------------------------------------------------------------------------------
kubectl debug -it $PODNAME  --image=xxradar/hackon --copy-to=my-debugger 
At the stage the pod is copied into pod my-debugger and a contaier is attached ... 2/2

NAME                  READY   STATUS    RESTARTS   AGE
my-debugger           2/2     Running   0          30s
www-6d49b97f5-4s7q9   1/1     Running   0          48m

kubectl describe po my-debugger
 ....

------------------------------------------------------------------------------------------------------------------------
kubectl debug  -it $PODNAME --image=xxradar/hackon  --copy-to=my-debugger2 --share-processes=true -- bash
 .... ps aux


------------------------------------------------------------------------------------------------------------------------
kubectl debug node/ip-10-1-2-55 -it  --image xxradar/hackon
...
ps aux
...
Accessing host filesystem
cd /host/home/ubuntu/
...
```
