# Architecting with Google Kubernetes Engine - Workloads: AK8S-12 Configuring Persistent Storage for Kubernetes Engine

## Objectives: In this lab, you learn how to perform the following tasks:

    - Create manifests for PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) for Google Cloud Platform (GCP) persistent disks (dynamically created or existing)

    - Mount GCP persistent disk PVCs as volumes in Pods

    - Use manifests to create StatefulSets

    - Mount GCP persistent disk PVCs as volumes in StatefulSets

    - Verify the connection of Pods in StatefulSets to particular PVs as the Pods are stopped and restarted

## Steps:

1.  Create PVs and PVCs

    1. create environment variables for the GCP zone and cluster name

       export my_zone=us-central1-a
       export my_cluster=standard-cluster-1

    2. Configure tab completion for the kubectl command-line tool.

       source <(kubectl completion bash)

    3. Create a VPC-native Kubernetes cluster.

       gcloud container clusters create $my_cluster \
   		--num-nodes 3 --enable-ip-alias --zone $my_zone

    4.Configure access to your cluster for kubectl:

        gcloud container clusters get-credentials $my_cluster --zone $my_zone

    # Create and apply a manifest with a PVC

    1. Enter the following command to clone the repository to the lab Cloud Shell.

       git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

    2. Change to the directory that contains the sample files for this lab.

       cd ~/training-data-analyst/courses/ak8s/12_Storage/

    3. To show that you currently have no PVCs, execute the following command:

       kubectl get persistentvolumeclaim

    4. To create the PVC, execute the following command:

       kubectl apply -f pvc-demo.yaml

    5. To show your newly created PVC, execute the following command:

       kubectl get persistentvolumeclaim

2.  Mount and verify GCP persistent disk PVCs in Pods

    1. To create the Pod with the volume, execute the following command:

       kubectl apply -f pod-volume-demo.yaml

    2. List the Pods in the cluster.

       kubectl get pods

    3. To verify the PVC is accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

       kubectl exec -it pvc-demo-pod -- sh

    4. To create a simple text message as a web page in the Pod enter the following commands:

       echo Test webpage in a persistent volume!>/var/www/html/index.html
       chmod +x /var/www/html/index.html

    5. Verify the text file contains your message.

       cat /var/www/html/index.html

    6. Enter the following command to leave the interactive shell on the nginx container.

       exit

    # Test the persistence of the PV

    1. Delete the pvc-demo-pod.

       kubectl delete pod pvc-demo-pod

    2. List the Pods in the cluster.

       kubectl get pods

       There should be no Pods on the cluster.

    3. To show your PVC, execute the following command:

       kubectl get persistentvolumeclaim

       Your PVC still exists, and was not deleted when the Pod was deleted.

    4. Redeploy the pvc-demo-pod.

       kubectl apply -f pod-volume-demo.yaml

    5. List the Pods in the cluster.

       kubectl get pods

    6. To verify the PVC is is still accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

       kubectl exec -it pvc-demo-pod -- sh

    7. To verify that the text file still contains your message execute the following command:

       cat /var/www/html/index.html

    8. Enter the following command to leave the interactive shell on the nginx container.

       exit

3.  Create StatefulSets with PVCs

    # Release the PVC

    1. Before you can use the PVC with the statefulset, you must delete the Pod that is currently using it. Execute the following command to delete the Pod:

       kubectl delete pod pvc-demo-pod

    2. Confirm the Pod has been removed.

       kubectl get pods

    3. To create the StatefulSet with the volume, execute the following command:

       kubectl apply -f statefulset-demo.yaml

    Verify the connection of Pods in StatefulSets

    1. Use "kubectl describe" to view the details of the StatefulSet:

       kubectl describe statefulset statefulset-demo

    2. List the Pods in the cluster.

       kubectl get pods

    3. To list the PVCs, execute the following command:

       kubectl get pvc

    4. Use "kubectl describe" to view the details of the first PVC in the StatefulSet:

       kubectl describe pvc hello-web-disk-statefulset-demo-0

4.  Verify the persistence of Persistent Volume connections to Pods managed by StatefulSets

    1. To verify that the PVC is accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

       kubectl exec -it statefulset-demo-0 -- sh

    2. Verify that there is no index.html text file in the /var/www/html directory.

       cat /var/www/html/index.html

    3. To create a simple text message as a web page in the Pod enter the following commands:

       echo Test webpage in a persistent volume!>/var/www/html/index.html
       chmod +x /var/www/html/index.html

    4. Verify the text file contains your message.

       cat /var/www/html/index.html

    5. Enter the following command to leave the interactive shell on the nginx container.

       exit

    6. Delete the Pod where you updated the file on the PVC.

       kubectl delete pod statefulset-demo-0

    7. List the Pods in the cluster.

       kubectl get pods

    8. Connect to the shell on the new statefulset-demo-0 Pod.

       kubectl exec -it statefulset-demo-0 -- sh

    9. Verify that the text file still contains your message.

       cat /var/www/html/index.html

    10. Enter the following command to leave the interactive shell on the nginx container.

        exit
