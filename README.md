# gcloud-data-notes
Notes for Data Engineer – Professional Certification Preparation for Google

## Google Cloud SDK

1. Download the [gcloud SDK](https://cloud.google.com/sdk)
1. Activate your project `gcloud init` or `gcloud config configurations activate [profile name]`
1. See [Managing SDK Configurations](https://cloud.google.com/sdk/docs/configurations) and 
other [All How-to Guides](https://cloud.google.com/sdk/docs/how-to)

### gcloud core commands

```bash

gcloud auth list
gcloud config list
gcloud info
gcloud help
gcloud help compute instances create

```

## Big Query

https://cloud.google.com/bigquery/docs/

https://github.com/cloudacademy/optimizing-bigquery



## Data Flow / Apache Beam

https://github.com/cloudacademy/beam

https://learning.oreilly.com/learning-paths/learning-path-apache/9781491995228/
    https://github.com/eljefe6a/beamexample
    https://docs.google.com/presentation/d/1ln5KndBTiskEOGa1QmYSCq16YWO9Dtmj7ZwzjU7SsW4/present?slide=id.g19b6635698_3_4


## Airflow

https://medium.com/bluecore-engineering/were-all-using-airflow-wrong-and-how-to-fix-it-a56f14cb0753




---

Composer/Airflow

* Composer Quickstart: https://cloud.google.com/composer/docs/quickstart
* Apache Airflow: http://airflow.apache.org/
* Concepts: https://airflow.apache.org/concepts.html   (Good one for overall concepts of airflow)
* Airflow Github: https://github.com/apache/incubator-airflow
* Common Pitfalls: https://cwiki.apache.org/confluence/display/AIRFLOW/Common+Pitfalls
 
* ETL best practices - https://gtoonstra.github.io/etl-with-airflow/
* Astronomer.io Best Practices: https://www.astronomer.io/guides/dag-best-practices/
* airflow-gcp-examples : https://github.com/alexvanboxel/airflow-gcp-examples

Blogs: 
* http://michal.karzynski.pl/blog/2017/03/19/developing-workflows-with-apache-airflow/
* https://medium.com/@nehiljain/lessons-learnt-while-airflow-ing-32d3b7fc3fbf
* https://medium.com/snaptravel/airflow-part-2-lessons-learned-793fa3c0841e
* http://soverycode.com/airflow-in-production-a-fictional-example/
* https://www.agari.com/email-security-blog/airflow-agari/
 

Testing DAGs

Google Composer is based on airflow. 

All operators, etc, they re-contribute back to airflow. 
The only issue is there is no Repo or tag or anything that allows you to pull down Composer’s version of airflow. 
They have some from airflow 1.9, some from 1.10 some from the master branch.

Composer is currently in 1.9 but has released 1.10 for beta. We are using the 1.9 right now.  Because of this, we use 
their ‘base’ Docker image for testing our dags. We build off of that and run tests. Here are a couple of Dag Testing 
blogs that we based our testing off of:

We test for dags (this is in addition to – and comes before -  e2e testing with behave

* Validity and Integrity of the dag
* Non Cyclic
* Workflow is going where we expect (task to task, etc.)
* We can query properties for the Dag and for each Operator (task) to test/verify them

Blogs:

* https://blog.usejournal.com/testing-in-airflow-part-1-dag-validation-tests-dag-definition-tests-and-unit-tests-2aa94970570c
* https://medium.com/wbaa/datas-inferno-7-circles-of-data-testing-hell-with-airflow-cef4adff58d8
 
Dataflow

I haven’t looked much into this since before I started.. But here are some Youtube videos etc that I looked at.

https://www.youtube.com/watch?v=7Q5VJzJMG3A
https://www.youtube.com/watch?v=ss6SaNBHLGc
