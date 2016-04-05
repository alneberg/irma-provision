# Playbooks for deploying NGI pipeline and Piper. 

Ansible playbooks for production usage on Irma to be run on irma3. 

## Usage 

Run `source /lupus/ngi/.bashrc` to setup environment variables for the virtual Ansible environment, and e.g. shortcuts for launching the NGI environment sourcing the NGI configs already on irma3. 

When the environment variables have been loaded you should be able to issue the command `ansibleenv` to load the Python environment for Ansible. Now you should be able to navigate to `/lupus/ngi/playbooks` and deploy (i.e. download, setup/compile, and provision configs) required software under the `/lupus/ngi/` directory tree. 

In the simplest case this is done with the command `ansible-playbook deploy.yml`. 

Important configuration values are at the moment set in the file `host_vars/127.0.0.1` (global values), and `roles/piper/defaults/main.yml` (piper specific values). (A corresponding file is missing for the role `ngi_pipeline`, instead it is mostly using values from `host_vars`. This should probably be changed a little bit.) 

When the software has been setup locally under `/lupus/ngi/` on irma3 we can sync it over to the rest of irma with the appropriate command. 

## Bootstrapping the playbook environment 

The deployment environment was basically setup by using the file `bootstrap/bashrc` in this repo. The latest version of virtualenv was downloaded and then locally run to setup a virtual Python environment where Ansible was installed with `pip install ansible` under `/lupus/ngi/ansible-env`. Then anaconda was downloaded manually (TODO: do this in the playbook instead?) and installed via the interactive guide under `/lupus/ngi/sw/anaconda2` (the role `ngi_pipeline` later uses anaconda to setup the NGI virtual environment.)

At the moment there has also been manual steps involved when downloading the test data under `/lupus/ngi/test_data`, as well as the gatk-bundle under `/lupus/ngi/piper_resources/gatk_bundle`. There is also a manually copied version of the NGI db copied into `/lupus/ngi/db/`. These things should probably be streamlined/changed in the future.  

## Some operational dependencies

- Requires the data record SQL database to be setup. Using `/lupus/ngi//db//isaks_db.sql` at the moment. 
- Requires a working connection to `charon` or `charon-dev` with suitable API token. For now we're using a reverse SSH tunnel + header-rewriting HTTP proxy for reaching charon-dev. 
- Requires a valid GATK key placed under `roles/piper/files`, and that the filename is specified in the `gatk_key` variable in `host_vars/127.0.0.1`. 

Quick steps: 

On irma login node, make sure that we have a Caddyfile with appropriate proxy settings. Running the binary `caddy` will load the configs and start proxying traffic between the two port numbers mentioned in the config file. 

For this to we also have to have an SSH tunnel up and running from nestor to irma. Something like `ssh johanhe@irma1 -R *:4242:charon-dev.scilifelab.se:80` on nestor will portforward 4242 on irma1 to charon-dev:80.

## Note on re-compiling software

Need to remove the file that is specified in the task's `creates=` field. Otherwise the compile step will be ignored.  
