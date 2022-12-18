# Basic setup of microk8s with nginx ingress and cert-manager

## Mincrok8s

<https://microk8s.io/docs/getting-started>
1 - microk8s

```bash
sudo snap install microk8s --classic --channel=1.26
```

2 - user

```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

su - $USER
```

3 - basic addons

```bash
microk8s enable dns hostpath-storage
```

4 - copy config

```bash
microk8s config
```

## Nginx + cert-manager

1 - CRDs cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.1.1/cert-manager.crds.yaml
```

2 - haproxy ingress, cert-manager

```bash
helmfile sync -f ./helmfile/helmfile.yaml -e infra
```

## Add New Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tg-notification-bot-{{ .Values.env }}
  annotations:
    kubernetes.io/ingress.class: haproxy    
    cert-manager.io/cluster-issuer: "letsencrypt-prod"

spec:
  tls:
    - hosts:
        - {{ .Values.host }}
      secretName: bot-ssl-{{ .Values.env }}
  rules:
    - host: {{ .Values.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tg-notification-bot-{{ .Values.env }}
                port:
                  number: 3009
```
