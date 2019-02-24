# Lifecycle Hooks

# Rolling Strategy (Pod-based Lifecycle Hook)

    kind: DeploymentConfig
    apiVersion: v1
    metadata:
    name: frontend
    spec:
    template:
        metadata:
        labels:
            name: frontend
        spec:
        containers:
            - name: helloworld
            image: openshift/origin-ruby-sample
    replicas: 5
    selector:
        name: frontend
    strategy:
        type: Rolling
        rollingParams:
        pre:
            failurePolicy: Abort
            execNewPod:
            containerName: helloworld 
            command: [ "/usr/bin/command", "arg1", "arg2" ] 
            env: 
                - name: CUSTOM_VAR1
                value: custom_value1
            volumes:
                - data

# Demo

## We use two containers in the example

In the example that follows, we are going to create a Postgres database schema and
load the default data using Liquibase change sets. If you haven’t come across this
library before, there are alternative database migration libraries such as Flyway that
may be familiar.

* A Postgres database container.

* The database configuration is supplied by the dbinit container. Configuration (via
liquibase) is layered into the Docker image at /deployments. The change set
records are exported to an XML file on a PVC as the last (post- hook) step.

By using two containers we can keep the database runtime and its configuration separate.

The Liquibase change sets allow the example to be rerun multiple times as the same
change set will not be applied twice.

Another spin on this example is to execute a pre lifecycle hook to initialize the database and a mid lifecycle hook to perform the database schema changes.

The example creates a schema called test in a Postgres database. The schema generation
uses annotated SQL scripts for data loading. The deployment hooks commands are specified in the JSON template file. If you examine the template file, you will see
that the Postgres database connection for Liquibase is specified using environment
variables.

## Create a Postgres database (Recreate deployment strategy)

     > Create New Project

     oc new-project postgres --display-name="postgres" --description="postgres"

     > Create the Prostgres Template

     oc create -f postgresql-persistent-template.json

     > Create the Prostgres Instance

     oc new-app --template=postgresql-persistent \
                -p POSTGRESQL_USER=user \
                -p POSTGRESQL_PASSWORD=password \
                -p POSTGRESQL_DATABASE=test

     oc patch dc postgresql -p '{"spec":{"strategy":{"type":"Recreate"}}}'

     > Set Environment Varriable

     oc set env dc postgresql POSTGRESQL_ADMIN_PASSWORD=password

     > Inspect Pods

     oc get pods

     NAME               READY   STATUS  RESTARTS    AGE
     postgresql-2-o662j 1/1     Running 0           5m

## Log in to the test database as follows

    oc rsh $(oc get pods -lapp=postgresql-persistent -o name)
    psql -H localhost -d test -U postgres

    psql (9.5.4)
    Type "help" for help.
    test=#

## Examine the database Using Postgres commands

    test=# \dt+
    No relations found.

    test=# \dn
    List of schemas
    Name | Owner
    --------+----------
    public | postgres
    (1 row)

    Enter Ctrl-D, Ctrl-D (or type \q, exit) to quit.

## Create PVC

    oc create -f dbinit-data-pvc.yaml

    oc new-app --name=dbinit --strategy=docker https://github.com/devops-with-openshift/liquibase-example.git

    oc delete dc dbinit

    oc process -f dbinit-deployment-config.json \
    -p="IMAGE_STREAM=$(oc export is dbinit --template='{{range .spec.tags}}{{.from.name}}{{end}}')" \
    |  oc create -f -

## trigger a deployment

    oc deploy dbinit --latest
    > Check Logs
    oc logs -f dc dbinit
    > Check Events
    oc get events | grep dbinit

## After DB init deployment again examine the database Using Postgres commands

    winpty oc rsh $(oc get pods -lapp=postgresql-persistent -o name)

    psql -H localhost -d test -U postgres

    test=# \dt+
    No relations found.

    test=# \dn
    List of schemas
    Name | Owner
    --------+----------
    public | postgres
    (1 row)

    Enter Ctrl-D, Ctrl-D (or type \q, exit) to quit.


## References :- 

* <https://raw.githubusercontent.com/openshift/openshift-ansible/master/roles/openshift_examples/files/examples/latest/db-templates/postgresql-persistent-template.json>
* <https://raw.githubusercontent.com/devops-with-openshift/liquibase-example/master/dbinit-data-pvc.yaml>