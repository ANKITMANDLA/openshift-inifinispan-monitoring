# Monitoring


## Name
Openshift template **Prometheus and Grafana**

## Description
Openshift template files  for Prometheus and Grafana for the purpose of installations on user managed openshift project. 

## prerequisite 
- Openshift project access
- Openshift Admin team provide access  **prometheus and Grafana Operator** on user  managed namespace
- Check if service account **grafana-serviceaccount** & **prometheus-k8s** exist on given namespace where we suppose to install monitoring solution. 
- Slack webhook URLs for prometheus alert notification. Get it provisioned from   slack workspace  owner. 
- slack channel available for prometheus alert. 

## prepration
- create Grafana service account in monitoring namespace
  >oc create sa infinispan-monitoring

- Give view permission to SA
  >oc adm policy add-role-to-user view -z infinispan-monitoring

- get Grafana SA token for **thanos-qurier data source.
  >oc serviceaccounts get-token infinispan-monitoring
- update variable **SA_TOKEN** on  file **grafana.parms** file with token value extracted from above steps. 
- create secret for slack notification in monitoring namespace.
  > oc create secret generic slack-notification --from-literal apiURL=<SLACK_WEBHOOK_URL>
  >
  > example: 
    > 
  >  oc create secret generic slack-notification --from-literal  apiURL=https://hooks.slack.com/services/****/****/*******

- Create secret for Datagird service monitor in monitoring namespace. Get the operator password from DG cluster secret. 
  > oc create secret generic rhdg-operator-secret --from-literal username=operator --from-literal password=< Operator password >

- Update  **grafana.parms** and **prometheus.parms** file with appropriate values.

## Installation
- Run below command to install **prometheus** in monitoring namespace
    >oc create -f promtetheus-template.yaml
    
    >oc process prometheus --param-file=prometheus.parms | oc apply -f -


- Run below command to install **Grafana** in monitoring namespace
    > oc create -f  grafana-template.yaml

    > oc process grafana --param-file=grafana.parms | oc apply -f -

    - create Grafana dashboards
    > oc create -f dashboard/grafana-infinispan-custom-dashboard.yml
    >
    > oc create -f dashboard/grafana-kube-dashboard.yml
    >
    > **Note** - create infinispan-custom dashboard only if this is being set up for RHDG
- **inifinispan-custom** dashboard will populate data only if DG monitoring is enabled. 

- If monitoring at DG cluster is  enabled, then one grafana dashboard called **inifinispan** should also be created. 
## Verification
- Check if all preometheus, alertmanager & grafana pods are up and running
    > oc get po

- Verify all Prometheus, Alertmanager & Grafana routes created
- Verify prometheus rules are created.
- Verify  grafana dashboard is created.

- login to Grafana dashboard with credential admin/welcome123, you should see infinispan-custom, kube-state-metrics-v2 created.

- Check if  prometheus metrics are available either from Grafana or Prometheus. 


## Templates for blackbox and Probes
- To create the templates we need to execute the following commands in monitoring namespace
    > cd blackbox

    > oc create -f blackbox-exporter-template.yaml
    
    > oc process blackbox-exporter --param-file=blackbox.parms | oc apply -f -

- Verify the blackbox exporter service, deployment and ConfigMap were created
- To create the prometheus custom probes please issue the following commands in monitoring namespace
    > cd probes

    > oc create -f probes-template.yaml

    > oc process prober --param-file=probes.parms | oc apply -f -

    > oc create -f probes-conn-template.yaml

    > oc process prober-conn --param-file=probes-conn.parms | oc apply -f -

    > oc create -f probes-cache-template.yaml

    > oc process prober-cache --param-file=probes-cache.parms | oc apply -f -
    
- Verify the 3 probes in Prometheus operators are created

## Deploy Prometheus Rules (This will be deployed along with prometheus.)
- This will be deployed along with prometheus. If rules needs to be created seprately then this can be used.
- Purpose of  this to update prometheus rules outside of prometheus deployment
- Dont run when are  you installing for the very first time. 
- To create the prometheus rules please issue the following commands in monitoring namespace. Update parm namespace & region in rules.parms file based on enviornment you are running.

    > cd prometheusRules

    > oc create -f rules-template.yaml

    > oc process prometheus-rules --param-file=rules.parms | oc apply -f -
    

- Verify the  prometheus rules in Prometheus operators are created


## Deploy All the other namespaces datasources
- Run the following commands to create each Thanos datasource

    > oc create -f thanos-qurier-template.yml

    > oc process thanos-datasource-template --param-file=thanos-qurier-rhdg.parms | oc apply -f -

    > oc process thanos-datasource-template --param-file=thanos-qurier-cpq.parms | oc apply -f -

    > oc process thanos-datasource-template --param-file=thanos-qurier-apis.parms | oc apply -f -

    > oc process thanos-datasource-template --param-file=thanos-qurier-dof.parms | oc apply -f -

    > oc process thanos-datasource-template --param-file=thanos-qurier-om.parms | oc apply -f -

    > oc process thanos-datasource-template --param-file=thanos-qurier-omf.parms | oc apply -f -


## Deploy the dashboards for all the other namespaces
- create Grafana dashboards for all the other namespaces
    > oc create -f dashboard/grafana-omf-kube-dashboard.yml
    >
    > oc create -f dashboard/grafana-om-kube-dashboard.yml
    >
    > oc create -f dashboard/grafana-cpq-kube-dashboard.yml
    >
    > oc create -f dashboard/grafana-dof-kube-dashboard.yml
    >
    > oc create -f dashboard/grafana-apis-kube-dashboard.yml
    >
    > create -f dashboard/grafana-infinispan-custom-dashboard.yml
    ## The infinispan custom will be recreated to add new functionalities

