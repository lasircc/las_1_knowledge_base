*************************
Biobank & Storage modules
*************************

The **Biobank** stores the aliquots information. Physical positions and container details are managed by the **Storage**.


Deleting an aliquot
###################

An automatic procedure is available at the URL http://las.ircc./biobank/canc/aliquot. The user has to insert a txt file listing all the genealogyIDs one per line. The procedure sets the the availability attribute equal to 0 in the table Aliquot of the biobank DB. Successively, it sets the endTimestamp in the Aliquot table of the Storage DB. Then, the tube is eliminated if it is set as "single use".

.. note:: Please remember that by default all the created tubes are set to **"single use"**.

Furthermore, users can delete aliquots on their own. Just go to **Aliquots -> Perform -> QC/QA -> Manual Revaluation** and set [DEFINE] to "Exhausted" during the re-evaulation procedure.


Aliquot Restore
###################
The automatic procedure to carry out the restore is accessible `here`_. This procedure is nothing but the dual of the above mentioned one for cancellation. At the end of this procedure, the aliquots regain the avaialbility and they become usable again. As for cancellation, the user is asked to provide a txt file with all the genealogyID he/she wants to restore written one per line.

.. _here: http://las.ircc.it/biobank/restore/aliquot



Changing aliquot pieces
#######################

