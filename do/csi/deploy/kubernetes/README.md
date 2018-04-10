1. Create a secret with your DigitalOcean API Access Token:


First encode your token in base64 (in this example, the string starting with
`a05...` is the API token):

```
$  echo -n "a05dd2f26b9b9ac2asdasdsd3fabf713560a129323890123cb5d1ec17513e06da" | base64
YTA1ZGQyZjI2YjliOWFjMmFzZGFzZHNkM2ZhYmY3MTM1NjBhMTI5MzIzODkwMTIzY2I1ZDFlYzE3NTEzZTA2ZGE=
```

Write a secret resource file with the base64 output above:

```
apiVersion: v1
kind: Secret
metadata:
  name: dotoken
type: Opaque
data:
  token: YTA1ZGQyZjI2YjliOWFjMmFzZGFzZHNkM2ZhYmY3MTM1NjBhMTI5MzIzODkwMTIzY2I1ZDFlYzE3NTEzZTA2ZGE=
```

and create the secret using kubectl:

```
$ kubectl create -f ./secret.yml
secret "dotoken" created
```


2. Deploy the CSI plugin and sidecars:

```
kubectl create -f csi-storageclass.yaml
kubectl create -f csi-attacher-rbac.yaml
kubectl create -f csi-attacher-do.yaml
kubectl create -f csi-provisioner-rbac.yaml
kubectl create -f csi-provisioner-do.yaml
kubectl create -f csi-nodeplugin-rbac.yaml
kubectl create -f csi-nodeplugin-do.yaml
```

This is based on the recommended mechanism of deploying CSI drivers on Kubernetes: https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md#recommended-mechanism-for-deploying-csi-drivers-on-kubernetes

Note that the proposal is still work in progress and not all of the written
features are implemented. When in doubt, as #sig-storage in Kubernetes Slack

3. Test and verify:

Create a PersistenVolumeClaim. This makes sure a volume is created and provisioned on behalf of you:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: do-block-storage
```

After that create a Pod that refers to this volume. When the Pod is created, the volume will be attached, formatted and mounted to the specified Container

**TODO(arslan): the below is not tested yet***

```
kind: Pod
apiVersion: v1
metadata:
  name: my-csi-app
spec:
  nodeName: "nodes-2"
  containers:
    - name: my-frontend
      image: busybox
      volumeMounts:
      - mountPath: "/data"
        name: my-csi-volume
      command: [ "sleep", "1000000" ]
  volumes:
    - name: my-csi-volume
      persistentVolumeClaim:
        claimName: csi-pvc 
```
