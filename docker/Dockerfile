FROM ghcr.io/tobitti0/docker-avisynthplus:4.3.1-ubuntu2004 as build

ENV        DEBIAN_FRONTEND=noninteractive

ADD modules /tmp/modules

RUN set -xe && \
    apt-get update && \
    apt-get install --no-install-recommends -y \
      curl git make gcc g++ cmake libboost-all-dev

# join_logo_scp_trial
RUN cd /tmp/modules/chapter_exe/src && \
    make && \
    mv chapter_exe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/logoframe/src && \
    make && \
    mv logoframe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/join_logo_scp/src && \
    make && \
    mv join_logo_scp /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/tsdivider/ && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release .. && \
    make && \
    mv tsdivider /tmp/modules/join_logo_scp_trial/bin/ && \
    mv /tmp/modules/join_logo_scp_trial /join_logo_scp_trial && \
    cd /join_logo_scp_trial

RUN set -xe && \
    git clone https://github.com/tobitti0/delogo-AviSynthPlus-Linux && \
    cd delogo-AviSynthPlus-Linux/src && \
    make && \
    cp libdelogo.so /join_logo_scp_trial

RUN set -xe && \
    curl -O -sL https://deb.nodesource.com/setup_10.x && \
    cp setup_10.x /join_logo_scp_trial

FROM ghcr.io/tobitti0/docker-avisynthplus:4.3.1-ubuntu2004 as release
ENV        DEBIAN_FRONTEND=noninteractive
MAINTAINER  Tobitti <mail@tobitti.net>
WORKDIR /join_logo_scp_trial
COPY --from=build /join_logo_scp_trial /join_logo_scp_trial

RUN bash setup_10.x && \
    apt-get update && \
    apt-get install --no-install-recommends -y nodejs libboost-filesystem-dev libboost-program-options-dev libboost-system-dev && \
    node -v && \
    npm --version &&\
    mv libdelogo.so /usr/local/lib/avisynth && \
    ls /usr/local/lib/avisynth && \
    npm install
ENTRYPOINT ["npm", "start", "--" ,"-i"]
CMD ["--help"]
