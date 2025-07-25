#########################################################################################################################
# This Makefile assumes you have Java 8 and blazegraph-runner, ncit-utils, dosdp-tools, robot, and owltools on your PATH.
# These can be obtained from GitHub:
# - https://github.com/balhoff/blazegraph-runner/releases
# - https://github.com/NCI-Thesaurus/ncit-utils/releases
# - https://github.com/INCATools/dosdp-tools/releases
# - https://github.com/ontodev/robot
# - https://github.com/owlcollab/owltools/releases
#########################################################################################################################

DATE=$(shell date +'%Y-%m-%d')
PREFIX=http://purl.obolibrary.org/obo/ncit
RELEASEPREFIX=$(PREFIX)/releases/$(DATE)

VERSION = "0.0.1"
IM=monarchinitiative/ncit

docker-build-no-cache:
	@docker build --no-cache -t $(IM):$(VERSION) . \
	&& docker tag $(IM):$(VERSION) $(IM):latest

docker-build:
	@docker build -t $(IM):$(VERSION) . \
	&& docker tag $(IM):$(VERSION) $(IM):latest

.PHONY: all
# ncit-plus.owl ncit-plus-negations.owl were dropped from here, see comments below
all: ncit.owl ncit.obo ncit-bridge-to-cl.owl ncit-bridge-to-uberon.owl ncit-negations.owl ncit-oncotree.ttl ncit-oncotree.obo

Thesaurus.owl:
 	curl -L -O https://evs.nci.nih.gov/ftp1/rdf/Thesaurus.owl

ncit-edit.owl: Thesaurus.owl oncotree.nt additions.ttl
	export JAVA_OPTS=-Xmx32G && export ROBOT_JAVA_ARGS=-Xmx32G &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=rdfxml Thesaurus.owl &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/remove-date.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/make_role_classes_obolete.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-properties.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/replace-properties-as-values.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/convert-subsets-to-obo-style.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/fix-multiple-defs.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-iri.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-ontology-version-iri.rq &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=turtle additions.ttl &&\
	blazegraph-runner load   --journal=blazegraph.jnl --informat=turtle oncotree.nt &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/update-iris-to-obo-style.rq &&\
	blazegraph-runner update --journal=blazegraph.jnl ../sparql/add-obo-compliance-notice.rq &&\
	blazegraph-runner dump   --journal=blazegraph.jnl --outformat=turtle ncit-edit.ttl &&\
	rm blazegraph.jnl &&\
	robot reason -i ncit-edit.ttl -r elk -o $@

tmp/revert-obsolete.txt: ncit-edit.owl
	robot query --input $< --query ../sparql/extract-obsolete-terms-notice.sparql tmp/revert-obsolete.txt

tmp/revert-obsolete.owl: tmp/ncit-released.obo tmp/revert-obsolete.txt
	robot filter -i tmp/ncit-released.obo --term-file tmp/revert-obsolete.txt --axioms all --select "self parents annotations properties" --signature true --trim true --output tmp/revert-obsolete.owl

ncit.owl: ncit-edit.owl tmp/ncit-released.obo tmp/revert-obsolete.txt tmp/revert-obsolete.owl
	sed -i  -e 's/ IMPORTANT NOTICE: The NCI Thesaurus contains biomedical terminologies that NCI does not own or control. This concept contains gender-related content that does not comply with Executive Order 14168.//g' \
	 	-e 's/IMPORTANT NOTICE: The NCI Thesaurus contains biomedical terminologies that NCI does not own or control. This concept contains gender-related content that does not comply with Executive Order 14168.//g' $<
	robot \
		remove -i ncit-edit.owl \
			--term-file tmp/revert-obsolete.txt \
			--axioms all \
			--select "self parents properties" \
			--signature true \
			--trim true \
	 	merge -i tmp/revert-obsolete.owl -i missing-classes.owl --output $@

#ncit-property-graph.ttl: ncit.owl
#	export JAVA_OPTS=-Xmx80G &&\
#	ncit-utils materialize-property-expressions ncit.owl $@.tmp ncit-property-graph-redundant.ttl.tmp &&\
#	mv $@.tmp $@ && mv ncit-property-graph-redundant.ttl.tmp ncit-property-graph-redundant.ttl &&\
#	robot annotate --input $@ --ontology-iri $(PREFIX)/$@ --output $@ &&\
#	robot annotate --input ncit-property-graph-redundant.ttl --ontology-iri $(PREFIX)/ncit-property-graph-redundant.ttl --output ncit-property-graph-redundant.ttl

#ncit-plus-property-graph.ttl: ncit-plus.owl
#	export JAVA_OPTS=-Xmx80G &&\
#	ncit-utils materialize-property-expressions ncit-plus.owl $@.tmp ncit-plus-property-graph-redundant.ttl.tmp &&\
#	mv $@.tmp $@ && mv ncit-plus-property-graph-redundant.ttl.tmp ncit-plus-property-graph-redundant.ttl &&\
#	robot annotate --input $@ --ontology-iri $(PREFIX)/$@ --output $@ &&\
#	robot annotate --input ncit-plus-property-graph-redundant.ttl --ontology-iri $(PREFIX)/ncit-plus-property-graph-redundant.ttl --output ncit-plus-property-graph-redundant.ttl
#
#ncit-graph.jnl: ncit.owl ncit-property-graph.ttl ncit-plus.owl ncit-plus-property-graph.ttl monarch-data
#	export JAVA_OPTS=-Xmx16G &&\
#	rm -f $@ &&\
#	blazegraph-runner load --journal=$@ --informat=rdfxml --use-ontology-graph ncit.owl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-property-graph.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-property-graph-redundant.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=rdfxml --use-ontology-graph ncit-plus.owl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-plus-property-graph.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncit-plus-property-graph-redundant.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph reactome.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph coriell.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph ncbigene.ttl &&\
#	blazegraph-runner load --journal=$@ --informat=turtle --use-ontology-graph bgee.ttl

ncit.obo: ncit.owl
	robot convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o tmp/$@
	grep -v ^owl-axioms tmp/$@ >$@

ncit-bridge-to-cl.owl: ../mappings/cl.tsv
	grep -Ev '^#' $< >../mappings/cl-filtered.tsv &&\
	export JAVA_OPTS=-Xmx16G &&\
	dosdp-tools generate --generate-defined-class true --infile=../mappings/cl-filtered.tsv --obo-prefixes=true --template=../mappings/equivalence_mapping.yaml --outfile=ncit-cl-bridge.ofn &&\
	robot annotate --input ncit-cl-bridge.ofn --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --output $@ && rm ncit-cl-bridge.ofn
	
ncit-bridge-to-uberon.owl: ../mappings/uberon.tsv ../mappings/uberon-previous.tsv
	grep -Ev '^#' ../mappings/uberon-previous.tsv >../mappings/uberon-previous-filtered.tsv &&\
	grep -Ev '^#' ../mappings/uberon.tsv >../mappings/uberon-filtered.tsv &&\
	export JAVA_OPTS=-Xmx16G &&\
	dosdp-tools generate --generate-defined-class true --infile=../mappings/uberon-previous-filtered.tsv --obo-prefixes=true --template=../mappings/equivalence_mapping.yaml --outfile=ncit-uberon-previous-bridge.ofn &&\
	dosdp-tools generate --generate-defined-class true --infile=../mappings/uberon-filtered.tsv --obo-prefixes=true --template=../mappings/equivalence_mapping.yaml --outfile=ncit-uberon-bridge.ofn &&\
	robot merge --input ncit-uberon-previous-bridge.ofn --input ncit-uberon-bridge.ofn --output ncit-uberon-bridge.ofn &&\
	robot annotate --input ncit-uberon-bridge.ofn --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --output $@ && rm ncit-uberon-bridge.ofn

#ncit-plus.owl: ncit.owl imports.ofn ../mappings/cl.tsv ../mappings/uberon.tsv ../mappings/uberon-previous.tsv #../mappings/go.tsv
#	export JAVA_OPTS=-Xmx16G && export ROBOT_JAVA_ARGS=-Xmx16G && export OWLTOOLS_MEMORY=16G &&\
#	ncit-utils replace-mapped-terms --mappings=../mappings/cl.tsv --ontology=ncit-edit.ttl --keep=object_id --drop=subject_id --output=ncit-plus-tmp.owl &&\
#	ncit-utils replace-mapped-terms --mappings=../mappings/uberon.tsv --ontology=ncit-plus-tmp.owl --keep=object_id --drop=subject_id --output=ncit-plus-tmp.owl &&\
#	ncit-utils replace-mapped-terms --mappings=../mappings/uberon-previous.tsv --ontology=ncit-plus-tmp.owl --keep=object_id --drop=subject_id --output=ncit-plus-tmp.owl &&\
#	owltools ncit-plus-tmp.owl imports.ofn --add-imports-from-supports --extract-module -c -s 'http://github.com/NCI-Thesaurus/thesaurus-obo-edition/imports.owl' -o import_extract.owl &&\
#	robot merge --input ncit-plus-tmp.owl --input import_extract.owl --output ncit-plus-tmp.owl &&\
#	robot \
#		query --input ncit-plus-tmp.owl --update ../sparql/remove-title-description.rq \
#		annotate --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --annotation-file ncit-plus-annotations.ttl \
#		reason -r elk -o $@ &&\
#	rm ncit-plus-tmp.owl

#	ncit-utils replace-mapped-terms --mappings=../mappings/go.tsv --ontology=ncit-plus-tmp.owl --keep=object_id --drop=subject_id --output=ncit-plus-tmp.owl &&\
	
ncit-negations.owl: ncit.owl
	blazegraph-runner load   --journal=ncit-negations.jnl --informat=rdfxml ncit.owl &&\
	blazegraph-runner update --journal=ncit-negations.jnl ../sparql/convert_exclude_to_complement.rq &&\
	blazegraph-runner dump   --journal=ncit-negations.jnl --outformat=rdfxml $@ &&\
	rm ncit-negations.jnl &&\
	robot annotate --input $@ --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ -o $@

#ncit-plus-negations.owl: ncit-plus.owl
#	blazegraph-runner load   --journal=ncit-plus.jnl --informat=rdfxml ncit-plus.owl &&\
#	blazegraph-runner update --journal=ncit-plus.jnl ../sparql/convert_exclude_to_complement.rq &&\
#	blazegraph-runner dump   --journal=ncit-plus.jnl --outformat=rdfxml $@ &&\
#	rm ncit-plus.jnl &&\
#	robot annotate --input $@ --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ -o $@

tumor_types.txt:
	curl -L -O http://oncotree.mskcc.org/api/tumor_types.txt

oncotree.nt: tumor_types.txt
	python3 ../python/oncotreestuff.py tumor_types.txt oncotree.nt

oncotree_terms.txt: tumor_types.txt
	set -e && cut -f 10 tumor_types.txt | tail -n +2 | sed 's~,~\'$$'\n~g' | sed '\~^\s*$$~d' | sed 's~^~http://purl.obolibrary.org/obo/NCIT_~' >$@

ncit-oncotree.ttl: ncit.owl oncotree_terms.txt
	robot extract --method MIREOT --input ncit.owl --lower-terms oncotree_terms.txt --output ncit-oncotree.tmp.ttl &&\
	robot annotate --input ncit-oncotree.tmp.ttl --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --annotation dc:title "NCIt OncoTree slim" --annotation rdfs:comment "Simplified hierarchy for terms used within OncoTree" -o $@ && rm ncit-oncotree.tmp.ttl

ncit-oncotree.obo: ncit-oncotree.ttl
	owltools ncit-oncotree.ttl -o -f obo $@.tmp && grep -v ^owl-axioms $@.tmp >$@

ncit-benign-malignant.ttl: ncit-graph.jnl
	export JAVA_OPTS=-Xmx16G &&\
	rm -f $@ &&\
	blazegraph-runner construct --journal=ncit-graph.jnl --outformat=turtle ../sparql/convert-benign-malignant-to-axioms.rq ncit-benign-malignant.tmp.ttl &&\
	robot merge --input benign-malignant-base.ttl --input ncit-benign-malignant.tmp.ttl annotate --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --annotation rdfs:comment "Conversion of benign and malignant annotations to logical axioms, for consistency testing." -o $@ && rm ncit-benign-malignant.tmp.ttl

#neoplasm-hierarchy-report.txt: ncit.owl ncit-plus.owl
#	export ROBOT_JAVA_ARGS=-Xmx16G &&\
#	robot query --format ttl --input ncit.owl --construct ../sparql/extract-neoplasm-hierarchy.rq standard-neoplasm.ttl &&\
#	robot query --format ttl --input ncit-plus.owl --construct ../sparql/extract-neoplasm-hierarchy.rq plus-neoplasm.ttl &&\
#	robot diff --left standard-neoplasm.ttl --right plus-neoplasm.ttl --output neoplasm-hierarchy-report.txt

Neoplasm_Core.txt:
	curl -1 -L -O https://evs.nci.nih.gov/ftp1/NCI_Thesaurus/Neoplasm/Neoplasm_Core.txt

neoplasm-terms.txt: Neoplasm_Core.txt
	set -e && iconv -f ISO-8859-1 -t ASCII//TRANSLIT Neoplasm_Core.txt | cut -f 1 | tail -n +2 | tr -d '\r' | sed '/^$$/d' | sed 's~^~http://purl.obolibrary.org/obo/NCIT_~' >$@

#neoplasm-core.owl: neoplasm-terms.txt ncit-plus.owl
#	export ROBOT_JAVA_ARGS=-Xmx16G &&\
#	robot extract --method BOT --input ncit-plus.owl --term-file neoplasm-terms.txt annotate --ontology-iri $(PREFIX)/$@ --version-iri $(RELEASEPREFIX)/$@ --annotation dc:title "NCIt Neoplasm Core Module" --annotation rdfs:comment "Module extract from NCIt OBO Edition (Plus) based on the NCIt Neoplasm Core. This module is based on the SLME BOT algorithm, starting with the list of Neoplasm Core terms." --output $@

coriell.ttl:
	curl -L -O https://data.monarchinitiative.org/ttl/coriell.ttl

reactome.ttl:
	curl -L -O https://data.monarchinitiative.org/ttl/reactome.ttl

bgee.ttl:
	curl -L -O https://data.monarchinitiative.org/ttl/bgee.ttl

ncbigene.ttl:
	curl -L -O https://data.monarchinitiative.org/ttl/ncbigene.ttl

monarch-data: coriell.ttl reactome.ttl bgee.ttl ncbigene.ttl

# Remove these here as there was no use case and they caused 
# problems in the build
#ncit-plus-negations.owl \
#ncit-plus.owl \
#neoplasm-core.owl \ (dependent on ncit-plus.owl)
#neoplasm-hierarchy-report.txt (dependent on ncit-plus.owl)

ASSETS=ncit-bridge-to-cl.owl \
ncit-bridge-to-uberon.owl \
ncit-negations.owl \
ncit-oncotree.obo \
ncit-oncotree.ttl \
ncit.obo \
ncit.owl


deploy_release:
	@test $(GHVERSION)
	ls -alt $(ASSETS)
	gh release create $(GHVERSION) --title "NCIT Release $(GHVERSION)" --draft $(ASSETS)

#################
### KGCL Diff ###
#################

upgrade_oak:
	pip install -U oaklib

.PHONY: kgcl-diff
kgcl-diff: kgcl-diff.md

tmp/ncit-released.obo:
	mkdir -p tmp
	wget http://purl.obolibrary.org/obo/ncit.obo -O $@

kgcl-diff.md: tmp/ncit-released.obo ncit.obo
	runoak -i simpleobo:tmp/ncit-released.obo diff -X simpleobo:ncit.obo -o $@ --output-type md

