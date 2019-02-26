# A/B Deployments

## 1. Create new project

    oc new-project cotd --display-name='A/B Deployment Example' \
        --description='A/B Deployment Example'

## 2. Deploy application version A  using our Cat of the Day application

    oc new-app --name='cats' -l name='cats' \
        php:5.6~https://github.com/devops-with-openshift/cotd.git \
        -e SELECTOR=cats

    oc expose service cats --name=cats -l name='cats'

## 3. Deploy application version B  using our City of the Day application

    oc new-app --name='city' -l name='city' \
        php:5.6~https://github.com/devops-with-openshift/cotd.git \
        -e SELECTOR=cities

    oc expose service city --name=city -l name='city'

## 4. Create  A/B Deploymnet route 

    oc expose service cats --name='ab' -l name='ab'

    oc annotate route/ab haproxy.router.openshift.io/balance=roundrobin

    oc set route-backends ab cats=100 city=0

## 5. Adjust traffic

    oc set route-backends ab --adjust city=+10%

    for i in {1..10}; do curl -s http://ab-cotd.192.168.1.121.nip.io/item.php | grep "data/.*/images" | awk '{print $5}'; done
