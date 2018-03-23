*****************
Cell Lines Module
*****************

The LAS allows the collection of new cell lines. The user must insert the tumor type, the cell line name and, optionally, a datasheet file and an invoice file. In the next interface, other parameters must be defined, as well as the position within the storage in which the line is going to be archived.


Cell Lines seemingly not shared
###############################
These simple queries allow to check if a cell line has been effectively shared between two different workgroups.

First of all open the mysql shell and use the ``Cells`` database. Then, access the table ``Cells`` containing all the general information about the cell lines generated so far.

Supposing to know the Genealogy_ID of the cell line apparently not shared, we retrieve all the info 

.. code:: sql
	
	mysql> use cells;
	mysql> select * from cells where genID ='Gen_ID';

+--------+----------------------------+----------+--------+----------------------+
| id     | genID                      | nickname | nickid | expansion_details_id |
+========+============================+==========+========+======================+
| *0123* | **Gen_ID**		      | abcde    |        |                 7823 |
+--------+----------------------------+----------+--------+----------------------+

Then, we look for all those cell lines in the ``Cells_WG`` table, coupling each cell line with its related working groups.
In particular we are interested in all those cell lines having the *gen_ID* retrieved before as *id_cell* attribute.

.. code:: sql
	
	mysql> select * from cells_wg where id_cell='0123';

+-------+----------+-------+
| id    | id_cell  | WG_id |
+=======+==========+=======+
| 24743 | **0123** |   *4* |
+-------+----------+-------+
| 24744 | **0123** |   *3* |
+-------+----------+-------+

.. note:: You can retrieve such data as well with a single query like:
	
	.. code:: sql
	
		select * from cells_wg where id_cell in (select id from cells where genID ='Gen_ID');


.. code:: sql
	
	mysql> select * from cellline_wg where id in (3, 4);

+----+-------------+----------+
| id | name        | owner_id |
+====+=============+==========+
|  3 | First_WG    |       98 |
+----+-------------+----------+
|  4 | Second_WG   |       99 |
+----+-------------+----------+

The cell line results already shared since it result associated to the desired WG's.

.. note:: The message "not shared entity", sometimes refers just to a cell line that has already been shared previously.
