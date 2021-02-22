OSG on AWS ECS
==============

This repo includes the files necessary to run jobs in AWS Elastic Container Service.

Prerequisites
-------------

You must install the AWS ECS command line.  Instructions are on the [AWS website](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).

You will need a Token from the OSG in order to connect the containerized worker node to the OSG Global Pool.

The [osgvo-docker-pilot](https://github.com/opensciencegrid/osgvo-docker-pilot) was customized in order to remove the non-root requirements.  Additionally, the location of tokens was moved in the container to `/etc/condor/tokens.d`

Initial Setup
-------------

First, you must initialize the Amazon ECS CLI.  Detailed instructions are on the [AWS website](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html).  

The two commands you will need to run are similar to:

Create the profile:

    $ ecs-cli configure profile --profile-name <profile_name> --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY

Create a cluster with a launch type using [FARGATE](https://aws.amazon.com/fargate/).

    $ ecs-cli configure --cluster <cluster_name> --default-launch-type FARGATE --config-name <config_name> --region <region>


Starting the Container
----------------------

Following the tutorial on the [AWS website](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-fargate.html)

1. Create the IAM role as documented in the tutorial.
2. Configure the AWS CLI, which you may have already done above in the [Intial Setup](#initial-website) section.
3. Create the cluster as described in the tutorial.  You will not need to create the security groups, as the OSG container only needs outgoing internet access.

        $ ecs-cli up --cluster-config <cluster> --ecs-profile <profile>

    Be sure to note the VPC and subnet IDs that are in the output from the above command.

4. The compose file in this repo has the skeleton needed to start the osg worker node.  You will need to put the token into an environment file named `.env` in the same directory as the `docker-compose.yml` file.

5. Start the container:

        $ ecs-cli compose --project-name <project> service up --create-log-groups --cluster-config <cluster> --ecs-profile <profile>

6. Check the status of the container:

        $ ecs-cli compose --project-name <project> service ps --cluster-config <cluster> --ecs-profile <profile>

7. You can view the logs from the container with:

        $ ecs-cli logs --task-id <task_id> --follow --cluster-config <cluster> --ecs-profile <profile>


Clean Up
--------

Delete the container with the `compose down` command.

    $ ecs-cli compose --project-name <project> service down --cluster-config <cluster> --ecs-profile <profile>

And delete the cluster

    $ ecs-cli down --force --cluster-config <cluster> --ecs-profile <profile>

