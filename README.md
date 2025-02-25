noWorkflow
==========

Copyright (c) 2015 Universidade Federal Fluminense (UFF).
Copyright (c) 2015 Polytechnic Institute of New York University.
All rights reserved.

The noWorkflow project aims at allowing scientists to benefit from provenance data analysis even when they don't use a workflow system. Also, the goal is to allow them to avoid using naming conventions to store files originated in previous executions. Currently, when this is not done, the result and intermediate files are overwritten by every new execution of the pipeline.

noWorkflow was developed in Python and it currently is able to capture provenance of Python scripts using Software Engineering techniques such as abstract syntax tree (AST) analysis, reflection, and profiling, to collect provenance without the need of a version control system or any other environment.

Installing and using noWorkflow is simple and easy. Please check our installation and basic usage guidelines below.

Team
----

The noWorkflow team is composed by researchers from Universidade Federal Fluminense (UFF) in Brazil and New York University (NYU), in the USA.

* Vanessa Braganholo (UFF)
* Fernando Chirigati (NYU)
* Juliana Freire (NYU)
* David Koop (NYU)
* Leonardo Murta (UFF)
* João Felipe Pimentel (UFF)

Publications
------------

* [MURTA, L. G. P.; BRAGANHOLO, V.; CHIRIGATI, F. S.; KOOP, D.; FREIRE, J.; noWorkflow: Capturing and Analyzing Provenance of Scripts. In: International Provenance and Annotation Workshop (IPAW), 2014, Cologne, Germany.] (https://github.com/gems-uff/noworkflow/raw/master/docs/ipaw2014.pdf)
* [PIMENTEL, J. F. N.; FREIRE, J.; MURTA, L. G. P.; BRAGANHOLO, V.; Collecting and Analyzing Provenance on Interactive Notebooks: when IPython meets noWorkflow. In: Theory and Practice of Provenance (TaPP), 2015, Edinburgh, Scotland.] (https://github.com/gems-uff/noworkflow/raw/master/docs/tapp2015.pdf)

Quick Installation
------------------

To install noWorkflow, you should follow these basic instructions:

If you have pip, just run:
```bash
$ pip install noworkflow[all]
```
This installs noWorkflow, PyPosAST, Flask, IPython Notebook and PySWIP.
The only requirement for running noWorkflow is PyPosAST. The other libraries are only used for provenance analysis.

If you only want to install noWorkflow and PyPosAST, please do:
```bash
$ pip install noworkflow
```

If you do not have pip, but already have Git (to clone our repository) and Python:
```bash
$ git clone git@github.com:JoaoFelipe/pyposast.git
$ cd pyposast
$ ./setup.py install
$ cd ..
$ git clone git@github.com:gems-uff/noworkflow.git
$ cd noworkflow/capture
$ ./setup.py install
```
This installs noWorkflow on your system.

Basic Usage
-----------

noWorkflow is transparent in the sense that it requires neither changes to the script, nor any laborious configuration. Run
```bash
now --help
```
to learn the usage options.

To run noWorkflow with a script called *simulation.py* with input data *data1.dat* and *data2.dat*, you should run
```bash
now run -v simulation.py data1.dat data2.dat
```
The *-v* option turns the verbose mode on, so that noWorkflow gives you feedback on the steps taken by the tool. The output, in this case, is similar to what follows.

```bash
$ now run -v simulation.py data1.dat data2.dat
[now] removing noWorkflow boilerplate
[now] setting up local provenance store
[now] collecting definition provenance
[now]   registering user-defined functions
[now] collecting deployment provenance
[now]   registering environment attributes
[now]   searching for module dependencies
[now]   registering provenance from 703 modules
[now] collecting execution provenance
[now]   executing the script
[now] the execution of trial 1 finished successfully
```
Each new run produces a different trial that will be stored with a sequential identification number in the relational database.

Verifying the module dependencies is a time consuming step, and scientists can bypass this step by using the *-b* flag if they know that no library or source code has changed. The current trial then inherits the module dependencies of the previous one.

It is possible to collect more information than what is collected by default, such as variable usages and dependencias.
To perform a dynamic program slicing and capture those information, just run
```bash
now run -e Tracer simulation.py data1.dat data2.dat
```

To list all trials, just run

```bash
now list
```
Assuming we run the experiment again and then run \texttt{now list}, the output would be as follows.

```bash
$ now list
[now] trials available in the provenance store:
  Trial 1: simulation.py data1.dat data2.dat
         with code hash aa49daae4ae8084af3602db436e895f08f14aba8
         ran from 2014-03-04 13:10:34.595995 to 2014-03-04 13:11:33.793083
  Trial 2: simulation.py data1.dat data2.dat
         with code hash aa49daae4ae8084af3602db436e895f08f14aba8
         ran from 2014-03-04 17:59:02.917920 to 2014-03-04 18:00:10.383637
```

To look at details of an specific trial, use
```bash
now show
```
This command has several options, such as *-m* to show module dependencies; *-d* to show function definitions; *-e* to show the environment context; *-a* to show function activations; and *-f* to show file accesses.

Running
```bash
now show -a 1
```
would show details of trial 1. Notice that the function name is preceded by the line number where the call was activated.

```bash
$ now show -a 1
[now] trial information:
  Id: 1
  Inherited Id: None
  Script: simulation.py
  Code hash: aa49daae4ae8084af3602db436e895f08f14aba8
  Start: 2014-03-04 13:10:34.595995
  Finish: 2014-03-04 13:11:33.793083
[now] this trial has the following function activation graph:
  42: run_simulation (2014-03-04 13:11:30.969055 -
                                2014-03-04 13:11:32.978796)
      Arguments: data_b = 'data2.dat', data_a = 'data1.dat'
      Globals: wait = 2
      Return value: [['0.0', '0.6'], ['1.0', '0.0'], ['1.0', '0.0'],
      ...
```

To restore files used by trial 1, run
```bash
$ now checkout -l -i 1
```

By default, the checkout command only restores the script used for the trial ("simulation.py"), even when it has imports and read files as input. Use the option "-l" to restore imported modules and the option "-i" to restore input files.
The checkout command track the evolution history. By default, subsequent trials are based on the previous Trial (e.g. Trial 2 is based on Trial 1). When you checkout a Trial, the next Trial will be based on the checked out Trial (e.g. Trial 3 based on Trial 1).


The remaining options of noWorkflow are *diff*, *export* and *vis*. The *diff* option compares two trials, and the *export* option exports provenance data of a given trial to Prolog facts, so inference queries can be run over the database.

The vis option starts a visualization tool that allows interactive analysis:
```bash
$ now vis -b
```
The visualization tool shows the evolution history, the trial information, an activation graph. It is also possible to compare different trials in the visualization tool.

The visualization tool requires Flask to be installed.
To install Flask, you can run
```bash
$ pip install flask
```

IPython Interface
-----------------

Another way to run, visualize, and query trials is to use IPython notebook.
To install IPython notebook, you can run
```bash
$ pip install ipython[all]
```

Then, to run ipython notebook, go to the project directory and execute:
```bash
$ ipython notebook
```

It will start a local webserver where you can create notebooks and run python code.

Before loading anything related to noworkflow on a notebook, you must initialize it:
```python
In  [1]: %load_ext noworkflow
    ...: import noworkflow.now.ipython as nip
```
It is equivalent to:
```python
In  [1]: %load_ext noworkflow
    ...: nip = %now_ip
```

After that, you can either run a new trial or load an existing object (History,Trial,Diff).

There are two ways to run a new trial:
1- Load an external file
```python
In  [1]: arg = 6

In  [2]: trial = %now_run script1.py $arg
    ...: trial
Out [2]: <Trial 5> # Loads the trial object represented as a graph
```
2- Load the code inside a cell
```python
In  [3]: %%now_run --name script2 --interactive
    ...: l = range(arg)
    ...: c = sum(l)
    ...: print(c)
         6
Out [3]: <Trial 6> # Loads the trial object represented as a graph

In  [4]: c
Out [4]: 6
```
Both modes supports all the `now run` parameters
The --interactive mode allows the cell to share variables with the notebook.

Loading existing trials, histories and diffs:
```python
In  [5]: trial = nip.Trial(2) # Loads trial with Id = 2
    ...: trial # Shows trial graph
Out [5]: <Trial 2>

In  [6]: history = nip.History() # Loads history
    ...: history # Shows history graph
Out [6]: <History>

In  [7]: diff = nip.Diff(1, 2) # Loads diff between trial 1 and 2
    ...: diff # Shows diff graph
Out [7]: <Diff 1 2>
```

There are attributes on those objects to change the graph visualization, width, height and filter values. Please, check the documentation by running the following code on ipython notebook
```python
In  [8]: trial?

In  [9]: history?
```

It is also possible to run prolog queries on IPython notebook. To do so, you will need to install SWI-Prolog with shared libraries and the pyswip module.

You can install pyswip module with the command:
```bash
$ pip install pyswip-alt
```

Check how to install SWI-Prolog with shared libraries at https://github.com/yuce/pyswip/blob/master/INSTALL

You can install pyswip

To query a specific trial, you can do:
```python
In  [10]: result = trial.query("activation(_, 550, X, _, _, _)")
    ...: next(result) # The result is a generator
Out [10]: {'X': 'range'}
```

To check the existing rules, please do:
```python
In  [11]: trial.prolog_rules()
Out [11]: [...]
```

Finally, it is possible to run the CLI commands inside ipython notebook:
```python
In  [12]: !now export ${trial.id}
Out [12]: %
     ...: % FACT: activation(trial_id, id, name, start, finish, caller_activation_id).
     ...: %
     ...: ...
```

Running


Included Software
-----------------

Parts of the following software were used by noWorkflow directly or in an adapted form:

The Python Debugger
Copyright (c) 2001-2013 Python Software Foundation.
All Rights Reserved.

Acknowledgements
----------------

We would like to thank CNPq, FAPERJ, and the National Science Foundation (CNS-1229185, CNS-1153503, IIS-1142013) for partially supporting this work.

License Terms
-------------

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

