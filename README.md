# NCIt OBO Edition
This project aims to develop an [OBO Library](http://obofoundry.org)-friendly release of the [NCI Thesaurus](https://ncit.nci.nih.gov/ncitbrowser/). Goals include adopting OBO conventions for term identifiers and ontology dereferencing, as well as improved semantic integration with existing OBO ontologies. We are working in collaboration with NCI staff under NCI/Leidos contract #17X118.

## Download
The OBO edition is currently a work in progress, so ontology content may change frequently. The in-progress ontology is available here: http://purl.obolibrary.org/obo/ncit.owl

Additional downloads may be available in the [project wiki](https://github.com/NCI-Thesaurus/thesaurus-obo-edition/wiki).

## Documentation
Please see the [project wiki](https://github.com/NCI-Thesaurus/thesaurus-obo-edition/wiki) for more information.

## Release process for Developers

1. Enter `src/ontology` directory
1. Build the docker image to get all the tools together (alternatively, install the tools locally, see top of `Makefile`)
1. Run the make build process
1. Deploy the release on GitHub. Make sure the `GHVERSION` starts with a lower case `v` and corresponds to the exact date the build process above was run.

```
cd src/ontology
make docker-build
sh odk.sh make all -B
make deploy_release GHVERSION=v2022-08-19
```

## Contact
More information about this project can be provided by Melissa Haendel (@mellybelly). For details on software implementing the OBO Edition, contact Jim Balhoff (balhoff@renci.org).
