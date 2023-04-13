# ScyllaDB feature store example (online storage)
This example project demonstrates a machine learning feature store use case for ScyllaDB.

You'll set up a database with flight data and use ScyllaDB to analyze flight delays.

# What is a feature store?

[![](https://mermaid.ink/img/pako:eNptkLtuwzAMRX-F4NDJ_gEPBdq62bI0mWp7ICRaFqqHQckpgjj_XtVJOxTlxMe5xCUvqKJmbHB08VNNJBmObR-gxFN3FAppjOJZwwOQMcKGcik0ZRqgrh_hhr50B3V2jtpnqCEGZwNDylHIMAx3ZMPXHWc1QRaywQazLUqcV2i7ffHhfif_qUamvAjDiVXZnVZ4vYsSy-mvZte9Mbk6W89A85wGrNCzeLK6XHv5RnvME3vusSmpJvnosQ_XwtGS4-EcFDZZFq5wmYtPbi0ZIY_NSC6VLmtbbOxv79u-WOFM4T3GH-b6BaPtdAY?type=png)](https://mermaid.live/edit#pako:eNptkLtuwzAMRX-F4NDJ_gEPBdq62bI0mWp7ICRaFqqHQckpgjj_XtVJOxTlxMe5xCUvqKJmbHB08VNNJBmObR-gxFN3FAppjOJZwwOQMcKGcik0ZRqgrh_hhr50B3V2jtpnqCEGZwNDylHIMAx3ZMPXHWc1QRaywQazLUqcV2i7ffHhfif_qUamvAjDiVXZnVZ4vYsSy-mvZte9Mbk6W89A85wGrNCzeLK6XHv5RnvME3vusSmpJvnosQ_XwtGS4-EcFDZZFq5wmYtPbi0ZIY_NSC6VLmtbbOxv79u-WOFM4T3GH-b6BaPtdAY)



## How ScyllaDB can help you build a feature store?
ScyllaDB is a real-time low latency NoSQL database that is best suited for feature stores where you require <10 ms latency consistently, and need peta-byte scalability.

* **Low latency**: ScyllaDB provides 15-30 ms P99 latency consistently. Low latency can speed up training time and leads to fatser model development.
* **High throughput**: Training requires huge amounts of data and processing large datasets with many millions of operations per second.
* **Large scale**: ScyllaDB can handle petabytes of data while still providing great performance.


# Get started

## Sing up for a ScyllaDB Cloud account
To complete this project sign up for a free trial account on S[cyllaDB Cloud](https://cloud.scylladb.com/user/signup).

If you prefer to use a self-hosted version of ScyllaDB, [see installation options here](https://www.scylladb.com/download/#open-source) like Docker.


## Prerequisites:
* cqlsh
* Python 3.7+


## Create table
Run the `schema.cql` file to create keyspace `feature_store` and table `flight_features`:
```bash
cqlsh "node-0.aws_us_east_1.xxxxxxxxx.clusters.scylla.cloud" 9042 -u scylla -p "password" -f schema.cql
```

Now let's connect to the database and make sure the table is created properly:
```bash
cqlsh "node-0.aws_us_east_1.xxxxxxxxx.clusters.scylla.cloud" 9042 -u scylla -p "password"

scylla@cqlsh> DESCRIBE TABLE pet;
```

```sql
create table feature_store.flight_features(
	FL_DATE TIMESTAMP,
	OP_CARRIER TEXT,
	OP_CARRIER_FL_NUM INT,
	ORIGIN TEXT,
	DEST TEXT,
	CRS_DEP_TIME INT,
	DEP_TIME FLOAT,
	DEP_DELAY FLOAT,
	TAXI_OUT FLOAT,
	WHEELS_OFF FLOAT,
	WHEELS_ON FLOAT,
	TAXI_IN FLOAT,
	CRS_ARR_TIME INT,
	ARR_TIME FLOAT,
	ARR_DELAY FLOAT,
	CANCELLED FLOAT,
	CANCELLATION_CODE TEXT,
	DIVERTED FLOAT,
	CRS_ELAPSED_TIME FLOAT,
	ACTUAL_ELAPSED_TIME FLOAT,
	AIR_TIME FLOAT,
	DISTANCE FLOAT,
	CARRIER_DELAY FLOAT,
	WEATHER_DELAY FLOAT,
	NAS_DELAY FLOAT,
	SECURITY_DELAY FLOAT,
	LATE_AIRCRAFT_DELAY FLOAT,
	PRIMARY KEY (OP_CARRIER_FL_NUM)
);
```

## Import the dataset into ScyllaDB
```bash
cqlsh "node-0.aws_us_east_1.xxxxxxxxx.clusters.scylla.cloud" 9042 -u scylla -p "password"

scylla@cqlsh> COPY feature_store.flight_features FROM 'flight_dataset.csv';
```

This will start ingesting data into your ScyllaDB instance:
```
op_carrier_fl_num|actual_elapsed_time|air_time|arr_delay|arr_time|cancellation_code|cancelled|carrier_delay|crs_arr_time|crs_dep_time|crs_elapsed_time|dep_delay|dep_time|dest|distance|diverted|fl_date            |late_aircraft_delay|nas_delay|op_carrier|origin|security_delay|taxi_in|taxi_out|weather_delay|wheels_off|wheels_on|
-----------------+-------------------+--------+---------+--------+-----------------+---------+-------------+------------+------------+----------------+---------+--------+----+--------+--------+-------------------+-------------------+---------+----------+------+--------------+-------+--------+-------------+----------+---------+
             4317|               96.0|    73.0|    -19.0|  2113.0|                 |      0.0|             |        2132|        2040|           112.0|     -3.0|  2037.0|MLI |   373.0|     0.0|2018-12-31 02:00:00|                   |         |OO        |DTW   |              |    5.0|    18.0|             |    2055.0|   2108.0|
             3372|               94.0|    74.0|     81.0|  1500.0|                 |      0.0|          0.0|        1339|        1150|           109.0|     96.0|  1326.0|RNO |   564.0|     0.0|2018-12-31 02:00:00|               81.0|      0.0|OO        |SEA   |           0.0|    3.0|    17.0|          0.0|    1343.0|   1457.0|
             1584|              385.0|   348.0|    -21.0|  2023.0|                 |      0.0|             |        2034|        1700|           394.0|     -9.0|  1658.0|SFO |  2565.0|     0.0|2018-12-31 02:00:00|                   |         |UA        |EWR   |              |   13.0|    33.0|             |    1731.0|   2019.0|
             4830|              119.0|    85.0|    -35.0|  1431.0|                 |      0.0|             |        1437|        1245|           136.0|    -13.0|  1232.0|MSP |   546.0|     0.0|2018-12-31 02:00:00|                   |         |OO        |MSP   |              |   16.0|    18.0|             |    1250.0|   1415.0|
             2731|              158.0|   146.0|    -19.0|  1911.0|                 |      0.0|             |        1930|        1800|           160.0|     -8.0|  1800.0|MSP |   842.0|     0.0|2018-12-31 02:00:00|                   |         |WN        |PVD   |              |    7.0|     7.0|             |    1807.0|   1904.0|
```

Now that you have some sample data to play with, let's see a decision tree example.

## Decision tree classification
Create a new virtual environment and activate it:
```bash
virtualenv env
source env/bi/activate
```

Install requirements:
```bash
pip install scikit-learn pandas scylla-driver
```

Connect to ScyllaDB:
```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier # Import Decision Tree Classifier
from sklearn.model_selection import train_test_split # Import train_test_split function
from sklearn import metrics #Import scikit-learn metrics module for accuracy calculation
from sqlalchemy import create_engine
from cassandra.cluster import Cluster, ExecutionProfile, EXEC_PROFILE_DEFAULT
from cassandra.policies import DCAwareRoundRobinPolicy, TokenAwarePolicy
from cassandra.auth import PlainTextAuthProvider

def pandas_factory(colnames, rows):
    return pd.DataFrame(rows, columns=colnames)

def getCluster():
    profile = ExecutionProfile(load_balancing_policy=TokenAwarePolicy(DCAwareRoundRobinPolicy(local_dc='AWS_US_EAST_1')),
                               row_factory=pandas_factory)
    return Cluster(
        execution_profiles={EXEC_PROFILE_DEFAULT: profile},
        contact_points=[
           ""
        ],
        port=9042,
        auth_provider = PlainTextAuthProvider(username="scylla", password="******"))


cluster = getCluster()
session = cluster.connect()
```

Query data into a dataframe:
```python
query = "SELECT * FROM demo.flight_features;"
rows = session.execute(query)
df = rows._current_rows
```

Split the dataset into features and target variable:
Feature attributes:
* `actual_elapsed_time`
* `air_time`
* `arr_time`
* `crs_arr_time` 
* `crs_dep_time`
* `crs_elapsed_time` 
* `dep_time` 
* `distance` 
* `taxi_in` 
* `taxi_out`
* `wheels_off`
* `wheels_on`
* `arr_delay`

Target attribute:
* `dep_delay`


```python
#Features
feature_cols = ["actual_elapsed_time", "air_time", 
                "arr_time", "crs_arr_time", "crs_dep_time", 
                "crs_elapsed_time", "dep_time", "distance", "taxi_in", "taxi_out", 
                "wheels_off", "wheels_on", "arr_delay"]

X = df[feature_cols]
y = df.dep_delay # Target variable

# Split dataset into training set and test set
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=1) # 70% training and 30% test
```

Create the classifier:
```python
# Create Decision Tree classifer object
clf = DecisionTreeClassifier()

# Train Decision Tree Classifer
clf = clf.fit(X_train,y_train)

#Predict the response for test dataset
y_pred = clf.predict(X_test)

print("Accuracy:",metrics.accuracy_score(y_test, y_pred))

Accuracy: 0.052884615384615384
```

Decision tree visualization:
```python
from sklearn.tree import export_graphviz
from six import StringIO  
from IPython.display import Image  
import pydotplus

dot_data = StringIO()
export_graphviz(clf, out_file=dot_data,  
                filled=True, rounded=True,
                special_characters=True,feature_names = feature_cols,class_names=['0','1'])
graph = pydotplus.graph_from_dot_data(dot_data.getvalue())  
graph.write_png('flight_delayed.png')
Image(graph.create_png())
```

## Jupyter notebook
You can run and modify the classifier script using the Jupyter notebook file in the repository. Before running it locally, make sure to edit the `config.py` file with your proper (local or ScyllaDB Cloud) ScyllaDB configuration.

You can also run the classifier in notebook form using [GitHub code editor](https://github.dev/scylladb/scylladb-feature-store/blob/master/decision_tree.ipynb).
