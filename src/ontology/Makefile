SRC=Thesaurus.owl
NCIT=ncit.owl

$(SRC):
	curl -L -O https://evs.nci.nih.gov/ftp1/NCI_Thesaurus/Thesaurus.owl

$(NCIT): $(SRC)
	export JAVA_OPTS=-Xmx8G &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=rdfxml $(SRC) &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-iri.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-iris-to-obo-style.rq &&\
	blazegraph-runner dump   --journal=blazegraph.jnl --outformat=rdfxml ncit.owl.tmp &&\
	mv ncit.owl.tmp ncit.owl && touch ncit.owl
