*************************
Storage module
*************************

Within the LAS, a “container” is defined as anything that is able to contain biological entities (i.e. samples) or other containers. Thus, we can think of the repository as a tree, where root containers
contain some objects, which in turn contain more objects. The nesting continues until the leaves of this tree are reached, representing biological samples.

All LAS containers are divided into four categories (generic types), listed from the root of each container hierarchically, we have:

+-------------------+
| Container Type    |
+===================+
| Freezer/Cabinet   |
+-------------------+
|  Rack/Drawer      |
+-------------------+
|    Plate/Box      |
+-------------------+
| Tube/Bio-Cassette |
+-------------------+


Although this order reflects the actual containment relationships (e.g. a freezer contains a rack), a few exceptions possible as well.
For example, a plate may directly contain some biological entities (rather than tubes). Every category is further divided into more container types.

For every container type a series of mutual interactions (i.e., which container type can host another one) has been defined, which can vary according to characteristics such as the layout or the laboratory procedure. For instance, a plate of a given manufacture and model may be able to host only some kind of tubes.
For every container type, several container instances exist, univocally identified by a barcode.

Every container has a geometry, defined by the number of its rows and columns. The geometry is used to visually render the container in the user interfaces. Furthermore every container can contain one or more types of biological entities (e.g. viable, RNA later, snap-frozen), which are tracked by the system.

Common operations for the help desk
###################################

Here is a collection of some of the most representative requests received by the LAS Help Desk.

Plate position change
*********************
Here the user basically requires to change the position of a plate within the father container. The steps described below holds as well for the insertion of a new plate.
Indeed, this is just a matter of updating the container, assigning a new ``idFatherContainer`` to the current position in which the plate is located.

Open the mysql shell and use the ``Storage`` database.

.. code:: sql

	mysql> use storage;

Thereafter, comes the update operation:

.. code:: sql

	mysql> update container set idFatherContainer = 'current_FatherID' position = 'current_position' where id = 'New_ID';

Now, to ensure coherence we must set to NULL both the ``idFatherContainer`` and the ``position`` attributes of the current position using an update query similar to the previous one.

As an immediate sanity check, we can simply control if the just-created container is actually full. To do so, it is enough to check the attribute ``full`` of the ``Container`` table (having value 1 if full, 0 if empty).

.. code:: sql
	
	mysql> select * from container where id = New_ID;

Resulting in the row depicted below.

+------------+-----------------+----------------------+------------+----------------------+---------+--------------+------+-------+---------+--------+
| id         | idContainerType | idFatherContainer    | idGeometry | position             | barcode | availability | full | owner | present | oneUse |
+============+=================+======================+============+======================+=========+==============+======+=======+=========+========+
| **New_ID** |               2 | **current_FatherID** |          2 | **current_position** | 01|D3   |            1 |  *1* | NULL  |       1 |      1 |
+------------+-----------------+----------------------+------------+----------------------+---------+--------------+------+-------+---------+--------+


Tubes swap
**********
Sometimes the user may want to swap the position of two tubes within the same plate.

Suppose to have the following scenario:
	- Tube **T101** in plate **P1**, position **POS_1**
	- Tube **T202** in plate **P1**, position **POS_2**

and want to swap the tubes. Please notice that both T101 and T202 are genealogyIDs.

Open the mysql shell and use the ``Storage`` database.

.. code:: sql

	mysql> use storage;

Then let's find the aliquot with the corresponding genealogyID. We basically select ``id_container``, ``barcode``, ``position``, ``id_aliquot`` and ``genealogyID`` where genealogyIDs T1 and T2 appears. Here we just look for a part of the genealogyID since it may be partial, or omitted in some parts by the user who may not specify the full GenealogyID code.

.. code:: sql

	mysql> select c.id, c.barcode, c.position, c1.barcode, a.id, genealogyID  from container c, container c1,  aliquot a where c.id=a.idContainer and c.idFatherContainer=c1.id and genealogyID like 'T1%';

+--------+------------+----------+------------+--------+----------------------------+
| id     | barcode    | position | barcode    | id     | genealogyID                |
+========+============+==========+============+========+============================+
| 258510 | NUKQ066731 | A11      | P2         | 387165 | T103                       |
+--------+------------+----------+------------+--------+----------------------------+
| 258511 | NUKQ066740 | A12      | P2         | 387166 | T105                       |
+--------+------------+----------+------------+--------+----------------------------+
| 258462 | NUKQ062425 | E12      | P3         | 387169 | T115                       |
+--------+------------+----------+------------+--------+----------------------------+
| 258273 | NUJC349169 | F5       | P1         | 387453 | T109                       |
+--------+------------+----------+------------+--------+----------------------------+
| 258274 | NUJC349178 | F6       | P1         | 387454 | T114                       |
+--------+------------+----------+------------+--------+----------------------------+
| 258275 | NUJC349187 | **POS_1**| **P1**     | 388631 | **T101**                   |
+--------+------------+----------+------------+--------+----------------------------+

And the same for tube **T2**.
Then we look at the content of positions **POS_1** and **POS_2** in plate **P1**.

.. code:: sql

	mysql> select c.id, c.barcode, c.position, c1.barcode, a.id, genealogyID  from container c, container c1,  aliquot a where c.id=a.idContainer and c.idFatherContainer=c1.id and c1.barcode = 'P1' and c.position in ('POS_1', 'POS_2');

+--------+------------+----------+------------+--------+----------------------------+
| id     | barcode    | position | barcode    | id     | genealogyID                |
+========+============+==========+============+========+============================+
| 258275 | NUJC349187 | **POS_1**| **P1**     | 388631 | **T101**                   |
+--------+------------+----------+------------+--------+----------------------------+
| 258286 | NUJC349293 | **POS_2**| **P1**     | 388642 | **T102**                   |
+--------+------------+----------+------------+--------+----------------------------+

Now it is just a matter of swapping the tubes positions (by means of a ``GenealogyID`` and ``idContainer`` swaps, which is simpler than operate on positions within a plate).

In case of a plate change, just go to table ``Container`` update the ``idFatherContainer`` as well.


Delete a tube
*************
This help desk case deals with the database operations to reflect the physical elimination of a container (a tube in this example) and its content.
Suppose that we want to delete tube whose barcode is 'P00001'.

As a preliminary operation we need to identify the tube content and make it unavailable to users. We do so interacting with both ``Storage`` and ``Biobank`` database:

.. code:: sql

	mysql> use storage;
	mysql> select * from biobanca.aliquot where uniqueGenealogyID in (select genealogyID from aliquot where idContainer in (select id from container where barcode='P00001'));

+--------+---------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| id     | barcodeID     | uniqueGenealogyID          | idSamplingEvent | idAliquotType | availability | timesUsed | derived | archiveDate |
+========+===============+============================+=================+===============+==============+===========+=========+=============+
| 155199 | **P00001**    | GBCNNDQBLH0000000000VT0100 |           00001 |             1 |            1 |         0 |       0 | 2019-06-10  |
+--------+---------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+

.. code:: sql

	mysql> update biobanca.aliquot set availability=0 where uniqueGenealogyID='GBCNNDQBLH0000000000VT0100';

Now it's time to delete the tube itself and to do so we exploit the containerID and barcode from the container table. 
We run the following query in this specified order to not violate any key constraint.

.. code:: sql

	mysql> delete from containerfeature where idContainer in (select id from container where barcode='P00001');
	mysql> delete from aliquot where idContainer in (select id from container where barcode='P00001');
	mysql> delete from container where barcode='P00001';

From now on, there is no more evidence of any container, since there is no aliquot or container in the Storage database but the aliquot is still alive in the graph. We have to clean after ourself in both graph and ``Biobank`` database.
We need to delete the relation between the aliquot and its WG, and the aliquot features from the ``aliquotfeature`` table. Only after these two operations we can delete the physical aliquot.

.. code:: sql

	mysql> use biobanca;
	mysql> delete from aliquot_wg where id_aliquot in (select id from aliquot where barcodeID='P00001');
	mysql> delete from aliquotfeature where idAliquot in (select id from aliquot where barcodeID='P00001');
	mysql> delete from aliquot where barcodeID='P00001';

On graph we delete the aliquot running the following query:

.. code:: cypher

	match (n:Aliquot {identifier: 'CRCNNDQBLH0000000000VT0100'}) detach delete n

Plate compatibility problems
****************************
At the moment of container creation, for every new instance the user must specify the type of biological content that this container will be allowed to contain.
It may happen sometimes, that users needs to modify such list afterwards when they move aliquots from a container to another.

There is a dedicated functionality to accomplish this task called "Change container features" available at /storage/plate/change/. In some cases anyhow, the user cannot act directly on the biological content types. That usually happens when some of the father containers does not allow a certain type of content by default (i.e. at the moment of creation). For this reason, a database update is required.

Let's see the example of making plate Plate01 compatibile with rack Rack99

Opening the mysql shell use the ``Storage`` database.

.. code:: sql

	mysql> use storage;

First of all identify the plate within the container table.

.. code:: sql

	mysql> select * from container where barcode='Plate01';

+--------+-----------------+-------------------+------------+----------+------------+--------------+------+-------+---------+--------+
| id     | idContainerType | idFatherContainer | idGeometry | position | barcode    | availability | full | owner | present | oneUse |
+========+=================+===================+============+==========+============+==============+======+=======+=========+========+
| 260128 |              16 |              NULL |         13 |          |   Plate01  |            1 |    1 | NULL  |       1 |      0 |
+--------+-----------------+-------------------+------------+----------+------------+--------------+------+-------+---------+--------+

We can see that ``idContainerType``=16, and querying the table ``genericcontainertype`` we should see its generic container type.

.. code:: sql

	mysql> select * from genericcontainertype where id in (select idGenericContainerType from containertype where id=16);

+----+-----------+--------------+
| id | name      | abbreviation |
+====+===========+==============+
|  3 | Plate/Box | plate        |
+----+-----------+--------------+

Now that we have confirmation of the container type, we can go on checking the content type of both container (the plate) and its father container (the rack i this case).

.. code:: sql

	mysql> select value from containerfeature where idContainer in (select id from container where barcode='Plate01') and value not in (select value from containerfeature where idContainer in (select * from container where barcode='Rack99'));

+-------+
| value |
+=======+
| FR    |
| FS    |
+-------+

So there are two content types (Frozen and Frozen Sediment) that can be contained by the plate, but not by its father container.
To overcome this problem, we perform an insert into the containerfeature table, remembering the father contaienr ID.

.. code:: sql

	mysql> insert into containerfeature (idFeature,idContainer,value) values (1,ID_Rack99,'FR'),(1,ID_Rack99,'FF');

In case the previous query would return an empty set (i.e. both container and its father can contain the same types), there is another table we should check.

Modify plate geometry
*********************
This procedure allows to change the geometry configuration of the containers within a plate.

Opening the mysql shell use the ``Storage`` database.

.. code:: sql

	mysql> use storage;

Then, access table ``Geometry``, that contains all the geometries created so far upon users requests.

.. code:: sql
	
	mysql> select id, name from geometry;

The following rows depicts a portion of the table, storing just the geometry ID along with its name.

+-----+-------+
| id  | name  |
+=====+=======+
|   2 | 1x1   |
+-----+-------+
|   4 | 28x4  |
+-----+-------+
|  13 | 8x12  |
+-----+-------+
|  14 | 4x6   |
+-----+-------+
|  15 | 2x3   |
+-----+-------+

From this table take note of the id's related to the geometry names we are interested in (i.e. the current one and the new one).

Using the table ``Container`` we have to change the geometry id. To do so, run a simple update query involving the Container ID whose geometry we want to edit.

The table is depicted below and it is sufficient to update the ``idGeometry`` field.

.. code:: sql

	mysql> select * from container where id = 'ContainerID';

+-----------------+-----------------+-------------------+------------+----------+-----------------------+--------------+------+-------+---------+--------+
| id              | idContainerType | idFatherContainer | idGeometry | position | barcode               | availability | full | owner | present | oneUse |
+=================+=================+===================+============+==========+=======================+==============+======+=======+=========+========+
| **ContainerID** |               1 |              NULL |      **4** |          | -80C COC Sanyo04 28x4 |            1 |    0 |       |       1 |      0 |
+-----------------+-----------------+-------------------+------------+----------+-----------------------+--------------+------+-------+---------+--------+


.. warning::  Beware that in this scenario we are not modifying the content of a plate at all. We are just changing the geometry configuration. Whether it is relatively safe to change in favor of a bigger geometry (i.e. we expand the plate), it is much more critical when the plate is shrunk. For this reason it is useful to perform a sanity check on data and on the content of a plate.


Change container name
*********************

Here we exemplify the name-change procedure using the case in which a user wants to change the name of a rack from **old_barcode** to **new_barcode**. 
A rack is usually barcoded manually by operators, hence they may require to change their barcodes according to tattributeheir needs.

To do so go to we use the ``Storage`` database.

.. code:: sql

	mysql> use storage;

Then, we firstly identify the rack, i.e. the container whose barcode matches the one specified by the user. Thereafter, we edit the container barcode with its new name via an update query.

.. code:: sql

	mysql> select * from Container where barcode = 'old_barcode';
	mysql> update Container set barcode = 'new_barcode' where barcode = 'old_barcode';

+--------+-----------------+-------------------+------------+----------+-----------------+--------------+------+-------+---------+--------+
| id     | idContainerType | idFatherContainer | idGeometry | position | barcode         | availability | full | owner | present | oneUse |
+========+=================+===================+============+==========+=================+==============+======+=======+=========+========+
| 268241 |              26 |              NULL |         72 |          | **old_barcode** |            1 |    0 | NULL  |       1 |      0 |
+--------+-----------------+-------------------+------------+----------+-----------------+--------------+------+-------+---------+--------+


The barcode is unique inside the system. The next operation is to insert the *father container* associated to the containers belonging to the one we just modified. Firstly it is useful to check the status of some of that containers via a simple select query. The ``FatherContainer`` field has to be updated from the NULL value to that of **new_barcode**. 

Change archiviation date
************************

The following operations exemplify a simple data change in the stored samples.
Suppose to have this scenario, with some imaginary values ad dates and a user asking to correct the archiviation date of the selected samples.

+-----------+-------------------+-------------------+------------+------------------+
| Cell Line | Aliquot           | Sample Barcode    | Wrong Date | Correct Date     |
+===========+===================+===================+============+==================+
| CL001     | Aliquot_GenID_1   | Barcode_1         | 09/05/2018 | 24/04/2018       | 
+-----------+-------------------+-------------------+------------+------------------+
| CL002     | Aliquot_GenID_2   | Barcode_2         | 09/05/2018 | 24/04/2018       | 
+-----------+-------------------+-------------------+------------+------------------+

Such changes require to act on three databases: ``Cells``, ``Storage`` and ``Biobanca``.
The first updates are on ``Cells``.

.. code:: sql

	mysql> use cells;


The dates we are about to modify are located under the attribute *application_date* of table ``archive_details``. In order to inspect them it is enough to issue a nested query on the *id* attribute, referenced by *archive_details_id* of table ``Aliquots``.

.. code:: sql

	mysql> select * from archive_details where id in (select archive_details_id from aliquots where gen_id in ('Aliquot_GenID_1','Aliquot_GenID_2'));

+------+------------------------+-----------+--------+-------------------------+
| id   | experiment_in_vitro_id | events_id | amount | application_date        |
+======+========================+===========+========+=========================+
| 8801 |                   NULL |     99927 |      9 | **2018-05-09 13:37:42** |
+------+------------------------+-----------+--------+-------------------------+
| 8802 |                   NULL |     99925 |      9 | **2018-05-09 13:37:42** |
+------+------------------------+-----------+--------+-------------------------+

Then we run an update querry correcting the wrong application dates. Remember to be coherent with the specified format, inserting a well-formed data (i.e. providing the application hour after the date).

.. code:: sql

	mysql> update archive_details set application_date='2018-04-24 13:37:42' where id in (select archive_details_id from aliquots where gen_id in ('Aliquot_GenID_1','Aliquot_GenID_2'));


Therefore, we have to inspect the ``Storage`` db to correct another timestamp associated to our aliquots.