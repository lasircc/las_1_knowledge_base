*************************
Biobank module
*************************

The **Biobank** stores the aliquots information. Physical positions and container details are managed by the **Storage**.


.. _deleting_an_aliquot:

Deleting an aliquot
###################

An automatic procedure is available at the URL http://las.ircc.it/biobank/canc/aliquot. The user has to insert a txt file listing all the genealogyIDs one per line, according to the following format::

	<GenealogyID>
	<GenealogyID>
	<GenealogyID>
	....
	

The procedure sets the the availability attribute equal to 0 in the table ``Aliquot`` of the Biobank DB. Successively, it sets the endTimestamp in the Aliquot table of the Storage DB. Then, the tube is eliminated if it is set as "single use".

.. note::  Please remember that by default all the created tubes are set to **"single use"**.

Furthermore, users can delete aliquots on their own. Just go to ``Aliquots => Perform -> QC/QA -> Manual Revaluation`` and set [DEFINE] to *"Exhausted"* during the re-evaluation procedure.


Aliquot Restore
###################
The automatic procedure to carry out the restore is accessible `here`_. This procedure is nothing but the dual of the above mentioned one for cancellation. At the end of this procedure, the aliquots regain the availability and they become usable again. As for cancellation, the user is asked to provide a txt file with all the genealogyID he/she wants to restore written one per line.

.. _here: http://las.ircc.it/biobank/restore/aliquot



Changing aliquot pieces
#######################
An automatic procedure is available at https://las.ircc.it/biobank/pieces/aliquot. The user must provide a TAB-separated txt file whose lines include the genealogyID and the number of pieces to assign to each sample.
Therefore, the resulting file should look like this: ::

	<GenealogyID>	<# pieces>
	<GenealogyID>	<# pieces>
	<GenealogyID>	<# pieces>
	....

If a sample does not have a number of pieces (for instance if it is a DNA sample or a cell line) the procedure does not perform any operation.


Changing patient code in a collection
######################################
In table ``Collection``, we must edit the *patientCode* field in the relative collection.
Hence, in the graph modify the attribute *localId* of the *hasAlias* relationship between *Patient* and *Project* nodes. 
In order to find the *Patient* node, we could query for the informed consensus (labeled as IC). Based on the value of *collectionEvent* attribute, in the table ``Collection`` it is easy yo find the related patient.

Thus, opening the neo4j shell:

.. code:: cypher

	match (n:IC)<-[r:signed]-(p:Patient)-[g:hasAlias]->(h:Project) where n.icCode='<consenso>' set g.localid='<codice_paziente>' return g


Funnel: wrong insertion of a block in the biobank
#################################################
In the email reporting the error, the attached JSON file includes all the data to be inserted in the system. For a manual insertion, it is enough to call the API ``biobank/api/blocks/`` posting the JSON file. 
Below is reported an example of API call: a solution may be to create a .py file built as follows and execute it after hardcoding the JSON data.
In detail, it is enough to insert your host URL where biobank is located in ``addr`` variable. Then put the specimen json ``{"ICcode": "code", ...}`` (without the specimen: key) into variable ``gen_dict[]``.

.. code::

	#!/usr/bin/env python3
	import json, requests

	gen_dict = ['insert json here']
	# addr = 'insert biobank location here'
	url = addr + '/api/blocks/'
	print("url is: " + url)
	values_to_send = json.dumps({'specimens':gen_dict})
	print("sent json is: " + values_to_send)
	r = requests.post(url, data=values_to_send, verify=False, headers={'Content-type': 'application/json'})
	print("r is: " + str(r))
	status = r.status_code
	print("status is: " + str(status))

The vast majority of wrong insertions is due to a missing initial portion of the block barcode. More precisely, we are referring to the first two letters, specifying the hospital where the samples have been collected. Of course the barcode needs to be complete when sent to the API.


Modify the creation date of a sample
####################################
The attribute *idSamplingEvent* in the ``Aliquot`` table points to the table ``SamplingEvent``, whose field *samplingDate* needs to be changed.

.. code:: sql

	update samplingevent set samplingDate ='<data>' where id=<id>;

Hence, a foreign key references the ``Series`` table, where the series data is stored. Whether the series points to that sampling event only, it is sufficient to change the data directly. Consequently, a new series must be created reporting the new data and pointing to the sampling we are considering.

.. code:: sql

	update serie set serieDate ='<data>' where id=<id>;

Thereafter, in the ``Storage.Aliquot`` table, the *startTimestamp* field has to be edited for each aliquot.

.. code:: sql

	update aliquot set startTimestamp='<data>' where genealogyID='<genealogy>';

- **If the examined sample is a derivate** (such as DNA or RNA)
    ::

    The initial date of the sampling procedure must be modified as well. To do so, just edit the field *initialDate* in the ``aliquotderivationschedule`` table as follows
  

    .. code:: sql

        update aliquotderivationschedule set initialDate ='<data>' where idAliquot=<id>
    
        
    Therefore, the measurement insertion date needs to be changed accordingly in the attribute *qualityevent*
    
    .. code:: sql
    
        update qualityevent set misurationDate ='<data>', insertionDate ='<data>' where     idAliquotDerivationSchedule =<id>
    
    Thereafter the derivation has to be edited accordingly in the *derivationevent* field.

- **If the sample comes from a mouse explant**
    ::

    As a first step, remember to modify its date of death as reported below.

    .. code:: sql

        update phys_mice set death_date ='<data>' where barcode ='<barcode>'

    Then, look for the explant details and edit the series date accordingly. If the series refers to that mouse only:

    .. code:: sql

        update series set date='<data>' where id=<id>

 - **If the sample is a cell line and is been archived using the Cell Lines module**
 	::

 	Here the procedure is slightly different. First of all, we edit the *application_date* and *end_date_time* attributes in tables ``archive_details`` and ``cell_details`` respectively.


Delete an experiment
####################
An experiment could be handled by an external module, i.e. not by the biobank, but by some other modules such as *realTime*, *Sanger* or *digitalPCR*.
To completely delete an experiment, we access to the ``Request`` table of the involved module. Then cancel the from table ``Aliquotexperiment`` all those lines in which a sample is related to the experiment we want to eliminate.

.. note::  To find all the samples involved in the experiment, having a look at the experimental notes may save you some time.


Derivation
##########
Derivation has a more complex structure. The main table is ``aliquotderivationschedule``, on which the audit is available. The table is composed by various attributes and foreign keys that are initially set to NULL and successively filled in as the derivation goes on.

The derivation measures are stored in table ``measurementevent`` referencing, among the others, the table ``qualityevent`` (that references ``aliquotderivationschedule``). Once the derivation its terminated in all its 4 steps, the new aliquots are created and a row is inserted in table ``derivationevent``. Such table references tables ``aliquotderivationschedule`` (to have a ) and ``samplingevent``. While the former allows to keep a constant connection with the procedure, the latter let us know which samples have been created. Such info is easy to be retrieved, since using the sampling event it is enough to load the ``aliqout`` table and look for the samples associated to that *idSamplingEvent*.


Slide preparation
#################

**Aliquots-> Slides preparation-> Plan**

This is a classic planning procedure, requiring the GenealogyID's or barcodes just for FF or OF [DEFINE]. 

**Aliquots-> Slides preparation-> Execute**

	1. Select protocol and samples for FF and OF to be cut.
	2. Validate samples reading their barcodes and click "Next Step".
	3. The value of thickness for each slice is retrieved directly from the DB. The number of sections of each slide tells you how many slices can be positioned at most on each slide. Each slide usually has one row and multiple columns. This value is fetched from the database in the table ``featureslideprotocol``. The number of section for each block accounts for how many slices are created by each FF or OF block in the current session. At the beginning, this value is set to 0. However you may change it according to your needs. In this case, as soon as you load a slide code, the systems automatically places a number of slices equal to the value you just set.
	4. Load a slide code. If it is not already registered in the LAS system, it is then created at the end of the current session along with the new aliquots.
	5. Click on a square inside the slide to place a slice.
	6. If needed, using the table at the bottom of the page, you can delete the samples you have placed.
	7. At the time of saving, the system creates a aliquots of type PS (Paraffin Section) if the block is an FF. Differently, if the block is an OF, the created aliquot will be of type OS (OCTSection). In addition, the slide instance is created in the Storage DB.

The DB table in which the entire procedure is saved is ``aliquotslideschedule``. In table ``slideprotocol``, you can find the protocol used to generate the slices and in ``featureslideprotocol`` the default values of each protocol are recorded. This last table translates a many-to-many relationship between ``feature`` and ``slideprotocol``.

.. note:: The code to generate this views is in ``tissue/slide.py``. The .js files are archived in ``tissue/tissue_media/JS/slide`` and the .html's are in ``tissue/Templates/tissue2/slide``.


Slides labelling
################

Taking in input a PS or a OS allows you insert a coloring or an antibody on a slide. The output of this operation is an LS (Labeled Section).

In order to execute the procedure, you must firstly set a protocol in ``Aliquots -> Slides labelling -> Define protocol``.
There you can insert the name, choose a technique and then define the related marker(s) by clicking on the "Create Marker" button. Be aware that this procedure changes according to the chosen technique:

	- **Histology:** a page opens up with a form in which the user can insert the features of a colorant. Some values are optional, but if you do not insert them now, you will not be able to do so until execution time.
	- **IF and IHC:** here you have to insert an antibody, specifying features such as the referring gene. The system has already recorded a list of antibodies, thus if you type their name in field "Marker Name", you can pick the right one choosing among the auto-complete results.
	- **FISH, CISH and RNAScope:** this category requires a probe. When you click "Create Marker" you are redirected to the *Annotation* module in which you can insert the probe name and the nucleotide sequence to find the right alignment on the genome.

**Aliquots-> Slides labelling-> Plan**

This is a canonical planning procedure, but only PS and OS aliquots are accepted.

**Aliquots-> Slides labelling-> Execute**

	1. Validate the samples via barcode reading and click "Next Step".
	2. In the left-hand side of the screen the slides are displayed, while the right-hand side is aimed to protocol choice of which, once selected, you will be able to see all parameters. The coloring takes place by clicking on the slide representing the desired aliquot. Thereafter, it changes color. Notice that each protocol has its own random color picked by the system on-the-fly. At the bottom of the page it is possible to delete all the operations form the beginning of the procedure.
	3. To avoid planning operations you can insert the slide directly and coloring jumping that phase. Just specify the GenealogyID or the slide code in the top-left field.
	4. At the end of the procedure the aliquot representing the colored slice is discarded and substituted with a new one of type LS.

**Aliquots-> Slides labelling-> Files-> Insert**

This screen shows the list of slides colored in the sessions that are still ongoing. Here you can also insert any file related to a slide that has been processed by the LAS system in the past and whose pictures have been already acquired. It is enough to insert the GenealogyID or the slide code in the top-left field. To consider each file unambiguously, at each file a new name is assigned (editable by the user if needed). this name is made up by the first figures of the GenealogyID, the coloring protocol name and, in closing, by the date.

From now on, the LAS will refer to that particular file using only this newly-created name. Whether the slide does not have any associated pictures, it is enough to check the voice "No File" and the insertion procedure terminates.

**Aliquots-> Slides labelling-> Files-> View/Download**

In the left-hand-hand side of the screen, the fields to create a slide are displayed (in the ``GenealogyID`` field you can insert only the first characters of the code). Once clicked "Search File", the slides appear on the right and by clicking on each of them, a windows pops up showing all the related files.

At this point, selecting one or more pictures one may decide to download them or to see them directly in the screen as a classic gallery.

**Aliquots-> Slides labelling-> Files-> Delete**

Once found the slide by means of the classic search filters, the related files become visible and the user can decide to delete one or more of them.

The file itself is not physically deleted: the system just appends its cancellation date and the operator identifier in the table ``labelfile``, containing the linking between the slide and the file. In such a way, that file will not be loaded anymore in the gallery of that slide.

**Aliquots-> Slides labelling-> Insert analysis result**

This procedure is still not complete. Indeed it is available in the trunk only. It allows to save the results of the analysis on a specific slide.

**Involved tables**:

	- ``aliquotlabelschedule`` is the table in which the procedure is saved.
	- ``labelprotocol`` stores the protocols.
	- ``labelfeature`` contains the features.
	- ``labelprotocollabelfeature`` translates the many-to-many relationship between the last two.
	- ``labelmarker`` containing the markers (antibodies, probes, ...).
	- ``labelmarkerlabelfeature`` translates the many-to-many relationship between ``labelmarker`` and ``labelfeature``.
	- ``labelconfiguration`` includes the created configurations saved with the common name *"configuration_n"*, where *n* is a progressive number.
	- ``labelfile`` lists the saved files and has a foreign key in ``aliquotlabelschedule`` towards ``labelconfiguration``.
	- ``labelconfigurationlabelfeature`` stores the specific values for each configuration.

.. note:: 
	You can find the code to design these views in ``tissue/label.py``. 
	The .js files are in ``tissue/tissue_media/JS/label``, while the .html's are in ``tissue/Templates/tissue2/label``.


Experiments
###########

The planning procedure is the same as in the *Derivation* module. This view has an additional section to insert files in ``Experiments-> Execute-> Other-> Upload results``. There you can see the pending experiments to which you can associate one or more files (of any kind) that are saved in Mongo via the *repmanager*. Using a drop-down menu, you can eventually select a label to identify the type of file loaded.

The view in ``Experiments-> View results-> Other`` allows to retrieve information about a specific experiment and download its related files (if loaded in the previous screen). the search can be improved using some dedicated filters.


**Involved tables**:

	- ``aliquotexperiment`` is the table in which the experiments are saved.
	- ``experimentfile`` stores the files and has a foreign key towards ``aliquotexperiment`` and ``filetype`` in order to easily understand the filetype chosen by the user.
	- ``filetypeexperiment`` is a controlled vocabulary to easily retrieve which filetypes are supported by each experiment. It basically translates a many-to-many relationship between ``filetype`` and ``experiment``..

.. note:: 
	The code to design these views is in ``tissue/experiment.py``. 
	The .js files are in ``tissue/tissue_media/JS/decrease`, while the .html's are in ``tissue/Templates/tissue2/update``.


Common operations for the help desk
###################################

Here is a collection of some of the most representative requests received by the LAS Help Desk.

Kit modifications
*****************

It may happen sometimes to run into a wrong kit insertion during the DNA extraction procedure.
Suppose that here we want to change the kit from "Kit_1" to "Kit_2". to do so, we select the``Biobank`` database and we identify the kits.

.. code:: sql

	mysql> use biobanca;
	mysql> select id, idKitType, name from derivationprotocol;

+----+-----------+-------------------------------------+
| id | idKitType | name                                |
+====+===========+=====================================+
|  1 |         1 | Kit_1					           |
|  2 |         3 | DNA extraction (ABC)                |
|  3 |         2 | RNA extraction (DEF)                |
|  4 |         4 | Kit_2                               |
|  5 |         5 | cDNA FF                             |
+----+-----------+-------------------------------------+

We now look for the derivations made by the user who asked for a kit change. We can query for all the derivations made by this user in the specified day, bearing in mind the ``idDerivationProtocol`` of the current kit (Kit_1) from the table above.

.. code:: sql

	mysql> select id, idAliquot, idDerivationSchedule, idDerivedAliquotType, idDerivationProtocol, idKit, operator, initialDate, validationTimestamp from aliquotderivationschedule where operator='username' and initialDate='2019-06-10' and idDerivationProtocol=1;

+-------+-----------+----------------------+----------------------+----------------------+-------+---------------+-------------+---------------------+
| id    | idAliquot | idDerivationSchedule | idDerivedAliquotType | idDerivationProtocol | idKit | operator      | initialDate | validationTimestamp |
+=======+===========+======================+======================+======================+=======+===============+=============+=====================+
| 25786 |    321105 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:00 |
| 25787 |    323913 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:01 |
| 25788 |    322091 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:02 |
| 25789 |    322056 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:03 |
| 25790 |    322050 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:04 |
| 25791 |    318828 |                 2684 |                    7 |                **1** |    31 |  **username** | 2019-06-10  | 2019-06-10 10:40:05 |
+-------+-----------+----------------------+----------------------+----------------------+-------+---------------+-------------+---------------------+

Now we can safely update the ``aliquotderivationschedule`` table inserting the new kit.

.. code:: sql

	mysql> update aliquotderivationschedule set idDerivationProtocol=2 where operator='username' and initialDate='2019-06-10' and idDerivationProtocol=1;