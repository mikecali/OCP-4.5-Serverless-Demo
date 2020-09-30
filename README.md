
# Abstract:

Red Hat serverless has three components, these are build, serving and events. Build is a plugable model for building artifacts like jar, files, zip or containers from source code. Serving is an event-driven model that serves the container with your application and can “scale to zero”. Events are the common infrastructure for consuming and producing events that will stimulate the application. For this lab, we will focus on building and serving only.

# Assumption: 
1. OCP 4.5 installed with 3 Masters and some worker nodes.
2. Machinesets are configured to allow node scaling.
3. A working Service Mesh operator deployed.

# Pre-requisites (Service Mesh deployment):
1. Deploy service mesh and the required components in your OCP 4.3 cluster (Make sure you are selecting the Red Hat     supported operator).
 
        a. Jaeger
        b. Elastic Search
        c. Kiali
        d. Openshift Service Mesh
![image](https://user-images.githubusercontent.com/17167732/94499298-c925b000-0258-11eb-907f-2c2d039da913.png)

2. Once Service mesh and the required components are installed, you now need to deploy the service mesh control plane. The service control plane needs to be deployed in istio-system namespace. 
        $ oc new-project istio-system

3. Deploy Service Mesh control plane to the newly created namespace istio-system. 

![image](https://user-images.githubusercontent.com/17167732/94499353-f1adaa00-0258-11eb-8f59-324a1959e79c.png)


a. Wait until the control plane pods are completely created before going to the next steps. Below are the pods that you need expect once service control plane deployment is completed.

![image](https://user-images.githubusercontent.com/17167732/94499397-13a72c80-0259-11eb-8458-152f2252a0cd.png)

5. This steps is optional - but you can test your service mesh deployment using this sample app from Istio website - https://istio.io/docs/examples/bookinfo/

# Serverless (Knative) Preparation:
1. Login as cluster-admin to your OCP 4.5 cluster

2. Navigate to Operator Hub and select Serverless Operator (by Red Hat) to install the operator.
![knative1](https://user-images.githubusercontent.com/17167732/75419766-7078f680-599b-11ea-81b4-f41e8dab78a1.png)

4. Create a namespace and name it knative-serving. This is the name-space that we will use to deploy knativeserving instance    using the serverless operator

        $oc new-project knative-serving

5. Deploy knative-native serving on the knative-serving namespace.

        $oc new-project knative-serving

![image](https://user-images.githubusercontent.com/17167732/94527569-abc20780-0293-11eb-90ba-91f20020581d.png)
      

Note to wait for the pods to be created before proceeding to the next steps. You should see something like below.

         $ oc get pods -n knative-serving
         NAME                                READY   STATUS    RESTARTS   AGE
         activator-78fb847596-kl8lp          1/1     Running   1          17h
         activator-78fb847596-q5c8t          1/1     Running   1          17h
         autoscaler-56d9fd956f-gc9fj         1/1     Running   2          17h
         autoscaler-hpa-68f6b9ddd4-4h5ph     1/1     Running   0          17h
         autoscaler-hpa-68f6b9ddd4-vrd2j     1/1     Running   0          17h
         controller-6cd8bc875b-hqqr8         1/1     Running   0          17h
         controller-6cd8bc875b-hxsl4         1/1     Running   0          17h
         kn-cli-downloads-5f555d9cf4-gw8fb   1/1     Running   0          17h
         webhook-684c455fd8-zs97d            1/1     Running   0          17h

6. Once all knative-serving pods are running, you can deploy an application. There are many ways of doing this and we will go through those samples in the next section.

# Serverless App deployment

As a requirements for this demo, first you need install `kn` commandline on your computer - 

        $ wget https://mirror.openshift.com/pub/openshift-v4/clients/serverless/latest/kn-linux-amd64-0.14.0.tar.gz
        $ tar -xvf kn-linux-amd64-0.14.0.tar.gz

Verify the installation

        $ kn version
          Version:      v0.14.0
          Build Date:   2020-06-25 16:32:56
          Git Revision: 5235b55
          Supported APIs:
          * Serving
            - serving.knative.dev/v1 (knative-serving v0.14.0)
          * Eventing
            - sources.knative.dev/v1alpha2 (knative-eventing v0.14.1)
            - eventing.knative.dev/v1alpha1 (knative-eventing v0.14.1


There are more than one way you can deploy a serverless application. These are using the following.

1. Using the usual yaml file.

        apiVersion: serving.knative.dev/v1alpha1
        kind: Service
        metadata:
          name: helloworld-go
          namespace: serverless-demo
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  env:
                    - name: TARGET
                      value: "Go Sample v1"

2. Using knative commandline `kn`  

          kn service create event-display-kn --image danielon30/quarkus-serverless:latest

3. Using OCP Developers perspective UI.
   With the developers perspective UI, you can deploy apps via `GIT`, `container image`, `Dockerfile`, `Yaml` or from `Catalog`

![image](https://user-images.githubusercontent.com/17167732/94523405-a1047400-028d-11eb-8f60-24cf1db82fa3.png)

Now that we know the different way of deploying a serverless application, let's  start deploy a simple helloworld golang application.

1. Create a namespace for the sample app that you will deploy - In this example I will use serverless-demo

        $ oc new-project serverless-demo
 
2. Deploy sample go hello world application. Below is the example code. Let’s call this service.yaml

        apiVersion: serving.knative.dev/v1alpha1
        kind: Service
        metadata:
          name: helloworld-go
          namespace: serverless-demo
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  env:
                    - name: TARGET
                      value: "Go Sample v1"


3.  Instantiate the application using the below command.

        $ oc apply -f service.yaml -n serverless-demo
        service.serving.knative.dev/helloworld-go created

4. Wait until the container is created. 

        $ oc get pods
        NAME                                              READY   STATUS              RESTARTS   AGE
        helloworld-go-2mj2m-deployment-5b5999d699-2dwwv   0/2     ContainerCreating   0          8s

5. Once the container is created, go to the developer console and select topology and check the status of your application. If it is successfully deployed using knative resource type, then you should see something like below:

        - In this view, you can see the following:
        - Resources of the application that you just deployed.
        - Routes of the application 
        - The pods that was created by the deployment 
        - The revisions where you can set traffic distribution.
        - Overall information that includes, labels, anotations, name of the application and the owner of the applications.

![image](https://user-images.githubusercontent.com/17167732/94513751-0b141d80-027c-11eb-9f40-969d0158c341.png)



6. To verify further, go to Serverless Tab on your admin console and check the services, revisions and routes. In the routes provided, you can access your sample application.

![image](https://user-images.githubusercontent.com/17167732/94522931-ed02e900-028c-11eb-9637-3ce8c7688994.png)


7. To test the serverless application, you can hit the routes provided and verify the status of the pods going to the developer console to see the status of the  pod being re-created. This is now the point where you have successfully deploy an application that can scale down to 0 and bring it back up again once the expose url is hit by a request.

![knative5](https://user-images.githubusercontent.com/17167732/75420293-9e126f80-599c-11ea-8e95-06fbc529cf91.png)

8. How about we explore the serverless serving a little bit? 
   First, lets find the serverless ingress gateway's public address
  
          $oc -n knative-serving-ingress get svc kourier
          NAME      TYPE           CLUSTER-IP     EXTERNAL-IP                                                                    PORT(S)                      AGE
          kourier   LoadBalancer   172.30.20.34   a1534e2a8f3e54314aed9b69ac64eefc-1280993366.ap-southeast-1.elb.amazonaws.com   80:32479/TCP,443:30946/TCP   24h

9. Then using `kn` command line, let us display the knative services

          $ kn service list
          NAME            URL                                                                             LATEST                AGE   CONDITIONS   READY   REASON
          event-display   http://event-display-bserverlessdemo-eventing.apps.cluster-e9f2.e9f2.example.opentlc.com   event-display-kthvp   9h    3 OK / 3     True 

# Spliting the Traffic to your Severlessi Application.

Now that your simple serverless application is actually working and has an ability to scale up and down (most importantly it can scale to 0), we now need to see how we can split the traffic to go lang application.

We will do this using commandline and developers perspective UI.

Any new change in the code or the service configuration triggers a revision, a snapshot of the code at a given time. For a service, you can manage the traffic between the revisions of the service by splitting and routing it to the different revisions as required.

1. Let's update the helloworld-go application

          $ kn service update helloworld-go --env RESPONSE="Hello OpenShift!"
          Updating Service 'helloworld-go' in namespace 'aserverlessdemo':

          4.128s Ready to serve.

          Service 'helloworld-go' updated to latest revision 'helloworld-go-gngxv-5' is available at URL:
          http://helloworld-go-aserverlessdemo.apps.cluster-e9f2.e9f2.example.opentlc.com

2. Then let's describe the serverless service itself using `kn` command. Notice an additional revision highlighted below.

           $ kn service describe helloworld-go
             Name:       helloworld-go
             Namespace:  aserverlessdemo
             Age:        9h
             URL:        http://helloworld-go-aserverlessdemo.apps.cluster-e9f2.e9f2.example.opentlc.com

             Revisions:  
                 *+  helloworld-go-gngxv-5 (current @latest) [3] (5m)*
                   Image:  gcr.io/knative-samples/helloworld-go (at 5ea96b)
                 100%  helloworld-go-vgcgh-2 #rev1 [2] (9h)
                   Image:  gcr.io/knative-samples/helloworld-go (at 5ea96b)

             Conditions:  
             OK TYPE                   AGE REASON
             ++ Ready                   5m 
   
3. Since we already have updated the application and a new revision was created *+  helloworld-go-gngxv-5 (current @latest) [3] (5m)* we are now ready to split the traffic. We can do this using commandline or developer perspective UI. 

*Using Developers Perspective UI:* 

![image](https://user-images.githubusercontent.com/17167732/94526843-9e584d80-0292-11eb-9242-5a3c3fbfe544.png)

![image](https://user-images.githubusercontent.com/17167732/94527056-ee371480-0292-11eb-9759-5216448e418e.png)

*Using Command Line

            kn service update helloworld-go --traffic @latest=10 --traffic helloworld-go-gngxv-5=50

# Serverless Eventing

When we talk about Serverless Eventing, we need to understand what is Knative Eventing. Knative Eventing on OpenShift Container Platform enables developers to use an event-driven architecture with serverless applications. An event-driven architecture is based on the concept of decoupled relationships between event producers that create events, and event sinks, or consumers, that receive them
In OpenShift, Knative Eventing uses standard HTTP POST requests to send and receive events between event producers and consumers.

The figure below, show the event-driven application architecture for direct connection of event source to sink bin (knative service/application).

![image](https://user-images.githubusercontent.com/17167732/94597465-17cf5a80-02ea-11eb-87f9-15683bde7a18.png)

This architecture works well, but what if you have multiple sources of event that you need to filter? There is another way of doing this and this is either via `channel & subscription` or `brokers and triggers`.

In this demo, we will just use the direct connection of event-source to sink bin (knative service/application). We will do this both via command line and use OpenShift Developers UI as well.

Prereqs:
- You have Knative Serving and Eventing installed.
- You have an installed `kn` command line tool.

Let's start!

1. Create a new project for eventing demo.
    
         $oc new-project aserverlessdemo-eventing

2. List all available sources using `kn`. You can see in the list below the different source type. In this demo we will use *PingSource*.
        
        kn source list-types
        TYPE              NAME                                            DESCRIPTION
        ApiServerSource   apiserversources.sources.eventing.knative.dev   Watch and send Kubernetes API events to a sink
        ApiServerSource   apiserversources.sources.knative.dev            Watch and send Kubernetes API events to a sink
        ContainerSource   containersources.sources.eventing.knative.dev   
        CronJobSource     cronjobsources.sources.eventing.knative.dev     
        *PingSource        pingsources.sources.knative.dev                 Send periodically ping events to a sink*
        SinkBinding       sinkbindings.sources.eventing.knative.dev       Binding for connecting a PodSpecable to a sink
        SinkBinding       sinkbindings.sources.knative.dev                Binding for connecting a PodSpecable to a sink

3. To verify that the `PingSource` is working, create a simple Knative service that dumps incoming messages to the service’s logs. For the sake of this demo, I will be deploying a simple  application.

       $ kn service create event-display --image quay.io/openshift-knative/knative-eventing-sources-event-display:latest
  
  You should see an output similar to below:
  
         Creating service 'event-display' in namespace 'aserverlessdemo':

         0.195s The Route is still working to reflect the latest desired specification.
         0.292s Configuration "event-display" is waiting for a Revision to become ready.
         7.264s ...
         7.393s Ingress has not yet been reconciled.
         8.112s Ready to serve.

        Service 'event-display' created to latest revision 'event-display-jcppt-1' is available at URL: http://event-display-aserverlessdemo.apps.cluster-e9f2.e9f2.example.opentlc.com

    
 4.  We then need to create a `Sink Binding` in the same namespace as the event consumer `event-display`. The command below will instantiate a sink binding called bind-hearbeat

         $ kn source binding create bind-heartbeat --subject Job:batch/v1:app=heartbeat-cron --sink svc:event-display
           Sink binding 'bind-heartbeat' created in namespace 'aserverlessdemo'
  
  5. We then need to create a cronjob. This will trigger an event based on the scheduled cronjob. Copy below to `cron.yaml` file
  
          apiVersion: batch/v1beta1
          kind: CronJob
          metadata:
             name: heartbeat-cron
          spec:
          spec:
          # Run every minute
          schedule: "* * * * *"
          jobTemplate:
            metadata:
               labels:
                 app: heartbeat-cron
            spec:
              template:
                 spec:
                   restartPolicy: Never
                   containers:
                     - name: single-heartbeat
                       image: quay.io/openshift-knative/knative-eventing-sources-heartbeats:latest
                       args:
                          - --period=1
                       env:
                         - name: ONE_SHOT
                           value: "true"
                         - name: POD_NAME
                           valueFrom:
                              fieldRef:
                                fieldPath: metadata.name
                         - name: POD_NAMESPACE
                           valueFrom:
                              fieldRef:
                                fieldPath: metadata.namespace
 
        Then apply this cronjob to instantiate the cronjob.

              $ oc apply -f cron.yaml 
                cronjob.batch/heartbeat-cron created
 
 6. Verify the sourcebinding.
 
             $ kn source binding describe bind-heartbeat
             
    You should expect similar output as below.
    
             Name:         bind-heartbeat
             Namespace:    aserverlessdemo
             Annotations:  sources.knative.dev/creator=user1, sources.knative.dev/lastModifier=user1
             Age:          1m
             Subject:      
                Resource:   Job (batch/v1)
                Selector:   
                app:      heartbeat-cron
             Sink:         
                Name:       event-display
                Resource:   Service (serving.knative.dev/v1)
             Conditions:  
               OK TYPE     AGE REASON
               ++ Ready     1m 
 
 
7. Wait and verify until SinkBinding (SBS) pods is created.

         $ oc get pods
         
   You should expect similar output below. 
        
         NAME                                                READY   STATUS      RESTARTS   AGE
         event-display-jcppt-1-deployment-7494c6cfbb-fwv4z   2/2     Running     0          6s
         heartbeat-cron-1601453760-4tg5d                     0/1     Completed   0          26s

8. You can verify that the Kubernetes events were sent to the Knative event sink by looking at the message dumper function logs.

        $ oc logs $(oc get pod -o name | grep event-display) -c user-container
        
   You should expect similar output below.     

        ☁️  cloudevents.Event
        Validation: valid
        Context Attributes,
          specversion: 1.0
          type: dev.knative.eventing.samples.heartbeat
          source: https://knative.dev/eventing-contrib/cmd/heartbeats/#aserverlessdemo/heartbeat-cron-1601453760-4tg5d
          id: 8bd109e1-0277-4cef-bc07-9033a3f7207f
          time: 2020-09-30T08:16:24.684270341Z
          datacontenttype: application/json 
        Extensions,
          beats: true
          heart: yes
          the: 42
        Data,
          {
            "id": 1,
            "label": ""
          }
 
 9. Or you can verify via OpenShift Developers perspective UI.

![image](https://user-images.githubusercontent.com/17167732/94660704-85b76880-0362-11eb-800d-446c06705803.png)

 
# Conclusion:

With OpenShift Serverless capabilities that includes Serving and Eventing as shown in this demo. This means that you can deploy any programming language of your choice and enable auto-scaling behavior, scaling up to meet demand and down even to zero. This is possible because of OpenShift Serverless Serving feature. Serving enables the deployment of applications and functions as serverless containers.

With Eventing and beyond the auto-scaling behavior (scale down to zero), Serverless eventing can use triggers from variety of sources and recieved events from Cronjob schedule as shown in this demo and other possible event sources such us Kafka messages, file upload to Storage as well as third party event sources like Salesforce, Service Now, E-mail, etc… powered by Camel-K 

 


# References: 
   Official Blog: <https://blog.openshift.com/knative-serving-your-serverless-services>  
   Knative Project: <https://knative.dev/docs/eventing/sources/>   



