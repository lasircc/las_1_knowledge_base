*************************
Biobank & Storage modules
*************************

The **Biobank** stores the aliquots information. Physical positions and container details are managed by the **Storage**.


Deleting an aliquot
###################

An automatic procedure is available at the URL http://las.ircc./biobank/canc/aliquot. The user has to insert a txt file listing all the genealogyIDs one per line, according to the following format::

	<GenealogyID>
	<GenealogyID>
	<GenealogyID>
	....
	

The procedure sets the the availability attribute equal to 0 in the table ``Aliquot`` of the biobank DB. Successively, it sets the endTimestamp in the Aliquot table of the Storage DB. Then, the tube is eliminated if it is set as "single use".

.. note:: Please remember that by default all the created tubes are set to **"single use"**.

Furthermore, users can delete aliquots on their own. Just go to ``Aliquots -> Perform -> QC/QA -> Manual Revaluation`` and set [DEFINE] to *"Exhausted"* during the re-evaulation procedure.


Aliquot Restore
###################
The automatic procedure to carry out the restore is accessible `here`_. This procedure is nothing but the dual of the above mentioned one for cancellation. At the end of this procedure, the aliquots regain the avaialbility and they become usable again. As for cancellation, the user is asked to provide a txt file with all the genealogyID he/she wants to restore written one per line.

.. _here: http://las.ircc.it/biobank/restore/aliquot



Changing aliquot pieces
#######################
An automatic procedure is available at https://las.ircc.it/biobank/pieces/aliquot. The user must provide a TAB-separated txt file whose lines include the genealogyID and the number of pieces to assign to each sample.
Therefore, the resulting file sholud look like this: ::

	<GenealogyID>	<# pieces>
	<GenealogyID>	<# pieces>
	<GenealogyID>	<# pieces>
	....

If a sample does not have a number of pieces (for instance if it is a DNA sample or a cell line) the procedure does not perform any operation.


Changing patient code in a collection
#######################
In table ``Collection``, we must edit the *patientCode* field in the relative collection.
Hence, in the graph modify the attribute *localId* of the *hasAlias* relationship between *Patient* and *Project* nodes. 
In order to find the *Patient* node, we could query for the informed consensus (labelled as IC). Based on the value of *collectionEvent* attribute, in the table ``Collection`` it is easy yo find the related patient.

Thus, opening the neo4j shell:

.. code:: cypher

	match (n:IC)<-[r:signed]-(p:Patient)-[g:hasAlias]->(h:Project) where n.icCode='<consenso>' set g.localid='<codice_paziente>' return g


Funnel: wrong insertion of a [] in the biobank
#######################
In the email reporting the error, the attached JSON file includes all the data to be inserted in the system. For a manual isnertion, it is enough to call the API ``biobank/api/blocks/`` posting the JSON file.
Below is reported an example of API call:

.. code::
	import json, requests
	from django.conf import settings
	gen_dict=[...]
	addr=settings.DOMAIN_URL+settings.HOST_URL
	url=addr+'/api/blocks/'
	values_to_send=json.dumps({'specimens':gen_dict})
	r=requests.post(url, data=values_to_send, verify=False, headers={'Content-type': 'application/json'})
	status=r.status_code

The vast majority of wrong insertions is due to a missing initial portion of the [] barcode. More precisely, we are referring to the first two letters, specifying the hospital where the samples have been collected. Of course the barcode needs to be complete when sent to the API.


Modify the creation date of a sample
#######################
The attribute ``idSamplingEvent`` in the ``Aliquot`` table points to the table ``SamplingEvent``, whose field ``samplingDate`` needs to be changed.

.. code:: sql
	update samplingevent set samplingDate ='<data>' where id=<id>;

Hence, a foreign key references the ``Series`` table, where the series data is stored. Whether the series points to that sampling event only, it is sufficient to change the data directly. Consequently, a new series must be created reporting the new data and pointing to the sampling we are considering.

..code:: sql
	update serie set serieDate ='<data>' where id=<id>;

Thereafter, in the ``Storage.Aliquot`` table, the ``startTimestamp`` field has to be edited for each aliquot.

..code:: sql
	update aliquot set startTimestamp='<data>' where genealogyID='<genealogy>';

- **If the examined sample is a derivate** (such as DNA or RNA), the initial date of the sampling procedure must be modified as well. To do so, just edit the field ``initialDate`` in the ``aliquotderivationschedule`` as follows.

	..code:: sql
		update aliquotderivationschedule set initialDate ='<data>' where idAliquot=<id>

	Therefore, the measurement insertion date needs to be changed accordingly in the attribute ``qualityevent``

	..code:: sql
		update qualityevent set misurationDate ='<data>', insertionDate ='<data>' where     idAliquotDerivationSchedule =<id>

	Thereafter the derivation has to be edited accordingly in the ``derivationevent`` field.

- **If the sample comes from a mouse explant**, remember to modify its date of death as reported below.

	..code:: sql
		update phys_mice set death_date ='<data>' where barcode ='<barcode>'

	Then, look for the explant details and edit the series date accordingly. If the series refers to that mouse only:

	..code:: sql
		update series set date='<data>' where id=<id>

	Conversely, if the sample is a cell line and is been archived using the Cell Lines module, the procedure is slightly different. First of all, we edit the ``application_date`` and ``end_date_time`` attributes in tables ``archive_details`` and ``cell_details`` respectively.


Delete an experiment
#######################
An experiment could be handled by an external module, i.e. not by the biobank, but by some other modules such as realTime, Sanger or digitalPCR.
To completely delete an experiment, we access to the ``Request`` table of the involved module. Then cancel the from table ``Aliquotexperiment`` all those lines in which a sample is related to the experiment we want to eliminate.

.. note:: To find all the samples involved in the experiment, having a look at the experimental notes may save you some time.