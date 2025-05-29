This is a PUBLIC repository, to have a basic multistore PrestaShop 8.2 setup for
testing purposes.

Uses PrestaShop git instead of the released zip archive to build, so its 
possible to test unreleased changes. However, the startup takes a
while since additional resources are downloaded and build, namely PHP packages,
npm modules and the webpack theme build.

In the docker compose file the IP is fixed and the DNS entries
`ps90.packingpanic.com` as well as `ps90.packingpanic.de` point to it. The
two domains will be used for a multistore setup. For other PS versions we
use different IPs so we can run the versions in parallel for comparisons.

## Setup

To avoid duplicated repository clones (the PrestaShop repository is 2.2GB),
its best to have a clone of the PrestaShop git repository at ../PrestaShop.

````
if test -d ../PrestaShop; then
  cp -a ../PrestaShop .
else
  git clone https://github.com/PrestaShop/PrestaShop.git
fi
( cd PrestaShop; git clean -xfd; git checkout 9.0.0-beta.1 )
````

Start PS with:

````
docker compose up -d
docker compose logs -f
````

After a while this should finish with:

````
prestashop-git-1  | 
prestashop-git-1  | ***
prestashop-git-1  | **
prestashop-git-1  | ** Front-office: http://ps90.packingpanic.com/
prestashop-git-1  | **  Back-office: http://ps90.packingpanic.com/admin-dev
prestashop-git-1  | **   Login with:
prestashop-git-1  | **     username: demo@prestashop.com
prestashop-git-1  | **     password: Correct Horse Battery Staple
prestashop-git-1  | **
prestashop-git-1  | ** To view sent emails point your browser to http://localhost:1080/
prestashop-git-1  | **
prestashop-git-1  | ***
prestashop-git-1  | 
prestashop-git-1  | 
prestashop-git-1  | * Starting web server now
prestashop-git-1  | 
prestashop-git-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
prestashop-git-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
prestashop-git-1  | [Thu May 29 01:38:23.023951 2025] [mpm_prefork:notice] [pid 1:tid 1] AH00163: Apache/2.4.62 (Debian) configured -- resuming normal operations
prestashop-git-1  | [Thu May 29 01:38:23.023984 2025] [core:notice] [pid 1:tid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
````

## Setup via admin UI

Login as admin at: http://ps90.packingpanic.com/admin-dev

Switch the theme to hummingbird

- Improve > Design > Theme & Logo > My theme .... > spot "Hummingbird" and   click Use this theme

Enable multistore by:

- Configure > Shop Parameters > General > Enable Multistore, set to Yes and save
- Advanced Parameters > Multistore > Add New Store
  - Enter name: "DE"
  - make sure theme "Hummingbird" is selected
  - keep everything else at default, that copies all data from the main store
- In the shop list click on: *Click here to set a URL for this shop.*
  - enter `ps90.pakcingpanic.de` as domain and SSL domain

Next, we enable only one language per shop.

- Improve > Localization > Languages > Click Edit on Language English > Store association, deselect DE and save
- Improve > Localization > Languages > Click Edit on Language German > Store association, deselect PrestaShop and save

## Redo build and installation

If you changed some settings for docker, you can redo the PrestaShop build
and installation. ATTENTION: this removes any changes you did within
PrestaShop.

This sequence only removes the database volume and keeps the other volumes
which are actiing as cache for the npm and composer artifact downloads.

```
docker compose down
docker volume rm $(basename $PWD | tr '[:upper:]' '[:lower:]')_db-data
( cd PrestaShop; git clean -xfd )
docker compose up -d
docker compose logs -f
```
