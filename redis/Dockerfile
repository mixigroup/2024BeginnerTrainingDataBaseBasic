FROM ruby:latest

WORKDIR /app
COPY Gemfile /app/Gemfile
COPY redis_pubsub_demo.rb /app/redis_pubsub_demo.rb

RUN bundle install

