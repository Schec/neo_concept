Neo_Concept
-----------

This software imports the ConceptNet Knowledge Base (version tested: 5.5) into neo4j 3.2.3. 
This project was inspired by [a post from Max De Marzi](http://maxdemarzi.com/2013/05/13/knowledge-bases-in-neo4j/) who also provided [the original software](https://github.com/maxdemarzi/neo_concept).

This rewrite does not use the [Bloom filter](http://en.wikipedia.org/wiki/Bloom_filter) anymore as according to ConceptNet wiki [the uri is unique among assertions](https://github.com/commonsense/conceptnet5/wiki/Edges) and [the provided CSV files](http://conceptnet5.media.mit.edu/downloads/current/) provide a list of assertions. 

Moreover, this software adds relations to nodes in the same hierarchy. For instance, if we have nodes
- A: "/c/en/able"
- B: "/c/en/able/a/having_the_necessary_means_or_skill_or_know-how_or_authority_to_do_something"

normally A and B are **not** connected through a relation. Therefore, we add the following two relations with weight 10.0 between these two nodes:
- A /r/DownHierarchy B
- B /r/UpHierarchy A

Optionally, it is possible to perform Part-Of-Speech (POS) tagging of the two concepts within the surface text of every relation by means of the Stanford CoreNLP software. 

- NEW!: Check out [Neo_Wordnet](https://github.com/redsk/neo_wordnet) to import Wordnet into neo4j!
- NEW!: Check out [Neo_Merger](https://github.com/redsk/neo_merger) to import Conceptnet and Wordnet into the same neo4j graph!

Pre-Requisites
--------------

- neo4j version 3.2.3 (needed for the [new import tool])
- [ConceptNet 5.5](https://s3.amazonaws.com/conceptnet/downloads/2017/edges/conceptnet-assertions-5.5.5.csv.gz) provides a list of assertions
- regular Python, no dependencies

Tested with neo4j-community-3.2.3.

How-To 
-------------------

    mkdir neo4j-kbs
    cd neo4j-kbs
    git clone https://github.com/redsk/neo_concept.git

    # get latest conceptnet from http://conceptnet5.media.mit.edu/downloads/
    mkdir conceptnet
    cd concepnet
    wget https://s3.amazonaws.com/conceptnet/downloads/2017/edges/conceptnet-assertions-5.5.5.csv.gz
    gunzip conceptnet-assertions-5.5.5.csv.gz
    ln -s conceptnet-assertions-5.5.5.csv csv_current
    cd ..

    # Usage:
    # python convertcn.py <input directory> [ALL_LANGUAGES]
    # If the flag ALL_LANGUAGES is not set, only English concepts will be converted
    # this will take a while
    python neo_concept/convertcn.py conceptnet

    # get latest neo4j, community edition, from https://neo4j.com/download/other-releases/#releases
    # unzip the file in any directory

    # do only one of the two import commands below. If you calculated the POS tags, edges.csv is no longer needed

    # import nodes.csv and edges.csv using the new import tool (WITHOUT POS TAGS!) -- this will take a while too
    mkdir concept-netgraph
    ~/Downloads/neo4j-community-3.3.1/bin/neo4j-import --into conceptnet-graph --nodes nodes.csv --relationships edges.csv --delimiter "TAB"


    # start neo4j
    # point neo4j to the conceptnet-graph directory you created


Goto localhost:7474 to see the graph. Create and index on Concepts for performance reasons:

    CREATE INDEX ON :Concept(id)

You can now query the database. Example:

    MATCH (sushi {id:"/c/en/sushi"}), sushi-[r]->other_concepts
    RETURN sushi.id, other_concepts.id, type(r), r.context, r.weight, r.surface
