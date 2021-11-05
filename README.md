# ansible-k8s

# Prerequistes
1. a desktop PC used as a server, in my case im lucky enough to have a Dell Optiplex 5080 micro, which is ridiculously powerfull, however, any hardware that has 16 GB of RAM and about 200G of disk would do just nicely.
2. a computer with anisble installed in it.
3. a managed firewall/router, dont freak out, you probably already have this if you have internet at home, this device MUST be able to do port forwarding if you want your services to be reachable form the internet (i have a fritzbox 6591).
4. Cloudflar

# set up

1. create a .vault_pass file.
2. create your own vault file and place it in group_vars/all/vault.
3. Encrypt it with ``` ansible-vault encrypt group_vars/all/vault ```
 
the vault file MUST HAVE THE FOLLOWING VALUES, except the last one, its not neccessarry unless you want to reach your cluster form the internet:

```
vault_localhost_sudo_password: YOUR LOCAL PASSWORD
vault_remote_sudo_password: REMOTE PASSWORD (WHAT YOU USE TO SSH TO YOUR NODES)
vault_ansible_user: REMOTE USER (THE USER YOU USE TO SSH TO YOUR NODES)
vault_ansible_local_user: YOU LOCAL USER, YOUR LAPTOP/DESKTOP USER, WHERE YOU ARE RUNNING THE SCRIPT FROM
vault_cloudflare_api_key: API_KEY 
vault_issuer_email: CLOUDFLARE EMAIL
vault_generated_files_dir: WHERE YOU WANT TO STORE YOUR CONFIG FILES
vault_home_dir: /home/ahmed/  HOME DR IN THE ANISBLE VMS, USUALLY (/home/user/)
vault_public_name: YOUR PUBLIC DOMAIN, IF YOU HAVE ONE
```
