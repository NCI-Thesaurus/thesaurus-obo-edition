###############################################################################
# This Makefile assumes you have blazegraph-runner and ncit-utils on your PATH.
# These can be obtained from GitHub:
# - https://github.com/balhoff/blazegraph-runner/releases
# - https://github.com/NCI-Thesaurus/ncit-utils/releases
###############################################################################

SRC=Thesaurus.owl
NCIT=ncit.owl

.PHONY: all
all: $(NCIT) ncit.obo ncit-graph.jnl

$(SRC):
	curl -1 -L -O https://evs.nci.nih.gov/ftp1/NCI_Thesaurus/Thesaurus.owl

$(NCIT): $(SRC)
	export JAVA_OPTS=-Xmx8G &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=rdfxml $(SRC) &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-properties.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-values.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/fix-multiple-defs.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-iri.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-version-iri.rq &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=turtle additions.ttl &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-iris-to-obo-style.rq &&\
	blazegraph-runner dump   --journal=blazegraph.jnl --outformat=turtle ncit-edit.ttl &&\
	mv ncit-edit.ttl $(NCIT) && touch $(NCIT)

ncit-property-graph.ttl: $(NCIT)
	export JAVA_OPTS=-Xmx80G &&\
	ncit-utils materialize-property-expressions ncit.owl $@.tmp &&\
	mv $@.tmp $@ && touch $@

ncit-graph.jnl: $(NCIT) ncit-property-graph.ttl
	rm -f $@
	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph $(NCIT) &&\
	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-property-graph.ttl

ncit.obo: $(NCIT)
	./owltools ncit.owl -o -f obo ncit.obo.tmp && grep -v ^owl-axioms ncit.obo.tmp >$@
