# Machine Learning with Big Data

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/elsdes3/big-data-ml)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/elsdes3/big-data-ml/master/3_combine_raw_data.ipynb)
![CI](https://github.com/elsdes3/big-data-ml/actions/workflows/main.yml/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/mit)
![OpenSource](https://badgen.net/badge/Open%20Source%20%3F/Yes%21/blue?icon=github)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
![prs-welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)
![pyup](https://pyup.io/repos/github/elsdes3/big-data-ml/shield.svg)

1. [Get word list](https://stackoverflow.com/a/45378529/4057186)
2. Tutorials
   - [1]](https://medium.com/@the.data.yoga/creating-word2vec-embeddings-on-a-large-text-corpus-with-pyspark-469007116551)
   - [2](https://gabefair.github.io/posts/2018/12/Understanding-Word2Vec-With-Pyspark/)
https://stackoverflow.com/a/56390537/4057186
https://stackoverflow.com/a/56391842/4057186

## [Table of Contents](#table-of-contents)
1. [About](#about)
2. [Project Organization](#project-organization)
3. [Pre-Requisites](#pre-requisites)
4. [Usage](#usage)
5. [Notes](#notes)

## [About](#about)

Use big-data tools ([PySpark](https://spark.apache.org/docs/latest/api/python/index.html)) to run topic modeling (unsupervised machine learning) on [Twitter data streamed](https://developer.twitter.com/en/docs/tutorials/stream-tweets-in-real-time) using [AWS Kinesis Firehose](https://aws.amazon.com/kinesis/data-firehose/).

## [Pre-Requisites](#pre-requisites)
1. The following AWS ([1](https://docs.aws.amazon.com/sdk-for-php/v3/developer-guide/guide_credentials_environment.html), [2](https://docs.aws.amazon.com/sdk-for-php/v3/developer-guide/guide_credentials_profiles.html)) and [Twitter Developer API](https://developer.twitter.com/en/docs/twitter-api/getting-started/getting-access-to-the-twitter-api) credentials
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_KEY`
   - `AWS_REGION`
   - `AWS_S3_BUCKET_NAME`
   - `TWITTER_API_KEY`
   - `TWITTER_API_KEY_SECRET`
   - `TWITTER_ACCESS_TOKEN`
   - `TWITTER_ACCESS_TOKEN_SECRET`

   must be stored in a [`.env` file](https://help.pythonanywhere.com/pages/environment-variables-for-web-apps/) stored one level up from the root directory of this project.i.e. one level up from the directory containing this `README.md` file.

## [Usage](#usage)
1. Create AWS resources
   ```bash
   make aws-create
   ```
   and, in
   ```bash
   inventories/production/host_vars/ec2host
   ```
   replace `...` in `ansible_host: ...` by the public IP address of the newly created EC2 instance from step 1.
2. Provision the EC2 host, excluding Python package installation
   ```bash
   make provision-pre-python
   ```
3. Install Python packages
   ```bash
   make provision-post-python
   ```
4. Start the Twitter streaming script locally
   ```bash
   make stream-local-start
   ```
5. Stop the Twitter streaming script running locally
   ```bash
   make stream-local-stop
   ```
6. Start the Twitter streaming script on the EC2 instance
   ```bash
   make stream-start
   ```
7. Stop the Twitter streaming script running on the EC2 instance
   ```bash
   make stream-stop
   ```
8. Run the Twitter streaming script locally, saving to a local CSV file but not to S3
   ```bash
   make stream-check
   ```
   Notes
   - there is no functionality to stop this script (it has to be stopped manually using Ctrl + C, or wait until the specified number of tweets, in `max_num_tweets_wanted` on line 217 of `twitter_s3.py`, have been retrieved)

   Pre-Requisites
   - the following environment variables must be manually set, before running this script, using
     ```bash
     export AWS_ACCESS_KEY_ID=...
     export AWS_SECRET_KEY=...
     export AWS_REGION=...
     export AWS_S3_BUCKET_NAME=...
     export TWITTER_API_KEY=...
     export TWITTER_API_KEY_SECRET=...
     export TWITTER_ACCESS_TOKEN=...
     export TWITTER_ACCESS_TOKEN_SECRET=...
     ```
9. Provision AWS SageMaker resources
   ```bash
   make sagemaker-create
   ```

   Pre-Requisites
   - an AWS IAM Role granting SageMaker full access to **one** pre-existing S3 bucket (the same bucket created in step 1.) must be created using the AWS console **before** running this step.
10. Run quantitative analysis
    ```bash
    make build
    ```
    and run the following two notebooks in order of the numbered prefix in their name
    - `3_combine_raw_data.ipynb` ([view](https://nbviewer.org/github/elsdes3/big-data-ml/blob/main/3_combine_raw_data.ipynb))
      - combines raw data into hourly CSVs
    - `4_data_processing.ipynb` ([view](https://nbviewer.org/github/elsdes3/big-data-ml/blob/main/4_data_processing.ipynb))
      - perform topic modeling (unsupervised machine learning) on combined hourly CSVs using PySpark and PySparkML
11. Destroy AWS SageMaker resources
    ```bash
    make sagemaker-destroy
    ```
10. Destroy AWS resources
    ```bash
    make aws-destroy
    ```

## [Notes](#notes)
1. Running the notebooks to create and destroy AWS resources in a non-interactive approach has not been verified. It is not currently known if this is possible.
2. The AWS credentials must be associated to a user group whose users have been granted programmatic access to AWS resources. In order to configure this for an IAM user group, see the documentation [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console).
3. The Twitter credentials must be for a user account with [elevated access](https://developer.twitter.com/en/support/twitter-api/v2) to the Twitter Developer API.

## [Project Organization](#project-organization)

    ├── LICENSE
    ├── .gitignore                          <- files and folders to be ignored by version control system
    ├── .pre-commit-config.yaml             <- configuration file for pre-commit hooks
    ├── .github
    │   ├── workflows
    │       └── main.yml                    <- configuration file for CI build on Github Actions
    ├── Makefile                            <- Makefile with commands like `make lint` or `make build`
    ├── README.md                           <- The top-level README for developers using this project.
    ├── ansible.cfg                         <- configuration file for Ansible
    ├── environment.yml                     <- configuration file to create environment to run project on Binder
    ├── manage_host.yml                     <- manage provisioning of EC2 host
    ├── read_data.py                        <- Python script to read streamed Twitter data that has been saved locally
    ├── streamer.py                         <- Wrapper script to control local or remote Twitter streaming
    ├── stream_twitter.yml                  <- stream Twitter data on EC2 instance
    ├── twitter_s3.py                       <- Python script to stream Twitter data locally or on EC2 instance
    ├── variables_run.yaml                  <- Ansible playbook variables
    ├── executed_notebooks
    |   └── *.ipynb                         <- executed notebooks, with output and execution datetime suffix in filename
    ├── data
    │   ├── raw                             <- The original, immutable data dump.
    |   └── processed                       <- Intermediate (transformed) data and final, canonical data sets for modeling.
    ├── 1_create_aws_resources.ipynb        <- create cloud resources on AWS
    ├── 2_create_sagemaker_resources.ipynb  <- create AWS SageMaker resources
    ├── 3_combine_raw_data.ipynb            <- combine raw tweets data in stored in S3 into CSV files
    ├── 4_data_processing.ipynb             <- unsupervised machine learning
    ├── 5_delete_aws_resources.ipynb        <- destroy cloud resources on AWS
    ├── 6_delete_aws_resources.ipynb        <- destroy AWS cloud resources
    ├── requirements.txt                    <- base packages required to execute all Jupyter notebooks (incl. jupyter)
    ├── inventories
    │   ├── production
    │       ├── host_vars                   <- variables to inject into Ansible playbooks, per target host
    │           └── ec2_host
    |       └── hosts                       <- Ansible inventory
    ├── src                                 <- Source code for use in this project.
    │   ├── __init__.py                     <- Makes src a Python module
    │   │
    │   ├── ansible                         <- Utilities to support Ansible orchestration playbooks
    |       ├── __init__.py                 <- Makes src.ansible a Python module
    │       └── playbook_utils.py
    │   │
    │   ├── cw                              <- Scripts to manage AWS CloudWatch Log Groups and Streams
    |       ├── __init__.py                 <- Makes src.cw a Python module
    │       └── cloudwatch_logs.py
    │   │
    │   ├── data                            <- Scripts to combine raw tweets data pre hour into a CSV file
    |       ├── __init__.py                 <- Makes src.data a Python module
    │       └── combine_data.py
    │   │
    │   ├── ec2                             <- Scripts to manage AWS EC2 instances and security groups
    |       ├── __init__.py                 <- Makes src.ec2 a Python module
    │       └── ec2_instances_sec_groups.py
    │   │
    │   ├── firehose                        <- Scripts to manage AWS Kinesis firehose data streams
    |       ├── __init__.py                 <- Makes src.firehose a Python module
    │       └── kinesis_firehose.py
    │   │
    │   ├── iam                             <- Scripts to manage AWS IAM
    |       ├── __init__.py                 <- Makes src.iam a Python module
    │       └── iam_roles.py
    │   │
    │   ├── keypairs                        <- Scripts to manage AWS EC2 SSH key pairs
    |       ├── __init__.py                 <- Makes src.keypairs a Python module
    │       └── ssh_keypairs.py
    │   │
    │   └── model_interpretation            <- Scripts to manage ML models
    |       ├── __init__.py                 <- Makes src.model_interpretation a Python module
    │       ├── import_export_models.py
    │       └── interpret_models.py
    │   │
    │   └── nlp                             <- Scripts to preform NLP tasks on text data
    |       ├── __init__.py                 <- Makes src.nlp a Python module
    │       ├── clean_text.py
    │   │
    │   └── s3                              <- Scripts to manage AWS S3 buckets
    |       ├── __init__.py                 <- Makes src.s3 a Python module
    │       ├── bucket_contents.py
    │       └── buckets.py
    │   │
    │   └── visualization                   <- Scripts to preform data visualization tasks
    |       ├── __init__.py                 <- Makes src.visualization a Python module
    │       ├── visualize.py
    │
    ├── papermill_runner.py                 <- Python functions that execute system shell commands.
    └── tox.ini                             <- tox file with settings for running tox; see https://tox.readthedocs.io/en/latest/

--------

<p><small>Project based on the <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. #cookiecutterdatascience</small></p>
