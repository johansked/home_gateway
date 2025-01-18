###
# Build step
###
FROM balenalib/raspberrypi5-debian-node:20.12-build as build

WORKDIR /usr/src/app
COPY ./app/package.json package.json

RUN JOBS=MAX npm install -g --production --unsafe-perm \
       node-red@4.0.8 \
       node-red-contrib-balena@1.0.1 \
       node-red-contrib-influxdb@0.7.0 \
       node-red-contrib-nordpool-api-plus@4.5.5 \
       node-red-contrib-modbus@5.43.0 \
       node-red-contrib-buffer-parser@3.2.2 \
       && JOBS=MAX npm install --unsafe-perm --production && npm cache clean --force

###
# Runtime image
###
FROM balenalib/raspberrypi5-debian-node:20.12-run

RUN apt-get update && apt-get install -yq --no-install-recommends \
  python3-dev \
  libatomic1 \
  git \
  openssh-client \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Defines our working directory in container
WORKDIR /usr/src/app

# Copy over the files created in the previous step, including lib/, bin/
COPY --from=build /usr/local/bin /usr/local/bin
COPY --from=build /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=build /usr/src/app/node_modules /usr/src/app/node_modules
COPY ./ssh_config /etc/ssh/ssh_config

# This will copy all files in our root to the working  directory in the container
COPY ./app ./

# server.js will run when container starts up on the device
CMD ["bash", "/usr/src/app/start.sh"]