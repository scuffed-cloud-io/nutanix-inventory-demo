# How To 

- Create a git project
  * Create collections/requirements.yml

`collections/requirements.yml`

```yaml
---
collections:
- name: nutanix.ncp.ntnx_prism_vm_inventory
Create inventory.yml
```

`inventory.yml`
```yaml
---
plugin: nutanix.ncp.ntnx_prism_vm_inventory
```

- commit and push this, you don't need more in here
- Create a custom credential type in Controller/Tower
  * Administration -> Credential Types -> Add
  * Define the ENV var fields documented from the nutanix plugin docs

`Input Configuration: (YAML)`
```yaml
fields:
  - id: nutanix_hostname
    type: string
    label: Nutanix Hostname
  - id: nutanix_port
    type: string
    label: Nutanix Port (Optional)
  - id: nutanix_username
    type: string
    label: Nutanix Username
  - id: nutanix_password
    type: string
    label: Nutanix Password
    secret: true
required:
  - nutanix_hostname
  - nutanix_username
  - nutanix_password
```

`Injector Configuration: (YAML)`
```yaml
env:
  NUTANIX_PORT: '{{ nutanix_port }}'
  NUTANIX_HOSTNAME: '{{ nutanix_hostname }}'
  NUTANIX_PASSWORD: '{{ nutanix_password }}'
  NUTANIX_USERNAME: '{{ nutanix_username }}'
```

- Create the Nutanix credential, now that you've defined the type
  * Resources -> Credentials -> Add -> Name it whatever, in the Credential Type dropdown, choose the one you just made and defined above
  * This is how we get the secret stuff like hostname/username/password to be secret/secure and inject into the plugin using ENV vars instead of cleartext typing it in the inventory.yml which is bad.
- Create the Project (where we will get the inventory.yml)
  * Resources -> Projects -> Add
  * Configure it to use the git repo you put the basic inventory.yml in
- Create the Inventory
  * Resources -> Inventories -> Add -> Add inventory
  * Name it whatever, pick an org it belongs to, don't need to fill out anything else since the magic is in the next part
- Add the Inventory Source to the Inventory
  * Inventory you made -> Sources tab
  * Add
  * Name it whatever, I called it Nutanix Plugin
  * Source: Sourced from a Project
  * Plug in your Credential you made, the Project you made, and in the Inventory file box, type inventory.yml. For whatever reason, the dropdown box is useless here. just type it and leave it be.
  * Enable Update on launch and Overwrite and save this source
  * Sync the inventory source


## Disconnected: Install the collection into an EE

- Log into `redhat.registry.io` with podman

```bash
podman login registry.redhat.io -u <Red Hat Username> -p <Red Hat Password>
```

- Run ansible-builder
```bash
ansible-builder build -f execution-environment.yml -t localhost/ansible-rhel9-ee:nutanix-inventory
```


- Save the container image as an exported oci-archive
```bash
podman save --format oci-archive -o ansible-rhel9-ee-nutanix-inventory.tar localhost/ansible-rhel9-ee:nutanix-inventory 
```

Write that container image file to disk/disc and bring it into your environment and add it to your image registry, such as Automation Hub.

```bash
# Load file into local cache
podman load -i ansible-rhel9-ee-nutanix-inventory.tar

# Auth to disconnected network's registry
podman login https://hub.yourdomain.internal:yourport

# Tag and Push up the image
podman tag localhost/ansible-rhel9-ee:nutanix-inventory hub.yourdomain.internal:yourport/ansible-rhel9-ee:nutanix-inventory
podman push hub.yourdomain.internal:yourport/ansible-rhel9-ee:nutanix-inventory
```
