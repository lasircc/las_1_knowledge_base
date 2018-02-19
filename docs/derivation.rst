DERIVATION
##########
Derivation has a more complex structure. The main table is ``aliquotderivationschedule``, on which the audit is available. The table is composed by various attributes and foreign keys that are initially set to NULL and successively filled in as the derivation goes on.

The derivation measures are stored in table ``measurementevent`` referencing, among the others, the table ``qualityevent`` (that references ``aliquotderivationschedule``). Once the derivation its terminated in all its 4 steps, the new aliqouts are created and a row is inserted in table ``derivationevent``. Such table referecnes tables ``aliquotderivationschedule`` (to have a ) and ``samplingevent``. While the former allows to keep a constant connection with the procedure, the latter let us know which samples have been created. Such info is easy to be retrieved, since using the sampling event it is enough to load the ``aliqout`` table and look for the samples associated to that *idSamplingEvent*.

********
HAMILTON
********

DERIVATION
##########

The procedure to derive aliquots using the **Hamilton** system consists of the steps illustrated below, starting from the *Aliquot* module

**Aliquots-> Derive-> Plan derivation**
As usual, the first step is the planning. Since it has not been modified, the required data are the same as in the manual derivation module.

**Aliquots-> Derive-> Automated derivation-> 1. Select aliquots and protocol**
The operations to be performed in this step are identical as those in the manual derivation and can be resumed as follows.
	1. Choice of the derivation protocol and selection of the samples to be derived.
	2. Sample validation through barcode read.
	3. Insertion of the amount of collected material for derivation. If necessary specify if an aliquot is exhausted.

**Aliquots-> Derive-> Automated derivation-> 2. Select kit**
Same as manual derivation.

**Aliquots-> Derive-> Automated derivation-> 3. Perform QC/QA**
This step aims to plan the quantification. The operations listed below are supposed to be applied after the sampling procedure via the automatic extractor and after physically placing the samples in the robot guide [LEFT??].

In the related view the samples are sorted in descending order of validation time (completed in Step 1 of this procedure).
Therefore, the sequence of operations to be performed is:

	1. Choose the quantification protocols to be executed.
	2. Insert a procedure name (even alphanumeric), to be used as a reference when the real procedure will be executed by the robot.
	3. Select the container type in which the samples to be derived are stored (those previously placed in the leftmost guide in Step 4).
	4. Mark as *"Failed"* the unsucessful derivations.
	5. Set the volume of samples to be derived, i.e. the amount of liquid poured by the extractor.
	6. Click "Confirm".

At this point the quantification is planned and the user just need to start the robot application to run it effectively.

**Aliquots-> Derive-> Automated derivation-> 4. Plan create derivatives**
This procedure allows to transfer to the LAS system the values of all the quantification made by the robot and to plan the dilution procedure. The steps can be summarized as reported below:

	1. Click on "Refresh sample list" to save the quantification data in the LAS application.
	2. The procedure name and the container type are automatically set, but can be edited if needed.
	3. Select the concentration to perform the sampling (e.g. fluo or UV), choosing from the drop-down menu in the leftmost table.
	4. Set the unsuccessful procedures (if any) or those that the user wants to terminate for whatever reason, checking the box "Stop Procedure".
	5. Set values for the aliquots that are about to be created. N aliquots will be generated, with N-1 similar aliquots and a last one acting as spare aliquot (left over).
		5. 1 The number of aliquots to be created is preset. If needed, the user can edit this value and then click on "Set aliquots to create" to make the changes efective.
		5. 2 It is possible to set the concentration value for the left over checking the option "Set left over concentration". This allows to have the same concentrations for all the N aliquots.
		5. 3 One may avoid to create the spare aliquot by unchecking "Save left over". In such a way N-1 aliquots will be saved and the spare material in the dilution can be discarded.

**Aliquots-> Derive-> Automated derivation-> 5. Create derivatives**
