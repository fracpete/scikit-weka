# sklearn-weka-plugin
Makes [Weka](https://www.cs.waikato.ac.nz/ml/weka/) algorithms available in [scikit-learn](https://scikit-learn.org/).

Built on top of the [python-weka-wrapper3](https://github.com/fracpete/python-weka-wrapper3) 
library, it uses the [javabridge](https://pypi.python.org/pypi/javabridge) library
under the hood for communicating with Weka objects in the Java Virtual Machine.


## Functionality

Currently available:

* Classifiers (classification/regression)
* Clusters
* Filters

Things to be aware of:

* You need to start/stop the JVM in your Python code before you can use Weka.
* Unlikely to work in multi-threaded/process environments (like flask).
* Jupyter Notebooks does not play nice with javabridge, as you might have to restart the kernel in order to be able 
  to restart the JVM (e.g., with additional packages).
* The conversion to Weka data structures involves guesswork, i.e., if targets are to be treated as nominal, you need 
  to convert the numeric values to strings (e.g., using `to_nominal_labels` and/or `to_nominal_attributes` functions 
  from `sklweka.dataset` or the `MakeNominal` transformer from `sklweka.preprocessing`).


## Installation

* create virtual environment

  ```commandline
  virtualenv -p /usr/bin/python3.7 venv
  ```
  
  or:
  
  ```commandline
  python3 -m venv venv
  ```

* install the *python-weka-wrapper3* library, see instructions here:

  http://fracpete.github.io/python-weka-wrapper3/install.html
  
* install the sklearn-weka-plugin  library itself

  * from local source

    ```commandline
    ./venv/bin/pip install .   
    ```
    
  * from Github repository

    ```commandline
    ./venv/bin/pip install git+https://github.com/fracpete/sklearn-weka-plugin.git   
    ```

## Examples

Here is a quick example (of which you need to adjust the paths to the datasets, of course):

```python
import sklweka.jvm as jvm
from sklweka.dataset import load_arff, to_nominal_labels
from sklweka.classifiers import WekaEstimator
from sklweka.clusters import WekaCluster
from sklweka.preprocessing import WekaTransformer
from sklearn.model_selection import cross_val_score

# start JVM with Weka package support
jvm.start(packages=True)

# regression
X, y, meta = load_arff("/some/where/bolts.arff", class_index="last")
lr = WekaEstimator(classname="weka.classifiers.functions.LinearRegression")
scores = cross_val_score(lr, X, y, cv=10, scoring='neg_root_mean_squared_error')
print("Cross-validating LR on bolts (negRMSE)\n", scores)

# classification
X, y, meta = load_arff("/some/where/iris.arff", "class_index=last")
y = to_nominal_labels(y)
j48 = WekaEstimator(classname="weka.classifiers.trees.J48", options=["-M", "3"])
j48.fit(X, y)
scores = j48.predict(X)
probas = j48.predict_proba(X)
print("\nJ48 on iris\nactual label -> predicted label, probabilities")
for i in range(len(y)):
    print(y[i], "->", scores[i], probas[i])

# clustering
X, y, meta = load_arff("/some/where/iris.arff", class_index="last")
cl = WekaCluster(classname="weka.clusterers.SimpleKMeans", options=["-N", "3"])
clusters = cl.fit_predict(X)
print("\nSimpleKMeans on iris\nclass label -> cluster")
for i in range(len(y)):
    print(y[i], "->", clusters[i])

# preprocessing
X, y, meta = load_arff("/some/where/bolts.arff", class_index="last")
tr = WekaTransformer(classname="weka.filters.unsupervised.attribute.Standardize", options=["-unset-class-temporarily"])
X_new, y_new = tr.fit(X, y).transform(X, y)
print("\nStandardize filter")
print("\ntransformed X:\n", X_new)
print("\ntransformed y:\n", y_new)

# stop JVM
jvm.stop()
```


See the example repository for more examples:

http://github.com/fracpete/sklearn-weka-plugin-examples

Direct links:

* [classifiers](https://github.com/fracpete/sklearn-weka-plugin-examples/blob/main/src/sklwekaexamples/classifiers.py)
* [clusters](https://github.com/fracpete/sklearn-weka-plugin-examples/blob/main/src/sklwekaexamples/clusters.py)
* [preprocessing](https://github.com/fracpete/sklearn-weka-plugin-examples/blob/main/src/sklwekaexamples/preprocessing.py)
* [pipeline](https://github.com/fracpete/sklearn-weka-plugin-examples/blob/main/src/sklwekaexamples/pipeline.py)

## Documentation

You can find the project documentation here:

https://fracpete.github.io/sklearn-weka-plugin/

