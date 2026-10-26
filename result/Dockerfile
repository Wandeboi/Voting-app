FROM node:22-slim

# add curl for healthcheck
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends curl tini && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app


# have nodemon available for local dev use (file watching)
RUN npm install -g nodemon

COPY package*.json ./

RUN npm ci && \
 npm cache clean --force && \
 mv /usr/local/app/node_modules /node_modules && \
 rm -rf /usr/local/lib/node_modules/npm /usr/local/bin/npm /usr/local/bin/npx

COPY . .

# create non-root user and give it ownership of the app
RUN addgroup --system appuser && adduser --system --ingroup appuser appuser && \
    chown -R appuser:appuser /usr/local/app

ENV PORT=5000
EXPOSE 5000

USER appuser

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
