NAME                       READY   STATUS    RESTARTS   AGE
pod/db-6c5c8947cd-djsl5    1/1     Running   0          17m
pod/web-f78cd89c-9c25r     1/1     Running   0          6m45s
pod/web-f78cd89c-rhlr7     1/1     Running   0          6m54s

NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/db    ClusterIP   10.96.29.209   <none>        5432/TCP   30m
service/web   ClusterIP   10.96.71.101   <none>        80/TCP     28m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db    1/1     1            1           30m
deployment.apps/web   2/2     2            2           28m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/db-6c5c8947cd    1         1         1       30m
replicaset.apps/web-6846548998   0         0         0       11m
replicaset.apps/web-7d868c9479   0         0         0       28m
replicaset.apps/web-f78cd89c     2         2         2       6m54s

NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/db-data   Bound    pvc-3d497094-8798-40b2-bcd3-a18d8e166e5f   1Gi        RWO            standard       <unset>                 30m
```

## 4. Experiment 2 — Persistent Data

Output of `curl http://localhost:8000/notes` after the database pod was deleted and recreated:

```text
[{"body":"hello from kubernetes","created_at":"2026-09-28T03:07:27.314549+00:00","id":1}]
```

The note remained available after the PostgreSQL pod was deleted and recreated, demonstrating that the database data persisted through the PersistentVolumeClaim.

## 5. Experiment 3 — Load Balancing

The web deployment was scaled to four replicas and multiple requests were sent to the Kubernetes web Service.

```text
deployment.apps/web scaled
Waiting for deployment "web" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "web" rollout to finish: 3 of 4 updated replicas are available...
deployment "web" successfully rolled out

{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-jzljl","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-gvlsz","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-rhlr7","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-gvlsz","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-9c25r","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-rhlr7","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-jzljl","service":"notes-app"}
```

The `served_by` values show requests being distributed among four different web pods:

- `web-f78cd89c-jzljl`
- `web-f78cd89c-gvlsz`
- `web-f78cd89c-rhlr7`
- `web-f78cd89c-9c25r`

## 6. Rolling Update

Output of `kubectl rollout history deployment/web` after the rolling update:

```text
deployment.apps/web
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>
```

The application was deployed using an immutable SHA-tagged image during the rolling-update experiment. The new image used for the update was:

```text
masjane99/notes-app:sha-397fae0
```

The rolling update completed successfully. A rollback was also tested during the experiment.

## 1. Successful GitHub Actions Run

Screenshot: `evidence/github-actions-success.png`

The GitHub Actions workflow completed successfully on the `main` branch and published the Docker image to Docker Hub.

## 2. Docker Hub Multi-Architecture Image

Screenshot: `evidence/dockerhub-tags.png`

The Docker Hub `masjane99/notes-app` repository contains the `latest` and SHA tags. The `latest` image supports:

- linux/amd64
- linux/arm64

## 3. Kubernetes Deployment

Output of `kubectl get all,pvc`:

```text
jane-claudiamasangu@MacBook-Air-4 notes-app % kubectl get all,pvc
NAME                      READY   STATUS    RESTARTS   AGE
pod/db-6c5c8947cd-djsl5   1/1     Running   0          17m
pod/web-f78cd89c-9c25r    1/1     Running   0          6m45s
pod/web-f78cd89c-rhlr7    1/1     Running   0          6m54s
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/db    ClusterIP   10.96.29.209   <none>        5432/TCP   30m
service/web   ClusterIP   10.96.71.101   <none>        80/TCP     28m
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db    1/1     1            1           30m
deployment.apps/web   2/2     2            2           28m
NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/db-6c5c8947cd    1         1         1       30m
replicaset.apps/web-6846548998   0         0         0       11m
replicaset.apps/web-7d868c9479   0         0         0       28m
replicaset.apps/web-f78cd89c     2         2         2       6m54s
NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/db-data   Bound    pvc-3d497094-8798-40b2-bcd3-a18d8e166e5f   1Gi        RWO            standard       <unset>                 30m
jane-claudiamasangu@MacBook-Air-4 notes-app % 

 curl http://localhost:8000/notes
[{"body":"hello from kubernetes","created_at":"2026-09-28T03:07:27.314549+00:00","id":1}]
jane-claudiamasangu@MacBook-Air-4 notes-app % 

kubectl scale deployment web --replicas=4
kubectl rollout status deployment/web
kubectl run curl --rm -it --restart=Never --image=curlimages/curl -- sh -c 'for i in 1 2 3 4 5 6 7 8; do curl -s http://web/; echo; done'
deployment.apps/web scaled
Waiting for deployment "web" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "web" rollout to finish: 3 of 4 updated replicas are available...
deployment "web" successfully rolled out
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-jzljl","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-gvlsz","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-rhlr7","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-gvlsz","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-9c25r","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-rhlr7","service":"notes-app"}
{"message":"Hello from Jane's notes app!","served_by":"web-f78cd89c-jzljl","service":"notes-app"}
Session ended, resume using 'kubectl attach curl -c curl -n notes-lab -i -t' command
pod "curl" deleted from notes-lab namespace
jane-claudiamasangu@MacBook-Air-4 notes-app % 

kubectl scale deployment web --replicas=2
deployment.apps/web scaled
jane-claudiamasangu@MacBook-Air-4 notes-app % 

jane-claudiamasangu@MacBook-Air-4 notes-app % kubectl rollout history deployment/web
deployment.apps/web 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>
jane-claudiamasangu@MacBook-Air-4 notes-app % 

