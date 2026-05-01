# 🚀 Jenkins CI/CD Pipeline with Kubernetes Deployment

## 📌 Project Overview

This project demonstrates a complete **CI/CD pipeline** using **Jenkins**, **Docker**, and **Kubernetes (Minikube)**.
The pipeline automates building, testing, containerizing, and deploying an application to a Kubernetes cluster.

---

## 🧱 Tech Stack

* **Jenkins** – CI/CD automation
* **Docker** – Containerization
* **Kubernetes (Minikube)** – Container orchestration
* **Maven** – Build tool
* **GitHub** – Source control

---



## ⚙️ Pipeline Workflow

The Jenkins pipeline automates the following steps:

1. ✅ Clone source code from GitHub
2. 🧪 Run Unit Tests using Maven
3. 🔨 Build the application
4. 🐳 Build Docker image from Dockerfile
5. 📤 Push image to Docker Hub
6. 🧹 Remove local Docker image
7. 📝 Generate Kubernetes deployment file
8. ☸️ Deploy application to Kubernetes cluster

---

## 🏗️ Architecture Diagram

---

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/5ae6b2be-f60a-428f-a723-66e1bf95f58a" />


---

## 🐳 Docker Hub

Images are pushed to:

```
zakyaakram/jenkins-app:<BUILD_NUMBER>
```

---

## ☸️ Kubernetes Deployment

The application is deployed using a dynamically generated `deployment.yaml` file.

### Deployment Features:

* 1 replica
* Container port: 8080
* Dynamic image versioning using Jenkins build number

---

## 🔐 Kubernetes Authentication (Service Account Token)

Instead of using kubeconfig, deployment is done using:

* Kubernetes API Server
* Service Account Token

### Steps:

| Step | Command |
| --- | --- |
| Create Service Account | ``kubectl ``create ``serviceaccount ``jenkins-sa ``-n ``ivolve`` |
| Assign Permissions | ``kubectl ``create ``clusterrolebinding ``jenkins-sa-binding ``--clusterrole=cluster-admin ``--serviceaccount=ivolve:jenkins-sa`` |
| Generate Token | ``kubectl ``create ``token ``jenkins-sa ``-n ``ivolve`` |
| Add Token to Jenkins | Add as **Secret Text** → ID: ``ServiceAccount-Token`` |


#### 4. Add Token to Jenkins

* Go to **Manage Jenkins → Credentials**
* Add as **Secret Text**
* ID: `ServiceAccount-Token`

---

## 🌐 Kubernetes API Access

Minikube was started with external API access:

```bash
minikube start --apiserver-ips=<YOUR-IP> --apiserver-name=localhost
```

Then retrieve IP:

```bash
minikube ip
```

Used in pipeline:

```
https://<MINIKUBE-IP>:8443
```

---

## 🔄 Jenkins Pipeline (Key Stage)

### Deploy to Kubernetes

```groovy
stage('Deploy to Kubernetes') {
    steps {
        withCredentials([
            string(credentialsId: 'ServiceAccount-Token', variable: 'TOKEN')
        ]) {
            sh '''
            kubectl apply -f deployment.yaml \
              --server=$KUBE_SERVER \
              --token=$TOKEN \
              --insecure-skip-tls-verify=true \
              --validate=false

            kubectl get pods -n $KUBE_NAMESPACE \
              --server=$KUBE_SERVER \
              --token=$TOKEN \
              --insecure-skip-tls-verify=true
            '''
        }
    }
}
```

---

## 📊 Post Actions

The pipeline includes post-build actions:

* ✅ Always: Print pipeline status
* 🎉 Success: Deployment completed
* ❌ Failure: Error notification

---
## 📂Pipline Flow
---
<img width="1818" height="245" alt="image" src="https://github.com/user-attachments/assets/a2db6a14-6954-4a0c-bf76-7dd538e31966" />


---

## 🧠 Key Learnings

* Building end-to-end CI/CD pipelines
* Docker image lifecycle management
* Kubernetes deployment automation
* Token-based authentication for Kubernetes
* Jenkins credential management

---


---

## 👨‍💻 Author

**Zakya Akram**

