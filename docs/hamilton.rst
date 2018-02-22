********
Hamilton
********

Derivation
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
	4. Mark as *"Failed"* the unsuccessful derivations.
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

		a. The number of aliquots to be created is preset. If needed, the user can edit this value and then click on "Set aliquots to create" to make the changes effective.
		b. It is possible to set the concentration value for the left over checking the option "Set left over concentration". This allows to have the same concentrations for all the N aliquots.
		c. One may avoid to create the spare aliquot by un-checking "Save left over". In such a way N-1 aliquots will be saved and the spare material in the dilution can be discarded.
		d. Set volume and concentration of each son-aliquot.
		e. Allocate one slot for each son-aliquot (left over included). This is the physical space inside the robot, containing the plate to store the son-aliquots.
		f. Click on "Apply Values".
		g. If the allotted material to derive new aliquots is not sufficient for the inserted values of volume and concentration, the samples rows will turn red. Otherwise the derivation can succeed and the row will appear green. **Be aware that the red lines have to be manually corrected**.
		h. To edit the affected row just click on it. A box will pop up showing the details of each single son-aliquot to look for. As in general planning, it is possible to change the number of aliquots to be created, set the concentration for the left over or to not save it at all. All the modifications made using this window will affect **only** the associated sample, without changing values for any of the others.
		i. The red-framed textboxes indicate an error, therefore their value must be modified. After editing the values of volume and concentration, click on "Recalculate Values" to compute values again.
		j. To confirm changes click "Ok" in the dialog box, otherwise click "Cancel" to discard all manual changes.
		k. Click on "Submit Data" to confirm all data and terminate the planning of the dilution procedure.

At this point the dilution has been effectively planned. Run the application in the robot environment to execute it materially.

**Aliquots-> Derive-> Automated derivation-> 5. Create derivatives**

Execute this procedure when the robot has finished the dilution steps.
the dedicated view lists the original derivated samples (and not the son-aliquots). The only required action is to click on "Submit Data" to let the LAS acquire the data produced by the robot. It then creates the derived aliquots according to their values of concentration and volume.

.. note:: 
	Looking under the hood, the code to implement all the views is located in ``tissue/derivation_robot.py`` and in ``tissue/derived.py``. The .js files are archived in ``tissue/tissue_media/JS/derive_robot``, while the .html's are in ``tissue/Templates/tissue2/derive_robot``.

The reference database is *hamilton* and it is linked to the application in ``settings.py``


Quantification
##############

**Aliquots-> Perform QC/QA-> Plan revaluation**

The first step is always the planning. In this scenario it has not been modified, so the required data are always the same.

**Aliquots-> Perform QC/QA-> Automated revaluation-> Plan**

Enter in this procedure when you have positioned all the samples in the right place inside the robot.

The view shows the planned samples that need to be validated as usual reading their barcodes. Then, after clicking "Next Step" a new screen (very similar to the derivation one) appears.

The steps to be followed are:

	1. Select the quantification protocols to be executed.
	2. Insert a procedure name (letters and numbers allowed), to be used as a reference when the procedure will be executed by the robot.
	3. Choose the container type to archive the samples.
	4. Click "Confirm".

Now the quantification is planned, so run the robot application to make it effective.

**Aliquots-> Perform QC/QA-> Automated revaluation-> Acquire data and save**

This procedure stores in the LAS system the values of the quantifications made by the Hamilton robot.

In case you applied both fluorescence and UV techniques, two different concentration values appear. Hence, the user must choose the values to be assigned to the sample. 

.. warning:: The numerical values are both saved in the LAS, but **only the selected one** accounts for the operational sample concentration.

To do so, just check one of the two checkboxes at the end of each line. Otherwise, if you want to use always the same kind of measurement, check the textbox in the table header.

Then, click on "Submit Data" to confirm.

.. note:: 
	You can find the code to design these views in ``tissue/revalue_robot.py`` and ``tissue/revalue.py``. 
	The .js files are in ``tissue/tissue_media/JS/revalue_robot``, while the .html's are in ``tissue/Templates/tissue2/revalue_robot``.


Split
#####

**Aliquots-> Split-> Plan splitting**

As usual, the first step is always to plan. In this scenario, the procedure has not been modified, so the required data are the same.

**Aliquots-> Split-> Automated splitting-> Plan**

You access to this procedure when you have positioned all the samples in the right place inside the robot.

The view shows the planned samples that need to be validated as usual reading their barcodes. Then, after clicking "Next Step" a new screen (very similar to the derivation one) appears.

Here are the steps to follow:

	1. Insert the procedure name and the container type with the samples to dilute.
	2. If any, set the unsuccessful procedures or those you want to abort, checking the voice "Stop Procedure".
	3. Set the values for the new aliquots:

		a. The number of aliquots to be created is pre-computed. If needed, the user can edit this value and then click on "Set aliquots to create" to make the changes effective.
		b. Set volume and concentration of each son-aliquot.
		c. Allocate one slot for each son-aliquot (left over included). This is the physical space inside the robot, containing the plate to store the new aliquots.
		d. Click on "Apply Values".
		e. The "Remaining Volume" column, reports the residual volume in the original instance, after the sampling.
		f. If the allotted material to derive new aliquots is not enough for the inserted values of volume and concentration, the samples rows will appear red. Otherwise the derivation can succeed and the row will turn green. **Be aware that the red lines have to be manually corrected**
		g. To modify the affected row just click on it. A box will pop up showing the details of each single derived aliquot to look for. As in general planning, it is possible to change the number of aliquots to be created. All the modifications made using this window will affect **only** the associated sample, without changing values for any of the others.
		h. The red-framed cells indicate an error, therefore their value must be modified. After editing the values of volume and concentration, click on "Recalculate Values" to compute values again.
		i. To confirm changes click "Ok" in the dialog box, otherwise click "Cancel" to discard all manual changes.

	4. Check "Exhausted" in each row the where sample is totally consumed after the the splitting process.
	5. Click on "Submit Data" to confirm all data and terminate the dilution planning.

At this point the dilution has been correctly planned and you just need to run the robot application to physically execute it.

**Aliquots-> Split-> Automated splitting-> Acquire data and save**

This procedure must be run when the robot finishes the dilution process.

This screen shows the original samples that have been diluted (and not the son aliquots). The only required action is to click on "Submit Data" in order to send the data produced by the robot to the LAS system. Thereafter, the system creates the new aliquots with the associated concentrations and volumes.

.. note:: 
	The code to design these views is located in in ``tissue/split_robot.py`` and ``tissue/split.py``. 
	The .js files are in ``tissue/tissue_media/JS/split_robot``, while the .html's are in ``tissue/Templates/tissue2/split_robot``.