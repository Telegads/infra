# nginx ingress

<https://microk8s.io/docs/getting-started>
1 - microk8s

```
sudo snap install microk8s --classic --channel=1.26
```

2 - user

```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

su - $USER
```

3 - basic addons

```
microk8s enable dns hostpath-storage
```

4 - copy config

```
microk8s config
```

wait if you see some warnings.

5 - CRDs cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.1.1/cert-manager.crds.yaml
```

6 - haproxy ingress, cert-manager

```
helmfile sync -f ./helmfile/helmfile.yaml -e infra
```

## Add New Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: nginx    
    cert-manager.io/cluster-issuer: "letsencrypt-prod"

spec:
  tls:
  - hosts:
    # host for certificate
    - kube.nikitin-petr.ru
    secretName: test
  rules:
  - host: kube.nikitin-petr.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # name of target service
            name: kubernetes-dashboard
            # port in target service
            port:
              number: 443

```
