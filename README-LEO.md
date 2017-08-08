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
OBS: "jq -S" sorts the output by key, making it easier to find a particular variable. 



# Demonstrate Environment Variables #

6. Set up two environment variables. 
```
oc set env dc/printenv APP_VAR_1=Value1 APP_VAR_2=Value2
```

7. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```

8. Delete the second environment variable.
```
oc set env dc/printenv APP_VAR_2-
```

9. View the environment variables values (step 7).
   
   
10. Change the value of APP_VAR_1 to VALUE1
```
oc set env dc/printenv --overwrite APP_VAR_1=VALUE1
```

11. View the environment variables values (step 7). 



# Demonstrate ConfigMap #

1. Create a ConfigMap with two environment variables (using Literal Values).
```
oc create configmap printenv-configmap \
    --from-literal=APP_VAR_3=Value3 \
    --from-literal=APP_VAR_4=Value4
```

2. Update the deployment configuration to retrieve these two variables from the ConfigMap.
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

3. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```


4. Update the ConfigMap with different values (VALUE3, VALUE4).
```
oc edit cm printenv-configmap
```

5. View the environment variables values (step 3). 



# Demonstrate Environment Variables from a text file #

1. Set an specific environment variable to read from the file.
```
oc set env dc/printenv READ_FROM_FILE=/temp/configfile.txt
```

2. Create a configuration file.
```
echo "This is the Config File" > temp/configfile.txt
```

3. Create a ConfigMap using this file (using Specific File).
```
oc create configmap printenv-configmap-file \
    --from-file=temp/configfile.txt
```

4. Mount the ConfigMap to the container at the /temp location.
```
oc set volume dc/printenv --add --overwrite --name=configmap-volume -m /temp/ -t configmap --configmap-name=printenv-configmap-file
```

5. Verify that now the application is returning the contents of the text file (do not use jq since the output is no longer JSON)
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv)
```
