# Installation-of-Metrics-Server-in-K8s-Cluster
This installation guide will help you to install metrics server in your K8s cluster.
### Why Do We Need Metrics Server?
Kubernetes does not provide a default metrics aggregator. We need to install Metrics Server which helps to collect the container level metrics like CPU or RAM usage. These metrics can then be used by other Kubernetes APIs and objects like the Horizontal Pod Autoscaler (HPA) or Vertical Pod Autoscaler (VPA).
### Installation Of Metrics Server on Master Node
To install all the components of Metrics Server from the components.yaml manifest file, run the following command.
>```
> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
>```
Now to check whether the metrics server is created or not, run the following command.
>```
> kubectl get pods -n kube-system
> ```
Now if we check the output of the above command, we can see that the mertrics server is running but it is not **Ready(0/1)**

To check why the Metrics Server is not Ready, Use the below Command:
> ```
>  kubectl -n kube-system describe pod ***(here copy the name of metrics server pod from the above output)***
> ```
If we check in the events section from the output of above command, we can see that
> The pod is not able to communicate with the API Server

We can fix this by overriding the metrics server command, we can do this by running the below command
>```
> kubectl -n kube-system edit deploy metrics-server
> ```
After running the above command press **"i"** (insert mode) and bring your cursor down to container section,add the below lines before **image tag**, Once done press esc:wq and come out of the file.
```yaml
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```
After adding the above lines it should look like
```yaml
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```
By running the above command we are editing and deploying the file at the same time

Now if we check the metrics server status by running the below command, we can see that the status as RUNNING and it is **READY 1/1**
>```
> kubectl get pods -n kube-system
> ```
Now to confirm wether the metrics server is running, we can check that by running top command
>```
> kubectl top nodes
> ```
By running the above command we can see the CPU & Memory Usage of the nodes.
