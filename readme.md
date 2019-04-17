# Let’s Encrypt and Rancher 2.0 with cert-manager

## 1 - Preparing the enviroment

* kubectl is running (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Download the kubeconfig file (```Rancher > Global > Clusters > [Cluster] > Download Kubeconfig File```)
* Put this into ```~/.kube/config```:

## 2 - Install cert-manager on Rancher

Install cert-manager on project level.

* `Rancher > [Cluster] > [Project] > Apps > Launch`
* Find `cert-manager` and [view details]
* Set false the option **Create Default Cluster Issuer**
* Set cluster as **Available Roles**
* Then Launch


## 3 - Install ingress-nginx



```yaml
# ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

# load balancer
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/cloud-generic.yaml
```

## 4 - Apply the issuer

The issuer is the the layer of comunication between rancher and the letsencrypt. 

The content of the file is in: `1-create-prod-issuer.yaml`

1. Change your e-mail on line **9**
2. Run: 

```yaml 
kubectl create -f 1-create-prod-issuer.yaml
```

To view the result, just run: `kubectl describe Issuer letsencrypt-staging`

## 5 - Create the ingress

This example uses two hosts: `apisslteste.brasilsabido.com.br` and `apisslteste2.brasilsabido.com.br`.

The file to run is `2-create-ingress.yaml`.

1. On line 4, set your ingress name (any name);
2. On line 12, set the domains list for the ingress;
3. On lines 18 and 26, is used theese domains;
4. On lines 22 and 30, set your service name.
5. Run:

```yaml
kubectl create -f 2-create-ingress.yaml
```

## 5.1 - Creating ingress by rancher interface

1. Configure the ingress
2. In ssl/tls, add certificate, then set your host
3. use the follow annotations:
```yml
kubernetes.io/ingress.class=nginx
certmanager.k8s.io/cluster-issuer=letsencrypt-prod
kubernetes.io/tls-acme="true"
```
4. Save
5. An anoing bug is that sometimes the value yaml for `spec > tls > hosts > secretName` is not filled. This means, that is necessary go [View/Edit Yaml] and add the secret name:
```yaml
#[...]
  tls:
  - hosts:
    - geoapi.brasilsabido.com.br
    # ADD_THE_NEXT_LINE
    secretName: letsencrypt-prod
status:
  loadBalancer:
#[...]
```


![Rancher](https://github.com/joaoverissimo/letsencrypt-rancher-cert-manager-config/raw/master/1554632302190.png "Rancher")

![Rancher](https://github.com/joaoverissimo/letsencrypt-rancher-cert-manager-config/blob/master/1554633150466.png?raw=true "Rancher")



## References

* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes
* https://www.idealcoders.com/posts/rancher/2018/06/rancher-2-x-and-lets-encrypt-with-cert-manager-and-nginx-ingress/

## Troubleshooting

If the message _customresourcedefinitions.apiextensions.k8s.io "certificates.certmanager.k8s.io" already exists_, is necessary delete the customresourcedefinition. 
```
kubectl get customresourcedefinition | certmanager
kubectl delete customresourcedefinition challenges.certmanager.k8s.io
```
