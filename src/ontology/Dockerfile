FROM obolibrary/odkfull:v1.3.1

LABEL maintainer="cjmungall@lbl.gov" 

# Install blazegraph
ENV BLAZEGRAPH=1.7
ENV PATH "/tools/blazegraph/bin:$PATH"
RUN wget -nv https://github.com/balhoff/blazegraph-runner/releases/download/v$BLAZEGRAPH/blazegraph-runner-$BLAZEGRAPH.tgz \
&& tar -zxvf blazegraph-runner-$BLAZEGRAPH.tgz \
&& mv blazegraph-runner-$BLAZEGRAPH /tools/blazegraph

# Install blazegraph
ENV NCITUTILS=0.6
ENV PATH "/tools/ncitutils/bin:$PATH"
RUN wget -nv https://github.com/NCI-Thesaurus/ncit-utils/releases/download/v$NCITUTILS/ncit-utils-$NCITUTILS.tgz \
&& tar -zxvf ncit-utils-$NCITUTILS.tgz \
&& mv ncit-utils-$NCITUTILS /tools/ncitutils

