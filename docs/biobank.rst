*************************
Biobank & Storage modules
*************************

The **Biobank** stores the aliquots information. Physical positions and container details are managed by the **Storage**.


Deleting an aliquot
###################

An automatic procedure is available at the URL las.ircc./biobank/canc/aliquot. The user has to insert a txt file listing all the genealogyIDs one per line. The procedure sets the the availability attribute equal to 0 in the table Aliquot of the biobank DB. Successively, it sets the endTimestamp in the Aliquot table of the Storage DB. Then, the tube is eliminated if it is set as "single use".


Changing aliquot pieces
#######################

