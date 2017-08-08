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

9. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```

10. Change the value of APP_VAR_1 to VALUE1
```
oc set env dc/printenv --overwrite APP_VAR_1=VALUE1
```

11. View the environment variables created. 
```
curl $(oc get route printenv | awk '{print $2}' | grep printenv) | jq -S
```


# Demonstrate ConfigMap #
