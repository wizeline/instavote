# Defining the Ingress Rule for Vote

Probably you have noticed that the External-IP shown in the LoadBalancer is not working

```sh
$ kubectl get svc -n flagger-system
NAME                                                            TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
flagger-loadtester                                              ClusterIP      10.98.253.194   <none>         80/TCP                       46m
flagger-prometheus                                              ClusterIP      10.107.35.2     <none>         9090/TCP                     46m
flagger-system-nginx-ingress-ingress-nginx-controller           LoadBalancer   10.107.7.146    10.107.7.146   80:31080/TCP,443:31443/TCP   19m
flagger-system-nginx-ingress-ingress-nginx-controller-metrics   ClusterIP      10.104.72.191   <none>         10254/TCP                    19m


$ curl 10.107.7.146
<html>
<head><title>404 Not Found</title></head>
<body
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

This is because we haven't configured any rule to route the trafic to our cluster. This is the goal for this chapter.

## Creating the Ingress Rule

For this we'll add the Ingress as part of the Kustomization for the `vote` application:

```sh
cd instavote-deploy/kustomize/vote/base
cat > ingress.yaml << EOF
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote
  namespace: instavote
  labels:
    app: vote
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: vote.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: vote
                port:
                  number: 80
EOF

# Now add the resource to the Kustomization
kustomize edit add resource ingress.yaml

# Not commit and publish the updates
git add .
git commit -m "feat: Add Ingress for Vote Application"
git push origin HEAD:refs/heads/main
```

### Verify the Ingres was deployed

Once the Reconciliation succeeds, the Ingres objects should be visible:

```sh
$ kubectl get ingress -n instavote
NAME          CLASS    HOSTS              ADDRESS         PORTS   AGE
vote          <none>   vote.example.com   10.103.190.27   80      10s

$ kubectl describe ingress/vote -n instavote
Name:             vote
Labels:           app=vote
                  kustomize.toolkit.fluxcd.io/name=vote-staging
                  kustomize.toolkit.fluxcd.io/namespace=instavote
Namespace:        instavote
Address:          10.107.7.146
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /   vote:80 (172.17.0.11:80,172.17.0.12:80,172.17.0.13:80 + 1 more...)
Annotations:  kubernetes.io/ingress.class: nginx
              supported-by: support@fluxcd.io
Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  Sync    47s (x3 over 4m48s)  nginx-ingress-controller  Scheduled for sync
```

Also now you should be able to visti the EXTERNAL-IP exposed by the LoadBalancer
