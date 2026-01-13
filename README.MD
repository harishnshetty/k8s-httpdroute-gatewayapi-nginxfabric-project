kubectl create namespace nginx-gateway
kubectl apply -f gateway.yaml
kubectl apply -f httproute.yaml
kubectl apply -f deployment.yaml


kubectl get pods -n application
kubectl get pods -n route-app
kubectl get svc -n application
kubectl get svc -n route-app



parentRefs:
- name: nginx-gateway          # Gateway name
  namespace: nginx-gateway     # Gateway's namespace (cross-namespace!)
  sectionName: harishshetty-http    # Which listener to use


  kubectl get httproute -A

  kubectl describe httproute app-route -n application


GATEWAY_IP=$(kubectl get gateway nginx-gateway -n nginx-gateway -o jsonpath='{.status.addresses[0].value}')


# Test each application
curl -k -H "Host: nginx.harishshetty.xyz" http://$GATEWAY_IP
curl -k -H "Host: routeapp.harishshetty.xyz" http://$GATEWAY_IP



# Check if label exists
kubectl get namespace application --show-labels

# Add missing label
kubectl label namespace application gateway-access=true
# Force route reconciliation
kubectl delete httproute app-route -n application
kubectl apply -f app-route.yaml



1. Use Namespace Selectors, Not from: All
❌ Avoid (insecure):

allowedRoutes:
  namespaces:
    from: All  # Anyone can attach!
✅ Use (secure):

allowedRoutes:
  namespaces:
    from: Selector
    selector:
      matchLabels:
        gateway-access: "true"
        environment: production  # Add more labels for granular control