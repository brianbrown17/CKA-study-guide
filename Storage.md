## Config Map
Used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

To create one:
```
$ kubectl create configmap mydata --from-literal=mykey=myvalue
```
You can also upload data from a file with --from-file

To mount onto a pod as an environment variable, run `kubectl run pod1 --image=nginx -oyaml --dry-run=client > pod.yaml` and add the following `env`:
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: nginx
      env:
        - name: MY_DATA
          valueFrom:
            configMapKeyRef:
              name: mydata
              key: mykey
```
Validate with `kubectl exec mypod -- env | grep mykey`.

To mount onto the pod as a volume (to be able to read/write to `/etc/mydata/*`), make the following edits instead:
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  volumes:
    - name: mydata
      configMap:
        name: mydata
  containers:
    - name: demo
      image: nginx
      volumeMounts:
        - name: mydata
          mountPath: /etc/mydata
```
Validate with `kubectl exec mypod -- cat /etc/mydata`