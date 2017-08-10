This simple Node.js application returns all of the available environment variables as JSON data, but if a specific environment variable READ_FROM_FILE is setted, it returns the contents of a local text file.

 
# Create the Project # 

1. Create an empty project.
```
oc new-project printenv --display-name="Print all environment variables" --description="Print all environment variables"
```

2. Create an application instance.
```
oc new-app https://github.com/leomachadorocha/PrintEnv
```

3. Create a route.
```
oc expose svc printenv
```

4. View the route.
```
oc get route printenv | awk '{print $2}' | grep printenv
```

5. Retrieve the available environment variables using the route that you just created.
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```
> OBS: "jq -S" sorts the output by key, making it easier to find a particular variable.    
   
 
 
# A - Demonstrate Environment Variables #

6a. Set up two environment variables. 
```
oc set env dc/printenv APP_VAR_1=Value1 APP_VAR_2=Value2
```

7a. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```

8a. Delete the second environment variable.
```
oc set env dc/printenv APP_VAR_2-
```

9a. View the environment variables values (step 7).
   
   
10a. Change the value of APP_VAR_1 to VALUE1
```
oc set env dc/printenv --overwrite APP_VAR_1=VALUE1
```
11a. View the environment variables values (step 7). 



# B - Demonstrate ConfigMap #

6b. Create a ConfigMap with two environment variables (using Literal Values).
```
oc create configmap printenv-configmap \
    --from-literal=APP_VAR_3=Value3 \
    --from-literal=APP_VAR_4=Value4
```

7b. Update the deployment configuration to retrieve these two variables from the ConfigMap.
```
oc edit dc printenv
```
```
spec:
  containers:
  - env:
    - name: APP_VAR_1
      value: Value1
    - name: APP_VAR_3
      valueFrom:
        configMapKeyRef:
          name: printenv-configmap
          key: APP_VAR_3
    - name: APP_VAR_4
      valueFrom:
        configMapKeyRef:
          name: printenv-configmap
          key: APP_VAR_4
```

8b. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```

9b. Update the ConfigMap with different values (VALUE3, VALUE4).
```
oc edit cm printenv-configmap
```

10b. View the environment variables values (step 3). 



# C - Demonstrate Environment Variables from a text file #

6c. Create a configuration file.
```
cd && \
mkdir labs && \
echo "This is the Config File" > labs/configfile.txt
```

7c. Create a ConfigMap using this file (using Specific File).
```
oc create configmap printenv-configmap-file \
    --from-file=labs/configfile.txt
```

8c. Mount the ConfigMap to the container at the /temp location.
```
oc set volume dc/printenv --add --overwrite --name=configmap-volume -m /folder-in-container/ -t configmap --configmap-name=printenv-configmap-file
```
> OBS:   
remove = `oc set volume dc/printenv --remove --name=configmap-volume`    
list   = `oc set volume dc/printenv`   


9c. Set an specific environment variable to read from the file.
```
oc set env dc/printenv READ_FROM_FILE=/folder-in-container/configfile.txt
```

10c. Verify that now the application is returning the contents of the text file (do not use jq since the output is no longer JSON).
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv)
```   
   
   
   
# Secrets #
Secrets can be added to a pod through [environment variables](https://github.com/leomachadorocha/PrintEnv/blob/master/README-LEO.md#d---secret-added-through-environment-variables) or [volumes](https://github.com/leomachadorocha/PrintEnv/blob/master/README-LEO.md#e---secret-added-through-volume-mount).

## D - Secret Added Through Environment Variables ##

6d. Create the files.
```
cd && \
mkdir labs && \
echo 'p4ssw0rd' > labs/password.txt && \
echo 'admin' > labs/user.txt
```

7d. Create a printenv-secret secret that contains a user ID field named app_user and a password field named app_password.
```
oc secret new printenv-secret app_user=labs/user.txt app_password=labs/password.txt
```

8d. Validate the secret.
```
oc get secret printenv-secret -o yaml
```
```
apiVersion: v1
data:
  app_password: cjNkaDR0MSEK
  app_user: YWRtaW4K
kind: Secret
...
```

9d. Verify that the secret is created by decoding the values in the secret.
```
echo "cjNkaDR0MSEK" | base64 --decode
echo "YWRtaW4K" | base64 --decode
```

10d. Add the secret to the PrintEnv application.
```
oc env dc/printenv --from=secret/printenv-secret
```

11d. Verify that it is added, listing all the environment variables in a deployment configuration.
```
oc env dc/printenv --list
```

12d. Add the secret again to the PrintEnv application prefixing both variables with MYSQL_.
```
oc env dc/printenv --from=secret/printenv-secret --prefix=MYSQL_
```

13d. Verify that it is added (step 5).



## E - Secret Added Through Volume Mount ##

6e. Create the files.
```
cd && \
mkdir labs && \
echo 'p4ssw0rd' > labs/dbpassword.txt && \
echo 'admin' > labs/dbuser.txt && \
echo 'http://postgresql:5432' > labs/dburl.txt
```

7e. Create a printenv-db-secret secret that contains a user ID, password, and database URL, naming these app_db_user, app_db_password, and app_db_url, respectively.
```
oc secret new printenv-db-secret \
  app_db_user=labs/dbuser.txt \
  app_db_password=labs/dbpassword.txt \
  app_db_url=labs/dburl.txt
```

8e. Mount the new database secret as a volume into the PrintEnv deployment configuration.
```
oc set volume dc/printenv --add --overwrite --name=db-config-volume -m /folder-in-container/ --secret-name=printenv-db-secret
```
> OBS:   
remove = `oc set volume dc/printenv --remove --name=db-config-volume`   
list   = `oc set volume dc/printenv`   
   
9e. Set an specific environment variable to read from the file (inside the container).
```
oc set env dc/printenv READ_FROM_FILE=/folder-in-container/app_db_url
```

10e. Verify that the environment variable was added.
```
oc env dc/printenv --list
```

11e. Verify that the data is being returned as expected.
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv)
```
