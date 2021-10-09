# CMLJobRunOperator
A simple custom Airflow operator definition to trigger jobs in the Cloudera Machine Learning (CML) service using the [v2 CML REST API](https://docs.cloudera.com/machine-learning/cloud/api/topics/ml-api-v2.html) from a Cloudera Data Engineering (CDE) [managed Arflow instance](https://docs.cloudera.com/data-engineering/cloud/manage-jobs/topics/cde-airflow-dag-pipeline.html) DAG.

## Summary
1. [Get a CML API Key](https://github.com/curtishoward/cml_operator/blob/main/README.md#generate-and-test-cml-api-key)
2. [Configure a CDE Airflow HTTP Connection to the CML API](https://github.com/curtishoward/cml_operator#configure-a-cde-airflow-http-connection-to-cml)
3. Add the [CMLJobRunOperator code](https://github.com/curtishoward/cml_operator/blob/main/cml_operator_example.py#L7-L60) to the beginning of your CDE Airflow DAG
4. Use the operator as follows, to trigger a CML job in DAG:
```
CMLJobRunOperator(task_id=cml_task_id,
    project='cml-project-name',
    job='cml-job-name',
    dag=your_dag)
```

## Generate and Test CML API Key
You'll first need to generate an authentication key for the CML REST API, based on these steps:
https://docs.cloudera.com/machine-learning/cloud/api/topics/ml-api-v2.html#autoId1

In short, you'll need to follow the menu options in the screen capture below, to generate the key.  Importantly, copy both the CML workspace URL hostname (```ml-5905fd52-3aa.curtis1.a465-9q4k.cloudera.site``` in the example below) and the **API Key** (the longer of the two keys generated after clicking __Create API Key__)
![api_key](api_key.png)

You can confirm access to the REST API using the ```curl``` command and the long API key noted from above:
```
export API_KEY="50695752efa4a278fbe399de050489e9660546dc22d4754966f56674fae65261.1e87e86d1d5cbeb608fefbed85b5410f74440e93af0e44ef1dc19d07a6a71ec5"
export CML_HOST="ml-5905fd52-3aa.curtis1.a465-9q4k.cloudera.site"
$ curl -X GET -H "authorization: Bearer $API_KEY" "https://${CML_HOST}/api/v2/projects" | python -m json.tool
{
    "projects": [
        {
            "id": "lv1g-4hu5-a359-wagc",
            "name": "curtis-proj",
            "owner": {
                "username": "curtis",
...
```

## Configure a CDE Airflow HTTP Connection to CML
Next, from the Airflow UI of your CDE virtual cluster, access the Admin -> Connections menu and add a new connection matching the values in the fields below (Conn ID=cml_rest_api, Type=HTTP, Schema=https):
![airflow_connection](airflow_connection.png)

Importantly, the 'Extras' fields should include the API key that you generated above in the following format:
```
{"authorization": "Bearer 50695752efa4a278fbe399de050489e9660546dc22d4754966f56674fae65261.1e87e86d1d5cbeb608fefbed85b5410f74440e93af0e44ef1dc19d07a6a71ec5"}
```

## Run the Example
Next, update the CML project and job name that should be triggered:
  ```
  cml_job = CMLJobRunOperator(
    task_id='cml_job_task',
    project='curtis-proj',
    job='cml_test_job',
    dag=dag)
  ```
  For example, the config above would trigger the following CML job:
  ![cml_project_job](cml_project_job.png)
  
 
Finally, create a a resource and Airflow job in CDE using the [CDE CLI](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html):
```
cde resource create --name cml-operator-resource
cde resource upload --name cml-operator-resource --local-path ./cml_operator_example.py
cde job create --name cml-operator-test-job --type airflow --mount-1-resource cml-operator-resource --dag-file cml_operator_example.py
```
  
This should trigger the Airflow DAG, and you should see a successful execution both from CDE Airflow:
![cde](cde.png)
  
... and from the CML Jobs UI:
![cml](cml.png)



