FROM python:alpine

# The following information is typically supplied by the DEV team
# Install CURL for testing purposes (NOT IN PRODUCTION)
RUN pip install redis && pip install requests && apk add curl

# Copy the source code from the git repo into the Docker image
COPY . .

# Metadata for the container
ENTRYPOINT ["python"]
CMD ["worker.py"]
