.. _ckan_harvesting:

########################
Harvesting configuration
########################

Adding an harvest source
========================

In order to add an harvest source, you need to log into CKAN web interface as an administrator.
You have to type the address for the harvesting page, since there is no direct link for it ::

   http://YOUR_SITE/harvest
   

.. figure:: /images/ckan_harvest_add.png
   :align: center
   :width: 640

Press the "Add Harvest Source" button and this page will open:

.. figure:: /images/ckan_harvest_geonetwork.png
   :align: center
 

The information needed to set up an harvester instance are:

* The URL endpoint (e.g. something like http://dati.provincia.fi.it/geonetwork/srv/ita/csw for a GeoNetwork CSW endpoint)
* A title/name for this source, for your reference
* The harvest source type, i.e. "CSW Server"

  You can choose among different types of remote service to retrieve data from:

   CKAN
      a remote CKAN instance.  
      You'll have only this kind of source type if you only installed the ``ckanext-harvest`` extension.   
   CSW
      a generic CSW server. 
      You will have this option if you installed the ``ckanext-spatial`` extension.
   GeoNetwork CSW server
      a CSW server implemented by a GeoNetwork instance. 
      You will have this option if you installed the ``ckanext-geonetwork`` extension.
      This source type will offer some more options than the bare CSW server.
       
* The update frequency 
* An optional configuration in JSON format (every source type accepts its own set of configuration items). 
* The owner Organization: all the dataset harvested from this source will be assigned to that Organization.

The JSON configuration allows these parameters:

Configuration params
====================
 
CKAN sources
------------

You can find `here <https://github.com/ckan/ckanext-harvest#the-ckan-harvester>`_ the doc about the configuration 
parameters that the CKAN harvester can handle. 

In particular, you may find useful these params:
 
* ``default_tags``: all dataset harvested from this source will have these tags attached.
* ``default_extras``: all dataset harvested from this source will have these key/value pairs attached. 

  The value may contain tokens in the form ``{token}``, where the allowed tokens are:
  
  * ``harvest_source_url``
  * ``harvest_source_title``
  * ``harvest_job_id``
  * ``harvest_object_id``

CSW sources
-----------

CSW sources may use the param 

* ``cql``: a CQL filter for harvesting only a subset of the records in the remote node

if your ``ckanext-spatial`` extension contains `this commit <https://github.com/ckan/ckanext-spatial/commit/55497f037e5add55f5890315e9c7c4f396cc49ac>`_
(which has been merged on 14 March 2014).

Should you need the ``default_tags`` and ``default_extras`` params, with the behavior described in `CKAN sources`_, you 
may need to include manually `this pull request <https://github.com/ckan/ckanext-spatial/pull/58>`_.

.. _geonetwork_harvester_params:

GeoNetwork sources
------------------

The *GeoNetwork CSW source type* inherits most of its behaviour from the CSW type, but it offers out of the box the 
handling of the ``default_tags`` and ``default_extras`` parameters.

Furthermore two ``extras`` fields will be added to each harvested dataset:

* ``gn_view_metadata_url``: contains a URL pointing to a page for displaying 
  the full metadata in the source GeoNetwork site (e.g. ``http://yourserver/geonetwork/srv/eng/metadata.show?uuid=aaa-bbb-ccc``) 
* ``gn_localized_url``: contains a URL pointing to the localized geonetowrk root (e.g. ``http://yourserver/geonetwork/srv/eng``)


