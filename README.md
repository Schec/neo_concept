Neo_Concept
-----------

This software helps to import the ConceptNet dataset (tested version 5.3) to neo4j 2.2. 
This project was inspired by [a post from Max De Marzi](http://maxdemarzi.com/2013/05/13/knowledge-bases-in-neo4j/) who also provided [the original software](https://github.com/maxdemarzi/neo_concept).

This rewrite does not use the [Bloom filter](http://en.wikipedia.org/wiki/Bloom_filter) anymore as according to ConceptNet wiki [the uri is unique among assertions](https://github.com/commonsense/conceptnet5/wiki/Edges) and [the provided CSV files](http://conceptnet5.media.mit.edu/downloads/current/) provide a list of assertions. 

Pre-Requisites
--------------

- neo4j version 2.2 (needed for the [new import tool](http://neo4j.com/docs/2.2.0-M03/import-tool.html))
- [ConceptNet 5.3](http://conceptnet5.media.mit.edu/downloads/current/) provide a list of assertions. 
- regular Python, no dependencies

Tested with neo4j-community-2.2.0-M02 and neo4j-community-2.2.0-M03.

How-To 
-------------------

    mkdir neo-conceptnet5
    cd neo-conceptnet5
    git clone https://github.com/redsk/neo_concept.git

    # get latest conceptnet from http://conceptnet5.media.mit.edu/downloads/
    wget http://conceptnet5.media.mit.edu/downloads/current/conceptnet5_flat_csv_5.3.tar.bz2
    tar jxvf conceptnet5_flat_csv_5.3.tar.bz2
    ln -s csv_<version> csv_current

    # "Usage:
    # python convertcn.py <input directory> [ALL_LANGUAGES]"
    # If the flag ALL_LANGUAGES is not set, only English concepts will be converted
    # this will take a while
    python neo_concept/convertcn.py csv_current/assertions/

    # get latest neo4j (tested with neo4j-community-2.2.0-M03)
    curl -O -J -L http://neo4j.com/artifact.php?name=neo4j-community-2.2.0-M03-unix.tar.gz
    tar zxf neo4j-community-2.2.0-M03-unix.tar.gz

    # import nodes.csv and edges.csv using the new import tool -- this will take a while too
    neo4j-community-2.2.0-M03/bin/neo4j-import --into neo4j-community-2.2.0-M03/data/graph.db --nodes nodes.csv --relationships edges.csv --delimiter "TAB"

    # start neo4j
    neo4j-community-2.2.0-M03/bin/neo4j start


Goto localhost:7474 to see the graph. Create and index on Concepts for performance reasons:

    CREATE INDEX ON :Concept(id)

You can now query the database. Example:

    MATCH (sushi {id:"/c/en/sushi"}), sushi-[r]->other_concepts
    RETURN sushi.id, other_concepts.id, type(r), r.context, r.weight, r.surface
