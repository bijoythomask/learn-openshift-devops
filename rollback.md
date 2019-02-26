
# Rollback

## 1. Create new porject
    
    oc new-project rollback --display-name='Rollback Deployment Example' \
        --description='Rollback Deployment Example'

## 2. Deploy Cats application 

    oc new-app --name='cotd' \
        -l name='cotd' php:5.6~https://github.com/devops-with-openshift/cotd.git \
        -e SELECTOR=cats

    oc expose service cotd --name=cotd -l name='cotd'

## 3. we can change the environment variable in our deployment configuration to cities, which will trigger a new deployment displaying cities instead of cats

    oc env dc cotd SELECTOR=cities

## 4. Let’s see what a rollback to revision 1 of our deployment will look like, but don’t perform the rollback:

    oc rollback cotd --to-version=1 --dry-run

    oc rollback cotd --to-version=1

    oc set triggers dc cotd --auto

    oc describe dc cotd