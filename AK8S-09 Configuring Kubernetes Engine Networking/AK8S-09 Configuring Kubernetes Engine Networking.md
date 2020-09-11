## Architectimg with Google Kubernetes Engine - Workloads: AK8S-06 Creating Kubernetes Engine Deployments

## Objectives:

In this lab, you learn how to perform the following tasks:

    - Create deployment manifests, deploy to cluster, and verify Pod rescheduling as nodes are disabled

    - Trigger manual scaling up and down of Pods in deployments

    - Trigger deployment rollout (rolling update to new version) and rollbacks

    - Perform a Canary deployment

## Steps:

1. Create deployment manifests and deploy to the cluster

   # Create a GKE cluster

   1. In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

      export my_zone=us-central1-a
      export my_cluster=standard-cluster-1

   2. Configure kubectl tab completion in Cloud Shell.

      source <(kubectl completion bash)

   3. In Cloud Shell, type the following command to create a Kubernetes cluster.

      gcloud container clusters create $my_cluster --num-nodes 3  --enable-ip-alias --zone $my_zone

   4. In Cloud Shell, configure access to your cluster for the kubectl command-line tool, using the following command:

      gcloud container clusters get-credentials $my_cluster --zone $my_zone

   5. In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.

      git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

   6. Change to the directory that contains the sample files for this lab.

      cd ~/training-data-analyst/courses/ak8s/06_Deployments/

   # Create a deployment manifest

   1. To deploy your manifest, execute the following command:

      kubectl apply -f ./nginx-deployment.yaml

   2. To view a list of deployments, execute the following command:

      kubectl get deployments

   3. Wait a few seconds, and repeat the command until the number listed for CURRENT deployments reported by the command matches the number of DESIRED deployments.

2. Manually scale up and down the number of Pods in deployments

   # Scale Pods up and down in the console

   # Scale Pods up and down in the shell

   1. In the Cloud Shell, to view a list of Pods in the deployments, execute the following command:

      kubectl get deployments

   2. To scale the Pod back up to three replicas, execute the following command:

      kubectl scale --replicas=3 deployment nginx-deployment

   3. To view a list of Pods in the deployments, execute the following command:

      kubectl get deployments

3. Trigger a deployment rollout and a deployment rollback

   # Trigger a deployment rollout

   1. To update the version of nginx in the deployment, execute the following command:

      kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record

   2. To view the rollout status, execute the following command:

      kubectl rollout status deployment.v1.apps/nginx-deployment

   3. To verify the change, get the list of deployments.

      kubectl get deployments

   4. View the rollout history of the deployment.

      kubectl rollout history deployment nginx-deployment

   # Trigger a deployment rollback

   1. To roll back to the previous version of the nginx deployment, execute the following command:

      kubectl rollout undo deployments nginx-deployment

   2. View the updated rollout history of the deployment.

      kubectl rollout history deployment nginx-deployment

   3. View the details of the latest deployment revision

      kubectl rollout history deployment/nginx-deployment --revision=3

4. Define the service type in the manifest

   # Define service types in the manifest

   1. In the Cloud Shell, to deploy your manifest, execute the following command:

      kubectl apply -f ./service-nginx.yaml

   # Verify the LoadBalancer creation

   1. To view the details of the nginx service, execute the following command:

      kubectl get service nginx

   2. When the external IP appears, open http://107.52.125.44:60000/ in a new browser tab to see the server being served through network load balancing.

5. Perform a canary deployment

   1. Create the canary deployment based on the configuration file.

      kubectl apply -f nginx-canary.yaml

   2. When the deployment is complete, verify that both the nginx and the nginx-canary deployments are present.

      kubectl get deployments

   3. Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You should continue to see the standard "Welcome to nginx" page.

   4. Switch back to the Cloud Shell and scale down the primary deployment to 0 replicas.

      kubectl scale --replicas=0 deployment nginx-deployment

   5. Verify that the only running replica is now the Canary deployment:

      kubectl get deployments
