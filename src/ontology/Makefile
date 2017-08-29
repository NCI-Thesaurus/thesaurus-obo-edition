##########################################################################################
# This Makefile assumes you have blazegraph-runner, ncit-utils, dosdp-tools, robot, and owltools on your PATH.
# These can be obtained from GitHub:
# - https://github.com/balhoff/blazegraph-runner/releases
# - https://github.com/NCI-Thesaurus/ncit-utils/releases
# - https://github.com/INCATools/dosdp-tools/releases
# - https://github.com/ontodev/robot
# - https://github.com/owlcollab/owltools/releases
##########################################################################################

SRC=Thesaurus.owl
NCIT=ncit.owl
DATE=$(shell date +'%Y-%m-%d')
PREFIX=http://purl.obolibrary.org/obo/ncit
RELEASEPREFIX=$(PREFIX)/releases/$(DATE)

.PHONY: all
all: $(NCIT) ncit.obo ncit-graph.jnl ncit-bridge-to-cl.owl ncit-plus.owl

$(SRC):
	curl -1 -L -O https://evs.nci.nih.gov/ftp1/NCI_Thesaurus/Thesaurus.owl

$(NCIT): $(SRC)
	export JAVA_OPTS=-Xmx8G && export ROBOT_JAVA_ARGS=-Xmx16G &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=rdfxml $(SRC) &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/make_role_classes_obolete.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-properties.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-values.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/fix-multiple-defs.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-iri.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-version-iri.rq &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=turtle additions.ttl &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-iris-to-obo-style.rq &&\
	blazegraph-runner dump   --journal=blazegraph.jnl --outformat=turtle ncit-edit.ttl &&\
	robot reason -i ncit-edit.ttl -r elk reduce -r elk -o $@

ncit-property-graph.ttl: $(NCIT)
	export JAVA_OPTS=-Xmx80G &&\
	ncit-utils materialize-property-expressions ncit.owl $@.tmp ncit-property-graph-redundant.ttl.tmp &&\
	mv $@.tmp $@ && touch $@ && mv ncit-property-graph-redundant.ttl.tmp ncit-property-graph-redundant.ttl && touch ncit-property-graph-redundant.ttl

ncit-graph.jnl: $(NCIT) ncit-property-graph.ttl
	rm -f $@
	blazegraph-runner load --journal=$@ --informat=rdfxml --use-ontology-graph $(NCIT) &&\
	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-property-graph.ttl
	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-property-graph-redundant.ttl

ncit.obo: $(NCIT)
	owltools ncit.owl -o -f obo ncit.obo.tmp && grep -v ^owl-axioms ncit.obo.tmp >$@

ncit-bridge-to-cl.owl:
	dosdp-tools generate --infile=../mappings/cl.tsv --obo-prefixes=true --template=../mappings/equivalence_mapping.yaml --outfile=ncit-cl-bridge.ofn &&\
	robot annotate --input ncit-cl-bridge.ofn --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --output $@ && rm ncit-cl-bridge.ofn

ncit-plus.owl: $(NCIT)
	ncit-utils replace-mapped-terms --mappings=../mappings/cl.tsv --ontology=ncit-edit.ttl --keep=2 --drop=0 --output=ncit-plus-tmp.owl &&\
	robot annotate --input ncit-plus-tmp.owl --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ reduce -r elk -o $@ && rm ncit-plus-tmp.owl
