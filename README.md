# smapeaqn.ml
*Machine Learning for Multi-Objective Performance Optimization of Self-Adaptive Systems modelled as SMAPEA Queuing Networks*.

## Description

**This project is related to the paper that I have submitted to the [SoSyM Theme Section *AI-enhanced Model-Driven Engineering*](https://www.sosym.org/theme_sections/cfp/cfp-SoSyM-TI-AI-MDE21.pdf)**. In particular, it supports the experimentation conducted in the paper, by providing everything that is needed, i.e. results, artifacts and replication instructions:
  - *SMAPEA-QN-emergency-handling.jsimg* contains the SMAPEA Queuing Network (QN) for the considered case study, as an xmi representation conforming to JSimGraph from [JMT](http://jmt.sourceforge.net/).
    [Note: A backup file is also provided, corresponding to the *-bkp* suffix.]
  - *res_sim_SMAPEA-QN-emergency-handling.jsim* is a temporary file used to store simulation results.
  - *Custom_NSGA-II_executions.zip* folder contains the results of step **(1) Meta-heuristic**, in particular:
     - *09_00625_ps_ne_det05_1min* subfolder refers to test sets building.
     - *09_00625_10_100_detx_1min* subfolder refers to test sets building.
  - *Datasets_building.zip* contains training and test sets resulting from step **(2) Training and Test Sets Builder** (manually performed).
  - *Auto-Weka_executions* folder is related to step **(3) Machine-Learning Classifiers**, in particular:
     - *Timeout_analysis* subfolder contains the results obtained through the execution of Auto-Weka aimed at choosing possibly convenient values for Auto-Weka timeouts. Such results are provided in the form of 2 *.zip* files, namely *Timeout_analysis_Part1* and *Timeout_analysis_Part2*
     - *2-30-5_funct_rules_trees* subfolder contains the results obtained through the execution of Auto-Weka aimed at recommending alternative solutions to the Controller Selection Policy problem defined on SMAPEA QNs. Such results are provided in the form of 3 *.zip* files, namely *CSP_solution_Part1*, *CSP_solution_Part2* and *CSP_solution_Part3*
  - *SoSyM_AISE_ThemeIssue_Experiment.xlsx* contains all the experimental results as an Excel spreadsheet.

## Experiment Replication
In order to replicate the experiment, please proceed as follows.

### (1) Meta-heuristic
  **1.** Follow the [Instructions for Running the cutom NSGA-II genetic algorithm](#nsga-run), thus obtaining a Run configuration.

  **2.** Run the obtained configuration multiple times, as follows:
  
    - One execution with population size 10 and number of evaluations 100, for each considered workload - i.e. det(x), 0.5 <= x <= 2.5 with step 0.25);
    - One execution with population size {10, 30, 60, 90, 120, 180, 360} and number of evaluations {60, 180, 360, 720, 1080, 1440, 1800}, for the heaviest workload - i.e. det(0.5).

  [**Note**: To save space, after performance evaluation of a solution, the corresponding *.jsimg* file is deleted from the file system. To maintain the generated SMAPEA QN models, remove the call to *deleteModelFiles* method within *CspSimpleNSGAIIRunner* class.]

### (2) Training and Test Sets Builder (currently manual)
Training and test sets *.arff* files must be created, based on the *.tsv* files returned by step (1). In particular, training sets shall be derived from point *6.a)* of step (1), whilst test sets shall be carried out from point *6.b)*. Please refer to *.arff* files provided in the *Datasets building* subfolder for the syntax of such files.

[**Note:** For each NSGA-II execution, *VARIABLES.tsv* contains near-Pareto CSP solutions in terms of routing probabilities for Normal and Critical MAPE job classes; *FITNESS.tsv*, instead, contains the corresponding mean system response times estimated for each solution, modulo Normal and Critical modes.]

### (3) Machine-Learning Classifiers
   **Pre-requisites**: [Auto-Weka v0.5](http://www.cs.ubc.ca/labs/beta/Projects/autoweka/) equipped with [SMAC v2.10.03](https://www.cs.ubc.ca/labs/beta/Projects/SMAC/v2.10.03/quickstart.html).

   [**Note**: A [*.zip* file](https://github.com/davewilsonfbc/smapeaqn.ml/blob/master/autoweka-0.5.zip) is available at this repository, providing a predefined Auto-Weka bundle]

#### a) Auto-Weka timeout analysis
In order to identify possibly convenient values for Auto-Weka timeouts, an analysis has been conducted, which can be replicated as follows.

[**Note**: Auto-Weka experiments that have been built for this step are contained in the subfolder *timeout analysis*.]

   **1.** An Auto-Weka experiment has been created with respect to different combinations of timeout values, by following the [Instructions for building Auto-Weka experiments](#auto-weka-build) and by providing training and test sets for *CriticalPlan* job class (i.e. the most demanding one) towards *CloudController* (i.e. the remote controller).
   In particular, the following combinations of Optimisation Timeout, Training Run Timeout and Attribute Selection Timeout, have been considered:
   {1, 10, 2}; {1, 30, 2}; {2, 20, 2}; {2, 30, 5}; {4, 60, 10}; {8, 60, 10}, {12, 60, 10}.

   **2.** Each of the 7 Auto-Weka experiments created in the previous step for *CriticalPlan* towards *CloudController* has been run, by following [Instructions for running Auto-Weka experiments](#auto-weka-run).

#### b) Auto-Weka execution for Controller Selection Policy problem
Once timeout values have been chosen, Auto-Weka has been exploited in order to obtain *S$\rightarrow$M* routing probabilities for MAPE job classes, as follows.

[**Note**: Auto-Weka experiments that have been built for this step are contained in the subfolder *2-30-5_funct_rules_trees*.]

   **1.** An Auto-Weka experiment has been created for each combination of (MAPE job class, controller), by following the [Instructions for building Auto-Weka experiments](#auto-weka-build) and properly providing training and test sets, with respect to the timeout values that have been previously chosen.

   **2.** Each of the ($8 \times 3 = 24$) Auto-Weka experiments created in the previous step for (MAPE job class, controller) combinations has been run, by following the [Instructions for running Auto-Weka experiments](#auto-weka-run).


#### c) Auto-Weka results post-processing
   **1.** Among several files created by Auto-Weka after an experiment run, a *predictions.0.csv* file is created in the experiment folder, containing predictions of the selected classifier with respect to the provided test set.
For each Auto-Weka experiment that has been run during the experimentation, average predicted values modulo the considered workloads have been calculated (directly into the corresponding *predictions.0.csv* file) and then normalized over the three controllers.

**2.** The obtained CSP solutions - one per each considered workload - have been simulated within JSimGraph tool from [JMT](http://jmt.sourceforge.net/) for sake of comparison.





## <a name="nsga-run">Instructions for Running the cutom NSGA-II genetic algorithm</a>

In the following, general instructions are provided for easing step **(1) Meta-heuristic**.

  **1.** Checkout (or download as *.zip* and extract) the GitHub project at https://github.com/davewilsonfbc/smapeaqn.moo, import the project into the Eclipse workspace and the related Maven dependencies.

  **2.** Create a folder named *experiment* in the main project folder

  **3.** Copy the file*SMAPEA-QN-emergency-handling.jsimg* into *experiment* folder

  **4.** Crete a new Run configuration with:

    *a)* Main Class: *it.univaq.disim.seagroup.smapeaqn.moo.runners.CspNSGAIIRunner*

    *b)* Program arguments: *./experiments/SMAPEA-QN-emergency-handling.jsimg 2 3 0.9 0.0625 pop numevals*, i.e.:
       2 system modes, 3 controllers, 0.9 (90%) and 0.0625 (6.25%) as crossover and mutation probabilities, pop and numevals (denoting population size and number of evaluations, respectively).

  **5.** Set the desired workload intensity by choosing one of the options below:

    *a)* Open *SMAPEA-QN-emergency-handling.jsimg* file with JSimGraph, go to "Define customer classes" and then edit the Interarrival Time Distribution for *Sense* class by providing a mean value, e.g. 0.5.

    *b)* Open *SMAPEA-QN-emergency-handling.jsimg* file through a text/XML editor and provide a workload intensity inside the `<value>` tag of subparameter "t" within "distrParam" (please find those tags at lines 30-32 circa), e.g. `<value>`0.5`</value>` .


## Instructions for building and running Auto-Weka experiments

Auto-Weka allows to create *experiments*, which specify particular parameter settings for the execution of a particular learning process.
First, an experiment must be *built* (created), then it can be *run* (executed).
In the following, general instructions are provided for easing step **(3) Machine-Learning Classifiers**.

### <a name="auto-weka-build">Building experiments</a>
Auto-Weka experiments can be created by means of **Experiment Builder** wizard, which consists of 3 steps [Click *Save* at the end]:

   **1. Dataset selection**: Provide the training and test set in the form of *.arff* files and select *Cross-Validation* as Instance generator; Click *Next*.

   **2. Classifier selection**: Select all classifiers belonging to *functions*, *rules* and *trees* families; Click *Next*.

   [Note: Other classifiers have been excluded from the experimentation due to fatal errors raised by Auto-Weka when they were selected.]

   **3. Experiment settings**:
       - Name the experiment.
       - Select the output folder (e.g. a folder named *experiment*);
       - Select *Root Mean Squared Error (Regression)* as result metric.
       - Select *SMAC* as optimisation method, then *Edit*:
          -  Select the file named *smac* inside SMAC v2.10.03 folder as SMAC Executable.
          [**Note**: Such file is inside *autoweka-0.5* folder if you downloaded the provided bundle.]
          - Initial Incumbent: Random.
          - Execution Mode: SMAC.
          - InitialN: 1.
       - Set Training Memory Limit to a value which is suitable for the machine onto which Auto-Weka is run.
       [**Note**: The default value of 1000 MB has been used in the experimentation.]
       - Check Use Attribute Selection option.
       - Set Optimisation Timeout, Training Run Timeout and Attribute Selection Timeout to the values chosen during step **(3.a)**.

       [**Note**: In the experimentation, they have been set to 2 hours, 30 minutes and 5 minutes, respectively.]

### <a name="auto-weka-run">Running experiments</a>
Auto-Weka experiments can be executed by means of **Experiment Runner** wizard [Click *Run* at the end]:
   - **Experiment folder**: Click *Open*, browse to the parent directory of an experiment folder and then select the latter.
[**Note**: You do not have to enter inside the experiment folder, but you just have to select it!]
   - **Seed**: Provide a seed.
 [**Note**: The default value of 0 has been used in the experimentation.]

[**IMPORTANT NOTE**: It might happen that Auto-Weka runs to infinite. For this reason, if an execution is taking much longer than the Optimisation timeout, it is recommended to stop a run it again.]
