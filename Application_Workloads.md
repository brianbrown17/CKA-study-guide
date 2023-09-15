# Application Workloads
Running containerized applications.
### Contaiers
command: ["sh", "-c"] for longer shell scripts
## Pod
Typically, it will be faster to create a pod declaratively, using `kubectl run`. For a quick reference when creating pods, type `kubectl run --help`.

This will display helpful examples like:
```
$ kubectl run --help
  # Start a nginx pod
  kubectl run nginx --image=nginx
```
For more control over the pod creation:
```
$ kubectl run -n nginx nginx --image=nginx -oyaml --dry-run=client > pod.yaml
```
The make the required edits.
### Static Pods
Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. They are bound to that kubelet.

First. create a file `static-web.yaml` in the `/etc/kubernetes/manifests` directory.
```
mkdir -p /etc/kubernetes/manifests/
cat <<EOF >/etc/kubernetes/manifests/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```
Next, edit the kulet config file (`/var/lib/kubelet/config.yaml`) with `staticPodPath: /etc/kubernetes/manifests`. Last, restart the kubelet `systemctl kubelet restart`

Note: The control plane components are run using static manifests.
### Scheduling
### Taints and toleration
#### Scheculing Priority
Priority indicates the importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the scheduler tries to preempt (evict) lower priority Pods to make scheduling of the pending Pod possible.

To use priority, imperatively create a PriorityClass (non-namespaced):
```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```
Priority `value` can go up to 1 billion. The `globalDefault` field indicates that the value of this PriorityClass should be used for Pods without a priorityClassName

Preemption occurs when a pod cannot schedule on the node. if a pod has `preemptionPolicy: Never` then it can still have a higher priority, but will not evict other pods.

### Affinity/Anti-Affinity
The `topologyKey` field is used by the scheduler to determine the domain for Pod placement.Nodes as being in the same topology. The scheduler tries to place a balanced number of Pods into each topology domain.

With podAffinity, a Pod will be scheduled in the same domain as the Pods that match the expression.
```
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
```
Since the domain is `topology.kubernetes.io/zone`, Pods will be scheduled in the same zone as a Pod that matches the expression. For podAntiAffinity, the opposite would be true.

Another common `topologyKey` is `kubernetes.io/hostname`.
### Pod Disruption Budgets
```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: zookeeper
```
You can specify only one of maxUnavailable and minAvailable in a single PodDisruptionBudget

Note: when you run `kubectl drain``, the tool tries to evict all of the Pods on the Node you're taking out of service, but it will not evict a pod with a replica number that is lower than its PDB