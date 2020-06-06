# Components

 - **Multi-Juicer** (https://github.com/iteratec/multi-juicer)
   - Multi-user OWASP Juice Shop (https://github.com/bkimminich/juice-shop)
 - **RootTheBox** for scoreboard

# Installing

## Server

This will require a fully working kubernetes server, which is well beyond the scope of this document.
It can easily run on a single-node server, with enough resources.

For resources, I would recommend:
 - At least 2 CPU, plus at least 0.25 CPU for each player
 - At least 1.5GB RAM, plus 200MB RAM for each player
 - Negligable storage space, over-and-above kubernetes requirements. 10GB is lots.

This setup requires a kubernetes ingress controller to exist on your kubernetes server.

Your user must have access to `kubectl` and `docker` commands on the server. Docker access through `sudo` is fine, as long as you have the ability to run Docker commands.

## Set up Scoreboard
We're using **RootTheBox** for a scoreboard, which is a bit of a challenge to set up in Kubernetes.
https://github.com/moloch--/RootTheBox

### Create a RootTheBox docker image
```
git clone https://github.com/moloch--/RootTheBox
cd RootTheBox
sudo docker build -t rootthebox:latest .
```

The tag of this docker image must match the image name in /scoreboard/kube-deployment/rootthebox.yml.


### Create a kubernetes deployment
Edit /scoreboard/kube-deployment/rootthebox.yml with the appropriate domain name (which you should point at your kubernetes server).

Also edit the ConfigMap section with any settings you'd like to apply. 
Help with configuration options can be found here: https://github.com/moloch--/RootTheBox/wiki/Configuration-File-Details

```
kubectl apply -f rootthebox.yml
```

This configuration does not save anything outside of the container, so if your container restarts, all progress and data will be lost. Setting up a volume in kubernetes can get a bit complicated, and is beyond the scope of this readme file.


## Set up Multi-Juicer
More detailed instructions can be found on their GitHub: https://github.com/iteratec/multi-juicer

You will need helm: https://helm.sh/

```
helm repo add multi-juicer https://iteratec.github.io/multi-juicer/
```
```
git clone https://github.com/iteratec/multi-juicer.git
```

Edit ./multi-juicer/helm/multi-juicer/values.yaml (or use the one from this repo).

**Things you should edit**: Change the ctfKey value before deploying. This will change the flag values so that they are unique to your event.

When it's configured to your satisfaction, install multi-juicer:

```
helm install -f ./multi-juicer/helm/multi-juicer/values.yaml multi-juicer ./multi-juicer/helm/multi-juicer/
```

### Deleting multi-juicer
To restart the multi-juicer environment, delete it, then re-install it:
```
helm delete multi-juicer
```

### Multi-Juicer admin

You may need to restart or delete instances of the Juice Shop, and for that you'll need to join the "admin" team.
The password for this team is random each time you deploy Multi-Juicer.
You can find the password by ssh'ing to the server and running the following command (with a user that has kubectl access)

```
kubectl get secrets juice-balancer-secret -o=jsonpath='{.data.adminPassword}' | base64 --decode
```

# Configuring for a CTF

Make sure you've customized the ctfKey in the Multi-Juicer config, so that yours is unique.

## Generate a Juice Shop CTF export file for the scoreboard

https://www.npmjs.com/package/juice-shop-ctf-cli
https://github.com/bkimminich/juice-shop-ctf

This does not need to be done on the server.

```
sudo apt install npm
sudo npm install -g juice-shop-ctf-cli
juice-shop-ctf
```

Follow the prompts.

The script reads the challenges from your JuiceShop server, so don't give it a Multi-Juicer url, or the script will fail. Use the default (https://juice-shop.herokuapp.com), or point it to a standalone instance of Juice Shop somewhere else. The important thing here is that your ctfKey matches the one in your config file.

Retrieve the file that this script makes, and import it into RootTheBox.
https://pwning.owasp-juice.shop/part1/ctf.html#running-rootthebox

**THIS FILE CONTAINS THE FLAGS IN PLAINTEXT SO BE CAREFUL WHERE YOU STORE IT.**

## Change the admin password on the scoreboard server

By default, the admin credentials are `admin/rootthebox`. You'll want to change this to something unique.
