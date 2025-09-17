# Ldap server for deployment

This is an example of how to deploy a simple ldap server on kubernetes.
It is configured with an initial bootstrap LDIF that creates two sample users (jdoe and asmith) with traefik.io domain.

## Openldap server deployment

Apply the manifests:

```shell
kubectl apply -f https://raw.githubusercontent.com/Fernando-Benegas/ldap-server-example/refs/heads/main/k8s/ldap.yaml
```

Verify the pod is running:

```shell
kubectl get pods -n ldap
```

Expected status:

```shell
NAME                        READY   STATUS    RESTARTS   AGE
openldap-6bbd87d5c5-gh4cq   1/1     Running   0          28m
```


### Testing
1. Port-forward the LDAP service

Forward LDAP port 389 from Kubernetes to your local machine:

```shell
kubectl port-forward -n ldap svc/openldap 389:389
```
2. Test admin bind

Authenticate using the admin DN (cn=admin,dc=traefik,dc=io) and password (admin123, from the secret):

```
ldapwhoami -x -H ldap://localhost:389 \
  -D "cn=admin,dc=traefik,dc=io" \
  -w admin123
```

Expected output:

```shell
dn:cn=admin,dc=traefik,dc=io
```

3. Query sample users

Search for all inetOrgPerson objects:

```shell
ldapsearch -x -H ldap://localhost:389 \
  -b "dc=traefik,dc=io" \
  -D "cn=admin,dc=traefik,dc=io" \
  -w admin123 "(objectClass=inetOrgPerson)"
```

Expected results include the bootstrap users:

```
John Doe

uid: jdoe

mail: jdoe@traefik.io

Alice Smith

uid: asmith

mail: asmith@traefik.io
```


## Whoami app deployment


Apply the manifests:

```shell
kubectl apply -f https://raw.githubusercontent.com/Fernando-Benegas/ldap-server-example/refs/heads/main/k8s/whoami.yaml
```

Verify the pod is running:

```shell
kubectl get pods -n apps
```

Expected status:

```shell
NAME                        READY   STATUS    RESTARTS   AGE
whoami-784cdc7fb4-7xxc2     1/1     Running   0          28m
```


### Testing

1. Send a request to the whoami app without autentication: 

```shell
curl -v "https://localhost/whoami"
```

Expected status:

```shell
* Request completely sent off
< HTTP/2 401 
< content-length: 0
```


2. Now, add the ldap credentials on Authentication header:

```shell
curl -v -X GET "https://localhost/whoami" -H "Authorization: Basic amRvZToxMjM0"
```

Expected status:

```shell
> GET / HTTP/2
> Host: localhost
> User-Agent: curl/8.12.1
> Accept: */*
> Authorization: Basic amRvZToxMjM0
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* Request completely sent off
< HTTP/2 200 
< content-type: text/html; charset=utf-8
```

