# ostankin-hosting
Scripts for managing my hosting on ostankin.net.

The blog is hosted on a Drupal + MySQL docker container pair. Both CMS and database are persistent and stored in `.data/cms` and `.data/db` directories respectively. Hosting files are located in `.data/files`.

## Prerequisites

The setup requires Docker and Docker Compose to work
```
sudo apt-get install docker docker.io python-pip
pip install docker-compose
```

## Restoring from backup

It is assumed that there is a `backup/latest` directory with the following two files:
* `myx_db.sql.gz` - gzipped database dump (prefix not used)
* `myx_files.tar.gz` - archived (with -p key!) CMS files

Running 'restore-from-backup' does the following:
* Stops containers hosting the blog (if they are running)
* Unpacks the backup files and prepares them for deployment
* Wipes `.data` directory, thus destroying all current data
* Deploys the backup data
* Generates new database passwords and updates the CMS config
* Creates and starts new containers
