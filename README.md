# passbolt

create new namespace:
```bash
kubectl create ns passbolt
```

Start mariadb server:
```bash
kubectl run mariadb --image=mariadb --expose --port=3306 \
  --env=MYSQL_ROOT_PASSWORD=passbolt \
  --env=MYSQL_DATABASE=passbolt \
  --env=MYSQL_USER=passbolt \
  --env=MYSQL_PASSWORD=passbolt
```

start passbolt:
```bash
kubectl run passbolt --image=passbolt/passbolt --expose --port=80 \
  --env=DATASOURCES_DEFAULT_HOST=mariadb \
  --env=DATASOURCES_DEFAULT_PASSWORD=passbolt \
  --env=DATASOURCES_DEFAULT_USERNAME=passbolt \
  --env=DATASOURCES_DEFAULT_DATABASE=passbolt \
  --env=APP_FULL_BASE_URL=https://passbolt.k8s.shubhamtatvamasi.com
```

Add Ingress:
```bash
kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: passbolt
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
      - passbolt.k8s.shubhamtatvamasi.com
    secretName: passbolt-tls
  rules:
  - host: passbolt.k8s.shubhamtatvamasi.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: passbolt
            port:
              number: 80
EOF
```

delete everything:
```bash
kubectl delete ns passbolt
kubectl delete po/mariadb svc/mariadb
kubectl delete po/passbolt svc/passbolt ing/passbolt
```

create admin user
```bash
kubectl exec -it passbolt -- su -m -c "bin/cake \
  passbolt register_user \
  -u info@shubhamtatvamasi.com \
  -f shubham \
  -l tatvamasi \
  -r admin" \
  -s /bin/sh www-data
```
> click the link to activate
