.. _install_ckan_other:

#####################
Other CKAN extensions
#####################

============
Introduction
============

In this document you'll only find specific information for installing some CKAN official and 
unofficial extensions.


====================
GeoNetwork harvester
====================

The GeoNetwork harvester extends the base CSW harvester type, adding some features 
as explained in :ref:`geonetwork_harvester_params`, such as:

* handling the ``default_tags`` and ``default_extras`` parameters;
* adding a couple of ``extras`` entries which contain URLs to GeoNetwork info. 


In order to install the extension, log in as user ``ckan``, activate the virtual env and check out the extension::

   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   git clone https://github.com/geosolutions-it/ckanext-geonetwork.git 
   cd ckanext-geonetwork
   python setup.py develop

Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] geonetwork_harvester
      
Restart supervisord::
      
   service supervisord restart 

.. _extension_tracker:

=======
Tracker
=======

Tracks visit to the site and to single datasets.

.. hint::
   Doc page at http://docs.ckan.org/en/tracking-fixes/tracking.html
    
Edit the file ``/etc/ckan/default/production.ini`` and add the line ::

   ckan.tracking_enabled = true
   
Then create a script ``tracker_update.sh`` like this::

    #!/bin/bash

    . /usr/lib/ckan/default/bin/activate

    paster --plugin=ckan tracking update         -c /etc/ckan/default/production.ini 
    paster --plugin=ckan search-index rebuild -r -c /etc/ckan/default/production.ini

You can use this file directly to run the index rebuilding, or use it in cron to make it run periodically.

As user ``root`` use ::

     crontab -e -u ckan
     
or as user ``ckan``::

     crontab -e
     
and add the line::

   0 * * * * /usr/lib/ckan/tracker_update.sh >>/var/log/ckan/tracker.log 2>&1


   