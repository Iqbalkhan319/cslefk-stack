# Step 1: Set the default namespace to 'efk' for all kubectl commands
kubectl config set-context --current --namespace=efk

# Step 2: Create the namespace (if it doesn't already exist)
kubectl create ns efk

# Step 3: Clone the EFK stack manifests from your GitHub repo
git clone https://github.com/Iqbalkhan319/cslefk-stack.git

# Step 4: Enter the cloned directory
cd cslefk-stack

# Step 5: Apply all manifests in the current directory
# This will deploy:
# - Elasticsearch StatefulSet
# - Kibana Deployment
# - ConfigMaps
# - Services
# - PVCs / PVs
kubectl apply -f .

# Step 6: Wait for Elasticsearch pod to be created
# You can check status with:
kubectl get pods

# Step 7: Open a shell into the Elasticsearch container
# Replace <elasticsearch-pod-name> with actual pod (usually elasticsearch-0)
kubectl exec -n efk -it <elasticsearch-pod-name> -- /bin/bash

# Step 8: Inside the container, generate a Kibana service account token
# This will output a JWT token (eyJ...), which you must copy
bin/elasticsearch-service-tokens create elastic/kibana kibana-token
# Example output:
# elastic/kibana/kibana-token => eyJhbGciOi...

# Step 9: Exit the pod shell
exit

# Step 10: Edit your ConfigMap to inject the token into kibana.yml
# Replace the value below with the actual token from step 8
kubectl edit configmap kibana-config

# Inside the editor, set:
# kibana.yml: |
#   server.port: 5601
#   server.host: "0.0.0.0"
#   elasticsearch.hosts: [ "http://elasticsearch:9200" ]
#   elasticsearch.serviceAccountToken: "eyJhbGciOiJIUzI1NiIsInR5..."

# Step 11: Restart the Kibana pod so it picks up the updated config
kubectl delete pod -l app=kibana

# Step 12: Access Kibana via NodePort (default is 30601)
# Open in browser:
# http://<your-node-ip>:30601

# Step 13: Log in using:
# Username: elastic
# Password: changeme (set in elasticsearch manifest)

# Note: Kibana now uses the service account token internally to talk to Elasticsearch.

