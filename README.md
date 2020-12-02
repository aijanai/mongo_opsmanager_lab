# Ops Manager bootstrap

Provisions Ops Manager (from scratch or from existing setup) and its agent onboard of nodes.

Full manual at: https://docs.google.com/document/d/1MoAGrNox3NRJu4dUhUhY8jWEUr_UFQ8mHbTGCejMhic/edit?usp=sharing


This repo includes a Vagrant lab for demo purposes:

- 1 Ops Manager server;
- 3 machine RS for Ops Manger DB;
- various machines to build test RS and Shards.

## Getting started

### Vagrant plugin requirements

* `vagrant-proxyconf` allows guests to go to the Internet, necessary in heavily proxied environments like Poste.
* `vagrant-hostmanager` manages the hosts file on guest machines ((reference)[https://github.com/devopsgroup-io/vagrant-hostmanager]).
* `vagrant-hostsupdater` adds an entry to your /etc/hosts file on the host system.
* `vagrant-scp` is nice to copy stuff from/to guests.

```
vagrant plugin install vagrant-hostmanager
vagrant plugin install vagrant-hostsupdater
vagrant plugin install vagrant-proxyconf
vagrant plugin install vagrant-scp
```

### Building the lab
Bring up infrastructure:
```
cd demo_infrastructure
vagrant up # watch out for sudo password requests
cd ..
```
Run `vagrant hostmanager` to update guests' `hosts` files.

Make sure to add hostname mapping in `/etc/hosts` before continuing since hosts need mutual visibility by name. `vagrant-hostmanager` helps in this.

Now configure Ops Manger server and its RS:
```
cd ansible
sudo ansible-playbook -i mongo.hosts mongo_ops.yml
```

Wait for Ops Manager to come up, takes some minutes due to first run schema initializations. You can `tail -f /opt/mongodb/mms/logs/*log` for updates.

Then, surf to `http://opsmanager:8080/`, add first user and configure Ops Manager. Refer to [official guide](https://docs.opsmanager.mongodb.com/current/tutorial/nav/install-application/).

After the server has been correctly initialized, create a new managed cluster:
1. Generate API key and take note of mms base url and cluster id.
2. Fill these info in `ansible/mongo_agent.yml`.
3. Run `ansible-playbook -i mongo.hosts mongo_agent.yml`

Verify the agents are started on the web UI.
