# Rolling Strategy Exercise

## 1. Create New Project and Appliction

    oc new-project welcome --display-name="Welcome"  --description="Welcome"

    oc new-app devopswithopenshift/welcome:latest --name=myapp

## 2. Set Readiness Probe

        oc set probe dc myapp --readiness\
        --open-tcp=8080 \
        --initial-delay-seconds=5 \
        --timeout-seconds=5

## 3. Set Liveness Probe

        oc set probe dc myapp --liveness -- echo ok

## 4. Expose Service

        oc expose svc myapp --name=welcome

## 5. Inspect the deployment config

        oc describe dc myapp

### Expected Result

        ...
        Replicas: 1
        Triggers: Config, Image(myapp@latest, auto=true)
        Strategy: Rolling
        ...

## Triggger new deployment

        oc rollout myapp --latest


