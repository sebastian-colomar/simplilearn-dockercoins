FROM ruby:alpine
RUN apk add build-base && gem install sinatra && gem install thin
COPY hasher.rb /src/
