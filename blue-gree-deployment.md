# Blue Green Deployment

## 1. Create Project for Blue/Green Deployment 

    oc new-project bluegreen --display-name="Blue Green Deployments" \
        --description="Blue Green Deployments"

## 2. Deploy App Version Blue

    oc new-app https://github.com/devops-with-openshift/bluegreen#master \
        --name=blue

## 3. Deploy App Version Green 

    oc new-app https://github.com/devops-with-openshift/bluegreen#green \
        --name=green

## 4. Expose blue-green Service

    oc expose service blue --name=bluegreen


## 5. switch service to green

    oc patch route/bluegreen -p '{"spec":{"to":{"name":"green"}}}'

## 6. switch back to blue again

    oc patch route/bluegreen -p '{"spec":{"to":{"name":"blue"}}}'
