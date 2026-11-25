# K8sDemo

Step 2: Create the Job
kubectl apply -f job.yaml


Step 3: Verify Job
kubectl get jobs


Expected output:
NAME            COMPLETIONS   DURATION   AGE
my-first-job    1/1           12s        20s


Step 4: Check Pod
kubectl get pods


Step 5: Check Job Logs
kubectl logs job/my-first-job


Expected output:
Hello Kubernetes
Job Completed


Step 6: Describe Job
kubectl describe job my-first-job


Step 7: Delete Job
kubectl delete job my-first-job


Important Job Parameters


------------------------------------part-2 ----------

2. Kubernetes Secret Creation
A Kubernetes Secret stores sensitive information such as:
- Database username and password
- API keys
- Application credentials
- Authentication tokens
For example, let's create a Secret for a MySQL database.
Method 1: Create Secret using kubectl command
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=MyDB@123


Step 2: Verify Secret
kubectl get secrets


Expected output:
NAME        TYPE     DATA   AGE
db-secret   Opaque   2      10s


Step 3: Describe Secret
kubectl describe secret db-secret


Expected output:
Name:         db-secret
Type:         Opaque
Data
====
password:  9 bytes
username:  5 bytes


Notice that Kubernetes does not display the actual values in the describe output.
Step 4: Decode Secret Values
Kubernetes stores Secret data in Base64 encoding by default.
kubectl get secret db-secret \
  -o jsonpath='{.data.username}' | base64 --decode


Output:
admin


Decode password:
kubectl get secret db-secret \
  -o jsonpath='{.data.password}' | base64 --decode


Output:
MyDB@123


Important security point
Base64 is encoding, not encryption. Enable Kubernetes encryption at rest and apply proper RBAC restrictions to protect Secrets.

3. Create Secret using YAML
Create a file named secret.yaml.

apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  username: YWRtaW4=
  password: TXlEQkAxMjM=



Here:
- YWRtaW4= = admin
- TXlEQkAxMjM= = MyDB@123
Create Secret:
kubectl apply -f secret.yaml


Verify:
kubectl get secrets



4. Use Secret inside a Kubernetes Pod
Now let's use the Secret inside an Nginx container as environment variables.

Create secret-pod.yaml.

apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-pod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: password



Create Pod:
kubectl apply -f secret-pod.yaml


Verify:
kubectl get pods


Check environment variables:
kubectl exec secret-demo-pod -- printenv DB_USERNAME


Output:
admin


Check password:
kubectl exec secret-demo-pod -- printenv DB_PASSWORD


Output:
MyDB@123
