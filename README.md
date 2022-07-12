# istio-multi-deploy-test

Repo to test why communication to services that has multiple deploys rolling out at the same time in combination
with istio-proxy termination duration produces intermittent 503 upstream errors.

Choose what flavour to test and follow the instructions below.

|flavour|description|
|-|-|
|`./istio`|`default istio setup`|
|`./istio-round-robin`|`istio with destination rule setting the load balancer to be of type ROUND_ROBIN`|
|`./no-istio`|`setup without istio`|

## prepare

Please set the environment variable `NAMESPACE` to an istio enabled namespace to deploy these tests in.  
This environment variable should also be set in all future shells running any of the tests / commands below.

## deploy

```shell
kubectl apply -n ${NAMESPACE} -k .
```

## run test

```shell
# shell 1 - start viewing logs.
kubectl logs -n ${NAMESPACE} -l app.kubernetes.io/name=multi-deploy-checker --timestamps --follow
```

Please wait to the initial creation is completed and 2 out of 2 pods are ready before starting the test.

```shell
# shell 2 - start the rotation.
kubectl rollout -n ${NAMESPACE} restart deploy multi-deploy 
sleep 110
kubectl rollout -n ${NAMESPACE} restart deploy multi-deploy
sleep 540
kubectl get -n ${NAMESPACE} pods | grep "multi-deploy" | grep -v "checker"
```

## cleanup

```shell
kubectl delete -n ${NAMESPACE} -k .
```
