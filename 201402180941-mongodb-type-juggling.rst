Type juggling with MongoDB
==========================

.. articleMetaData::
   :Where: London, UK
   :Date: 2014-02-18 09:41 Europe/London
   :Tags: blog, php, mongodb
   :Short: mdb-typed

As PHP developer you likely know that all GET and POST variables are
represented as strings through the ``$_GET`` and ``$_POST`` super globals.
PHP's weak typing system allows you to do calculations with numbers that are
stored in strings as well as with normal numbers. For example the following
works just fine, and outputs ``131.88``::

	<?php
	$_GET['life'] = "42";
	echo $_GET['life'] * "3.14", "\n";
	?>

Similarly we can compare a number stored in a string with a real number just
as easily::

	<?php
	$a = "1701";
	$b = 1701;

	echo $a == $b ? "the same" : "similar", "\n";
	?>

Which echos ``the same``.

When storing data in a relational database, you should always take care of
escaping data, or, of course use prepared statements. As an example, you would
store the same GET parameter from above with something like::

	<?php
	$db = new PDO( "mysql:host=localhost;dbname=test" );
	$stmt = $db->prepare( "INSERT INTO test(value) VALUES(?)" );
	$stmt->bindparam( 1, $_GET['life'] );
	$stmt->execute();
	?>

Because the arguments to prepared statements are not sent as part of the
string, the database will just cast it to the type that is defined in the
database through ``CREATE TABLE``.

And to select this record again, we can run a ``SELECT`` query, again either
with the number in the string, or as a PHP integer::

	<?php
	$db = new PDO( "mysql:host=localhost;dbname=test" );
	$stmt = $db->prepare( "SELECT * FROM TEST WHERE value = ?" );
	$stmt->bindparam( 1, "42" );
	// or
	$stmt->bindparam( 1, 42 );
	$stmt->execute();
	?>

And in both cases it will find the record, again, because the database knows
which type the value is stored at through it's table schema.

In SQL prepared statements are required to prevent SQL injections. If we
look at MongoDB, you will see that because we are not creating an SQL string
to be executed against the database, we do not have to be worried about (No)SQL
injections::

	<?php
	$m = new MongoClient;
	$m->demo->test->insert( [ 'value' => $_GET['life'] ] );
	?>

Because MongoDB is schemaless, the type of each field has not been defined
and hence you can store the value as any type you would like::

	<?php
	$m = new MongoClient;
	$m->demo->test->insert( [ 'value' => "42" ] );
	$m->demo->test->insert( [ 'value' => 42 ] );
	$m->demo->test->insert( [ 'value' => 42.0 ] );
	?>

MongoDB stores data as typed values, and the above three inserts result in the
following three documents in the database (as seen through the MongoDB Shell)::

	> db.test.find();
	{ "_id" : ObjectId("52f5691544670a8077b0dc51"), "value" : "42" }
	{ "_id" : ObjectId("52f5691544670a8077b0dc52"), "value" : NumberLong(42) }
	{ "_id" : ObjectId("52f5691544670a8077b0dc53"), "value" : 42 }

Because they are stored as three different documents, essentially all three
with a different schema each, it becomes important to use the correct type
when querying the data as well.

To find the string and integer variant, we have to run::

	<?php
	$m = new MongoClient;
	var_dump( $m->demo->test->findOne( [ 'value' => "42" ] )['value'] );
	var_dump( $m->demo->test->findOne( [ 'value' => 42 ] )['value'] );
	?>

Which outputs::

	string(2) "42"
	int(42)

However, if we want to find the document where ``value`` is stored as a
floating point number we fail to do that with::

	<?php
	$m = new MongoClient;
	var_dump( $m->demo->test->findOne( [ 'value' => 42.0 ] ) );
	?>

Which outputs::

	array(2) {
	  '_id' =>
	  class MongoId#6 (1) {
		public $$id =>
		string(24) "52f5691544670a8077b0dc52"
	  }
	  'value' =>
	  int(42)
	}

As you can see it finds the variant with the integer value first. When using
``find()`` to find **all** documents that match the query, we see the document
with the floating point value turn up as well::

	<?php
	$m = new MongoClient;
	foreach ( $m->demo->test->find( [ 'value' => 42.0 ] ) as $r )
	{
		var_dump( $r['value'] );
	}
	?>

Which outputs::

	int(42)
	double(42)

It is possible to get only the document back where the value is a floating
point number by enforcing the value is of type float::

	<?php
	$m = new MongoClient;
	$r = $m->demo->test->findOne(
		[ '$and' => [
			[ 'value' => 42.0 ],
			[ 'value' => [ '$type' => 1 ] ]
		] ]
	);
	var_dump( $r['value'] );
	?>

Which then outputs the expected::

	double(42)

The values for the ``$type`` operator can be found in the MongoDB documentation__.

__ http://docs.mongodb.org/manual/reference/operator/query/type/

**Conclusion**

MongoDB stores values in the type that values have been inserted with.
Unlike relational databases MongoDB does not coalesce this into the "defined"
type, as there is no type defined.

To find documents, make sure that you use the same type as what you inserted
the value as, otherwise MongoDB will **not** find the document. The execption
here is that the three numerical types (32 bit integer, 64 bit integer and
double) are interchangable—as long as the value is the same.
