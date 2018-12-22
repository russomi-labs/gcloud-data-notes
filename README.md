# gcloud-data-notes

Study notes for [Data Engineer – Professional Certification Preparation for Google](https://cloudacademy.com/learning-paths/data-engineer-professional-certification-preparation-for-google-83/)

## Install and Configure Google Cloud SDK 

1. Download the [gcloud SDK](https://cloud.google.com/sdk)
1. Activate your project `gcloud init` or `gcloud config configurations activate [profile name]`
1. See [Managing SDK Configurations](https://cloud.google.com/sdk/docs/configurations) and 
[All How-to Guides](https://cloud.google.com/sdk/docs/how-to)
1. The gcloud config files are located under `~/.config/` by default.

### Core Commands

```bash
gcloud auth list
gcloud config list
gcloud info
gcloud help
gcloud help compute instances create
```

## Google Big Data Products

* https://cloud.google.com/products/big-data/
* https://cloud.google.com/docs/tutorials#big_data

## Google Training

* https://www.coursera.org/specializations/gcp-data-machine-learning
* https://www.coursera.org/learn/serverless-data-analysis-bigquery-cloud-dataflow-gcp
* https://github.com/GoogleCloudPlatform/training-data-analyst

## Big Query

* https://cloud.google.com/bigquery/docs/
* https://github.com/cloudacademy/optimizing-bigquery

## Data Flow / Apache Beam

This file contains text you can copy and paste for the examples in Cloud Academy's _Introduction to Google Cloud Dataflow_ course.  

See [Apache Beam Java SDK Quickstart](https://beam.apache.org/get-started/quickstart-java/) for additional tutorials.

### Before you Begin

**Note:** Creating a gcloud service account is required to run locally for `gs` buckets.  

See [Before you Begin](https://cloud.google.com/dataflow/docs/quickstarts/quickstart-java-maven#before-you-begin) for steps 
to follow to create a service account, assign it a role, and download the key.

Set the environment variable `GOOGLE_APPLICATION_CREDENTIALS` to the file path of the JSON file that contains your 
service account key. This variable only applies to your current shell session, so if you open a new session, set the 
variable again.

```bash
gcloud auth application-default login

export GOOGLE_APPLICATION_CREDENTIALS="~/.config/gcloud/application_default_credentials.json"
```

### What is a pipeline?

Taken from slides for [Learn Stream processing with Apache Beam](https://goo.gl/DlJvJL)

* A pipeline is a ...
    * A Directed Acyclic Graph (DAG) of data transformations
    * Possibly unbounded collections of data flow on the edges
    * May include multiple sources and multiple sinks
    * Optimized and executed as a unit

* A pipeline describes ...
    * What are you computing?
    * Where in event time?
    * When in processing time?
    * How do refinements relate?

* A pipeline describes ...
    * What = Transformations
    * Where = Windowing
    * When = Watermarks + Triggers
    * How = Accumulation

### Building and Running a Pipeline

Installing on your own computer: https://cloud.google.com/dataflow/docs/quickstarts  
Transforms: https://beam.apache.org/documentation/sdks/javadoc/2.0.0/org/apache/beam/sdk/transforms/package-summary.html

```bash
git clone https://github.com/cloudacademy/beam.git
cd beam/examples/java8
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalLineCount
```
```bash
gsutil cat gs://dataflow-samples/shakespeare/kinglear.txt | wc
```

### Deploying a Pipeline on Cloud Dataflow

Setup Project and Bucket environment variables...

```bash
export PROJECT=enter project name
export BUCKET=gs://dataflow-$PROJECT

# or ...

nano ~/.profile
    PROJECT=[Your Project ID]
    BUCKET=gs://dataflow-$PROJECT
```

Create a Google Storage bucket...
 
```bash
gsutil mb $BUCKET
cd ~/beam/examples/java8
```

Run the MinimalLineCount example:

```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalLineCountArgs \
  -Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp"
```

Run the line LineCount example:

```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.LineCount \
  -Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp \
  --output=$BUCKET/linecount"
```

### Custom Transforms

```bash
cd ~/beam/examples/java8
```
```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
  -Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp \
  --output=$BUCKET/wordcounts"
```

### Composite Transforms

```bash
cd ~/beam/examples/java8
```
```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.complete.game.UserScore \
-Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp/ \
  --output=$BUCKET/scores"
```

### Windowing

```bash
cd ~/beam/examples/java8
```
```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.complete.game.HourlyTeamScore \
-Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp/ \
  --output=$BUCKET/scores \
  --startMin=2015-11-16-16-00 \
  --stopMin=2015-11-17-16-00"
```

### Running LeaderBoard

```bash
bq mk game
```

API Console Credentials: https://console.developers.google.com/projectselector/apis/credentials

```bash
export GOOGLE_APPLICATION_CREDENTIALS="[Path]/[Credentials file]"
cd ~/beam/examples/java8
```
```bash
mvn compile exec:java  -Dexec.mainClass=org.apache.beam.examples.complete.game.injector.Injector \
-Dexec.args="$PROJECT game none"
```
```bash
cd ~/beam/examples/java8
```
```bash
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.complete.game.LeaderBoard \
-Dexec.args="--runner=DataflowRunner \
  --project=$PROJECT \
  --tempLocation=$BUCKET/temp/ \
  --output=$BUCKET/leaderboard \
  --dataset=game \
  --topic=projects/$PROJECT/topics/game"
```


### Resources
* [Dataflow Templates](https://github.com/GoogleCloudPlatform/DataflowTemplates)
* https://github.com/cloudacademy/beam
* https://learning.oreilly.com/learning-paths/learning-path-apache/9781491995228/
* https://github.com/eljefe6a/beamexample
* https://docs.google.com/presentation/d/1ln5KndBTiskEOGa1QmYSCq16YWO9Dtmj7ZwzjU7SsW4/present?slide=id.g19b6635698_3_4
* https://www.youtube.com/watch?v=7Q5VJzJMG3A
* https://www.youtube.com/watch?v=ss6SaNBHLGc

## Composer / Airflow

Google Composer (Apache Airflow) Notes

### Resources

* Composer Quickstart: https://cloud.google.com/composer/docs/quickstart
* Apache Airflow: http://airflow.apache.org/
* Concepts: https://airflow.apache.org/concepts.html   (Good one for overall concepts of airflow)
* Airflow Github: https://github.com/apache/incubator-airflow
* Common Pitfalls: https://cwiki.apache.org/confluence/display/AIRFLOW/Common+Pitfalls 
* :raised_hands: ETL best practices - https://gtoonstra.github.io/etl-with-airflow/
* Astronomer.io Best Practices: https://www.astronomer.io/guides/dag-best-practices/
* airflow-gcp-examples : https://github.com/alexvanboxel/airflow-gcp-examples

### Blogs

* http://michal.karzynski.pl/blog/2017/03/19/developing-workflows-with-apache-airflow/
* https://medium.com/@nehiljain/lessons-learnt-while-airflow-ing-32d3b7fc3fbf
* https://medium.com/snaptravel/airflow-part-2-lessons-learned-793fa3c0841e
* http://soverycode.com/airflow-in-production-a-fictional-example/
* https://www.agari.com/email-security-blog/airflow-agari/
* https://medium.com/bluecore-engineering/were-all-using-airflow-wrong-and-how-to-fix-it-a56f14cb0753
 
### Testing DAGs

Google Composer is based on airflow. All operators, etc, they re-contribute back to airflow. 

The only issue is there is no Repo or tag or anything that allows you to pull down Composer’s version of airflow. 
They have some from airflow 1.9, some from 1.10 some from the master branch.  Composer is currently in 1.9 but has 
released 1.10 for beta. We are using the 1.9 right now.  

Because of this, use their ‘base’ Docker image for testing DAGs. We build off of that and run tests. 

#### What to Verify

* Validity and Integrity of the dag
* Non Cyclic
* Workflow is going where we expect (task to task, etc.)
* We can query properties for the Dag and for each Operator (task) to test/verify them

#### Resources

* https://blog.usejournal.com/testing-in-airflow-part-1-dag-validation-tests-dag-definition-tests-and-unit-tests-2aa94970570c
* https://medium.com/wbaa/datas-inferno-7-circles-of-data-testing-hell-with-airflow-cef4adff58d8

