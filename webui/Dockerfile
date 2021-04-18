FROM node:4-slim

# The following information is typically supplied by the DEV team
# Install CURL for testing purposes (NOT IN PRODUCTION)
RUN npm install express && npm install redis && apk add curl

# Copy the source code from the git repo into the Docker image
COPY . .

# Metadata for the container
ENTRYPOINT ["node"]
CMD ["webui.js"]
EXPOSE 8080
