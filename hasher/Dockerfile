FROM ruby:alpine
RUN apk add build-base
RUN gem install sinatra
RUN gem install thin
COPY hasher.rb /src/
