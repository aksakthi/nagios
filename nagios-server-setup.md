# Description: Nagios Core
- Nagios is an enterprise class, open source software that can be used for network and infrastructure monitoring. Using 
  Nagios, we can monitor servers, switches, applications and services etc. It alerts the System Administrator when 
  something goes wrong and also alerts back when the issues have been rectified.

### Features
* Monitor entire IT infrastructure.
* Identify problems before they occur.
* Know immediately when problems arise.
* Reduce downtime and business losses.

### Prerequisites
* LAMP server
* build-essential
* libgd2-xpm-dev
* libgd2-xpm-dev

### Create Nagios User And Group
```
sudo useradd -m nagios
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
sudo usermod -a -G nagcmd www-data
```

### Download Nagios And Plugins
```
# Download nagios core 
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.3.4.tar.gz

# Download nagios plugin
wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
```

### Install Nagios
```
# Extract downloaded nagios package
tar xzf nagios-4.3.4.tar.gz
cd nagios-4.3..4/

# Compile and Install Nagios.
sudo ./configure --with-command-group=nagcmd
sudo make all
sudo make install
sudo make install-init
sudo make install-config
sudo make install-commandmode
sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf

# Create a nagiosadmin account for logging into the Nagios web interface. 
# Notedown the password you assign to this account in order to login in to nagios web interface..
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

# Restart Apache
sudo service apache2 restart
```

### Install Nagios Plugins
```
# Extract Nagios plugins
tar xzf nagios-plugins-2.2.1.tar.gz
cd nagios-plugins-2.2.1/

# Compile and install Nagios Plugins
sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagios
sudo make
sudo make install
```

### Configure Nagios
#### Configure Email ID
- Nagios sample configuration files is available on path /usr/local/nagios/etc directory.
- Edit the /usr/local/nagios/etc/objects/contacts.cfg config file with a text editor and change the email address 
  associated with the nagiosadmin contact definition to the address you’d like to use for receiving alerts.

```
Edit Contacts.cfg
sudo nano /usr/local/nagios/etc/objects/contacts.cfg

# Find the following line and update the email id
define contact {
    contact_name                    nagiosadmin             ; Short name of user
    use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
    alias                           Nagios Admin            ; Full name of user

    email                           email_id@example.com    ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
}

# Save and close the file.
```

#### Configure IP Based Access
- Either use Apache Allow-Deny rules or AWS Security Groups to restrict access from a particular set of IPs. 
- Configure Apache Allow Deny rules to restrict access to Nagios administrative console from a particular set of IPs.

```
# Edit file /etc/apache2/sites-enabled/nagios.conf
sudo nano /etc/apache2/sites-enabled/nagios.conf

# Allow nagios administrative access from 192.168.0.0/24 series only

## Comment the following lines ##
#   Order allow,deny
#   Allow from all

## Uncomment and Change lines as shown below ##
Order deny,allow
Deny from all
Allow from 127.0.0.1 192.168.0.0/24
```

#### Enable Apache’s rewrite and cgi modules
```
sudo a2enmod rewrite
sudo a2enmod cgi
sudo service apache2 restart
```


#### Restart Nagios 
```
# Check nagios conf file for any syntax errors
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

If there is no error, start nagios service.
sudo /etc/init.d/nagios start
```

### Access Nagios Web Interface
- Open any web browser and navigate to `http://nagios-server-ip/nagios`. 
- Enter the username as nagiosadmin and the corresponding password which was created earlier.

```
# Access Nagios Web Interface Using nagiosadmin and its password.
http://example.com/nagios
```


### Add a Host to Monitor
* Nagios Server IP: example.com
* Ubuntu Host IP: 192.168.2.123

```
# Connect to host using ssh
ssh 192.168.2.123

# Install NRPE Service.
sudo apt-get install nagios-nrpe-server nagios-plugins

# Edit the nrpe file /etc/nagios/nrpe.cfg.
sudo vim /etc/nagios/nrpe.cfg

# Add Nagios Server IP example.com
allowed_hosts=127.0.0.1 example.com

# Start nrpe service
sudo /etc/init.d/nagios-nrpe-server restart
```
### Add read only user 

```
#Add user by using following steps
sudo htpasswd  /usr/local/nagios/etc/htpasswd.users newusername

#Edit cgi.cfg file.
sudo vi /usr/local/nagios/etc/cgi.cfg

#Add newuser name into below lines.
authorized_for_all_services=nagiosadmin,newusername
authorized_for_all_hosts=nagiosadmin,newusername

#Restart nagios and check by using this login.
sudo /etc/init.d/nagios start
```
### Allow new user to read only few hosts 

```
#Add user by using following steps
sudo htpasswd  /usr/local/nagios/etc/htpasswd.users newusername

#Create new contact from contact.cfg file.
define contact{
        contact_name                    newusername                ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Network Admin            ; Full name of user
        }
        
#Add useraname into contact group. This user can control all the hosts under this contact group.
define contactgroup{
        contactgroup_name       network-team
        alias                   network-team
        members                 newusername
        }
#Restart nagios and check by using this login.
sudo /etc/init.d/nagios start       
```
