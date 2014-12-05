=========
phpSesame
=========

Welcome to the phpSesame project, this is a client library for Sesame 2.x which utilises the REST API that it implements.

Requirements
============

- `PHP 5+ <http://php.net/>`_ - (There shouldn't be any subversion dependencies, but I haven't checked thoroughly)
- `HTTP_Request2 <http://pear.php.net/package/HTTP_Request2>`_

This project is a rewrite for Sesame 2.x of the `phesame library <http://www.hjournal.org/phesame/>`_ written by Michele Barbera and Riccardo Giomi.

Support
=======

Unfortunately the project is unsupported, there is some links below that document the library though. Feel free to contact me with any bugs or suggestions though.

Examples
========

I am assuming at this point you have installed and configured Sesame, have a repository set up and the REST API functioning correctly. If not then please consult the `Sesame documentation <http://www.openrdf.org/doc/sesame2/users/>`_.

Using the Library
-----------------

To get the library up and running all you need is::

	require_once "path/to/phpSesame/phpSesame.php";

	$sesame = array('url' => 'http://localhost:8080/openrdf-sesame', 'repository' => 'exampleRepo');
	$store = new phpSesame($sesame['url'], $sesame['repository']);

You can change the repository you are working on at any time by calling::

	$store->setRepository("newRepo");

Querying a Store
----------------

The simplest way to query a store is::

	$sparql = "PREFIX foaf:<http://xmlns.com/foaf/0.1/>
	SELECT ?s ?o WHERE { ?s foaf:name ?o } LIMIT 100";
	$resultFormat = phpSesame::SPARQL_XML; // The expected return type, will return a phpSesame_SparqlRes object (Optional)
	$lang = "sparql"; // Can also choose SeRQL (Optional)
	$infer = true; // Can also choose to explicitly disallow inference. (Optional)

	$result = $store->query($sparql, $resultFormat, $lang, $infer);

	if($result->hasRows()) {
		foreach($result->getRows() as $row) {
			echo "Subject: " . $row['s'] . ", Object: " . $row['o'] . ".";
		}
	}

The library only supports SPARQL_XML at present, however Sesame supports a number of `Content Types <http://www.openrdf.org/doc/sesame2/system/ch08.html#d0e609u>`_ and the library could do with some more development.

Namespaces
----------

Sesame 2.x can store commonly used namespaces and associated prefixes so that they do not have to be explicitly defined along with each query; these are held in a repository specific list::
	
	$namespace = $store->getNS("rdf");
	$store->setNS("newrdf", $namespace);
	$store->deleteNS("newrdf");

Managing Data
-------------

Sesame allows you to either to append, overwrite or delete data. You can perform these actions on a specific context or on the entire repository. As such the second parameter is optional but it defaults to all. (I may change this though!)::

	$index = array("http://www.example.com/users/joe_bloggs" => array(
	    "foaf:name" => array("Joe Bloggs"),
	    "foaf:age" => array(21),
	    "foaf:knows" => array("http://www.example.com/users/mary_smith")
	));

	$conf = array('ns' => array('rdf' => 'http://www.w3.org/1999/02/22-rdf-syntax-ns#', 'owl' => 'http://www.w3.org/2002/07/owl#'));
	$serializer = ARC2::getRDFXMLSerializer($conf);

	$rdfxml = $serializer->getSerializedIndex($index);
	$context = "http://www.example.com/users/joe_bloggs"; // Optional - defaults to entire repository though.
	$inputFormat = phpSesame::RDFXML; // Optional - defaults to RDFXML

	$store->append($rdfxml, $context, $inputFormat);

Sesame supports a number of input types, the library supports: RDFXML, N3, NTRIPLES, TURTLE, TRIX and TRIG. Please see `Sesame's documentation <http://www.openrdf.org/doc/sesame2/system/ch08.html#d0e609>`_ for more information.

This example uses the `ARC2 library <https://github.com/semsol/arc2/wiki>`_ to provide serialization::

	$rdfxml = $serializer->getSerializedIndex($index);
	$context = "http://www.example.com/users/joe_bloggs";

	$store->overwrite($rdfxml, $context);

If at a later date you want to modify that data you must recreate the entire data item and overwrite it, unfortunately there is no easier way!

More Information
================

If you are looking for a good RDF Serializer I use the `ARC2 library <https://github.com/semsol/arc2/wiki>`_ for managing the data into the format required for this library to use.

API Documentation
=================

Please use `PHP Documentor <http://www.phpdoc.org/>`_ to generate the API documentation.
