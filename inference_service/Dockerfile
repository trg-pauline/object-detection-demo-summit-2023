FROM registry.access.redhat.com/ubi9/python-39:latest

# This will avoid people forgetting to set no-cache-dir when building images
ENV PIP_NO_CACHE_DIR=1

USER 0

# Update base image for latest security updates
# (ubi images are not released that often)
RUN dnf -y  --setopt=tsflags=nodocs update && \
    dnf -y clean all --enablerepo='*' && \
    mkdir -pv log/vars log/images && \
    chown -R 1001:1001 log && \
    chmod -R 755 log

USER 1001

# Install micropipenv to deploy packages from Pipfile.lock
RUN pip install -U "micropipenv[toml]"

WORKDIR /opt/app-root/src

# Copy files
COPY Pipfile.lock api.py grpc_predict_v2_pb2_grpc.py \
    grpc_predict_v2_pb2.py remote_infer_grpc.py ./

COPY config config

COPY feature_extractor feature_extractor

# Install packages and cleanup
# (all commands are chained to minimize layer size)
RUN echo "Installing softwares and packages" && \
    # Install Python packages \
    micropipenv install && \
    rm -f ./Pipfile.lock && \
    # Fix permissions to support pip in Openshift environments \
    chmod -R g+w /opt/app-root/lib/python3.9/site-packages && \
    fix-permissions /opt/app-root -P 

CMD [ "python", "./api.py"]
