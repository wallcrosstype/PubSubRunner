# PubSubRunner
Python Boilerplate for Google Cloud PubSub running task in python

## Table of Contents
- [Setup](#setup)
- [Config](#config)
- [Test](#test)
- [Run](#run)
- [Debug](#debug)

## Setup
1. create service account in google cloud project

    - add permission pub/sub admin to account
    - pubsub must be enabled

2. create key

    - create json key , copy as services.json at root folder of project

3. set ENV

    - GOOGLE_APPLICATION_CREDENTIALS=services.json
    - CLOUD_PROJECT=your-project
    - CLOUD_PUBSUB_SUBSCRIBE_TOPIC=subscribe's topic
    - CLOUD_PUBSUB_SUBSCRIBE_SUBSCRIPTION=subscription of subscribe's topic
    - CLOUD_PUBSUB_PUBLISH_TOPIC=publish's topic

4. add in requirements.txt
    - -e git+https://github.com/roticagas/PubSubRunner.git#egg=PubSubRunner

## Config 
By using Environment variable 

in testing you can directly set with 

    os.environ['CLOUD_PUBSUB_CHECK'] = 'false'

###### PORT (8080)
port of Runner, This is standalone application.

###### CLOUD_PROJECT () 
*NEED TO BE SET*
 
google cloud project name

###### CLOUD_PUBSUB_SUBSCRIBE_TOPIC (from) 
topic name of subscription

###### CLOUD_PUBSUB_SUBSCRIBE_SUBSCRIPTION (from-subscription)
subscription name

###### CLOUD_PUBSUB_PUBLISH_TOPIC (to) 
*OPTIONAL*

topic name to be publish after task succeed
###### CLOUD_PUBSUB_DEAD_LETTER_TOPIC (dead-letter) 
*OPTIONAL*

topic name to be publish if message is not in json format 

###### CLOUD_PUBSUB_MAX_LEASE_DURATION (7200)
seconds for task to complete before pubsub not acknowledge response

###### CLOUD_PUBSUB_MAX_DEADLINE (600)
seconds for deadline 

###### CLOUD_PUBSUB_CHECK (true)
if true: check topic and subscription name and create it if not exist

###### CLOUD_PUBSUB_ACK (true)
if true: ack message after publish message or dead letter message

## Test
*TBD using pycharm*

## Run 

To run with sample code

add messenger_main.py with this code 

    import json
    import logging
    import os
    import time
    
    from PubSubRunner.cloud_util import CloudUtil
    from PubSubRunner.runner_application import RunnerApplication
    from PubSubRunner.runner_config import RunnerConfig
    
    logging.basicConfig(level=logging.DEBUG)
    
    
    def deliver(message):
        logging.debug('deliver: {}'.format(message))
        if int(message['delay']) > 0:
            time.sleep(int(message['delay']))
        logging.debug('deliver: {} done.'.format(message))
    
    
    def task(message, runner=deliver):
        """
        :param message: json from pubsub
        :param runner:
        :return: message: 'directory':output path relative with output_directory for 'filename' input
        inside directory contains hls processed files inside.
        """
        logging.debug('receive: {}'.format(message))
        assert 'delay' in message, 'KeyError: filename'
        assert len(message['delay']) > 0
        runner(message)
        logging.debug('send: {}'.format(message))
        return message
    
    
    if __name__ == '__main__':
        # intend to run by python3 messenger_main.py
        # check console with LINE: INFO:root:publish : {"delay": "{delay}"} like
        # > INFO:root:publish : {"delay": "1"}
        # and look around for warning like
        # > WARNING:google.cloud.pubsub_v1.subscriber._protocol.leaser:Dropping 2 items because they were leased too long.
        #
        # when complete 600 seconds (in case create by this lib) it will shown
        # > INFO:google.cloud.pubsub_v1.subscriber._protocol.streaming_pull_manager:Observed recoverable stream error
        # 504 Deadline Exceeded
        #
        os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = '../services.json'  # expect file to be here
        os.environ['CLOUD_PROJECT'] = 'your-project'  # TODO: change this name, topic and subscription will auto gen
        os.environ['CLOUD_PUBSUB_SUBSCRIBE_TOPIC'] = 'delay-03'
        os.environ['CLOUD_PUBSUB_SUBSCRIBE_SUBSCRIPTION'] = 'delay-03-1'
        os.environ['CLOUD_PUBSUB_PUBLISH_TOPIC'] = 'delay-04'
        os.environ['CLOUD_PUBSUB_MAX_LEASE_DURATION'] = '60'  # play with this value
        os.environ['CLOUD_PUBSUB_ACK_DEADLINE'] = '30'  # play with this value
    
    
        def job():  # this function is run after subscribe topic for easier testing.  
            config = RunnerConfig()
            CloudUtil.publish_data(config.cloud_project, config.cloud_pubsub_subscribe_topic, json.dumps({'delay': '1'}))
            CloudUtil.publish_data(config.cloud_project, config.cloud_pubsub_subscribe_topic, json.dumps({'delay': '15'}))
            CloudUtil.publish_data(config.cloud_project, config.cloud_pubsub_subscribe_topic, json.dumps({'delay': '25'}))
            CloudUtil.publish_data(config.cloud_project, config.cloud_pubsub_subscribe_topic, json.dumps({'delay': '45'}))
    
    
        RunnerApplication(task).run(job=job)


run

    python messenger_main.py
     
or
     
    python3 messenger_main.py 

## Debug
set logging level

    import logging
    logging.basicConfig(level=logging.DEBUG)

it will shown like

    DEBUG:root:HINT: gcloud pubsub topics list
    DEBUG:google.auth.transport.requests:Making request: POST https://oauth2.googleapis.com/token
    DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): oauth2.googleapis.com:443
    DEBUG:urllib3.connectionpool:https://oauth2.googleapis.com:443 "POST /token HTTP/1.1" 200 None
    