Managing schema changes with MongoDB
====================================

.. articleMetaData::
   :Where: Paris, France
   :Date: 2013-11-19 09:32 Europe/Paris
   :Tags: blog, mongodb, php
   :Short: mongodbschema

In an `earlier article`_ I explained that although MongoDB_ stores documents
in a "schema-less" collection you still have to think about how you store your
data if you want to get the best out of the database.

.. _`earlier article`: /introduction-to-document-databases.html
.. _MongoDB: http://mongodb.org

However, sometimes you have to change your mind on how a document should look
like. For example when you realize that your `firstname and surname fields`_
really should have been just one ``name`` field, when you need to be able to
store more than one phone number or address, or when you decide that hashing
passwords is a good idea after all.

.. _`firstname and surname fields`: http://www.w3.org/International/questions/qa-personal-names

Relational Databases
--------------------

With traditional relational databases it is often necessary to make these
database schema changes as part of a deployment process. There are various
methods for doing so, and they most often consist of a process of applying SQL
DDL_ to the schema and then when necessary a specific script to move data
around. Tools like Phinx_ and Liquibase_ can help with that but of course you
can also do it manually. 

.. _DDL: http://en.wikipedia.org/wiki/Data_Definition_Language
.. _Phinx: http://phinx.org/
.. _Liquibase: http://www.liquibase.org/

Let's have a look at the following schema:

**Users**

==== ========= ========= ======== =============
*id* password  firstname lastname phone number
==== ========= ========= ======== =============
1    sekret    Derick    Rethans  +447551569555
2    elephpant Rasmus    Lerdorf  NULL
==== ========= ========= ======== =============

In this schema for users, we have records containing a name, a password and a
phone number. Now let's imagine we want to be able to store multiple contact
details and we also want to hash the passwords. The new schema will look like:

**Users**

==== ========= ========= ========
*id* password  firstname lastname
==== ========= ========= ========
1    7f1afdbe  Derick    Rethans 
2    ae9c300e  Rasmus    Lerdorf 
==== ========= ========= ========

**Contacts**

====== ======= =============
userid method  value
====== ======= =============
1      phone   +447551569555
2      twitter derickr
2      twitter rasmus
====== ======= =============

To make this transition, you could run the following SQL statements:

 #. The new table:

    ::

        CREATE TABLE Contacts(userid INT, method VARCHAR(16), value VARCHAR(250));
        ALTER TABLE Contacts ADD FOREIGN KEY (userid) REFERENCES Users(id);

 #. Moving the phone number:

    ::

        INSERT INTO Contacts
            SELECT id, "phone", phonenumber FROM Users WHERE phonenumber IS NOT NULL;

        ALTER TABLE Users DROP phonenumber;

 #. Hashing the password (please don't use CRC32 in your code, it is here as
    an example):

    ::

        UPDATE Users SET password = CRC32(CONCAT(firstname,lastname,password));

Reverting this "migration" is problematic. While you can easily move the phone
numbers back to the ``Users`` table, you can not "unhash" the password—as
that's the whole idea behind a hash. If things go wrong, you have a bit of a
problem and you will probably need to reset all the passwords.

MongoDB
-------

So how do you deal with schema changes in MongoDB? First, let's have a look at
how we would store the original documents:

**Users**

::

    {
        _id: 1,
        password: "sekret",
        firstname: "Derick",
        lastname: "Rethans",
        phonenumber: "+447551569555",
    },
    {
        _id: 2,
        password: "elephpant",
        firstname: "Rasmus",
        lastname: "Lerdorf",
    }

And now secondly after the migration:

**Users**

::

    {
        _id: 1,
        password: "7f1afdbe", 
        firstname: "Derick",
        lastname: "Rethans",
        contacts: [
            {
                method: "phone",
                value: "+447551569555",
            }
        ]
    },
    {
        _id: 2,
        password: "ae9c300e",
        firstname: "Rasmus",
        lastname: "Lerdorf",
    }

As you can see we store the contacts as an array unlike as in the relational
model where we used a second table. 

To migrate the data between the two schemas we can use the same approach as in
the relational example from above: write a migration script to convert the
data from the old to the new schema. We don't really have to do a schema
change, but we do need to update each document. MongoDB however does not allow
you to update fields according to the value of other fields so we can not
simply do a::

    $set: { 'contacts' : { method: 'phone', value: '$phonenumber' } }

Although you can retrieve the data like this through the Aggregation
Framework::

    db.Users.aggregate( [
        { '$group': {
            '_id': '$_id',
            'firstname' : { $first: '$firstname' },
            'lastname' : { $first: '$lastname' },
            'contacts' : { $push : { method: { $concat: [ 'phone' ] }, value: '$phonenumber' } }
        } }
    ] );

*Note that I am abusing `$concat`_ here. From MongoDB 2.6 (or 2.5.2 if you use
a development version) you can use `$literal`_ instead.*

.. _`$concat`: http://docs.mongodb.org/manual/reference/operator/aggregation/concat/#exp._S_concat
.. _`$literal`: https://jira.mongodb.org/browse/SERVER-5782

But this still requires you to read all the documents in a script, in which you
also have to do the password encoding, and subsequently write the documents
back to collection. 

Reading all the documents and writing them back takes a great toll on your
server. In some situations you might not even care about a large amount of
your records. Take for example a forum with 500.000 total users, but
with only 5.000 active users. You could argue that you are wasting 99% of the
time that it takes to update 495.000 records that are never going to be used.

An alternative way of doing this is by using a versioning system for your
documents. This allows you to have documents of both versions at the same
time. An application should support reading documents in all versions and
update/write only the latest version. 

Let's have a look at how this might look at in the collection::

    {
        _id: 1,
        schema_version: 1,
        password: "sekret", 
        firstname: "Derick",
        lastname: "Rethans",
        phonenumber: "+447551569555",
    },
    {
        _id: 2,
        schema_version: 2,
        password: "ae9c300e",
        firstname: "Rasmus",
        lastname: "Lerdorf",
        contacts [
            {
                method: 'twitter',
                value: 'rasmus',
            }
        ]
    }

Here the record for me has the old layout still, whereas the one for Rasmus has
been updated to *schema version 2*. Your model layer could read User documents
in the following way::

    class Model
    {
        public function fetchUser( $id )
        {
            $raw = $this->Users->find( array( '_id' => (int) $id ) );
            return User::createFromDB( $raw );
        }
    }

    class User
    {
        private $id;
        private $firstname;
        private $lastname;
        private $password; // hashed password
        private $contacts; // array of contacts with method and value

        private static function createFromDBv1( $raw )
        {
            $n = new User;
            $n->firstname = $raw['firstname'];
            $n->lastname  = $raw['lastname'];
            $n->password  = hash( 'crc32', $raw['password'] );

            $contact = array(
                'method' => 'phone',
                'value'  => $raw['phonenumber'],
            );
            $n->contacts = array( $contact );

            return $n;
        }

        private static function createFromDBv2( $raw )
        {
            $n = new User;
            $n->firstname = $raw['firstname'];
            $n->lastname  = $raw['lastname'];
            $n->password  = $raw['password'];
            $n->contacts  = $raw['contacts'];

            return $n;
        }

        static function createFromDB( $raw )
        {
            $fname = "createFromDB{$raw['schema_version']}";

            $n = $this->$fname( $raw );
            $n->id = $raw['_id'];

            return $n;
        }
    }

The above (greatly simplified) example can read documents of both version 1 and
2. Internally it is stored as a one-to-one translation of the version 2
document, where for version 1 we fake the contacts as an array and hash the
password upon **reading**.

When storing the data, we will only write version 2 documents::

    class Model
    {
        public function saveUser( User $user )
        {
            $internal = $user->hydrate();
            $this->Users->update( array( '_id' => (int) $id ), $internal );
        }
    }

    class User
    {
        private $id;
        private $firstname;
        private $lastname;
        private $password; // hashed password
        private $contacts; // array of contacts with method and value

        function hydrate()
        {
            $internal = array();
            $internal['schema_version'] = 2;
            $internal['firstname'] = $this->firstname;
            $internal['lastname'] = $this->lastname;
            $internal['password'] = $this->password;
            $internal['contacts'] = $this->contacts;

            return $internal;
        }
    }

I realise that this of course a simplified example, but the general idea
should be applicable in many situations.

**Closing Words**

Among the benefits of using versioning for your document's structure is that no
extra processing is needed on the database side as the structure is only
updated when they are changed, which has a similar inpact as in normal
operations. All of the maintenance and testing burdens are placed on the
developer that decides to change the structure in the first place. It means
that you don't have to create a long running script to update all the
documents, and neither is there a situation where you have to roll back if
something goes wrong during a migration. It might take a *long* time before all
the documents in a collection are updated, depending on their use case. And
later, you can decide to not support really old versions if the maintenance
burden becomes too great, or perhaps even get rid of documents of an older
version altogether.
