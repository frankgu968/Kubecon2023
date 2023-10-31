# Kubecon2023
My Kubecon2023 talk on "How We Optimized Developer Productivity with Telepresence"

# Environment Set up 

## Commands
```bash
kubectl create ns kubecon2023
kubectl apply -f k8s-billing-gateway.yaml -n kubecon2023
kubectl apply -f k8s-converter.yaml -n kubecon2023

## To delete
# kubectl delete -f k8s-billing-gateway.yaml
# kubectl delete -f k8s-converter.yaml
```

# Telepresence Commands
Please follow [these instructions](https://www.getambassador.io/docs/telepresence-oss/latest/install) to set up Telepresence. 
Special care should be taken if you intend to use the file mounting feature. See [this article](https://www.getambassador.io/docs/telepresence-oss/latest/troubleshooting#volume-mounts-are-not-working-on-macos) for more details.
1. Clear any existing Telepresence sessions with `telepresence quit`
2. Connect to an existing kubernetes cluster with `telepresence connect`

### Intercepting a service
```bash
mkdir -p /tmp/billing-gateway # Set up tmp directory to store env file and mounts

# Launch a shell with port forwarding
telepresence intercept billing-gateway -n kubecon2023 --port 8080:app --env-file /tmp/billing-gateway/.env --mount /tmp/telepresence-mounts/billing-gateway -- /bin/sh
ln -s /tmp/telepresence-mounts/billing-gateway/tmp/billing-gateway/mounts /tmp/billing-gateway/mounts
```

Once an intercept is active, you can consider your local machine "inside" the target k8s cluster: 
- DNS to `.cluster.local` endpoints will resolve correctly to the appropriate ClusterIP
- Telepresnce will forward traffic on the local port (eg. 8080) to the target port on the Deployment (eg. app)
- Environment variables are injected into the target process (eg. /bin/sh)
- Environment variables are stored in a dotenv file (eg. /tmp/.env.billing-gateway-telepresence)
- Pod file mounts are also mounted on the local machine at the specified mount point (eg. /tmp/billing-gateway)