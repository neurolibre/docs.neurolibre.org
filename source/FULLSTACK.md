# Terraform deployment

Both `preprint` and `preview` flavors of the full-stack server can be deployed and managed using Terraform. This modular approach allows for easy scaling and customization of the server infrastructure. Currently, the available providers support OpenStack only, yet the modular approach allows for easy integration with other cloud providers.

## Deploy in 6 steps

Assuming that you have the terraform CLI installed, have ssh access to the target OpenStack project, and have the necessary permissions to create instances, volumes, and other resources, you can deploy the full-stack server in 6 steps:

- Clone infralibre: `git clone https://github.com/neurolibre/infralibre.git`
- Copy over the files from `terraform-modules/providers/openstack-neurolibre-server` to a directory on your local machine
- Source the OpenStack credentials and set necessary variables
- Run `terraform init` to initialize the working directory
- Run `terraform plan` to see the execution plan
- Run `terraform apply` to apply the changes

that's it!

- Run `terraform destroy` to destroy the infrastructure

Ability to create/destroy the server easily should not give you license not to understand what you are doing. The rest of this section is dedicated to explaining what each of the server components is responsible for and how they interact with each other.

The only glitch you may have to deal with is mounting the external-data volume. The volume will be created and attached to the instance if you specified NOT to use an existing volume (by UUID). In this case, it will not be formatted with ext4, so you will have to do that with the following command:

```
lsblk # Figure out the name of the volume, e.g. `/dev/vdc`
sudo mkfs.ext4 /dev/vdc # Format the volume
sudo mount /dev/vdc /DATA # Mount the volume
sudo chown -R ubuntu:ubuntu /DATA # Make sure the volume is owned by the user
sudo chmod -R 755 /DATA # Make sure the volume is writable
```

# Comprehensive documentation

## Full-stack server components

NeuroLibre operates two servers dedicated to serving static files and API endpoints. These servers are designed to provide essential resources and functionality for various purposes, including testing, technical reviews of NeuroLibre Reproducible Preprints (NRPs), and post-publication access, where the resources are reserved for published NRPs only.

The purpose of this section is to offer an overview of each component within the NeuroLibre full-stack server. This overview will provide a solid understanding of its architecture before delving into hands-on development. It serves as a helpful guide to familiarize oneself with the system's structure and functionalities, facilitating a smoother and more informed development process.

### Data Storage for Test Server Stack (May 2024)

As of May 2024, all data associated with the test server stack is stored in the neurolibre-data volume, which has a capacity of 1TB and is identified by the ID 05fe3035-**. This volume includes:

* The book-artifacts directory, which contains the static HTML files generated from myst/jupyter-book builds.
* Other directories storing submission data, managed by the repo2data tool.

Needless to say, please do NOT delete this volume. It would not be the end of the world, yet this would hamper ongoing submissions. 

#### Volume Mounting

On the instance where the `neurolibre/full-stack-server` is deployed with preview configurations, the `neurolibre-data` volume is mounted on `/DATA`. You can verify this configuration by viewing the `/etc/fstab` file with the command:

```
sudo cat /etc/fstab
```

#### Network File Sharing (NFS) Setup

The /DATA directory is shared between both the full-stack-server and binder-test instances via a Network File Sharing (NFS) protocol. In this setup:

* The `full-stack-server` acts as the NFS server.
* The `binder-test` instance acts as the NFS client.

This configuration ensures that both instances have access to the same data directory (`/DATA`), facilitating seamless data management and sharing.

##### Setting up the NFS server 

######  Mounting and Installing NFS Server

1. Ensure `/DATA` is Mounted

Confirm that the `/DATA` directory has been properly mounted the `full-stack-server` instance.

2. Install NFS Kernel Server

Update the package list and install nfs-kernel-server:

```
sudo apt-get update
sudo apt install nfs-kernel-server
sudo systemctl start nfs-kernel-server 
```

confirm the status:

```
sudo systemctl start nfs-kernel-server 
```

> [!NOTE]
> Make sure that the instance is spawned from a [Ubuntu Cloud image](https://cloud-images.ubuntu.com/) with **generic** kernel. Instances spawned from a linux-kvm (Kernel-based Virtual Machine) optimized kernel will not support the installation of nfs-kernel-server due to the lack of necessary NFS drivers (nfsd).

###### Configure NFS Exports

3. Edit the /etc/exports File

Open the /etc/exports file for editing:

```
sudo nano /etc/exports
```

4. Add the following line to share the `/DATA` directory over the internal network:

```
/DATA 192.168.73.0/24(rw,sync,no_root_squash,no_all_squash)
```

Put the following content in `/etc/exports` (use `sudo nano /etc/exports`):

```
/DATA 192.168.73.0/24(rw,sync,no_root_squash,no_all_squash)
```

This configuration shares the `/DATA` directory with all machines on the `192.168.73.0/24` subnet, allowing read and write access while disabling root squashing.

##### Setting up an NFS client

1. Install `nfs-common`:

```
sudo apt-get update
sudo apt install nfs-common
```

2. To mount shared volume temporarily (disappears upon reboot of the client):

```
sudo mount -t nfs 192.168.73.179:/DATA /DATA
```

If you are asking the heck is that `192.168.73.179`, it is the internal IP address of the nfs server. You can either find it out on the cloudflare dashboard or by running `hostname -i` on the nfs server instance.  

2. To mount shared volume permanently:

Put the following content in the last line of the `/etc/fstab` (use `sudo nano /etc/fstab`):

```
192.168.73.179:/DATA      /DATA      nfs rw,noatime,nolock,hard,tcp 0 0
```

**NOTE:** If you are deploying BinderHub test server using terraform, you don't have to deal with this manually, just ensure that you pass the IP address using the following variable:

```json
variable "sftp_ip_address" {
  description = "Internal IP address of the SFTP instance on openstack."
}
```

For more details on deploying BinderHub on openstack using Terraform, see the relevant section of the developer documentation.

### Static files

Static files are the reproducible preprint content (HTML, CSS, JS, etc.) that are generated in one of the following cases:

1. The user front-end of the RoboNeuro web application (https://roboneuro.herokuapp.com/)
2. The technical screening process on the NeuroLibre review repository (https://github.com/neurolibre/neurolibre-reviews/issues)
3. The finalized version of the reproducible preprint.

Cases 1-2 are handled on the `preview` server (on [Compute Canada Arbutus](https://arbutus.cloud.computecanada.ca/) to `preview.conp.cloud`), while case 3 is handled on the `production` server (on [NeuroLibre's own cloud](https://opennebula.conp.cloud) to `preprint.conp.cloud`), both making the respective content available to the public internet.

Under the hood, we use [NGINX](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/) to serve static content. To manage the [DNS records](https://www.cloudflare.com/learning/dns/dns-records/) for the domain `conp.cloud` over which NGINX serves the content, we are using [Cloudflare](https://www.cloudflare.com). Cloudflare also provides [SSL/TLS encryption](https://www.cloudflare.com/learning/ssl/what-is-ssl/) and [CDN](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) (content delivery network, not Cotes-des-Neiges :), a tiny Montrealer joke).

A good understanding of these concepts is essential for successfully deploying NeuroLibre's reproducible preprints to production. Make sure you have a solid grasp of these concepts before proceeding with the deployment instructions.

### API endpoints

An application programming interface (API) endpoint is a specific location within NeuroLibre server (e.g., `preview.conp.cloud/api/books/all`) that provides access to resources and functionality that are available (e.g., list reproducible preprints on this server):

* Some of the resources and functions available on the `preview` and `production` servers differ. For instance, only the `production` server is responsible for archiving the finalized preprint content on Zenodo, while JupyterBook builds are currently only executed on the `preview` server.

* On the other hand, there are `common` resources and functions shared between the`preview` and `production` servers, such as retrieving a reproducible preprint.

There is a need to reflect this separation between `preview`, `production`, and `common` tasks in the logic of how NeuroLibre API responds to the HTTP requests. To create such a framework, we are using [Flask](https://flask.palletsprojects.com). Our Flask framework is defined by three python scripts: 

```
full-stack-server/
├─ api/
│  ├─ neurolibre_common.py
│  ├─ neurolibre_preview.py
│  ├─ neurolibre_production.py
```

Even though Flask includes a built-in web server that is suitable for development and testing, it is not designed to handle the high levels of traffic and concurrency that are typically encountered in a production environment.

[Gunicorn](https://gunicorn.org/), on the other hand, is a production-grade application server that is designed to handle large numbers of concurrent tasks. It acts as a web service gateway interface (WSGI) that knows how to talk Python to Flask. As you can infer by its name, it is an "interface" between Flask and something else that, unlike Gunicorn, knows how to handle web traffic.

That something else is a [reverse proxy server](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/), and you already know its name, NGINX! It is the gatekeeper of our full-stack web server. NGINX decides whether an HTTP request is made for static content or the application logic (encapsulated by Flask, served by Gunicorn).

I know you are bored to death, so I tried to make this last bit more fun:

This Flask + Gunicorn + NGINX trio plays the music we need for a production-level NeuroLibre full-stack web server. Of these 3, NGINX and Gunicorn always have to be all ears to the requests coming from the audience. In more computer sciency terms, they need to have their own [daemons](https://en.wikipedia.org/wiki/Daemon_(computing)) 👹.

NGINX has its daemons, but we need a unix systemD (d for daemon) ritual to summon daemons upon Gunicorn 🕯👹👉🦄🕯. To do that, we need go to the `/etc/` dungeon of our ubuntu virtual machine and drop a service spell (`/systemd/neurolibre.service`). This will open a portal (a unix socket) through which Gunicorn's daemons can listen to the requests 24/7. We will tell NGINX where that socket is, so that we can guide right mortals to the right portals.

Let's finish the introductory part of our full-stack web server with reference to [this Imagine Dragons song](https://www.youtube.com/watch?v=mWRsgZuwf_8):

```
  When the kernels start to crash
  And the servers all go down
  I feel my heart begin to race
  And I start to lose my crown

  When you call my endpoint, look into systemd
  It's where my daemons hide
  It's where my daemons hide
  Don't get too close, /etc is dark inside
  It's where my daemons hide
  It's where my daemons hide

  I see the error messages flash
  I feel the bugs crawling through my skin
  I try to debug and fix the code
  But the daemons won't let me win (you need sudo)
```

P.S. No chatGPT involved here, only my demons.

### Security

On Cloudflare, we activate [full(strict)](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/) encryption mode for handling SSL/TLS certification. In addition, we disable legacy TLS versions of  `1.0` and `1.1` due to [known vulnerabilities](https://www.acunetix.com/blog/articles/tls-vulnerabilities-attacks-final-part/). With these configurations, we receive a solid SSL Server Rating of A from [SSL Labs](https://www.ssllabs.com/ssltest/analyze.html).

While implementing SSL is a fundamental necessity for the security of our server, it is not sufficient on its own. SSL only addresses the security of the communication channel between a website and its users, and does not address other potential security vulnerabilities. For example, any web server will be subjected to brute-force attacks typically coming from large botnets. To deal with this, we are using `fail2ban`, which is a tool that monitors our nginx log files and bans IP addresses that show malicious activity, such as repeated failed login attempts.

#### What else? - Future considerations 

Another consideration is client-side certificate authorization. In this approach, clients (e.g., `roboneuro`) are required to present a digital certificate as part of the authentication process when they attempt to access a server or service. The server then verifies the certificate to determine whether the client is authorized to access the requested resource. This would require creating a client certificate on Cloudflare, then adding that to the server block :

```
ssl_client_certificate  /etc/nginx/client-ca.crt;
ssl_verify_client optional;
```

Verification must be location-optional, as it works against serving static files. To achieve this only for api endpoints, the config would look like this:

```
location /api/ {
...
if ($ssl_client_verify != "SUCCESS") { return 403; }
...
}
```

This is currently NOT implemented due to potential issues on Heroku, where our web apps are hosted. 

Alternatively, Cloudflare provides [API Shield](https://developers.cloudflare.com/api-shield/) for enterprise customers and [mutual TLS](https://developers.cloudflare.com/api-shield/security/mtls/) for anyone.

### Performance

<details>
  <summary>Expand this tab to see the list of key configurations that determine the performance of serving static files with nginx</summary>
  <ul>
    <li><code>worker_processes</code>: This directive specifies the number of worker processes that nginx should use to handle requests. By default, nginx uses one worker process, but you can increase this number if you have a multi-core system and want to take advantage of multiple cores.</li>
    <li><code>worker_connections</code>: This directive specifies the maximum number of connections that each worker process can handle. Increasing this value can improve the performance of nginx if you have a high number of concurrent connections.</li>
    <li><code>sendfile</code>: This directive enables or disables the use of the <code>sendfile()</code> system call to send file contents to clients. Enabling <code>sendfile</code> can improve the performance of nginx when serving large static files, as it allows the kernel to copy the data directly from the filesystem cache to the client without involving the worker process.</li>
    <li><code>tcp_nopush</code>: This directive enables or disables the use of the <code>TCP_NOPUSH</code> socket option, which can improve the performance of nginx when sending large responses to clients by allowing the kernel to send multiple packets in a single batch.</li>
    <li><code>tcp_nodelay</code>: This directive enables or disables the use of the <code>TCP_NODELAY</code> socket option, which can improve the performance of nginx by disabling the Nagle algorithm and allowing the kernel to send small packets as soon as they are available, rather than waiting for more data to be buffered.</li>
    <li><code>gzip</code>: This directive enables or disables gzip compression of responses. Enabling gzip compression can improve the performance of nginx by reducing the amount of data that needs to be transmitted over the network.</li>
    <li><code>etag</code>: This directive enables or disables the use of <code>ETag</code> headers in responses. Enabling <code>ETag</code> headers can improve the performance of nginx by allowing clients to cache responses and reuse them without making additional requests to the server.</li>
    <li><code>expires</code>: This directive sets the <code>Expires</code> header in responses, which tells clients to cache responses for a specified period of time. Enabling <code>Expires</code> headers can improve the performance of nginx by allowing clients to reuse cached responses without making additional requests to the server.</li>
    <li><code>keepalive_timeout</code>: This directive sets the timeout for keepalive connections, which allows clients to reuse connections for multiple requests. Increasing the value of <code>keepalive_timeout</code> can improve the performance of nginx by reducing the overhead of establishing new connections.</li>
    <li><code>open_file_cache</code>: This directive enables file caching, which can improve the performance of nginx by allowing it to reuse previously opened files rather than opening them anew for each request.</li>
  </ul>
</details>

For further details on tuning NGINX for performance, see these blog posts about [optimizing nginx configuration](https://www.nginx.com/blog/tuning-nginx/) and [load balancing](https://www.nginx.com/blog/load-balancing-with-nginx-plus).

You can use [GTMetrix](https://gtmetrix.com/) to test the loading speed of individual NeuroLibre preprints. The loading speed of these pages mainly depends on the content of the static files they contain. For example, pages with interactive plots rendered using HTML may take longer to load because they encapsulate all the data points for various UI events.

## Deploy and configure NeuroLibre servers

Clone this repository to `home` directory (typically `/home/ubuntu`):

```
cd ~
git clone https://github.com/neurolibre/full-stack-server.git
```

Be careful not to run these commands (or anything else in this section) as the `root` user. If you ssh'd into the VM as root, you can switch to ubuntu by executing the `su ubuntu` command in the remote terminal. 

> Throughout the rest of this section, **`<type>`** refers to either `preview` or `preprint`.

### Install prerequisites 

* Update the package lists 

```
sudo apt-get update
```

* Install some basic dependencies in one line

```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates gpg lsb-release
```

### Install Redis 

Simply follow [these instructions](https://redis.io/docs/getting-started/installation/install-redis-on-linux/) to install Redis on Ubuntu.

Our server will use Redis both as message broker and backend for Celery asynchronous task manager. What a weird sentence, is not it? I tried to explain above what these components are responsible for.

### Install Docker

#### Why?

Docker containers are a critical component of the NeuroLibre workflow. These containers, created by `repo2docker` as part of the BinderHub builds, store all the dependencies for respective preprints in Docker images located in NeuroLibre's private container registry (https://binder-registry.conp.cloud).

**On the test (preview) server**, Docker is required to pull these images from the registry to build MyST-formatted articles by spawning a Jupyter Hub. These operations are managed by the `myst_libre` Python package. As of May 2024, Jupyter-Book-based builds are handled by BinderHub. If ongoing support for Jupyter Books is needed, these builds should also be managed using `myst_libre`.

**On the production (preprint) server**, Docker is necessary to pull images and archive them on Zenodo.

#### Installation steps

These steps are adapted from the [official documentation](https://docs.docker.com/engine/install/ubuntu) for Ubuntu 24.04. Please keep it up to date with the Ubuntu version. 

* Set up Docker's apt repository:

```
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

* Select the OLDEST version among the available versions in the stable repository (just a convention):

```
VERSION_DOCKER=$(apt-cache madison docker-ce | awk '{ print $3 }' | sort -V | head -n 1)
VERSION_CONTAINERD=$(apt-cache madison containerd.io | awk '{ print $3 }' | sort -V | head -n 1)
```

* Install docker and contai_nerd_ (container runtime): 

```
sudo apt install -y containerd.io=$VERSION_CONTAINERD docker-ce=$VERSION_DOCKER docker-ce-cli=$VERSION_DOCKER
```

#### Docker configurations

To see the current docker configurations you can run `docker info`. 

One important setting is the root directory where docker will save the images. To see this `docker info | grep 'Docker Root Dir`. 

This is by default set to `/var/lib/docker`, which is on the system partition with limited memory. As the number of docker images grow (e.g., there are 10 ongoing submissions), this will raise problems. 

On openstack, you can use the ephemeral storage (the HDD allocation that will be destroyed when the instance is destroyed) that is mounted to `/mnt` directory for this purpose. To achieve that:

```
sudo nano /etc/docker/daemon.json
```

with the following content:

```
{
  "data-root": "/mnt"
} 
```

* Stop the docker service:

```
sudo systemctl stop docker
```

Ideally you should perform these modifications **before** pulling images. If this is the case, skip this step. Otherwise, copy them over to the new location not to lose those images (this may take some time):

```
sudo rsync -aP /var/lib/docker/ /mnt
```

After this you can `sudo rm -r /var/lib/docker` to reclaim that space. 

* Start the docker service:

```
sudo systemctl start docker
```

* Confirm the changes have been applied by `docker info | grep 'Docker Root Dir`.

### Install MyST markdown dependencies (preview server only)

* As the test (preview) server should be capable of building MyST formatted preprints, certain Node dependencies must be present. For the latest version dependencies please see [here](https://mystmd.org/guide/quickstart).


* Install node version manager (NVM):

Either close and reopen your terminal to start using nvm or run the following to use it right away:

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

Confirm installation:

```
nvm -v
```

* Use NVM to install node 

```
nvm install 20
```

confirm by `node -v`. Also check `npm -v` to ensure the minimum version requirement is met.

### Flask, Gunicorn, Celery, and other Python dependencies 

This documentation assumes that the server host is a Ubuntu VM. To install Python dependencies,
we are going to use virtual environments.

Check out which version of python is installed where:

```
which python3
python --version
```

For the server app, you may need a specific version of Python that differs from the one bundled with your Ubuntu distribution. We will manage this using virtual environments.

##### Installing virtualenv

To install virtualenv, use the following command:

```
sudo apt install python3-virtualenv
```

##### Installing an Older Python Version

If you need an older version of Python, such as Python `3.8` on Ubuntu `24.04`, you might need to add the deadsnakes PPA (Personal Package Archive).

First, check if the desired Python version (e.g., `3.8`) is available in the default repositories. You can do this by trying to install it directly:

```
sudo apt install python3.8
```

If you encounter an error like the one below, you will need to add the 💀🐍 deadsnakes PPA (or another PPA of you choice that has the desired python distribution):

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package python3.8
E: Couldn't find any package by glob 'python3.8'
```

Follow these steps to add the deadsnakes PPA and install the required Python version:

* Add the deadsnakes PPA:

```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```

* Now install the geriatric python version you want: 

```
sudo apt install python3.8
```

* Confirm installed location:

```
which python3.8
```

##### Creating a virtual env

Create a new folder (`venv`) under the home directory and inside that folder, create a virtual environment named `neurolibre`:

```
mkdir ~/venv
cd ~/venv
virtualenv neurolibre --python=/usr/bin/python3.8
```

> Note: Please do not replace the virtual environment name above (`neurolibre`) with something else. You can take a look at the `systemd/neurolibre-<type>.service` configuration files as to why. 

If successful, you should see `~/venv/neurolibre` created. Now, activate this virtual environment to the install dependencies in the right place:

```
source ~/venv/neurolibre/bin/activate
```

If successful, the name of the environment should appear on bash, something like `(neurolibre) ubuntu@neurolibre-sftp:~/venv$`. **Ensure that the (neurolibre) environment is activated when you are executing the following commands**:

```
pip3 install --upgrade pip setuptools wheel
pip3 install -r ~/full-stack-server/api/requirements.txt
```

You can confirm the packages/versions via `pip3 freeze`.

##### Add working environment secret variables 

Based on the `env.<type>.template` file located at the `/api/` folder under this repository (`~/full-stack-server/api`). create a `~/full-stack-server/api/.env` file and fill out the respective variable values:

```
cp ~/full-stack-server/api/env.<type>.template ~/full-stack-server/api/.env
nano .env
```

> Note, this file will be ignored by git as it MUST NOT be shared. Please ensure that the file name is correct (`~/full-stack-server/api/.env`).

#### Configure the server as a systemd service

Depending on the server **type** [`preview` or `preprint`], copy the respective content from `systemd` folder in this repository into `/etc/systemd/system`:

```
sudo cp ~/full-stack-server/systemd/neurolibre-<type>.service /etc/systemd/system/neurolibre-<type>.service
```

If the python virtual environment and its dependencies are properly installed, you can start the service by:

```
sudo systemctl start neurolibre-<type>.service
```

You can check the status by

```
sudo systemctl status neurolibre-<type>.service
```

This should start multiple `gunicorn` workers, each one of them binding our flask application to a `unix socket` located at `~/full-stack-server/api/neurolibre_<type>_api.sock`. You can check the existence of the `*.sock` file at this directory. The presence of this socket file is of key importance as in the next step, we will register it to nginx as an upstream server! 

> Reminder: Replace the **`<type>`** in the commands above either with `preprint` or `preview` depending on the server (e.g., `neurolibre-preview.service`) you are configuring. Note that this is not only a naming convention, but also defines a functional separation between the roles of the two servers.

#### Configure Celery as a systemd service

For Celery async task queue manager to work, there are two requirements:
1. Redis properly installed and running 
2. `neurolibre-<type>.service` is up and running (see previous step)

#### Preprint <--> Preview serve data sync configurations

After technical screening process, the final version of the Jupyter Book and respective data will be transferred from the preview (source) to the preprint (destination) server. At least as for the current convention. To achieve this, we preferred [`rsync`](https://linux.die.net/man/1/rsync) that uses ssh for communication between the source and destination.

Whenever the public IP of either server changes and/or the VMs are re-spawned from scratch, please ensure that the following configuration is valid.  

1. Create an ssh keypair _on the destination (preprint) server_ `ssh-keygen -t rsa`
2. Add the **public** key (`*.pub`) to the `~/.ssh/authorized_keys` file _in the source (preview) server_. This will allow production server to pull files from the preview server.
3. Confirm that you can ssh **into** the source (preview) server **from** the destination (preprint) server `ssh -i ~/.ssh/key ubuntu@preview.server.ip`.
4. Create an ssh configuration file `~/.ssh/config` _on the destination (preprint) server_ to recognize preview (source) server as a host. The content of the configuration will be:

```
Host neurolibre-preview
        HostName xxx.xx.xx.xxx
        User ubuntu
```

Ensure that the you replaced `xxx.xx.xx.xxx` with the public IP address of the preview server. The first line of the configuration above declares the alias `neurolibre-preview`. If you change this name, you will need to make respective changes in the `neurolibre_preprint_api.py`. 

5. Test file transfer. SSH into the destination (preprint) server and pull an example file from the source server: 

```
rsync -avR neurolibre-preview:/DATA/foo.txt /
```

Provided that the `/DATA/foo.txt` exists on the source (preview) server and you successfully configured ssh, you should see the same file appearing at the same destination (directory syncing, see more [here](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories)) on the destination (preprint) server.

#### Cloud-level considerations

#### NGINX installation and configurations

To install and configure `nginx`:

```
sudo apt install nginx
```

Allow HTTP (80) and HTTPS (443) ports:

```
sudo ufw allow 80,443/tcp
```

Create the following folders: 

```
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

##### Update the `nginx.conf` and add `neurolibre_params`

Replace the default nginx configuration file with the one from this repository:

```
sudo cp ~/full-stack-server/nginx/nginx.conf /etc/nginx/nginx.conf
```

Add proxy pass parameters for the upstream server that is gunicorn/flask:

```
sudo cp ~/full-stack-server/nginx/neurolibre_params /etc/nginx/neurolibre_params
```

##### Add server-specific configuration files

Depending on the server **type** [`preprint` or `preview`], copy `/nginx/neurolibre-<type>.conf` file to `/etc/nginx/sites-available`:

```
sudo cp ~/full-stack-server/nginx/neurolibre-<type>.conf /etc/nginx/sites-enabled/neurolibre-<type>.conf
```

> Reminder: Replace the **`<type>`** in the commands above either with `preprint` or `preview` depending on the server (e.g., `neurolibre-preview.service`) you are configuring.

##### Create SSL certificates

* Login to the cloudflare account, got to the respective site domain (e.g. `conp.cloud` or `neurolibre.org`), under the `SSL/TLS` --> `Origin Server` --> `Create Certificate`.

* Use the default method (RSA 2048), leave the host names as is (or define a single subdomain, your call). Click create. 

* This will create two long strings, one for `certificate` (first) and one for the private `key`. Create two files under `/etc/ssl` directory:
     *  `cd /etc/ssl`
     * `sudo nano /etc/ssl/conp.cloud.pem` --> Copy the `certificate` key here and save 
     * `sudo nano /etc/ssl/conp.cloud.key` --> Copy the `key` string here and save.

> Note: `conp.cloud.pem` and `conp.cloud.key` can be changed with any alternative name, such as `neurolibre.pem` and `neurolibre.key` as long as the origin certificate content is accurate AND if your `nginx.conf` is configured to look for that new file name:

```
    ssl_certificate     /etc/ssl/conp.cloud.pem;
    ssl_certificate_key    /etc/ssl/conp.cloud.key;
```

Remember that the same directives also exist in the `/etc/nginx/sites-available/neurolibre-<type>.conf` configuration files, both for `preview` and `preprint`. If you decide to change the certificate name, you will need to update these configs as well.

##### A tiny hack to serve swagger ui static assets over upstream 

This is a bit tricky both because a funny `_` (what python gives) vs `-` (what nginx expects) mismatch, also because we will be serving the swagger UI over a convoluted path. When you run the flask app locally, it will know where to locate UI-related assets and serve the UI on your localhost. But when we attempt it from `https://<type>.neurolibre.org/documentation`, our NGINX server will not be able to locate them, so we help it: 

```
sudo mkdir /etc/nginx/html/flask-apispec
sudo cp -r ~/venv/neurolibre/lib/python3.6/site-packages/flask_apispec/static /etc/nginx/html/flask-apispec/.
```

This is required for both server types.

##### Start the server

When you symlink the configuration file from `sites-available` to `sites-enabled`, it will take effect: 

```
sudo ln -s /etc/nginx/sites-available/neurolibre-<type>.conf /etc/nginx/sites-enabled/neurolibre-<type>.conf
```

then

```
sudo systemctl restart nginx
```

That's it! The server should be accessible at the domain you configured (e.g. https://preview.neurolibre.org)

This is required for both server types.

> Remember to use the correct name (`neurolibre-<type>.conf`) for the respective (`preprint` or `preview`) server you are configuring.

> Also, if your upstream server, i.e. the gunicorn socket, is not active, the webpage will not load. Ensure that `sudo systemctl status neurolibre-<type>.service` shows active status for the respective server. 

### Newrelic installation

We will deploy New Relic Infrastructure (`newrelic-infra`) and the NGINX integration for New Relic (`nri-nginx`,[source repo](https://github.com/newrelic/nri-nginx)) to monitor the status of our host virtual machine (VM) and the NGINX server. 

With these tools, we will be able to track the performance and availability of our host and server, and identify and troubleshoot any issues that may arise. By using New Relic and the NGINX integration, we can manage and optimize the performance of our system from a single location.

> You need credentials to login to [NewRelic portal](https://one.newrelic.com/). Otherwise you cannot proceed with the installation and monitoring. 

Ssh into the VM (`ssh -i ~/.ssh/your_key root@full-stack-server-ip-address`) and follow these instructions:

1. Install new relic infrastructure agent 

After logging into the newrelic portal, click `+ add data`, then type `ubuntu` in the search box. Under the `infrastructure & OS`, click `Linux`. When you click the `Begin installation` button, the installation command with proper credentials will be generated. Simply copy/paste and execute that command on the VM terminal.  

Alternatively, you can replace `<NEWRELIC-API-KEY-HERE>` and `<NEWRELIC-ACCOUNT-ID-HERE>` with the respective content below (please do not include the angle brackets).

```bash
curl -Ls https://download.newrelic.com/install/newrelic-cli/scripts/install.sh | bash && sudo NEW_RELIC_API_KEY=<NEWRELIC-API-KEY-HERE> NEW_RELIC_ACCOUNT_ID=<NEWRELIC-ACCOUNT-ID-HERE> /usr/local/bin/newrelic install
```

After successful installation, the newrelic agent should start running. Confirm its status by:

```bash
sudo systemctl status newrelic-infra.service
```
If the installer prompted you to add additional packages including `NGINX`, `Golden Signal Alerts` etc. , you may skip the step 2. below. Nevertheless, go through the second bullet point (of step 2) to confirm successful `nri-nginx` installation.  

2. Install new relic nginx integration

* Download the `nri-nginx_*_amd64.deb` from the assets of the latest (or a desired) [nri-nginx release](https://github.com/newrelic/nri-nginx/releases). You can get the download link by right clicking the respective release asset:

```
wget https://github.com/newrelic/nri-nginx/releases/download/v3.2.5/nri-nginx_3.2.5-1_amd64.deb -O ~/nri-nginx_amd64.deb
```

* Install the package

```
cd ~
sudo apt install ./nri-nginx_amd64.deb
```

* If the installation is successful, you should see `nginx-config.yaml.sample` upon: 

```
ls /etc/newrelic-infra/integrations.d
```

For the next step, confirm that the `stab_status` of the nginx is properly exposed to `127.0.0.1/status` by:

```bash
curl 127.0.0.1/status
```

The output should look like:

```
Active connections: 1 
server accepts handled requests
 126 126 125 
Reading: 0 Writing: 1 Waiting: 0 
```

3. Configure the nginx agent

We will use the default configuration provided in the sample configuration by copying it to a new file:

```
cd /etc/newrelic-infra/integrations.d
sudo cp nginx-config.yml.sample nginx-config.yml
```

This action will start the `nri-nginx` integration. Run `sudo systemctl status newrelic-infra.service` to confirm successful. You should see the _"Integration health check finished with success"_ message for _integration_name=nri-nginx_.

### Fail2ban installation and configuration 

* Install 

```
sudo apt-get install fail2ban
``` 

* Copy over fail2ban configurations from this repository to where they should be:

```
sudo cp -R ~/full-stack-server/fail2ban/* /etc/fail2ban
```

> Note that these configurations assume that `/home/ubuntu/nginx-error.log` and `/home/ubuntu/nginx-access.log` are where they should be and configured as error/access logs for the nginx server.

* Activate NGINX jails:

```
sudo systemctl restart fail2ban.service
```

* Confirm that the service is ready:

```
sudo systemctl status fail2ban.service
```

* See the number and the list of jails set up:

```
sudo fail2ban-client status
```

* Be a responsible guardian and take a look at those jails. For example, to see the list of blocked IPs due to suspicious authentication retries:

```
sudo fail2ban-client status nginx-http-auth
```

You can check other jails (e.g., `nginx-noproxy`,`nginx-nonscript`,`sshd`). 

* Use your get out of the jail card: 

In case you trapped yourself while testing if the jail works:

```
sudo fail2ban-client set nginx-http-auth unbanip ip.address.goes.here
```

See this [documentation](https://www.digitalocean.com/community/tutorials/how-to-protect-an-nginx-server-with-fail2ban-on-ubuntu-14-04) for further details regarding the configurations.

## Monitor, Debug, and Improve

### Use Newrelic

Login to the NewRelic portal where you can take a look at all the entities from both `preview` and `preprint` server. These entities could be specific to NGINX or the hosts events. You can take a look at a variety of logs, and see if there are any errors or critical warnings thrown.

NewRelic not only provides centralized monitoring of multiple resources, but also allows you to set alert conditions! You can install the mobile application to your iPhone/Android and get immediately notified when things are out of control.

### Know your logs 

We have several `systemd` services that are critical. You can use `journalctl` to take a look at what's going on with each one of them. For example, if you need to take a look at the logs from gunicorn (through which Flask logs are forwarded): 

```
journalctl -xn 20 -u neurolibre-preview.service
```

The above would help you understand what went wrong if the service failed to restart. Note that `sudo systemctl status neurolibre-preview.service` is not going to explain what went wrong at the level you expect.

Here, `-xn` is the number of last N lines of log with application context and `-u` is followed by the name of the service (e.g., nginx.service). For further details, see [journalctl reference](https://www.freedesktop.org/software/systemd/man/journalctl.html).

## Hosting DashBoards with Dokku

### Create a Dokku instance

* Create a Ubuntu VM and associate a floating IP (vm.floating.ip).

* Install Dokku to the VM by following [debian installation instructions](https://dokku.com/docs/getting-started/install/debian/).

* Add an shh key to the Dokku. To achieve this, you need root access (`sudo -i`). If an ssh keypair does not exist (~/.ssh/id_rsa), create one (`ssh-keygen`): 

```
dokku ssh-keys:add <name> ~/.ssh/id_rsa 
```

* Add global domain

```
dokku domains:add-global db.neurolibre.org
```

* On Cloudflare, add an A record for wildcard nested domain `*.db.neurolibre.org`. This will require total TLS to issue individual certificates for every proxied hostname (paid feature). Otherwise, SSL termination will fail.

* On Cloudflare: 
  * Under the SSl/TLS --> Edge Certificates --> *.db.neurolibre.org (Type Advanced) should be active.
  * SSL/TLS encryption mode must be set to FULL 
  * `A record` added for the floating IP address (*.db.neurolibre.org 206.xxx.xx.xx) must be proxied (orange cloud)

* Install letsencrypt plugin:
```
sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

* Create a DNS edit token for zone neurolibre.org
  * Login to cloudflare
  * My Profile --> API Tokens --> Create Token
  * Next to  `Edit Zone DNS`, click `use this template`
  * First row: Zone - DNS - Edit
  * Second row: Include - Specific zone - neurolibre.org
  * Continue to summary, create token and note it down for the next step

Note: You can check [supported lego providers](https://go-acme.github.io/lego/dns/cloudflare/) for up-to-date environment variable names. Note that dokku letsencrypt plugin does not support `_FILE` suffixes to read values from a file. If you are living up to your parents' societal expectations, your lego provider probably refers to a real logo store where you buy some interlocking plastic bricks to entertain your kids.

* Configure the plugin:
  *  `dokku letsencrypt:set --global email conp.dev@gmail.com`
  *  `dokku letsencrypt:set --global dns-provider cloudflare`
  *  `dokku letsencrypt:set --global dns-provider-CLOUDFLARE_DNS_API_TOKEN <add-dns-api-token-here>`

### Deploy a dashboard

* Now we are ready to deploy applications (PHEW). Clone a compatible repository, e.g., `my-dashboard` and cd into it: 

```
cd ~/my-dashboard
dokku apps:create my-dashboard
```

> If your app needs, you'll need to create [service plugins](https://dokku.com/docs/community/plugins/#official-plugins-beta) at this step.

* Add git remote for your application: 

```
git remote add dokku dokku@[vm.floating.ip]:my-dashboard
```

* Deploy the application 

```
git push dokku <reference_branch>:master
```

Replace the `<reference_branch>` with the branch from which you want to deploy the app.

* The push command above will start the deployment. If the VHOST is not enabled by default, you may not see URL being printed at the end of the deployment. In either case, add domain for the app: 

```
dokku domains:add my-dashboard db.neurolibre.org
```

Confirm that it is enabled with: 

```
dokku domains:report my-dashboard
```

enable otherwise: 

```
dokku domains:enable my-dashboard
```

* Add certificates to the application: 

```
dokku letsencrypt:enable my-dashboard
```

* That's it! If successful, the app should be live on https://my-dashboard.db.neurolibre.org 

* Check all the resources to see if things are in order:

```
dokku report my-dashboard
```

> Needless to say, `my-dashboard` here is just an example name. The repository you'll clone should have basic requirements (e.g., source code, a procfile to indicate what to execute and runtime dependency declarations such as requirements.txt, Gemfile, package.json, pom.xml, etc.) to deploy itself as an application to dokku.

### Troubleshooting

* If the application appears to be running on the VM, but now accessible via web
  * Ensure that a proper `Security Group` is added to the instance to allow outside connections
  * Click on the instance --> Interfaces --> Under the Actions Tab --> Edit Security Groups
  * Move a suitable security group from the left panel (all) to the right panel (port)

* If there's not a security group available
    * Network --> Security Groups --> Create

| Direction | Ether Type | Protocol | Port Range  | Remote IP Prefix |
|-----------|------------|----------|-------------|------------------|
| Egress    | IPv4       | Any      | Any         | 0.0.0.0/0        |
| Egress    | IPv6       | Any      | Any         | ::/0             |
| Ingress   | IPv4       | ICMP     | Any         | -                |
| Ingress   | IPv4       | ICMP     | Any         | 192.168.73.30/32 |
| Ingress   | IPv4       | TCP      | Any         | 192.168.73.30/32 |
| Ingress   | IPv4       | TCP      | 1 - 65535   | -                |
| Ingress   | IPv4       | TCP      | 1 - 65535   | 192.168.73.30/32 |
| Ingress   | IPv4       | TCP      | 22 (SSH)    | 0.0.0.0/0        |
| Ingress   | IPv4       | TCP      | 80 (HTTP)   | 0.0.0.0/0        |
| Ingress   | IPv4       | TCP      | 443 (HTTPS) | 0.0.0.0/0        |
| Ingress   | IPv4       | UDP      | 1 - 65535   | -                |
| Ingress   | IPv4       | UDP      | 1 - 65535   | 192.168.73.30/32 |

Note: This is a pretty loose security group example. Depending on the type of connections you expect to the instance, please revise before applying them. 

* Each application on Dokku will run in a container, named as "dynos" in Heroku. If you connect this VM to NewRelic (see instructions above), you can monitor each container/application/load and set alert conditions.

* Permanent redirect from `*.dashboards.neurolibre.org` to `*.db.neurolibre.org`
  * Cloudflare --> Rules --> Page Rules 
  * `URL`: *.dashboards.neurolibre.org/
  * Forwarding URL & 301 Permanent redirect 
  * `Destination URL`: https://$1.db.neurolibre.org