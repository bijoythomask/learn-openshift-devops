# Triggers

## Removing all triggers

        oc set triggers dc myapp --remove-all

        oc set triggers dc myapp

## Enable and disable Config Change Trigger

        oc set triggers dc myapp --from-config --remove
        oc set triggers dc myapp

        oc set triggers dc myapp --from-config
        oc set triggers dc myapp

## Enable and disable Image Change Triggers

### import image stream into our namespace

        oc import-image docker.io/busybox:latest --confirm

### Add an image trigger to a deployment config

        oc set triggers dc myapp --from-image=welcome/busybox:latest \
                --containers=myapp

### Add our myapp image trigger back as well

        oc set triggers dc myapp --from-image=welcome/myapp:latest \
                --containers=myapp

        oc set triggers dc myapp

        NAME TYPE VALUE AUTO
        deploymentconfigs/myapp config true
        deploymentconfigs/myapp image busybox:latest (myapp) true
        deploymentconfigs/myapp image myapp:latest (myapp) true
