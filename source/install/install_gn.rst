.. _install_gn:

##########################
Installing GeoNetwork 2.10
##########################

============
Introduction
============

In this document you'll only find specific information for installing GeoNetwork.

It is expected that the base system has already been properly installed and configured as described in :ref:`setup_system`.

In such document there are information about how to install some required base components, such as PostgreSQL, 
Apache HTTPD, Oracle Java, Apache Tomcat.

=====================
Installing GeoNetwork
=====================

.. hint::
   GeoNetwork project page at http://geonetwork-opensource.org/
      

Download packages
-----------------

Download the `.war` files needed for a full GeoNetwork installation::

   cd /root/download
   wget http://garr.dl.sourceforge.net/project/geonetwork/GeoNetwork_opensource/v2.10.3/geonetwork.war
   wget http://84.33.2.27/download/iso19139.rndt.zip 

Setup tomcat base
-----------------

Create catalina base directory for MapStore::

   cp -a /var/lib/tomcat/base/       /var/lib/tomcat/geonetwork
   cp /root/download/geonetwork.war  /var/lib/tomcat/geonetwork/webapps/


.. _gn_create_db:

Create user and DB for GeoNetwork
---------------------------------

Create a PostgreSQL DB for GeoNetwork::

   su - postgres -c "createuser -S -D -R -P -l geonetwork"

Annotate the user password.   
   
Create the DB::
   
   su - postgres -c "createdb -O geonetwork geonetwork -E utf-8"

Add the spatial extension to the ``geonetwork`` DB::

   # su - postgres -c "psql geonetwork"
   geonetwork=# CREATE EXTENSION postgis;
   geonetwork=# GRANT ALL PRIVILEGES ON DATABASE geonetwork TO geonetwork;
   geonetwork=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO geonetwork;

Create GN data dir
------------------

Some GN dirs can be externalized.

We'll put such dirs in ``/var/lib/tomcat/geonetwork/gn``, in this structure::

    gn
    ├── data
    │   ├── metadata_data
    │   ├── metadata_subversion
    │   └── resources
    │       └── images
    │           └── logos
    ├── index
    │   └── nonspatial
    └── upload


Create the directory hierarchy::

   cd /var/lib/tomcat/geonetwork/
   mkdir -p gn/data/{metadata_subversion,metadata_data}
   mkdir -p gn/data/resources/images/logos/
   mkdir -p gn/data/upload
   mkdir -p gn/index/nonspatial/

Create the override file:: 

   vim /var/lib/tomcat/geonetwork/gn/config-overrides.xml

and insert :download:`this content <../resources/gn-config-overrides.xml>`.

You will have to customize at least:

* the ``site.host`` element, setting the IP address or the server host name;
* the password for the geonetwork DB

You may also want to customize:

* the site name
* the bounding box and the layers for the search map.
  Please note that there are 2 sets of map definition:

  * ``<mapSearch>`` is about the search map 
  * ``<mapViewer>`` is about the preview map

setenv.sh
---------

Create the file ``setenv.sh``. 
We'll set here some system vars used by tomcat, by the JVM, and by the webapp itself::

   vim /var/lib/tomcat/geonetwork/bin/setenv.sh

Insert this content::

   # Set tomcat vars
   export CATALINA_BASE=/var/lib/tomcat/geonetwork
   export CATALINA_HOME=/opt/tomcat/  
   export CATALINA_PID=$CATALINA_BASE/work/pidfile.pid
  
   # Configure memory and system stuff   
   export JAVA_OPTS="$JAVA_OPTS -Xms1024m -Xmx2048m -XX:MaxPermSize=512m"
   export JAVA_OPTS="$JAVA_OPTS -Dorg.apache.lucene.commitLockTimeout=60000"

   # Configure GeoNetwork  
   export GN_EXT_DIR=$CATALINA_BASE/gn

   # Configure override file  
   export GN_OVR_PROPNAME=geonetwork.jeeves.configuration.overrides.file
   export GN_OVR_FILE=$GN_EXT_DIR/config-overrides.xml 
   export JAVA_OPTS="$JAVA_OPTS -D$GN_OVR_PROPNAME=$GN_OVR_FILE"
  
   #export JAVA_OPTS="$JAVA_OPTS -Dgeonetwork.dir=$GN_DATA_DIR"
  
   # Configure data dirs
   export GN_CTX=geonetwork.  
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}data.dir=$GN_EXT_DIR/data/metadata_data"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}resources.dir=$GN_EXT_DIR/data/resources"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}svn.dir=$GN_EXT_DIR/data/metadata_subversion"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}lucene.dir=$GN_EXT_DIR/index"
   
and make it executable::

   chmod +x /var/lib/tomcat/geonetwork/bin/setenv.sh


Edit server.xml
---------------

We need to assign 3 ports to this catalina instance.

Edit file ::

   vim /var/lib/tomcat/geonetwork/conf/server.xml

and change the connection ports in this way: 

- 8007 for commands to catalina instance
- 8082 for the HTTP connections
- 8011 for the AJP connections

See also :ref:`application_ports`.

Tomcat dir ownership
--------------------

Set the ownership of the ``geonetwork/`` related directories to user tomcat ::

   chown tomcat: -R /var/lib/tomcat/geonetwork
 

Automatic startup
-----------------

Create the file ``/etc/init.d/geonetwork`` and insert :download:`this content <../resources/geonetwork>`.

Once downloaded, make it executable ::

   chmod +x /etc/init.d/geonetwork

and set it as autostarting  ::

   chkconfig --add geonetwork

.. note::    
   If using Ubuntu, you have to use this command instead::
  
      update-rc.d geonetwork start 90 2 3 4 5 . stop 10 0 1 6 .
      
   
Configure httpd
---------------
   
Create the file ``/etc/httpd/conf.d/80-geonetwork.conf`` and insert these lines::

   ProxyPass        /geonetwork   ajp://localhost:8011/geonetwork                                                                                                                                                                                                                           
   ProxyPassReverse /geonetwork   ajp://localhost:8011/geonetwork

.. note::    
   If using Ubuntu, you have to put these lines in file ::
   
      vim /etc/apache2/sites-available/ckan 
      
   just before the ``ProxyPass`` directive redirecting the ``/``.    


Then reload the configuration for apache httpd::

   service httpd reload


=============
Further setup
=============

Once GeoNetwork is up and running, you have to perform some other steps using the web interface.

Login as ``admin`` / ``admin``.

Change the admin pw
-------------------

Go to "Administration" >  "Users and groups" >  "Change password".
Change and annotate the new password.

Check the system configuration
------------------------------

Go to "Administration" >  "Catalogue settings" >  "System configuration".

Check if the right values are set in these fields:

* Site name
* Site organization
* Host
* Port (you may want to put ``80`` here) 

You then may want to:

* Disable Z39.50 server
* Enable search statistics
* Enable INSPIRE
* Enable INSPIRE view
* Setup the CSW server info

Default language
----------------

The only way to change de default UI language is to edit the index.html file::

   vim webapps/geonetwork/index.html
   
The default language is set as a 3 letters ISO code in this line::
   
   window.location="srv/eng/home" + search;
   
so you may for instance change the string to ``srv/ita/home`` to have Italian as default language. 

=========================
Installing schema plugins
=========================

You may want to add additional schemas to GeoNetwork.

For instance let's add the RNDT profile.

You need the zip file containing the definition of the schema, or a URL pointing to such file.

Go to "Administration" >  "Metadata & Template" >  "Add a metadata schema/profile".

Set as schema name ``iso19139.rndt``

Then check the "URL of Schema Zip Archive" option, and set ``http://84.33.2.27/download/iso19139.rndt.zip``.

Then press the "Add" button.

Now in "Administration" you can "Add templates" and "Add sample metadata" for the new schema.

============
Known issues
============

* site name and site URL set in the override file are not put in the DB during the initialization, 
  so a manual setup in the configuration page is required. 

==============
Other settings
==============

Log file location
-----------------

GeoNetwork log setting are set to create the log files into ``CURRENT_DIRECTORY/logs/geonetwork.log``.
It means that, running GeoNetowrk with the configuration explained in this document, you'll get the log files into
``/home/tomcat/logs/geonetwork.log``.  

If you wish to customize the log location, you'll have to edit the file ``WEB-INF/log4j.cfg``. 

You may want to path the log4j configuration file before running the GeoNetowrk service hte first time, in order 
not to have temp log files places in unwanted places. 

- Expand the war file ::

   cd /var/lib/tomcat/geonetwork/webapps/
   mkdir geonetwork
   cd geonetwork
   jar xvf /root/geonetwork-main-2.10.xxxxx.war

Edit the file ``WEB-INF/log4j.cfg``, setting the property ``log4j.appender.jeeves.file`` as follows::

   log4j.appender.jeeves.file = ${catalina.base}/logs/geonetwork.log

Make sure you have the ``${catalina.base}`` part. In this way, the logfile should be created in the direcotry   
 ``/var/lib/tomcat/geonetwork/logs/``.


.. _gn_web_config:


Logo
----

You may customize the site logo in the administration page. 

In the option group "Catalog configuration" select "Logo configuration".
Upload the image you wish to use as the site logo. Once loaded, select it and click on "Use for this catalog".

Note: 
In previous GeoNetowrk releases you had to use the non-interactive procedure:
You had to identify the site UUID (in the info page -- "Info" link on hte toolbar). 
Then you had to copy the ``gif`` file into the directory ``images/logos``, 
with name ``SITE_UUID.gif``.

===============================
Reconfiguring GN on a cloned VM
===============================

For information about cloning the VM, follow the instuctions on page :ref:`cloning_vm`.

Next sections will show how to reconfig GN on a cloned VM already reconfigured.


Config on files
---------------

WMS Layer for the search map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit WMS server, layer and bounding box in the externalized file, as described above.  

Config on UI and DB
-------------------

DB configurations include the automatic UUID site generation. This info does not apply to you if you are using 
the RNDT schema plugin, because you will be setting the site ID by hand anyway.  

Anyway, to be sure you have a clean GeoNetowrk DB, you'd better create a brand new DB.

Stop the GeoNetwork instance
----------------------------

Since we are reconfiguring a cloned machine, where GeoNetwork has been set to start at boot, we'll have 
to stop the geonetwork service :: 

   service geonetwork stop

Database setup
--------------

As user postgres drop the DB ::

   dropdb geonetwork
   
Then recreate a DB from scratch, following the steps described in section :ref:`gn_create_db`::

   createdb -O geonetwork  geonetwork
   psql -W -U geonetwork -d geonetwork -c "CREATE EXTENSION postgis;"
   psql -W -U geonetwork -d geonetwork -c "CREATE EXTENSION postgis_topology;"
   
Restart  GeoNetwork
-------------------
::

    service geonetwork start
    
At boot, GeoNetwork will regenerate the DB schema and will populate it with the initial data. 
    
Settings in UI
--------------

Follow the steps in  :ref:`gn_web_config`.



