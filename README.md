
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
        
![image](https://user-images.githubusercontent.com/17167732/94499539-700a4c00-0259-11eb-89a9-b2a86617445a.png)


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


# Using Developer Console.
This is to show that you can create a serverless application using Developer Console.

1. Select container image. 
![knative6](https://user-images.githubusercontent.com/17167732/75420401-da45d000-599c-11ea-91a6-96aa43ddb461.png)

2. Make sure you verify the image by clicking the button.
![knative7](https://user-images.githubusercontent.com/17167732/75420412-ddd95700-599c-11ea-9ab7-affc667d6184.png)

3. Select Knative Service as your resource type.
![knative8](https://user-images.githubusercontent.com/17167732/75420569-25f87980-599d-11ea-80de-e727857eb854.png)


4. You can choose the number of pods as you Max number of pods when scaling up as part of you serverless option and click create.
![knative9](https://user-images.githubusercontent.com/17167732/75420580-2a249700-599d-11ea-98d7-82ef3c8bd930.png)

5. You will automatically be directed to Topology view. You can then monitor the status of your deployment.
![kNATIVE10](https://user-images.githubusercontent.com/17167732/75420586-2e50b480-599d-11ea-8022-b25028fea5df.png)

6. Once the container is running, you can test the application by selecting the routes and see if you can access exposed routes. If you are succesful, you can wait for a few minutes and see if the pods will scale to zero. Once the pods has scaled to zero, hit the routes again and monitor if the pods are re-created by the knative serving. Once you see it re-created, your demo is a success!! 


# References: 
    https://blog.openshift.com/knative-serving-your-serverless-services
    https://github.com/tnscorcoran/openshift-servicemesh



