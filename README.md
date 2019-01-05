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

---

## Google Big Data Products

* https://cloud.google.com/products/big-data/
* https://cloud.google.com/docs/tutorials#big_data

## Google Training

* https://www.coursera.org/specializations/gcp-data-machine-learning
* https://www.coursera.org/learn/serverless-data-analysis-bigquery-cloud-dataflow-gcp
* https://github.com/GoogleCloudPlatform/training-data-analyst

---

## Big Query

* https://cloud.google.com/bigquery/docs/
* https://github.com/cloudacademy/optimizing-bigquery

---

## Data Flow / Apache Beam

This file contains text you can copy and paste for the examples in Cloud Academy's _Introduction to Google Cloud Dataflow_ course.  

See [Apache Beam Java SDK Quickstart](https://beam.apache.org/get-started/quickstart-java/) for additional tutorials.
* [Apache Beam WordCount Examples](https://beam.apache.org/get-started/wordcount-example/)
* [Apache Beam Example Pipelines](https://github.com/apache/beam/tree/master/examples/java)

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

See [Setting up a Java Development Environment for Apache Beam on Google Cloud Platform](https://medium.com/google-cloud/setting-up-a-java-development-environment-for-apache-beam-on-google-cloud-platform-ec0c6c9fbb39).


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
export PROJECT=$(gcloud config get-value core/project)
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

## Create a new java project with the `mvn archetype:generate` command

```bash
mvn archetype:generate \
      -DarchetypeGroupId=org.apache.beam \
      -DarchetypeArtifactId=beam-sdks-java-maven-archetypes-examples \
      -DarchetypeVersion=2.9.0 \
      -DgroupId=com.russomi \
      -DartifactId=word-count-beam \
      -Dversion="0.1" \
      -Dpackage=com.russomi.beam.examples \
      -DinteractiveMode=false
```

## Execute the Wordcount sample locally

```bash
mvn compile exec:java -Dexec.mainClass=com.russomi.beam.examples.WordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts" -Pdirect-runner
```

See the [Java Quickstart](https://beam.apache.org/get-started/quickstart-java/) for more.

---

### Dataflow Templates

#### [Overview](https://cloud.google.com/dataflow/docs/guides/templates/overview)

> Cloud Dataflow templates allow you to stage your pipelines on Cloud Storage and execute them from a variety of environments. You can use one of the Google-provided templates or create your own.

> Templates provide you with additional benefits compared to traditional Cloud Dataflow deployment, such as:

    * Pipeline execution does not require you to recompile your code every time.
    * You can execute your pipelines without the development environment and associated dependencies that are common with traditional deployment. This is useful for scheduling recurring batch jobs.
    * Runtime parameters allow you to customize the execution of the pipeline.
    * Non-technical users can execute templates with the Google Cloud Platform Console, gcloud command-line tool, or the REST API.

#### [Creating Templates](https://cloud.google.com/dataflow/docs/guides/templates/creating-templates)

> Cloud Dataflow templates use runtime parameters to accept values that are only available during pipeline execution. 
To customize the execution of a templated pipeline, you can pass these parameters to functions that run within the 
pipeline (such as a `DoFn`).

##### Creating and staging templates

```bash
   mvn compile exec:java \
     -Dexec.mainClass=com.example.myclass \
     -Dexec.args="--runner=DataflowRunner \
                  --project=[YOUR_PROJECT_ID] \
                  --stagingLocation=gs://[YOUR_BUCKET_NAME]/staging \
                  --templateLocation=gs://[YOUR_BUCKET_NAME]/templates/MyTemplate"
```

#### [Executing Templates](https://cloud.google.com/dataflow/docs/guides/templates/executing-templates)

> This example creates a batch job from a template that reads a text file and writes an output text file.

```bash
    gcloud dataflow jobs run [JOB_NAME] \
        --gcs-location gs://[YOUR_BUCKET_NAME]/templates/MyTemplate \
        --parameters inputFile=gs://[YOUR_BUCKET_NAME]/input/my_input.txt,outputFile=gs://[YOUR_BUCKET_NAME]/output/my_output
```

> Google provides a set of open-source Cloud Dataflow templates.
  
* [Google Provided Templates](https://cloud.google.com/dataflow/docs/guides/templates/provided-templates)
* [DataflowTemplates Repo on Github](https://github.com/GoogleCloudPlatform/DataflowTemplates)

### Dataflow / Apache Beam Resources

* [Setting up a Java Development Environment for Apache Beam on Google Cloud Platform](https://medium.com/google-cloud/setting-up-a-java-development-environment-for-apache-beam-on-google-cloud-platform-ec0c6c9fbb39)
* [Dataflow Templates](https://github.com/GoogleCloudPlatform/DataflowTemplates)
* [Data Engineer – Professional Certification Preparation for Google](https://cloudacademy.com/learning-paths/data-engineer-professional-certification-preparation-for-google-83/)
* [cloudacademy/beam](https://github.com/cloudacademy/beam)
* [Learning Path: Apache Beam for Stream Processing](https://learning.oreilly.com/learning-paths/learning-path-apache/9781491995228/)
* [Learn stream processing with Apache Beam - Strata 2017](https://docs.google.com/presentation/d/1ln5KndBTiskEOGa1QmYSCq16YWO9Dtmj7ZwzjU7SsW4/present?slide=id.g19b6635698_3_4)
* [Tutorial - Learn stream processing with Apache Beam - Strata 2017](https://github.com/eljefe6a/beamexample)
* [Advancing Serverless Data Processing in Cloud Dataflow (Cloud Next '18)](https://www.youtube.com/watch?v=7Q5VJzJMG3A)
* [Apache Beam and Google Cloud Dataflow](https://www.youtube.com/watch?v=ss6SaNBHLGc)
* [Performing ETL from a Relational Database into BigQuery](https://cloud.google.com/solutions/performing-etl-from-relational-database-into-bigquery)
* [GoogleCloudPlatform/bigquery-etl-dataflow-sample](https://github.com/GoogleCloudPlatform/bigquery-etl-dataflow-sample)
* [Quickstart Using Java and Apache Maven](https://cloud.google.com/dataflow/docs/quickstarts/quickstart-java-maven)
* [All Google Cloud Tutorials](https://cloud.google.com/docs/tutorials)
* [jorwalk/data-engineering-gcp](https://github.com/jorwalk/data-engineering-gcp)

---

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

---

# [Setting up a Java Development Environment for Apache Beam on Google Cloud Platform](https://medium.com/google-cloud/setting-up-a-java-development-environment-for-apache-beam-on-google-cloud-platform-ec0c6c9fbb39)

> These instructions will show you how to get a development environment up and running to start developing Java Dataflow 
jobs. By the end you’ll be able to run a Dataflow job locally in debug mode, execute code in a REPL to speed your 
development cycles, and submit your job to Google Cloud Dataflow.

## Google Cloud Setup
export PROJECT=$(gcloud config get-value core/project)

## Create a bucket
gsutil mb -c regional -l us-central1 gs://$PROJECT
export BUCKET=$PROJECT

## Create a BigQuery dataset
bq mk DataflowJavaSetup
bq mk java_quickstart

## Cloning the Dataflow Templates Repo

git clone https://github.com/GoogleCloudPlatform/DataflowTemplates

# Run the Apache Beam Pipeline Locally

mvn clean && mvn compile

## Create a Run/Debug configuration for the class that defines this Apache Beam pipeline

Add Application
working directory drop down select %MAVEN_REPOSITORY%
Choose class
Add arguments

```bash
--project=sandbox
--stagingLocation=gs://sandbox/staging
--tempLocation=gs://sandbox/temp
--templateLocation=gs://sandbox/templates/GcsToBigQueryTemplate.json
--runner=TestDataflowRunner
--javascriptTextTransformGcsPath=gs://sandbox/resources/UDF/sample_UDF.js
--JSONPath=gs://sandbox/resources/schemas/sample_schema.json
--javascriptTextTransformFunctionName=transform
--inputFilePattern=gs://sandbox/data/data.txt
--outputTable=sandbox:java_quickstart.colorful_coffee_people
--bigQueryLoadingTemporaryDirectory=gs://sandbox/bq_load_temp/
```

## Set a break point and run the debugger

line 93 of TextIOToBigQuery.java

Right Click > “Debug TextIOToBigQuery”

Right Click and choose “Evaluate expression” 

# Run the Apache Beam Pipeline on the Google Cloud Dataflow Runner

sample_schema.json:
```
{
 “BigQuery Schema”: [
 {
 “name”: “location”,
 “type”: “STRING”
 },
 {
 “name”: “name”,
 “type”: “STRING”
 },
 {
 “name”: “age”,
 “type”: “STRING”
 },
 {
 “name”: “color”,
 “type”: “STRING”
 },
 {
 “name”: “coffee”,
 “type”: “STRING”
 }
 ]
}
```

sample_UDF.js:

```javascript
function transform(line) {
 var values = line.split(‘,’);
 var obj = new Object();
 obj.location = values[0];
 obj.name = values[1];
 obj.age = values[2];
 obj.color = values[3];
 obj.coffee = values[4];
 var jsonString = JSON.stringify(obj);
 
return jsonString;
}
```


data.txt:

````text
US,joe,18,green,late
CAN,jan,33,red,cortado
MEX,jonah,56,yellow,cappuccino
````

## Stage these files in Google Cloud Storage:

```bash
gsutil cp ./sample_UDF.js gs://$BUCKET/resources/UDF/
gsutil cp ./sample_schema.json gs://$BUCKET/resources/schemas/
gsutil cp ./data.txt gs://$BUCKET/data/

mvn compile exec:java -Dexec.mainClass=com.google.cloud.teleport.templates.TextIOToBigQuery -Dexec.cleanupDaemonThreads=false -Dexec.args=” \
--project=$PROJECT \
--stagingLocation=gs://$BUCKET/staging \
--tempLocation=gs://$BUCKET/temp \
--templateLocation=gs://$BUCKET/templates/GcsToBigQueryTemplate.json \
--runner=DataflowRunner”
```

## Submit the job

```bash
gcloud dataflow jobs run colorful-coffee-people-gcs-test-to-big-query \
--gcs-location=gs://$BUCKET/templates/GcsToBigQueryTemplate.json \
--zone=us-central1-f \
--parameters=javascriptTextTransformGcsPath=gs://$BUCKET/resources/UDF/sample_UDF.js,JSONPath=gs://$BUCKET/resources/schemas/sample_schema.json,javascriptTextTransformFunctionName=transform,inputFilePattern=gs://$BUCKET/data/data.txt,outputTable=$PROJECT:java_quickstart.colorful_coffee_people,bigQueryLoadingTemporaryDirectory=gs://$BUCKET/bq_load_temp/
```
