# ASMonAzure
![](images/title.png)  
Update: April 30, 2020

## Introduction

This cookbook will walk you through the process of installing **Anypoint Service Mesh** on **Microsoft Azure Cloud Platform**. You will deploy a demo application and secure using Anypoint Service Mesh.

***To log issues***, click here to go to the [github](https://github.com/mulesoft-consulting/ASMonAzure/issues) repository issue submission form.

## Objectives

- [Create Kubernetes Cluster on Azure Kubernetes Services (AKS)](#installaks)
- [Install Istio](#installistio)
- [Deploy Demo Application](#deploydemo)
- [Install Anypoint Service Mesh](#installasm)
- [Apply API Management Policies](#applypolicy)

## Required Artifacts

- The following lab requires an Enterprise Azure account.
- Enable Anypoint Service Mesh in your STGX Organization (this is required before GA)

<a id="installaks"></a>
## Create Azure Kubernetes Services Cluster

### **STEP 1**: Create Azure Virtual Network

- From any browser, go to the URL to access Azure Portal:

   <https://portal.azure.com/>

- Under Azure services, select **Virtual networks**.

    ![](images/image1.png)

- Click on **+ Add** at the left top corner or **Create virtual network** if there isn't any already.

	![](images/image2.png)

- Select **Resource Group** Enter **Name** for instance and select **Region** for Virtual Network creation.

    ![](images/image3.png)

- Select and remove the **default** subnet. Add a new subnet with more available IP addresses.

	![](images/image4.png)


- The lower the subnet number is like /16, the more IP addresses are available.

	![](images/image5.png)

- Click **Next** to **Review + create**. Ensure the validation passed & click on **Create**.

    ![](images/image6.png)
    
- Wait till the deployment is complete.

	![](images/image7.png)

### **STEP 2**: Create Kubernetes Cluster

- Back to the Azure Portal home page. Select **Kubernetes services**

    ![](images/image8.png)

- Click on **+ Add** at the left top corner of **Kubernetes services**, or **Create Kubernetes service** if there isn't any already. Select **Resource group**, enter a unique **Kubernetes cluster name**, select Region, and click on Change size for **Node size**.
	
    ![](images/image9.png)

- Select **B4ms**, per [documentation](https://beta.docs.stgx.mulesoft.com/beta-service-mesh/service-mesh/1.0/prepare-to-install-service-mesh#hardware-requirements)

    ![](images/image10.png)

- Keep everything default till the **Networking** step. Switch **Network configuration** from **Basic** to **Advanced**. Select the **Virtual network** created in **STEP 1**. Enter **Kubernetes service address range** and **Kubernetes DNS service IP address** allowed in the selected **Virtual network**.

    ![](images/image11.png)
    
- At the **Tags** step at some tags that help identify the resources you're creating, like **owner** -> **Email**. 

	![](images/image12.png)
	
- Move on to the last step, **Review + create**. Ensure the validation passed & click on **Create**.

	![](images/image13.png)	
	
- Wait till the deployment is complete.

	![](images/image14.png)

### **STEP 3**: Verify Cluster and Connect

- From the Kubernetes Services page, launch the **az** command line from the newly created AKS clluster using either *Bash* or *Powershell*. Make sure your account's initialized successfully from the Cloud Shell. 

	![](images/image15.png)	

- Open Terminal window. If you don't already have the **Azure CLI** installed following the [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) to first install Azure CLI.

- Next running the following command to verify that you cluster is running.

```bash
kubectl get namespaces
```

![](images/image16.png)

<a id="installistio"></a>
## Install Istio

### **STEP 4**: Download and Install Istio CLI

- To install **Istio** we will be using the **Istio CLI**. For completed instructions [Istio Docs](https://istio.io/docs/setup/install/istioctl/)

- Use the following command to download **Istio CLI** into your directory of choice and supported by ASM (1.4.6 at this time).

```bash
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=<x.x.x> sh -
```

![](images/image17.png)

- Change into newly downloaded directory (the Istio version downloaded and to be installed)

```bash
cd istio-<x.x.x>/
```

- Add current directory directly to path

```bash
export PATH=$PWD/bin:$PATH
```

![](images/image18.png)

### **STEP 5**: Install Istio using CLI
- To install **Istio** we will be using the **Istio CLI**. From the **istio** directory run the following command

```bash
istioctl manifest apply --set profile=demo --set values.global.disablePolicyChecks=false
```

![](images/image19.png)

- Verify that **Istio** has been installed. You should now see the **istio-system** namespace

```bash
kubectl get namespaces
```

![](images/image20.png)

<a id="deploydemo"></a>
## Deploy Demo Application

### **STEP 6**: Clone Demo Application

- For our demo application will will be using **Northern Trail Outfitters** shopping cart application. This web based UI will call several services to complete the order.

- Clone the demo application git repository onto your local machine.

```bash
git clone https://github.com/mulesoft-consulting/ServiceMeshDemo
```

- Change to the **ServiceMeshDemo** directory and list out the contents to verify that the repository has been created correctly

```bash
cd ServiceMeshDemo/
ls
```

![](images/image21.png)

### **STEP 7**: Deploy Demo Application

- We will now deploy the demo application to your kubernetes cluster. The deployment script takes the namespace as a parameter. We will be using **nto-payment** for namespace

```bash
./deployAll.sh nto-payment
```

![](images/image22.png)

- You can monitory the deployment with the following commands

```bash
kubectl get pods -n nto-payment
kubectl get services -n nto-payment
```

![](images/image23.png)

- Once all services are running you can test out the application. To access the application open you browser and go to the following URL

```bash
http://<EXTERNAL-IP>:3000
```

![](images/image24.png)

![](images/image25.png)

- To test out the application follow these steps:

    - Select Item to purchase
    - Click **Add to Cart**
    - Click **Checkout**
    - Leave default email and click **CONTINUE**
    - Click **AUTHORIZE PAYMENT**
    - Last click **PLACE ORDER**

![](images/image26.png)

<a id="installasm"></a>
## Install Anypoint Service Mesh

### **STEP 8**: Install Anypoint Service Mesh

For complete instructions and documentation please visit [MuleSoft Docs](https://beta.docs.stgx.mulesoft.com/beta-service-mesh/service-mesh/1.0/service-mesh-overview-and-landing-page)

- First lets enable API Analytics by setting the **disableMixerHttpReports** flag to false:

```bash
kubectl -n istio-system get cm istio -o yaml | sed -e 's/disableMixerHttpReports: true/disableMixerHttpReports: false/g' | kubectl replace -f -
```

![](images/image27.png)

- Download the latest Anypoint Service Mesh CLI and make it executable

Download from Staging environment (STGX) for Pre-GA:

```bash
mkdir -p $HOME/.asm && curl -Ls https://stgx.anypoint.mulesoft.com/servicemesh/xapi/v1/install > $HOME/.asm/asmctl && chmod +x $HOME/.asm/asmctl && export PATH=$PATH:$HOME/.asm
```

Download from GA:

```bash
mkdir -p $HOME/.asm && curl -Ls http://anypoint.mulesoft.com/servicemesh/xapi/v1/install > $HOME/.asm/asmctl && chmod +x $HOME/.asm/asmctl && export PATH=$PATH:$HOME/.asm
```

- Since ASM is Pre-GA we will be using a staging environment. To have the **asmctl** connect to the correct environment set the variable **SERVICEMESH_PLATFORM_URI**

```bash
export SERVICEMESH_PLATFORM_URI=https://stgx.anypoint.mulesoft.com
```

- Now we are ready to install Anypoint Service Mesh. To do this we will call **asmctl install**. This command requires 3 parameters
    - Client Id
    - Client Secret
    - Service Mesh license

- If you are not familiar with how to get environment Client Id and Secret please visit [MuleSoft Docs](https://docs.mulesoft.com/access-management/environments)

```bash
./asmctl install
```

![](images/image28.png)

- Verify that Anypoint Service Mesh has been installed correctly with the following command

```bash
kubectl get pods -n service-mesh
```

![](images/image29.png)

### **STEP 9**: Install Anypoint Service Mesh Adapter

- Next we want to deploy the Anypoint Service Mesh adapter in each namespace that we want to monitor APIs. For this example we will just be doing the **nto-payment** namespace that contains the demo application.

- To deploy the ASM Adapter we will be using a Kubernetes custom resource definition (CRD). In the **ServiceMeshDemo** repository we have create the file **nto-payment-asm-adapter.yaml** that can modified.

    ![](images/imageX.png)

- Replace **```<CLIENT ID>```** and **```<CLIENT SECRET>```** with values for your environment. Save file and run the following command

```bash
kubectl apply -f nto-payment-asm-adapter.yaml
```

![](images/imageX.png)

- Use the following command to monitor the progress. Wait for status to change to **Ready**

```bash
asmctl adapter list
```

![](images/imageX.png)

### **STEP 10**: Create APIs

- We will now use now use Anypoint Service Mesh auto discovery to create API's in Anypoint Platform. We will create API's for Customer, Inventory, Order and Payments services that are used by the demo application.

- Modify the Kubernetes custom resource definition (CRD) file **demo-apis.yaml**. 

- For each API, replace **```<ENV ID>```**, **```<USER>```** and **```<PASSWORD>```** with the values for your environment. If you are unsure how to get the environment Id check out this [article](https://help.mulesoft.com/s/question/0D52T00004mXPvSSAW/how-to-find-cloud-hub-environment-id). Save the file and run the following command

***NOTE: *** If you run this multiple times you might need to change the version number since Anypoint Platform will keep it around for 7 days.

```bash
kubectl apply -f demo-apis.yaml
```

![](images/imageX.png)

- Use the following command to monitor the progress. Wait for status to change to **Ready**

```bash
asmctl api list
```

![](images/imageX.png)

- You can also verify that the API's have been created in Anypoint Platform. Go to Anypoint Platform and navigate to **API Manager**

    ![](images/imageX.png)

### **STEP 11**: Binding API's with Services

- The last step is to bind the Kubernetes Services with the Anypoint Platform API's. To do this you will use the binding definition file **demo-bind-apis.yaml**. Execute the following command

```bash
kubectl apply -f demo-bind-apis.yaml
```

![](images/imageX.png)


- Use the following command to monitor the progress. Wait for status to change to **Ready**

```bash
asmctl api binding list
```

![](images/imageX.png)

- If you go may to **API Management** in Anypoint Platform and refresh the page you will see that the API's are now **Active**. 

- You have completed the installation of Anypoint Service Mesh. In the next section we will walk through applying some policies against the kubernetes services.

<a id="applypolicy"></a>
## Apply API Management Policies

### **STEP 12**: Apply Rate Limiting Policy to Customer API

- From the **API Management** Screen in Anypoint Platform click on the version number for **customer-api**

    ![](images/imageX.png)

- Click **Policies** and then click **Apply New Policy**. Expand **Rate Limiting** select newest version and click **Configure Policy**. 

    ![](images/imageX.png)

- We will configure the rate limit to be 1 call per minute. Click **Apply**

    ![](images/imageX.png)

- You should now see your new **Rate limiting** policy. To test this out run through the order process in the demo application. Try to run through it 2 times within a minute. The second time through you will get **Account Retrieval Failed** error.

    ![](images/imageX.png)

- Before moving onto the next step remove the **Rate Limiting** policy.

### **STEP 13**: Apply Client ID enforcement Policy to Payment API

- Navigate back to the ***API Administration** page. Click on the version number for **payment-api**.

- Click **Policies** and then click **Apply New Policy**. Expand **Client ID enforcement** select newest version and click **Configure Policy**. 

    ![](images/imageX.png)

- Leave all defaults and click **APPLY**

- You should now see your new **Client ID enforcement** policy. Once again run through the demo application but this time you should see **Payment Authorization Failed** when you click **AUTHORIZE PAYMENT**

    ![](images/imageX.png)

**CONGRATULATIONS!!!** You have completed install Anypoint Service Mesh and applying policies to kubernetes services via Anypoint Platform.

