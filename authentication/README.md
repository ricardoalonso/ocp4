# HTPASSWD authentication

##### Creating an HTPasswd file

```
# htpasswd -c -B -b </path/to/htpasswd> <user_name> <password>
```

The command generates a hashed version of the password

##### Add more users or update already existing users to the HTPasswd file

```
# htpasswd -b </path/to/htpasswd> <USER> <PASSWORD>
```

##### Create a Secret to hold the HTPasswd file

```
# oc create secret generic htpasswd-secret --from-file=htpasswd=</path/to/htpasswd> -n openshift-config
```

##### Create the custom resource

```
# oc create -f htpasswd-copnfig.yaml
```

### Updating the HTPasswd file

##### Dump current htpasswd secret content to htpasswd file

```
# oc get secret htpasswd-secret -n openshift-config -o jsonpath={.data.htpasswd} | base64 -d > ocp-htpasswd
```

##### Add or update user passwords

```
# htpasswd -Bb ocp-htpasswd <USER> <PASSWORD>
```

##### Patch htpasswd secret data with content from file

```
# oc patch secret htpasswd-secret -n openshift-config -p '{"data":{"htpasswd":"'$(base64 -w0 ocp-htpasswd)'"}}'
