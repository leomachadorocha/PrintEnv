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

# Environment Variables #

6. Increase the weights for one service by 20% and verify that you retrieve seven images of one type and three of the other for every ten calls to the route. 
```
oc edit route ab-cotd-route
```
