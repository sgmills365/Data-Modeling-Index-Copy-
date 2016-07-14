.. index:: index; multikey
.. _index-type-multi-key:
.. _index-type-multikey:

================
Multikey Indexes
================

.. default-domain:: mongodb

.. contents:: On this page
   :local:
   :backlinks: none
   :depth: 1
   :class: singlecol

For indexing a field that holds an array value, MongoDB creates an index
key for each element in the array. These *multikey* indexes support
efficient queries against array fields. Multikey indexes can be
constructed over arrays that hold both scalar values (e.g. strings,
numbers) *and* nested documents.

.. include:: /images/index-multikey.rst

Create Multikey Index
---------------------

To create a multikey index, use the
:method:`db.collection.createIndex()` method:

.. code-block:: javascript

   db.coll.createIndex( { <field>: < 1 or -1 > } )

MongoDB automatically creates a multikey index if any indexed field is
an array; you do not need to necessarily  specify the multikey type.

Index Bounds
------------

If an index is multikey, then computation of the index bounds follows
special rules. For details on multikey index bounds, see
:doc:`/core/multikey-index-bounds`.

Limitations
-----------

Compound Multikey Indexes
~~~~~~~~~~~~~~~~~~~~~~~~~

For a :ref:`compound <index-type-compound>` multikey index, each
indexed document can have *at most* one indexed field whose value is an
array. As such, you cannot create a compound multikey index if more than one
to-be-indexed field of a document is an array. Or, if a compound
multikey index already exists, you cannot insert a document that would
violate this restriction.

For example, consider a collection that contains the following document:

.. code-block:: javascript

   { _id: 1, a: [ 1, 2 ], b: [ 1, 2 ], category: "AB - both arrays" }

You cannot create a compound multikey index ``{ a: 1, b: 1 }`` on the
collection since both the ``a`` and ``b`` fields are arrays.

But consider a collection that contains the following documents:

.. code-block:: javascript

   { _id: 1, a: [1, 2], b: 1, category: "A array" }
   { _id: 2, a: 1, b: [1, 2], category: "B array" }

A compound multikey index ``{ a: 1, b: 1 }`` is permissible since for
each document, only one field indexed by the compound multikey index is
an array; i.e. no document contains array values for both ``a`` and
``b`` fields. After creating the compound multikey index, if you
attempt to insert a document where both ``a`` and ``b`` fields are
arrays, MongoDB will fail the insert.

Shard Keys
~~~~~~~~~~

You **cannot** specify a multikey index as the shard key index.

.. versionchanged:: 2.6

   However, if the shard key index is a :ref:`prefix
   <compound-index-prefix>` of a compound index, the compound index is
   allowed to become a compound *multikey* index if one of the other
   keys (i.e. keys that are not part of the shard key) indexes an
   array. Compound multikey indexes can have an impact on performance.

Hashed Indexes and Covered Queries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Hashed </core/index-hashed>` indexes **cannot** be multikey. 
As well, multikey indexes cannot support a covered query. 


Query on the Array Field as a Whole
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a query filter specifies an :ref:`exact match for an array as a
whole <array-match-exact>`, MongoDB can use the multikey index to look
up the first element of the query array but cannot use the multikey
index scan to find the whole array. Instead, after using the multikey
index to look up the first element of the query array, MongoDB
retrieves the associated documents and filters for documents whose
array matches the array in the query.

For example, consider an ``inventory`` collection that contains the
following documents:

.. code-block:: javascript

   { _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
   { _id: 6, type: "food", item: "bbb", ratings: [ 5, 9 ] }
   { _id: 7, type: "food", item: "ccc", ratings: [ 9, 5, 8 ] }
   { _id: 8, type: "food", item: "ddd", ratings: [ 9, 5 ] }
   { _id: 9, type: "food", item: "eee", ratings: [ 5, 9, 5 ] }

The collection has a multikey index on the ``ratings`` field:

.. code-block:: javascript

   db.inventory.createIndex( { ratings: 1 } )

The following query looks for documents where the ``ratings`` field is
the array ``[ 5, 9 ]``:

.. code-block:: javascript

   db.inventory.find( { ratings: [ 5, 9 ] } )

MongoDB can use the multikey index to find documents that have ``5`` at
any position in the ``ratings`` array. Then, MongoDB retrieves these
documents and filters for documents whose ``ratings`` array equals the
query array ``[ 5, 9 ]``.

Examples
--------

Index Basic Arrays
~~~~~~~~~~~~~~~~~~

Consider a ``survey`` collection with the following document:

.. code-block:: javascript

   { _id: 1, item: "ABC", ratings: [ 2, 5, 9 ] }

Create an index on the field ``ratings``:

.. code-block:: javascript

   db.survey.createIndex( { ratings: 1 } )

Since the ``ratings`` field contains an array, the index on ``ratings``
is multikey. The multikey index contains the following three index
keys, each pointing to the same document:
 ``2``, ``5``, and ``9``.

Index Arrays with Embedded Documents
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create multikey indexes on array fields that contain nested
objects.

Consider an ``inventory`` collection with documents of the following
form:

.. code-block:: javascript

   {
     _id: 1,
     item: "abc",
     stock: [
       { size: "S", color: "red", quantity: 25 },
       { size: "S", color: "blue", quantity: 10 },
       { size: "M", color: "blue", quantity: 50 }
     ]
   }
   {
     _id: 2,
     item: "def",
     stock: [
       { size: "S", color: "blue", quantity: 20 },
       { size: "M", color: "blue", quantity: 5 },
       { size: "M", color: "black", quantity: 10 },
       { size: "L", color: "red", quantity: 2 }
     ]
   }
   {
     _id: 3,
     item: "ijk",
     stock: [
       { size: "M", color: "blue", quantity: 15 },
       { size: "L", color: "blue", quantity: 100 },
       { size: "L", color: "red", quantity: 25 }
     ]
   }

   ...

The following operation creates a multikey index on the ``stock.size``
and ``stock.quantity`` fields:

.. code-block:: javascript

   db.inventory.createIndex( { "stock.size": 1, "stock.quantity": 1 } )

The compound multikey index can support queries with predicates that
include both indexed fields as well as predicates that include only the
index prefix ``"stock.size"``, as in the following examples:

.. code-block:: javascript

   db.inventory.find( { "stock.size": "M" } )
   db.inventory.find( { "stock.size": "S", "stock.quantity": { $gt: 20 } } )

For details on how MongoDB can combine multikey index bounds, see
:doc:`/core/multikey-index-bounds`. For more information on behavior of
compound indexes and prefixes, see :ref:`compound indexes and prefixes
<compound-index-prefix>`.

The compound multikey index can also support sort operations, such as
the following examples:

.. code-block:: javascript

   db.inventory.find( ).sort( { "stock.size": 1, "stock.quantity": 1 } )
   db.inventory.find( { "stock.size": "M" } ).sort( { "stock.quantity": 1 } )

> For more information on behavior of compound indexes and sort
operations, see :doc:`/tutorial/sort-results-with-indexes`.

.. class:: hidden

   .. toctree::
      :titlesonly:

      /core/multikey-index-bounds
