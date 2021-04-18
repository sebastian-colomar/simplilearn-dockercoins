FROM ruby:alpine

# The following information is typically supplied by the DEV team
# Install CURL for testing purposes (NOT IN PRODUCTION)
RUN apk add build-base && gem install sinatra && gem install thin && apk add curl

# Copy the source code from the git repo into the Docker image
COPY . .

# Metadata for the container
ENTRYPOINT ["ruby"]
CMD ["hasher.rb"]
EXPOSE 8080
