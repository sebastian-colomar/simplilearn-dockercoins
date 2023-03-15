FROM node:4-slim

# The following information is typically supplied by the DEV team
RUN npm install express && npm install redis

# Copy the source code from the git repo into the Docker image
COPY . .

# Metadata for the container
ENTRYPOINT ["node"]
CMD ["webui.js"]
EXPOSE 8080
