# ansible-k8s

This is an Ansible playbook that bootstraps a fully functioning k8s cluster + an nginx ingress controller + cert manager for managing certs + a test service (just a normal nginx image which can be accessed using the browser once everything is set up correctly).

The resulting cluster consists of 1 master node and 2 workers. if you would like to have more master or worker nodes, just edit the hosts file accordingly and the playbook will funtion teh same way (hopefully).

Note: This was tested with ubuntu 20.04

# Must haves to run the playbook successfully
1. Four ubuntu 20.04 virtual machines.
2. A computer with anisble installed in it.
3. A managed firewall/router, dont freak out, you probably already have this if you have internet at home (if you dont, then you should, take this as an opportunity buy one and replace the one that you currently have, you will not regret it :)), this device MUST be able to do port forwarding if you want your services to be reachable form the internet (i have a fritzbox 6591).
4. All of machines mentioned above MUST layer 3 connectivity(translation: they should be able to ping each other.

# Virtual machines specs
1. Specs for both master and worker nodes (RAM: 4G, DISK: 30G).
Note: This is not the minumum requirment, rather a comfortable set up. Im sure you can get away with less.

# Must haves for the cert-manager role to work
1. a cloudflare account.
2. an API Key or token with sufficient rights.
3. domain name that you own.

# What if you dont have a cloudflare account?
Do you want TLS?
# No:
Really? you should. But if you insist, then you really dont need the cert-manager role and the create-service role, you can just comment them out in the ``` site.yml ``` file and run the playbook normally. Then create a service and access it by ip to test the cluster like this: 
1. ``` kubectl create deployment nginx --image=nginx ```
2. ``` kubectl get pods -l app=nginx ```
3. ``` kubectl expose deployment nginx --port 80 --type NodePort ```
4. ``` NODE_PORT=$(kubectl get svc nginx --output=jsonpath='{range .spec.ports[0]}{.nodePort}') ```
5. ``` EXTERNAL_IP=WORKER_IP ```, yes, any worker.
6. ``` curl -I http://${EXTERNAL_IP}:${NODE_PORT} ```
7. you should get this something like this:


```
HTTP/1.1 200 OK
Server: nginx/1.19.10
Date: Sun, 02 May 2021 05:31:52 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 13 Apr 2021 15:13:59 GMT
Connection: keep-alive
ETag: "6075b537-264"
Accept-Ranges: bytes
```

# Yes:
Good.
There is an alternative here here, but this will require some work from your side. you need to edit, specifically the issuers file located in ``` roles/cert-manager/templates/issuers.yml ```

1. Read this: https://cert-manager.io/docs/tutorials/acme/http-validation/
2. Edit the ``` roles/cert-manager/templates/issuers.yml ```
3. Edit the ``` roles/cert-manager/tasks/install-cert-manager.yml ```, you need to remove or comment the ``` create cloudflare secrets file ``` 
4. Replace the entry ```manager.io/issuer```located in ```roles/create-service/files/nginx-test.yml``` to fit the name of the issuer you created.  
5. Move on to the next section of this document

<!-- # Prerequistes for running the cluster successfully
1. Understanding PKI's and how certificates based authentication work
2. An understanding of basic network concepts 
3. An understanding of what containers are and how they work -->

# Set up

1. Create a .vault_pass file and write a password.
2. Create your own vault var file and place it in group_vars/all/vault.
3. Encrypt it with ``` ansible-vault encrypt group_vars/all/vault ```
 
the vault file MUST HAVE THE FOLLOWING VALUES, except vault_public_name, its not neccessarry unless you want to reach your cluster form the internet:

```
vault_localhost_sudo_password: "YOUR LOCAL PASSWORD"
vault_remote_sudo_password: "REMOTE PASSWORD (WHAT YOU USE TO SSH TO YOUR NODES)"
vault_ansible_user: "REMOTE USER (THE USER YOU USE TO SSH TO YOUR NODES)"
vault_ansible_local_user: "YOU LOCAL USER, YOUR LAPTOP/DESKTOP USER, WHERE YOU ARE RUNNING THE SCRIPT FROM"
vault_cloudflare_api_key: "API_KEY"
vault_issuer_email: "CLOUDFLARE EMAIL"
vault_generated_files_dir: "WHERE YOU WANT TO STORE YOUR CONFIG FILES (EX. /home/user/folder), NOTE:NO SLASH AT THE END"
vault_home_dir: "HOME DIR IN THE ANISBLE VMS, USUALLY (/home/user/) NOTE: WITH A SLASH AT THE END"
vault_public_name: "YOUR PUBLIC DOMAIN, IF YOU HAVE ONE (this will be added to the API server certificate, in case you want to access it by DNS name)"
vault_LAN: "YOUR HOME NETWORK CIDR, check your router, or issue ```ip a``` on linux or ```ifconfig/all``` on windows"
vault_kubernetes_public_address: "{{ hostvars['loadbalancer-0'].ansible_host }}" "we will use the loadbalancer to forward traffic to our api servers, This way if one fails, another takes over."
```
4. Change the service file located in ``` roles/create-service/files/nginx-test.yml ```. (you need to change the host entries in the ingress part, instead of ``` nginx-k8s.ombada.tech ```, it should be whatever name that you give it). 
6. Make sure that the run_playbook file is executable ``` sudo chmod +x run_playbook ``` should do the trick
7. Run the playbook ```./run_playbook ```
8. After the playbook runs, you can verify by issuing the following commands:
 ``` kubectl get nodes ```
 ``` kubectl get pods --all-namespaces ```
 ``` kubectl get svc --all-namespaces ```
 ``` kubectl get ing ```
 ``` kubectl get certs ```
 9. Configure an entry in your internal dns server or hosts file (in linux: ``` /etc/resolv.conf ```, in windows: ```C:\Windows\System32\drivers\etc\hosts ```, its hidden under windows, so you need to say show all files, and you MUST run your editor as an administrator to make changes)
 10. If you've done everything correctly, you should be able to reach your service by name from the browser.
 
 # Bugs and Issues
 Open a new Issue and i will try to solve it :).
