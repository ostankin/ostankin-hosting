# ostankin-hosting
Scripts for managing my hosting on ostankin.net.

Sites are routed via Nginx. Each site is configured in a separate file under `nginx.conf` directory. Each site may be:

* a static file tree, each located in a separate directory under `.data/files`;
* a CMS, represented by a CMS container, linked to MySQL database container.

Both CMS and database are persistent and stored in `.data/cms` and `.data/db` directories respectively.

## Prerequisites

The setup requires Docker and Docker Compose to work (as `root`):
```
apt-get install docker docker.io python-pip
pip install docker-compose
```

## Configuration

The following elements are responsible for hosting configuration:

* `backup` - a location for backup files;
* `config.d` - metadata for CMS sites;
* `nginx.conf` - set of Nginx config files for each site;
* `docker-compose.yml` - configuration of containers and their interdependencies.

## Restoring from backup

Static sites do not need any complex backup-restore mechanisms,
`rsync` is just enough to keep them safe. CMS'es need more attention.

There are two ways to restore a CMS-site:

* rebuild the whole hosting from scratch, using site backups;
* restore from backup a single site.

It is assumed that there is a `backup/latest` directory
with the following two files(for each site):

* `<site_name>_db.sql.gz` - gzipped database dump (prefix not used);
* `<site_name>_files.tar.gz` - archived (with -p key!) CMS files.

### Rebuilding the whole hosting from scratch

Running `./restore-from-backup -a` does the following:

1. Stops all containers (if they are running).
1. Unpacks the backup files and prepares them for deployment.
1. Wipes `.data` directory, thus destroying all current data.
1. Deploys the backup data.
1. Generates new database passwords and updates the CMS configs.
1. Starts new containers.
1. Restores the databases from backup and creates users to access them.

### Restoring a single CMS site

Running `./restore-from-backup <site_name>` does the following:

1. Wipes the existing CMS files.
1. Restores CMS files from backup.
1. Recreates the corresponding database.

Since passwords are not recreated, the script does not allow restoring from
backups, created before the whole rebuild was done (and new passwords were
generated).

## Creating a backup

Running `./create-backup` does the following:

1. Takes a snapshot of each CMS site database.
1. Creates a tarball from all CMS files for each site separately.
1. Puts backup files into `backup/latest` directory.

