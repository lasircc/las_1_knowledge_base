*********************
The Xenografts module
*********************

The *Xenografts module* is responsible for all the operations related to Xeno experiments. Blah Blah Blah...






The Implant
###########

- **Process description** some aliquots are implanted on a biological mouse bound to physical mouse.
- **DBMS involved**: Neo4j, MySQL.





The Explant
###########

- **Process description** some aliquots are created from a biological mouse. The relative physical mouse is sacrificed.
- **DBMS involved**: Neo4j, MySQL

Data created

In the biobanca a new record for each aliquot is created in the table ``biobanca.aliquot``. Each aliquot has a reference to the relative sampling event (e.g., ``54398``).

.. code:: sql

    mysql> select * from biobanca.aliquot where uniqueGenealogyID like 'CRC0177LMX0C03005%';


+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| id     | barcodeID  | uniqueGenealogyID          | idSamplingEvent | idAliquotType | availability | timesUsed | derived | archiveDate |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293941 | 20334      | CRC0177LMX0C03005TUMFF0100 |           54398 |             4 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293942 | 1156025086 | CRC0177LMX0C03005TUMRL0100 |           54398 |             3 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293943 | 1156025087 | CRC0177LMX0C03005TUMRL0200 |           54398 |             3 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293944 | 1156025088 | CRC0177LMX0C03005TUMRL0300 |           54398 |             3 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293945 | 1156025089 | CRC0177LMX0C03005TUMRL0400 |           54398 |             3 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293946 | 1156004068 | CRC0177LMX0C03005TUMSF0100 |           54398 |             2 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293947 | 1156004069 | CRC0177LMX0C03005TUMSF0200 |           54398 |             2 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293948 | 1156004070 | CRC0177LMX0C03005TUMSF0300 |           54398 |             2 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+
| 293949 | 1156004058 | CRC0177LMX0C03005TUMSF0400 |           54398 |             2 |            1 |         0 |       0 | NULL        |
+--------+------------+----------------------------+-----------------+---------------+--------------+-----------+---------+-------------+


The sampling event has a reference to a collection. The collection record is not usually created during the explant session. Therefore, if you need to delete explanted aliquots, **do not delete** the collection. The sampling event has a reference to a series.

.. code:: sql

    mysql> select * from biobanca.samplingevent where id = 54398;


+-------+--------------+--------------+----------+---------+--------------+
| id    | idTissueType | idCollection | idSource | idSerie | samplingDate |
+-------+--------------+--------------+----------+---------+--------------+
| 54398 |            2 |          148 |        7 |   12081 | 2018-02-02   |
+-------+--------------+--------------+----------+---------+--------------+


The series...

.. code:: sql

    mysql> select * from biobanca.serie where id = 12081;

+-------+-------------------+------------+
| id    | operator          | serieDate  |
+-------+-------------------+------------+
| 12081 | user.name         | 2018-02-02 |
+-------+-------------------+------------+

Besides, each aliquot has features in ``biobanca.aliquotfeature``

.. code:: sql

    mysql> select * from biobanca.aliquotfeature where idAliquot in (select id from aliquot where uniqueGenealogyID like 'CRC0177LMX0C03005%');


+--------+-----------+-----------+-------+
| id     | idAliquot | idFeature | value |
+--------+-----------+-----------+-------+
| 462254 |    293941 |         4 |     1 |
+--------+-----------+-----------+-------+
| 462255 |    293942 |         3 |     1 |
+--------+-----------+-----------+-------+
| 462256 |    293943 |         3 |     1 |
+--------+-----------+-----------+-------+
| 462257 |    293944 |         3 |     1 |
+--------+-----------+-----------+-------+
| 462258 |    293945 |         3 |     1 |
+--------+-----------+-----------+-------+
| 462259 |    293946 |         2 |     1 |
+--------+-----------+-----------+-------+
| 462260 |    293947 |         2 |     1 |
+--------+-----------+-----------+-------+
| 462261 |    293948 |         2 |     1 |
+--------+-----------+-----------+-------+
| 462262 |    293949 |         2 |     1 |
+--------+-----------+-----------+-------+


In the storage:...



In neo4j: ...



Common operations for the help desk
###################################

Users usually request some of the following operations.

Adding aliquots to a performed explant
**************************************

**Problem**: After an explant session, the mouse is not available anymore and the user cannot attach new explanted aliquots.

You can load other aliquots in the following way:

- Go to ``Biological Experiments`` > ``Xenografts`` > ``Batch``
- Select ``Add aliquots to explant``
- Load the file with new aliquots. Click on ``Template`` to get the required format.

Recovering an explanted mouse
*****************************

...

Recreate an implant
*******************

Here we exemplify the implant recovery procedure to restore the implant-related data to a consistent state in the database. The operations we are going to explain here are useful in all those cases in which the save operation has failed for whatever reason.
First of all, we identify the experimental series the implant belongs to. In detail, there is a one-to-many relationship between a series and their related sampling events.

.. code:: sql

	query to identify the experimental series

Once the experimental series has been identified, we use the ``Storage`` database

.. code:: sql
	
	mysql> use storage;

Since the aliquots can be collected in multiple fashions, they must be treated accordingly during this recreation procedure.

	- **FFPE blocks:** delete both the container and the aliquot.
	- **Tubes:** make them available again (empty them setting attribute *full*=0 in the ``Container`` table) and delete the associated aliquot.

In order to access the containers related to the explants, we use their *GenealogyIDs* in the table ``Container``

.. code:: sql
	
	mysql> update Container set full=0 where id in (select idContainer from Aliquot where GenealogyID like 'Gen_ID%');

To delete the FFPE block, we delete its record in the ``Contanier`` table, along with the associated information in the ``ContainerFeature`` table.

.. code:: sql

	mysql> delete from containerfeature where idContainer in (select idContainer from Aliquot where GenealogyID like 'Gen_ID%');

To simply delete an aliquot an automatic procedure is available at the URL http://las.ircc.it/biobank/canc/aliquot. Please refer to  :ref:`deleting_an_aliquot` section.
