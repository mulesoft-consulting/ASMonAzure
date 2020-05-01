# ASMonAzure
Walk through on how to install Anypoint Service Mesh on Azure Kubernetes Servies
![](images/title.png)  
Update: April 30, 2020

## Introduction

This cookbook will walk you through the process of installing **Anypoint Service Mesh** on **Azure**. You will deploy a demo application and secure using Anypoint Service Mesh.

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
