# Install PowerDNS on Ubuntu 18.04, 20.04, & 22.04 | phoenixNAP KB

*   Install PowerDNS on Ubuntu 18.04, 20.04, & 22.04 | phoenixNAP KB
*   Why Use PowerDNS?
*   Installing PowerDNS on Ubuntu 18.04, 20.04, & 22.04
*   Step 1: Install and Configure MariaDB Server
*   Step 2: Install PowerDNS
*   Step 3: Configure PowerDNS
*   Step 4: Install PowerDNS Admin Dependencies
*   Step 5: Configure and Run PowerDNS Admin
*   Step 6: Create PowerDNS Admin Service
*   Step 7: Install and Configure Nginx
*   Step 8: Configure PowerDNS API

##### Links

*   [high availability](https://phoenixnap.com/blog/what-is-high-availability)
*   [data redundancy](https://phoenixnap.com/glossary/data-redundancy)
*   [install and configure MariaDB](https://phoenixnap.com/kb/how-to-install-mariadb-ubuntu)
*   [database servers](https://phoenixnap.com/kb/what-is-a-database-server)
*   [Install Node.js](https://phoenixnap.com/kb/install-latest-node-js-and-nmp-on-ubuntu)
*   [Git repository](https://phoenixnap.com/kb/what-is-a-git-repository)
*   [Install Nginx](https://phoenixnap.com/kb/install-nginx-on-ubuntu)
*   [DNS record types](https://phoenixnap.com/kb/dns-record-types)
*   [DNS security best practices](https://phoenixnap.com/kb/dns-best-practices-security)

##### Marks

Introduction

PowerDNS is an open-source [DNS](https://phoenixnap.com/kb/what-is-domain-name-system) server solution that helps resolve namespaces. PowerDNS supports [high availability](https://phoenixnap.com/blog/what-is-high-availability), [data redundancy](https://phoenixnap.com/glossary/data-redundancy), and various backends, which makes it a flexible and robust solution.

**This guide shows how to install PowerDNS and the PowerDNS Admin interface on Ubuntu.**

![Install PowerDNS on Ubuntu](https://phoenixnap.com/kb/wp-content/uploads/2022/09/install-powerdns-on-ubuntu.png)

Prerequisites

*   Access to the terminal.
*   Access to the root user.
*   A text editor, such as [nano](https://phoenixnap.com/kb/use-nano-text-editor-commands-linux).
*   A web browser to access PowerDNS Admin.

Why Use PowerDNS?
-----------------

PowerDNS provides two nameserver solutions:

*   The **Authoritative Server**, which uses the database to resolve queries about domains.
*   The **Recursor**, which consults with other authoritative servers to resolve queries.

Other nameservers combine the two functions automatically. PowerDNS offers them separately, and allows the mix of the two solutions seamlessly for a modular setup.

Additionally, PowerDNS is open source, works equally well for small and large query volumes, and offers many possibilities for backend solutions.

Installing PowerDNS on Ubuntu 18.04, 20.04, & 22.04
---------------------------------------------------

Follow the steps below to install and configure PowerDNS with the MariaDB server as a backend [database](https://phoenixnap.com/kb/what-is-a-database). Additionally, the steps guide users through the setup of the PowerDNS Admin web interface and API.

### Step 1: Install and Configure MariaDB Server

To [install and configure MariaDB](https://phoenixnap.com/kb/how-to-install-mariadb-ubuntu), do the following:

1\. Update and upgrade system packages:

sudo apt update && sudo apt upgrade

2\. Install the MariaDB server and client with:

sudo apt install mariadb-server mariadb-client

**Note:** Other possible [database servers](https://phoenixnap.com/kb/what-is-a-database-server) include PostgreSQL, MySQL, and other relational databases.

Wait for the installation to finish before continuing.

3\. Connect to MariaDB with:

sudo mysql

![sudo mysql MariaDB](https://phoenixnap.com/kb/wp-content/uploads/2022/09/sudo-mysql-mariadb.png)

The terminal connects to a database session.

4\. Create a database for the PowerDNS nameserver:

create database pda;

**Note:** If using a different database name, change all consequent commands accordingly.

5\. Grant all privileges to the **`pda`** user and provide the user password:

grant all privileges on pda.\* TO 'pda'@'localhost' identified by 'YOUR\_PASSWORD\_HERE';
flush privileges;

6\. Connect to the database:

use pda;

![use pda mysql output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/use-pda-mysql-output.png)

7\. Use the following SQL queries to create tables for the _pda_ database:

CREATE TABLE domains (
  id                    INT AUTO\_INCREMENT,
  name                  VARCHAR(255) NOT NULL,
  master                VARCHAR(128) DEFAULT NULL,
  last\_check            INT DEFAULT NULL,
  type                  VARCHAR(6) NOT NULL,
  notified\_serial       INT UNSIGNED DEFAULT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  PRIMARY KEY (id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE UNIQUE INDEX name\_index ON domains(name);

CREATE TABLE records (
  id                    BIGINT AUTO\_INCREMENT,
  domain\_id             INT DEFAULT NULL,
  name                  VARCHAR(255) DEFAULT NULL,
  type                  VARCHAR(10) DEFAULT NULL,
  content               VARCHAR(64000) DEFAULT NULL,
  ttl                   INT DEFAULT NULL,
  prio                  INT DEFAULT NULL,
  change\_date           INT DEFAULT NULL,
  disabled              TINYINT(1) DEFAULT 0,
  ordername             VARCHAR(255) BINARY DEFAULT NULL,
  auth                  TINYINT(1) DEFAULT 1,
  PRIMARY KEY (id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE INDEX nametype\_index ON records(name,type);
CREATE INDEX domain\_id ON records(domain\_id);
CREATE INDEX ordername ON records (ordername);

CREATE TABLE supermasters (
  ip                    VARCHAR(64) NOT NULL,
  nameserver            VARCHAR(255) NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (ip, nameserver)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE TABLE comments (
  id                    INT AUTO\_INCREMENT,
  domain\_id             INT NOT NULL,
  name                  VARCHAR(255) NOT NULL,
  type                  VARCHAR(10) NOT NULL,
  modified\_at           INT NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  comment               TEXT CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE INDEX comments\_name\_type\_idx ON comments (name, type);
CREATE INDEX comments\_order\_idx ON comments (domain\_id, modified\_at);

CREATE TABLE domainmetadata (
  id                    INT AUTO\_INCREMENT,
  domain\_id             INT NOT NULL,
  kind                  VARCHAR(32),
  content               TEXT,
  PRIMARY KEY (id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE INDEX domainmetadata\_idx ON domainmetadata (domain\_id, kind);

CREATE TABLE cryptokeys (
  id                    INT AUTO\_INCREMENT,
  domain\_id             INT NOT NULL,
  flags                 INT NOT NULL,
  active                BOOL,
  content               TEXT,
  PRIMARY KEY(id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE INDEX domainidindex ON cryptokeys(domain\_id);

CREATE TABLE tsigkeys (
  id                    INT AUTO\_INCREMENT,
  name                  VARCHAR(255),
  algorithm             VARCHAR(50),
  secret                VARCHAR(255),
  PRIMARY KEY (id)
) Engine\=InnoDB CHARACTER SET 'latin1';

CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);

8\. Confirm the tables have been created with:

show tables;

![MariaDB show tables pda database output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/mariadb-show-tables-pda-database-output.png)

The output lists the available tables.

9\. Exit the database connection:

exit;

![database exit terminal return](https://phoenixnap.com/kb/wp-content/uploads/2022/09/database-exit-terminal-return.png)

The command returns the session to the terminal.

### Step 2: Install PowerDNS

To install PowerDNS on Ubuntu, do the following:

1\. Switch to the root user:

sudo su -

![sudo su - terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/sudo-su-terminal-output.png)

The terminal session changes to the root user.

2\. The **`systemd-resolved`** service provides the name resolutions to local applications. PowerDNS uses its own service for name resolutions.

Disable the **`systemd-resolved`** service with:

systemctl disable --now systemd-resolved

![systemctl disable systemd-resolved terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/systemctl-disable-systemd-resolved-terminal-output.png)

The output confirms the service removal.

3\. Delete the system service configuration file with:

rm -rf /etc/resolv.conf

4\. Create the new _resolv.conf_ file:

echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

![new resolv.conf file google nameserver](https://phoenixnap.com/kb/wp-content/uploads/2022/09/new-resolv.conf-file-google-nameserver.png)

Appending the Google nameserver ensures DNS resolution.

5\. Install the PowerDNS server and database backend packages with:

apt-get install pdns-server pdns-backend-mysql -y

Wait for the installation to complete before continuing.

### Step 3: Configure PowerDNS

Configure the local PowerDNS file to connect to the database:

1\. Open the configuration file for editing:

nano /etc/powerdns/pdns.d/pdns.local.gmysql.conf

2\. Add the following information to the file:

\# MySQL Configuration
#
\# Launch gmysql backend
launch+=gmysql

\# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=pda
gmysql-user=pda
gmysql-password=YOUR\_PASSWORD\_HERE
gmysql-dnssec=yes
\# gmysql-socket=

![pdns.local.gmysql.conf file contents](https://phoenixnap.com/kb/wp-content/uploads/2022/09/pdns.local_.gmysql.conf-file-contents.png)

Exchange the database name, user, and password with the correct parameters if using different ones. Save and close the file.

3\. Change the file permissions:

chmod 777 /etc/powerdns/pdns.d/pdns.local.gmysql.conf

4\. Stop the _**pdns**_ service:

systemctl stop pdns

5\. Test the connection to the database:

pdns\_server --daemon=no --guardian=no --loglevel=9

![pdns_server database connection test](https://phoenixnap.com/kb/wp-content/uploads/2022/09/pdns_server-database-connection-test.png)

The output shows a successful connection. Press **CTRL**+**C** to exit the test.

6\. Start the service:

systemctl start pdns

7\. Check the connection with the [ss command](https://phoenixnap.com/kb/ss-command):

ss -alnp4 | grep pdns

![ss pdns listening port 53](https://phoenixnap.com/kb/wp-content/uploads/2022/09/ss-pdns-listening-port-53.png)

Verify the [TCP](https://phoenixnap.com/glossary/transmission-control-protocol-tcp)/[UDP](https://phoenixnap.com/glossary/what-is-udp) port **`53`** is open and in **`LISTEN`**/**`UCONN`** state.

### Step 4: Install PowerDNS Admin Dependencies

The PowerDNS Admin helps manage PowerDNS through a web interface. To install the dashboard locally, do the following:

1\. Install the Python development package:

apt install python3-dev

2\. Install dependencies:

apt install -y git libmysqlclient-dev libsasl2-dev libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev libffi-dev pkg-config apt-transport-https python3-venv build-essential curl

3\. Fetch the Node.js setup:

curl -sL https://deb.nodesource.com/setup\_14.x | sudo bash -

![nodejs setup fetch terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/nodejs-setup-fetch-terminal-output.png)

4\. [Install Node.js](https://phoenixnap.com/kb/install-latest-node-js-and-nmp-on-ubuntu) with:

apt install -y nodejs

5\. Next, install the Yarn package manager. Fetch the Yarn public key and add it to **apt**:

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -

![add yarn key terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/add-yarn-key-terminal-output.png)

Yarn helps build the asset files.

6\. Add Yarn to the list of sources:

echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

7\. Update the apt sources list:

apt update \-y

8\. [Install Yarn](https://phoenixnap.com/kb/how-to-install-yarn-ubuntu) with:

apt install yarn -y

9\. Clone the PowerDNS Admin [Git repository](https://phoenixnap.com/kb/what-is-a-git-repository) to _/opt/web/powerdns-admin_:

git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /opt/web/powerdns-admin

![git clone powerdns admin repository terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/git-clone-powerdns-admin-repository-terminal-output.png)

If using a different directory, exchange the destination directory in the command and in all subsequent appearances.

10\. Navigate to the cloned Git directory:

cd /opt/web/powerdns-admin

11\. Create a Python virtual environment:

python3 -mvenv ./venv

12\. Activate the virtual environment with:

source ./venv/bin/activate

![python venv activate](https://phoenixnap.com/kb/wp-content/uploads/2022/09/python-venv-activate.png)

13\. Upgrade pip to the latest version:

pip install \--upgrade pip

The [pip](https://phoenixnap.com/kb/how-to-install-pip-on-ubuntu) package manager helps install additional Python requirements.

14\. Install the requirements from the _requirements.txt_ file:

pip install -r requirements.txt

![pip install requirements powerdns admin](https://phoenixnap.com/kb/wp-content/uploads/2022/09/pip-install-requirements-powerdns-admin.png)

After installing all the requirements, the PowerDNS Admin requires additional configuration before running.

### Step 5: Configure and Run PowerDNS Admin

To configure and start PowerDNS Admin on a local instance, do the following:

1\. Use the [cp command](https://phoenixnap.com/kb/how-to-copy-files-directories-linux) to copy the example _development.py_ Python file to _production.py_:

cp /opt/web/powerdns-admin/configs/development.py /opt/web/powerdns-admin/configs/production.py

2\. Open the _production.py_ file for editing:

nano /opt/web/powerdns-admin/configs/production.py

3\. Edit the following lines:

#import urllib.parse

SECRET\_KEY = 'e951e5a1f4b94151b360f47edf596dd2'

SQLA\_DB\_PASSWORD = 'changeme'

4\. Uncomment the library import, provide a randomly generated [secret key](https://phoenixnap.com/glossary/secret-key), and provide the correct database password.

import urllib.parse

SECRET\_KEY \= '\\x19\\xc7\\xd8\\xa7$\\xb6P\*\\xc6\\xb8\\xa1E\\x90P\\x12\\x95'

SQLA\_DB\_PASSWORD = 'YOUR\_PASSWORD\_HERE'

![production.py powerdns admin config contents](https://phoenixnap.com/kb/wp-content/uploads/2022/09/production.py-powerdns-admin-config-contents.png)

**Note:** Generate a random key using Python:

python3 -c "import os; print(os.urandom(16))"

Copy and paste the output into the **`SECRET_KEY`** value.

Save and close the file.

5\. Export the production app configuration variable:

export FLASK\_CONF=../configs/production.py

6\. Export the flask application variable:

export FLASK\_APP=powerdnsadmin/\_\_init\_\_.py

7\. Upgrade the database schema:

flask db upgrade

![flask db upgrade terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/flask-db-upgrade-terminal-output.png)

8\. Install project dependencies:

yarn install \--pure-lockfile

![yarn install --pure-lockfile terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/yarn-install-pure-lockfile-terminal-output.png)

9\. Build flask app assets:

flask assets build

![flask assets build terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/flask-assets-build-terminal-output.png)

Wait for the build to complete.

10\. Run the application with:

./run.py

![powerdns admin local run terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin-local-run-terminal-output.png)

Leave the application running.

11\. The application currently runs on localhost on port **`9191`**. Visit the following address:

http://localhost:9191

![powerdns admin login page local](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin-login-page-local.png)

The login screen for PowerDNS Admin shows. Currently, there are no users, and the first user you register shall be the administrator account.

12\. In the terminal, exit the virtual environment and log out of the root user with:

exit

The terminal returns to a regular state.

### Step 6: Create PowerDNS Admin Service

Configure PowerDNS Admin to run on startup:

1\. Create a systemd service file for PowerDNS Admin:

sudo nano /etc/systemd/system/powerdns-admin.service

2\. Add the following contents:

\[Unit\]
Description\=PowerDNS-Admin
Requires\=powerdns-admin.socket
After\=network.target

\[Service\]
User\=root
Group\=root
PIDFile\=/run/powerdns-admin/pid
WorkingDirectory\=/opt/web/powerdns-admin
ExecStartPre\=/bin/bash -c '$$(mkdir -p /run/powerdns-admin/)'
ExecStart\=/opt/web/powerdns-admin/venv/bin/gunicorn --pid /run/powerdns-admin/pid --bind unix:/run/powerdns-admin/socket 'powerdnsadmin:create\_app()'
ExecReload\=/bin/kill -s HUP $MAINPID
ExecStop\=/bin/kill -s TERM $MAINPID
PrivateTmp\=true

\[Install\]
WantedBy\=multi-user.target

![powerdns-admin.service file contents](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin.service-file-contents.png)

3\. Create a unit file:

sudo systemctl edit \--force powerdns-admin.service

4\. Append the following:

\[Service\]
Environment\="FLASK\_CONF=../configs/production.py"

![powerdns-admin.service.d override contents](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin.service.d-override-contents.png)

5\. Create a [socket](https://phoenixnap.com/glossary/what-is-a-socket) file:

sudo nano /etc/systemd/system/powerdns-admin.socket

6\. Insert the following information:

\[Unit\]
Description\=PowerDNS-Admin socket

\[Socket\]
ListenStream\=/run/powerdns-admin/socket

\[Install\]
WantedBy\=sockets.target

![powerdns-admin.socket contents](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin.socket-contents.png)

7\. Create an environment file:

sudo nano /etc/tmpfiles.d/powerdns-admin.conf

8\. Add the following information:

d /run/powerdns-admin 0755 pdns pdns -

9\. Reload the daemon:

sudo systemctl daemon-reload

10\. Start and enable the service and socket:

sudo systemctl start powerdns\-admin.service powerdns\-admin.socket

sudo systemctl enable powerdns-admin.service powerdns-admin.socket

![enable powerdns-admin service and socket terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/enable-powerdns-admin-service-and-socket-terminal-output.png)

11\. Check the status with:

sudo systemctl status powerdns-admin.service powerdns-admin.socket

![powerdns-admin service and socket status](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin-service-and-socket-status.png)

The services show as running without any errors.

### Step 7: Install and Configure Nginx

To configure PowerDNS Admin to run on Nginx, do the following:

1\. [Install Nginx](https://phoenixnap.com/kb/install-nginx-on-ubuntu) with:

sudo apt install nginx -y

2\. Edit the Nginx configuration file:

sudo nano /etc/nginx/conf.d/pdns-admin.conf

3\. Add the following information:

server {
  listen \*:80;
  server\_name               localhost;

  index                     index.html index.htm index.php;
  root                      /opt/web/powerdns-admin;
  access\_log                /var/log/nginx/powerdns-admin.local.access.log combined;
  error\_log                 /var/log/nginx/powerdns-admin.local.error.log;

  client\_max\_body\_size              10m;
  client\_body\_buffer\_size           128k;
  proxy\_redirect                    off;
  proxy\_connect\_timeout             90;
  proxy\_send\_timeout                90;
  proxy\_read\_timeout                90;
  proxy\_buffers                     32 4k;
  proxy\_buffer\_size                 8k;
  proxy\_set\_header                  Host $host;
  proxy\_set\_header                  X-Real-IP $remote\_addr;
  proxy\_set\_header                  X-Forwarded-For $proxy\_add\_x\_forwarded\_for;
  proxy\_headers\_hash\_bucket\_size    64;

  location ~ ^/static/  {
    include  /etc/nginx/mime.types;
    root /opt/web/powerdns-admin/powerdnsadmin;

    location ~\*  \\.(jpg|jpeg|png|gif)$ {
      expires 365d;
    }

    location ~\* ^.+.(css|js)$ {
      expires 7d;
    }
  }

  location / {
    proxy\_pass            http://unix:/run/powerdns-admin/socket;
    proxy\_read\_timeout    120;
    proxy\_connect\_timeout 120;
    proxy\_redirect        off;
  }

}

If using a different server name, change **`localhost`** to your server address.

4\. Confirm the file has no syntax errors:

nginx -t

![sudo nginx -t terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/09/sudo-nginx-t-terminal-output.png)

5\. Change the ownership of **`powerdns-admin`** to **`www-data`**:

sudo chown -R www-data:www-data /opt/web/powerdns-admin

6\. Restart the Nginx service:

sudo systemctl restart nginx

7\. Access the PowerDNS admin page through the [browser](https://phoenixnap.com/glossary/web-browser-definition):

localhost

If linking to a different address, use the address provided in the Nginx configuration file.

### Step 8: Configure PowerDNS API

To configure the PowerDNS API, do the following:

1\. Log into the PowerDNS Admin via the browser. If running for the first time, create a new user first. The first user is automatically the administrator.

2\. Open the [API Keys](https://phoenixnap.com/glossary/api-key) tab on the left menu.

![PowerDNS Admin page menu API keys](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-admin-page-menu-api-keys.png)

3\. Click the **Add Key+** button.

![add API key UI PowerDNS Admin](https://phoenixnap.com/kb/wp-content/uploads/2022/09/add-api-key-ui-powerdns-admin.png)

4\. The _Role_ field defaults to the **Administrator** user. Add an optional description for the key.

5\. Click **Create Key** to generate an API key.

![create API key PowerDNS Admin UI](https://phoenixnap.com/kb/wp-content/uploads/2022/09/create-api-key-powerdns-admin-ui.png)

6\. A popup window prints the key. **Copy the key** and press **Confirm** to continue.

![api key generated powerdns admin ui](https://phoenixnap.com/kb/wp-content/uploads/2022/09/api-key-generated-powerdns-admin-ui.png)

7\. Navigate to the **Dashboard** page.

8\. Enter the domain and API key. Save the changes.

![PowerDNS pdns API settings UI](https://phoenixnap.com/kb/wp-content/uploads/2022/09/powerdns-pdns-api-settings-ui.png)

9\. Enable the API in the PowerDNS configuration. Open the configuration file in the terminal:

nano /etc/powerdns/pdns.conf

10\. Uncomment and change the following lines:

api\=yes
api-key\=yoursecretekey
webserver\=yes

11\. Save the changes and close nano. The API is set up and ready to use.

Conclusion

After going through the steps in this guide, you've set up PowerDNS, the PowerDNS Admin web interface on Nginx, and connected the PowerDNS API.

Next, learn about the different [DNS record types](https://phoenixnap.com/kb/dns-record-types) or [DNS security best practices](https://phoenixnap.com/kb/dns-best-practices-security).
