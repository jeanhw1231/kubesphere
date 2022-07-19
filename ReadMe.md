# Monitoring container resources with KubeSphere

## 1. Launch Huawei's Cloud Container Engine (CCE) Kubernetes Cluster

Click [here](https://support.huaweicloud.com/intl/en-us/qs-cce/cce_qs_0001.html) for the instructions to create CCE kubernetes cluster on Huawei Cloud.

To use KubeSphere, the following requirements need to be met:
-   To install KubeSphere 3.3.0 on Kubernetes, your Kubernetes version must be v1.19.x, v1.20.x, v1.21.x, v1.22.x, and v1.23.x (experimental support).
-   Ensure the cloud computing network for your Kubernetes cluster works, or use an elastic IP when you use  **Auto Create**  or  **Select Existing**. You can also configure the network after the cluster is created. Refer to  [NAT Gateway](https://support.huaweicloud.com/en-us/productdesc-natgateway/en-us_topic_0086739762.html).
-   Select  `s3.xlarge.2` `4-core｜8GB`  for nodes and add more if necessary (3 and more nodes are required for a production environment).

Refer basic cluster requirements from [KubeSphere website](https://kubesphere.io/docs/v3.3/installing-on-kubernetes/hosted-kubernetes/install-kubesphere-on-huaweicloud-cce/)

Alternatively, you may use the ***terraform script*** here to launch the CCE cluster on Huawei Cloud.

Once the CCE cluster has been launched, you will see the default csi storage being created. 
![cce storage](https://github.com/jeanhw1231/kubesphere/blob/main/pics/1_cce_storage.png)

> Huawei CCE built-in Everest CSI provides StorageClass `csi-disk` which uses SATA (normal I/O) by default, but the actual disk that is used for Kubernetes clusters is either SAS (high I/O) or SSD (extremely high I/O). Therefore, it is suggested that you create an extra StorageClass and set it as **default**. Refer to the official document - [Use kubectl to create a cloud storage](https://support.huaweicloud.com/en-us/usermanual-cce/cce_01_0044.html).

Please make sure that you run this--> [storageclass yaml file](https://github.com/jeanhw1231/kubesphere/blob/main/kubectl-kubesphere/csi-disk-sas.yaml), else the workload will fail. For more info on the ks-installer, refer to - https://github.com/kubesphere/ks-installer
![csi-disk-sas yaml](https://github.com/jeanhw1231/kubesphere/blob/main/pics/2a_cce_kubesphere_pods_csi-disk-sas.png)
If you don't run the storage class configuration, you will see the log message indicating "Default storage class was not found":
![ks-installer-failed](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3c_ks-installer.png)


## 2. Install KubeSphere
Once your CCE cluster is "available", run the following kubectl commands:

    kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/kubesphere-installer.yaml

	kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/cluster-configuration.yaml

After the command has been executed, the KubeSphere pods and deployment can be seen in the console:
![Kubesphere pods](https://github.com/jeanhw1231/kubesphere/blob/main/pics/2_cce_kubesphere_pods.png)
![deployment](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3e_deployed_kubesphere1.png)

You can also verify whetherr KubeSphere was installed successfully, by running the following command to inspect the log:

    kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-installer -o jsonpath='{.items[0].metadata.name}') -f

KubeSphere comprises of ks-controller-manager, ks-console, ks-apiserver services:
![KubeSphere Services](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3f_deployed_kubesphere3_network.png)

## 3. Configure ELB service for KubeSphere

Configure ELB service for KubeSphere
![Configure ELB service for KubeSphere](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3g_cce_create_service_elb.png)
Set the ELB access port and container port as follow:
![Configure ELB service for KubeSphere](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3f_cce_create_service_elb_port.png)
Sometimes it takes a while for the EIP to show up for the ELB service. You can check it by running kubectl:
![Kubectl - External IP not showing](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3f_kubectl_service_kubesphere.png)Or, you may also check out the EIP for KubeSphere via the console:
![EIP for KubeSphere ELB](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3h_cce_create_service_list_console.png)
With the newly obtained EIP, access KubeSphere via the browser:
![KubeSphere console - EIP:portnum](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3i_access_kubesphere_browser.png)Get the default admin logon via the terminal (black screenshot on the left). Once you logon, you will be prompted to change to a more complex password


## 4. Explore KubeSphere

### I) View overall resource usage on the main console:
![default_Kubesphere console](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3j_access_kubesphere_console_default.png)
### II) View Cluster Nodes:
![cluster nodes](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3k_access_kubesphere_console_clusterNodes.png)
### III) View the projects created on the cluster:
![project view](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3k_access_kubesphere_console_projectview.png)
![project listing](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3m_access_kubesphere_console_projlisting.png)
- you can also create new project within the KubeSphere console as well:
![create project](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3l_access_kubesphere_console_createnewproj.png)
Perform filtering based on application namespace:
![filter app namespace](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3n_access_kubesphere_console_filterservice.png)
View deployed routes of Nginx Ingress Controller:

If you follow the demo for previous ***NGINX ingress controller deployment***, you would be able to see that the routes have been configured for hotel.jeanuinespace.com and also cafe.jeanuinespace.com.

- nginx-ingress route details for hotel.jeanuinespace.com
![nginx-ingress controller](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3o_access_kubesphere_console_nginxRoutes.png)
![nginx-ingress route details](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3p_access_kubesphere_console_nginxRoutes_details.png)
![nginx ingress annotations](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3q_access_kubesphere_console_nginxRoutes_annotations.png)
- nginx-ingress route details for cafe.jeanuinespace.com
![cafe route](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3r_access_kubesphere_console_cafe_nginxRoutes.png)
![cafe route details](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3s_access_kubesphere_console_cafe_nginxRoutes_details.png)
![cafe route annotations](https://github.com/jeanhw1231/kubesphere/blob/main/pics/3t_access_kubesphere_console_cafe_nginxRoutes_annotations.png)
### IV) View CCE Cluster Status via KubeSphere:
![Cluster Status view1](https://github.com/jeanhw1231/kubesphere/blob/main/pics/4_Cluster_Status.png)
![Cluster status view2](https://github.com/jeanhw1231/kubesphere/blob/main/pics/4a_Cluster_Status_Request_Monitoring.png)
