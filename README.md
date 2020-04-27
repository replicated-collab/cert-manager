Cert-Manager + LetsEncrypt
==================

Example project showcasing a solution to use cert-manager and LetsEncrypt to automatically generate certificates for an NGINX ingress. h

## How Charts were downloaded (TGZ placed in manifest directory)
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm fetch jetstack/cert-manager
helm fetch stable/nginx-ingress

## Infra Setup (DNS, Static IP, Cluster Firewall)
(For some reason, CRDs need to be applied *before* helm chart is installed?)
```
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.13/deploy/manifests/00-crds.yaml

gcloud compute addresses create helloweb-ip --global
IP_ADDR=$(gcloud compute addresses describe helloweb-ip --global | awk 'NR==1 {print $2}')
IP_ADDR=gcloud compute addresses describe helloweb-ip --global --format 'value(address)'
gcloud dns --project=smart-proxy-839 record-sets transaction start --zone=materiography
gcloud dns --project=smart-proxy-839 record-sets transaction add $IP_ADDR --name=hello2.materiography.com. --ttl=300 --type=A --zone=materiography
gcloud dns --project=smart-proxy-839 record-sets transaction execute --zone=materiography
gcloud compute firewall-rules create k8s-cert-manager --source-ranges 172.16.0.0/28 --target-tags gke-austins-cluster-15-844f99b4-node --allow TCP:6443 
```

## Validating (should show 'Ready' with cert info)
```
kubectl get certificate hello2-materiography-com-tls --namespace cert-manager
kubectl describe certificate hello2-materiography-com-tls --namespace cert-manager
```
