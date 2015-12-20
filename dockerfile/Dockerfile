FROM ubuntu:15.10

RUN apt-get update && apt-get -y upgrade && apt-get -y install gcc g++ make automake autoconf curl bzip2 unzip libbz2-dev python-dev graphviz imagemagick zlib1g-dev openjdk-8-jdk curl libboost-all-dev libtool && apt-get clean

WORKDIR /opt

RUN curl -o moses.zip https://codeload.github.com/moses-smt/mosesdecoder/zip/master && unzip moses.zip && rm moses.zip
RUN cd mosesdecoder-master && ./bjam -j4 && mkdir -p tools

RUN curl -o gizapp.zip https://codeload.github.com/moses-smt/giza-pp/zip/master && unzip gizapp.zip && rm gizapp.zip
RUN cd giza-pp-master && make && cp GIZA++-v2/GIZA++ GIZA++-v2/snt2cooc.out mkcls-v2/mkcls /opt/mosesdecoder-master/tools

RUN apt-get -y install mecab mecab-ipadic mecab-ipadic-utf8 mecab-jumandic mecab-jumandic-utf8 mecab-naist-jdic mecab-naist-jdic-eucjp mecab-utils && apt-get clean

RUN mkdir corpus && cd corpus && curl -o examples.utf.gz ftp://ftp.monash.edu.au/pub/nihongo/examples.utf.gz
RUN cd /opt/corpus && gzip -dc examples.utf.gz | grep ^A: | cut -f1 | sed 's/^A: //' | mecab -Owakati > tanaka.ja
RUN cd /opt/corpus && gzip -dc examples.utf.gz | grep ^A: | cut -f2 | sed 's/#.*$//' > tanaka.en
RUN cd /opt/corpus && /opt/mosesdecoder-master/bin/lmplz -o 5 -S 80% -T /tmp < tanaka.ja > tanaka.ja.arpa

RUN ./mosesdecoder-master/scripts/training/train-model.perl \
    --root-dir . \
    --corpus corpus/tanaka \
    --f en \
    --e ja \
    --lm 0:5:/opt/corpus/tanaka.ja.arpa \
    --external-bin-dir ./mosesdecoder-master/tools

CMD /bin/bash
