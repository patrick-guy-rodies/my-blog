# Using Hugo with OpenFaaS as CI with TLS

This repo contains Yaml file to install Hugo statics pages 

## Key features

* LetsEncrypt certificate
* DNS Azure service
* Domain name from Free service called freenom
* No use of Helm - :-)


## Possible Improvements
* Disaster Recovery
* MakeFile

## Pre-requisites

1. A Kubernetes 1.10+ cluster with role-based access control (RBAC) enabled

1. The kubectl command-line tool installed on your local machine and configured to connect to your cluster. You can read more about installing kubectl in the official documentation.

1. OpenFaas installed and configured - [patrickguyrodies/openfaas/](bitbucket.org:patrickguyrodies/openfaas.git) repo

1. Nginx Ingress controller

### Clone Repository

                $ git clone https://patrickguyrodies@bitbucket.org/patrickguyrodies/myblog.git

### Install Hugo as an OpenFaaS template

 Pull in my template using its URL

                $ faas template pull https://github.com/matipan/openfaas-hugo-template

1. Now create a Hugo site called `myblog`

                $ faas new --lang hugo myblog

This will create a folder called myblog where we will place the content for the site along with its configuration and any themes we may want.

The myblog.yml file is called a stack file and can be used to configure the deployment on OpenFaaS.

The following steps are based upon the Hugo [quick-start guide](https://gohugo.io/getting-started/quick-start/#step-2-create-a-new-site):

                $ cd example-site

                # Create a new site
                $ hugo new site .

                # Add a custom theme

                $ git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke

                # Append the theme to the config file
                $ echo 'theme = "ananke"' >> config.toml

When you are developing new content you’ll probably want to see what it looks like before deploying it. To do this, you can use the hugo server command inside the function’s directory:

                $ hugo server
                Watching for changes in /home/capitan/src/gitlab.com/matipan/openfaas-hugo-blog/blog/{archetypes,content,static}
                Watching for config changes in /home/capitan/src/gitlab.com/matipan/openfaas-hugo-blog/blog/config.toml
                Environment: "development"
                Serving pages from memory
                Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
                Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
                Press Ctrl+C to stop

Now go to http://localhost:1313 and check out what your site looks like.

Edit to config.toml and set the baseURL to the custom DNS domain that you will be using.

1. Check your OpenFaaS Yaml stack file to have correct settings

                $ nano myblog.yml

1. Login to your repo hub, Docker or Azure repo

                $ docker login pgr095.azurecr.io -- replace with your docker hub url

1. Login to OpenFaaS interface - Create a pass txt file with your password

                $ cat ~/faas_pass.txt | faas-cli login -u admin --password-stdin --gateway https://openfaas.pgr095.tk

Now deploy your site: You can also run faas-cli build, push & deploy if you want to separate the calls

                $ export OPENFAAS_URL="www.pgr095.tk" # Set when installing OpenFaaS

                $ faas-cli up -f myblog.yml

1. Create a deployment and service for your new site
Check yaml file myblog.yaml

                $ nano myblog.yaml
                # Deploy replacd container info with correct registry and name of the image
                $kubectl apply -f myblog.yaml

1. Map the custom domain with a Ingress
Now it’s time to map our Hugo blog to the custom domain above. We can do that with a FunctionIngress record. Check block_ingress.yaml

                $ nano myblog_ingress.yaml

Now apply the file with 

                $ kubectl apply -f myblog_ingress.yaml

1. Check your implementation

    1. Check your new ingress:

                $ kubectl get ingress -n openfaas
    
    1. Check new Service

                $  kubectl -n openfaas get svc myblog
    
    1. Check the certificate:

                $ kubectl get cert -n openfaas

Navigate to your domain and check out your new site.


> Service and Deployment are installed inside openfaas namespace

The implementation of TLS and controller are out of scope for this document, please check [patrickguyrodies/openfaas/](bitbucket.org:patrickguyrodies/openfaas.git) repo for more information around certificate.

### Updating Hugo site

Running faas-cli up command after any update will update your image.

1. Updating image
                $ kubectl -n openfaas set image deployment.v1.apps/myblog myblog=pgr095.azurecr.io/hugoblog --record

2. Checking update

                $ kubectl rollout status deployment.v1.apps/myblog -n openfaas

3. Checking logs from service

                $ kubectl logs <ingress-nginx-controller-name> -n ingress-nginx -f