# AutoDQM
AutoDQM parses DQM histograms and identifies outliers by various statistical tests for further analysis by the user. Its output can be easily parsed by eye on an AutoPlotter-based html page which is automatically generated when you submit a query from the AutoDQM GUI. Full documentation for AutoDQM can be found on our [wiki](http://github.com/jkguiang/AutoDQM/wiki).

1. [Features](#features)

## Features

###### AutoDQM.py
- [x] Outputs histograms that clearly highlight outliers
- [x] Creates a .txt file along with each .pdf with relevant information on it
- [x] Allows user to easily change input
- [x] Seeks and accurately finds outliers

###### index.php
- [x] Previews input in a readable way
- [x] Gives a clear indication of the status of a user's query 

###### plots.php
- [x] Dynamically displays text files below AutoPlotter toolbar
- [x] Unique url's for sharing plots pages with the data and reference data set names

## Using AutoDQM

### Setup:

This shows how to set up AutoDQM to be served from your local machine or a machine on CERN OpenStack. This was written based on a fresh CC7 VM on CERN OpenStack. 

You'll need a CERN User/Host certificate authorized with the CMS VO. CMS VO authorization can take ~8 hours so bear that in mind. Certificates can be aquired either from https://cern.ch/ca or, on a CC7 machine, by using auto-enrollment https://ca.cern.ch/ca/Help/?kbid=024000.

Install docker according to https://docs.docker.com/install/ and docker-compose through pip because CC7 has an old versions in it's repositories.
Enable+start the docker service, and be sure to add your user to the docker group.
```sh
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce -y
sudo yum install python-pip -y
sudo pip install docker-compose
```
```sh
sudo gpasswd -a [user] docker
sudo systemctl enable --now docker
```

You may need to relog into your account before the group settings take effect.

Store your CERN certificate into docker secrets. You may need to extract your cert from PKCS12 format:
```sh
openssl pkcs12 -in cern-cert.p12 -out cern-cert.public.pem -clcerts -nokeys
openssl pkcs12 -in cern-cert.p12 -out cern-cert.private.key -nocerts -nodes
```
```sh
docker swarm init
docker secret create cmsvo-cert.pem cern-cert.public.pem
docker secret create cmsvo-cert.key cern-cert.private.key 
```
Then initialize a docker swarm, build the autodqm image with docker-compose, and deploy the image as a docker stack
```sh
docker-compose build
docker stack deploy --compose-file=./docker-compose.yml autodqm
```

You can now view AutoDQM at `http://127.0.0.1`. If you would like to make your instance of AutoDQM public, oopen port 80 to http traffic on your firewall. For example, on CC7:
```sh
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload
```

After making changes to configuration or source code, rebuild and redeploy the newly built image:
```sh
docker-compose build
docker stack rm autodqm
docker stack deploy --compose-file=./docker-compose.yml autodqm
```

## Using AutoDQM Offline

The `./run-offline.py` script can retrieve run data files and process them without needing a web server. Run `./run-offline.py --help` for all the options.

1. Supply certificate files to the environment variables below. Alternatively, the default uses the files produced when running voms-proxy-init, so that may work instead.
2. Use `./run-offline.py` to process data with AutoDQM

## Environment Variables

- `ADQM_CONFIG` location of the configuration file to use
- `ADQM_DB` location to store downloaded root files from offline DQM
- `ADQM_TMP` location to store generated temporary pdfs, pngs, etc
- `ADQM_SSLCERT` location of CMS VO authorized public key certificate to use in querying offline DQM
- `ADQM_SSLKEY` location of CMS VO authorized private ky to use in querying offline DQM
- `ADQM_CACERT` location of a CERN Grid CA certificate chain, if needed

### AutoDQM GUI
###### Main Page:
1. Select series and sample
2. Enter data and reference information
3. Click submit.
    - After process is complete, you will be automatically redirected to the "plots" page
