##Ldap server for deployemnt

This is just an example of how to deploy a simple ldap server on kubernetes.
It is configured with an initial bootstrap LDIF that creates two sample users (jdoe and asmith) with traefik.io domain.

#Deployment

Apply all manifests:

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
Load the ldif data into the openldap container:

```shell
kubectl exec -n ldap deploy/openldap -c openldap -- ldapadd -x -D "cn=admin,dc=traefik,dc=io" -w admin123 -f /ldif-data/bootstrap.ldif
```

Testing
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

