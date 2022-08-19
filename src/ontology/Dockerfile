FROM obolibrary/odkfull:v1.3.1

LABEL maintainer="cjmungall@lbl.gov" 

# Install blazegraph
ENV BLAZEGRAPH=1.7
ENV PATH "/tools/blazegraph/bin:$PATH"
RUN wget -nv https://github.com/balhoff/blazegraph-runner/releases/download/v$BLAZEGRAPH/blazegraph-runner-$BLAZEGRAPH.tgz \
&& tar -zxvf blazegraph-runner-$BLAZEGRAPH.tgz \
&& mv blazegraph-runner-$BLAZEGRAPH /tools/blazegraph


