# Architecting with Google Kubernetes Engine - Workloads: AK8S-10 Creating Services and Ingress Resources

## Objectives:

In this lab, you learn how to perform the following tasks:

    - Observe Kubernetes DNS in action.

    - Define various service types (ClusterIP, NodePort, LoadBalancer) in manifests along with label selectors to connect to existing labeled Pods and deployments, deploy those to a cluster, and test connectivity.

    - Deploy an Ingress resource that connects clients to two different services based on the URL path entered.

    - Verify Google Cloud Platform network load balancer creation for type=LoadBalancer services.

## Steps:

1.  Deploy a Google Kubernetes Engine (GKE) cluster and test DNS

    # Create a GKE cluster

    1. In Cloud Shell, type the following command to create environment variables for the GCP zone and cluster name that will be used to create the cluster for this lab.

       export my_zone=us-central1-a
       export my_cluster=standard-cluster-1

    2. Configure tab completion for the kubectl command-line tool.

       source <(kubectl completion bash)

    3. Create a VPC-native Kubernetes cluster.

       gcloud container clusters create $my_cluster \
   		--num-nodes 3 --enable-ip-alias --zone $my_zone

    4. Configure access to your cluster for kubectl:

       gcloud container clusters get-credentials $my_cluster --zone $my_zone

2.  Create Pods and services to test DNS resolution

    # Clone the source file repository and deploy the sample application pods

    1. In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.

       git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

    2. Change to the directory that contains the sample files for this lab.

       cd ~/training-data-analyst/courses/ak8s/10_Services/

    3. Create the service and Pods.

       kubectl apply -f dns-demo.yaml

    4. Verify that your Pods are running.

       kubectl get pods

    # Access Pods and services by FQDN

    1.  You can find the IP address for dns-demo-2 by displaying the details of the Pod.

        kubectl describe pods dns-demo-2

            # You will see the ip-address in the first section of the below the status, before the details of the individual containers.

    2.  Ping dns-demo-2.

        ping dns-demo-2.dns-demo.default.svc.cluster.local

            # The ping fails because you are only in the Cloud Shell VM, not inside the cluster itself.

    3.  To get inside the cluster, open an interactive session to bash running from dns-demo-1.

            kubectl exec -it dns-demo-1 /bin/bash

    4.  Update apt-get and install a ping tool.

        apt-get update
        apt-get install -y iputils-ping

    5.  Ping dns-demo-2:

        ping dns-demo-2.dns-demo.default.svc.cluster.local

    6.  Press Ctrl+C to abort the ping command.

    7.  Ping the dns-demo service's FQDN, instead of a specific Pod inside the service:

        ping dns-demo.default.svc.cluster.local

    8.  Press Ctrl+C to abort the ping command.

    9.  Leave the interactive shell on dns-demo-1 running.

3.  Deploy a sample workload and a ClusterIP service

    1. In the Cloud Shell menu bar, click the plus sign (+) icon to start a new Cloud Shell session.

    2. In the second Cloud Shell tab, change to the directory that contains the sample files for this lab.

       cd ~/training-data-analyst/courses/ak8s/10_Services/

    3. To create a deployment from the hello-v1.yaml file, execute the following command:

       kubectl create -f hello-v1.yaml

    4. To view a list of deployments, execute the following command:

       kubectl get deployments

    # Define service types in the manifest

    1. To deploy the ClusterIP service, execute the following command:

       kubectl apply -f ./hello-svc.yaml

    2. Verify that the service was created and that a Cluster-IP was allocated:

       kubectl get service hello-svc

    # Test your application

    1. In the Cloud Shell, attempt to open an HTTP session to the new service using the following command:

       curl hello-svc.default.svc.cluster.local

    2. Return to your first Cloud Shell window, which is currently redirecting the stdin and stdout of the dns-demo-1 Pod.

    3. Install curl so you can make calls to web services from the command line.

       apt-get install -y curl

    4. Use the following command to test the HTTP connection between the Pods.

       curl hello-svc.default.svc.cluster.local

4.  Convert the service to use NodePort

    1. Return to your second Cloud Shell window. This is the window NOT connected to the stdin and stdout of the dns-test Pod.

    2. To deploy the manifest that changes the service type for the hello-svc to NodePort , execute the following command:

       kubectl apply -f ./hello-nodeport-svc.yaml

    3. Enter the following command to verify that the service type has changed to NodePort:

       kubectl get service hello-svc

    # Test your application

    1. In the Cloud Shell, attempt to open an HTTP session to the new service using the following command:

       curl hello-svc.default.svc.cluster.local

    2. Return to your first Cloud Shell window, which is currently redirecting the stdin and stdout of the dns-test Pod.

    3. Use the following command to test the HTTP connection between the Pods.

       curl hello-svc.default.svc.cluster.local

5.  Deploy a new set of Pods and a LoadBalancer service

    1. Return to your second Cloud Shell window. This is the window NOT connected to the stdin and stdout of the dns-test Pod.

    2. To deploy the manifest that creates the hello-v2 deployment execute the following command:

       kubectl create -f hello-v2.yaml

    3. To view a list of deployments, execute the following command:

       kubectl get deployments

    # Define service types in the manifest

    1. Make sure you are still in the second Cloud Shell window. This is the window NOT connected to the stdin and stdout of the dns-test Pod.

    2. To deploy your LoadBalancer service manifest, execute the following command:

       kubectl apply -f ./hello-lb-svc.yaml

    3. Verify that the service was created and that a node port was allocated:

       kubectl get services

    # Test your application

    1. In the Cloud Shell, attempt to open an HTTP session to the new service using the following command:

       curl hello-lb-svc.default.svc.cluster.local

    2. Try the connection again using the External IP address associated with the service. Type the following command and substitute [external_IP] with your service's external IP address.

    3. Return to your first Cloud Shell window, which is currently redirecting the stdin and stdout of the dns-demo-1 Pod.

    4. Use the following command to test the HTTP connection between the Pods.

       curl hello-lb-svc.default.svc.cluster.local

    5. Try the connection again within the Pod using the External IP address associated with the service. Type the following command and substitute [external_IP] with your service's external IP address.

       curl [external_IP]

    6. Exit the console redirection session to the Pod by typing exit.

       exit

6.  Deploy an Ingress resource

    # Create an ingress resource

    1. To deploy this ingress resource, execute the following command:

       kubectl apply -f hello-ingress.yaml

    # Test your application

    1. To get the external IP address of the load balancer serving your application, execute the following command:

       kubectl describe ingress hello-ingress

    2. Use the External IP address associated with the Ingress resource, and type the following command, substituting 106.65.103.187 with the Ingress resource's external IP address. Be sure to include the /v1 in the URL path.

       curl 106.65.103.187/v1

    3. Now test the v2 URL path from Cloud Shell. Use the External IP address associated with the Ingress resource, and type the following command, substituting 106.65.103.187 with the Ingress resource's external IP address. Be sure to include the /v2 in the URL path.

       curl 106.65.103.187/v2
