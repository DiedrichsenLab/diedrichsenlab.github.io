---
layout: page
title: Writing clean code
---

## Introduction
Some people like to code - others do not. Often we write code as a thinking tool, so we can understand a specific problem better. That is well-spent time. But most often, we write code to solve uninteresting analysis problems that we (and others) have already solved before, which is a time sink. As a lab, we want to minimize our total time spent on writing boring code, so we can focus on the interesting problems.

Clean code is easy to read and to understand. It easy to modify and extend. Clean code is easy to test and to debug. Clean code should feel natural and intuitive when you use it.  Most often, however, it is not. Too often, it is faster to write your own code to solve a problem than to understand somebody else's code, making sure it works as you think it works, and then use it. That's a sign that the original code was not good.

Writing clean code is therefore an essential skill. Like writing clear papers, there are a lot of skills, techniques, and habits that can all be learned. Mastering them it will save you (down the line) and others (immediately) a lot of time and frustration. It allows our lab to do more complicated things faster. Writing clean code requires clear thinking about the structure of the problem, about data organization, and about the scientific question itself. Writing clean code is not just a service to others, it will make your own science better.

## Spotting bad code

Don't worry - of course *your* code is good. It works, right? You understand what it does and how to call it? It does exactly what you want it to do in one line? So it must be good - no? Unfortunately, you are likely the least qualified person to judge the quality of *your* code.

![WTF/minute](/assets/coding_wtfm.jpg)

A better judge of your code is a new student to the lab who wants to build upon your project. It's the person that is trying to replicate your paper. It's the person that wants to use your analysis technique. So even if you already write pretty clean code - it can always be better. And that takes work and attention to detail.

Here are some signs of code that requires improvement:

* You find yourself writing the same kind of code over and over again
* You find yourself copy-pasting code from one project to another
* You have to look up the name, input or output arguments of a function you wrote
* You discover that you have written two functions that do very similar things
* You know that you have solved this problem before, but you cannot find the code or remember how you did it
* Sharing your code online upon publication fills you with dread (or you secretly hope nobody will ever look at it)
* Other people in the lab who do very related things write their own code, rather than using yours

One principle to improve your code is code review. Companies like Google and Facebook have a very strict code review process. Every line of code that is written is reviewed by at least one other person. In the lab, we do not have the resources to do this - at least not for all the code that is written. But in general, show you code to others. Ask them to use it. Improve your code based on their feedback. And if you see somebody write bad code, tell them and help them to do it better.

![Code review](https://imgs.xkcd.com/comics/code_quality_2x.png)

## Different types of software projects
Not all code requires the same level of scrutiny and care. I think we can distinguish the following levels of code projects:

| Level | Description | shared with | Version control | Documentation |
| --- | --- | --- | --- | --- |
| 1. Private  | Your dirty underwear | Nobody | - | - |
| 2. Project  | Notebooks, workflows, and functions for a single project | A few people - public upon publication | Github | Readme |
| 3. Internal | Functions and classes that benefits multiple projects | Mostly lab and close collaborators | Branch control | Readme + Sphinx docs |
| 4. External | Software packages we support for the community | SUIT, RSA, PcmPy, nitools, etc | Code review, releases, PiPy packaging | Readthedocs |
| --- | --- | --- | --- | --- |

Many projects and repositories contain code at different levels. Keep them separate. A good practice is to place general tools (classes and functions) in modules at the root of the directory, so they can be easily imported. Place project-specific code in a `scripts` or `notebooks` directory. If you find that you want to use the same code in multiple scripts, it belongs in a module. Place your dirty underwear in a appropriately named directory - `scrapheap` or  `scratch`  comes to mind.

## Meaningful Names
Name things in a way that makes clear what they are. There are many rules on naming out there, but remember that the main point is to make the code easily readable. The better your names are, the less comments you need to write.

### Variables

Variable names should tell the reader what the variable is. `median_rt` is a better name than `mrt`. `condition` is better than `c`.

On the other hand, long variable names do not make the code automatically more readable. `number_of_trials_per_block` maybe very clear, but it makes the lines of code very long. When using shorter forms like `n_trials`, make sure that you use these forms **consistently**. Do not mix `n_trials`, `num_conditions`, `number_of_blocks`, and `N_runs` in the same project.

Some guides say to avoid single-letter variable names. I disagree: Local variables in loops can definitely be single letter - for example `i` and `j` for indices. `for i,pid in enumerate(participant_id):` is perfectly clear and makes the code within the loop more readable.
Variables with defined mathematical notations can also be single letter.  `design_matrix` maybe more explicit than `X`. But I find

```python
inv(X @ inv(V) @ X.T) @ X.T @ inv(V) @ Y
```
much more readable than
```
numpy.linalg.inv(design_matrix @ numpy.linalg.inv(covariance_estimate) @
design_matrix.T) @ design_matrix.T @
numpy.linalg.inv(covariance_estimate) @ activity_data
```

By convention, we use capital letters for matrices / tensors and lower case letters for vectors / scalars.
When you use short forms, just make sure the meaning of the variable is defined in the comment section of the code.

### Functions
Where variable names should begin with a noun, function names should start with a verb. `get_data` retrieves some data. `set_param` sets a parameters. `compute_t_stats` computes a t_statistics. `func1`, `my_secret_weapon`, or `tmp` are not good names.

### Classes
Classes describe *Objects*, who have both features (attributes) and behaviors (function). Class names should be nouns, start with a capital letter, in the lab code we use CamelCase for them (where otherwise we use understroke). So `Nifti1Volume` is a good class name.

Often you have a whole collection of classes, all inheriting from a superclass. It is good practice to start the class name with the superclass. `AtlasSurface` and `AtlasVolume` is good, as this naming works well with tab-completion: By typing `Atlas<tab>` you can see immediately all available Atlas classes. While `SurfaceAtlas` and `VolumeAtlas` roll better of the tongue, tab completion does not work well.

### Projects, Repositories and Packages
Projects are hard to name. Make sure that the project name you choose is unique and will make clear to sombody in the lab what it is. `PhD_Experiment_1` obviously is not very descriptive.

We use a abbreviated form of the project name as an identifier for functions, datafiles, and directories. So the `tendigit` mapping project contains experiments `td1`-`td5`.

For repositories for general use (level 3 and 4), you may want to check if the name is already taken on PiPy, the Python Package manager. Ideally your repository and the package have the same name. Sometimes this is not possible, so the repository `nitools` has to be installed as `pip neuroimagingtools`, because `nitools` was already taken.

### Do not be afraid of renaming
Naming everything clearly and consistently is hard. Make it a habit to inspect your naming convention from time to time. Is it consistent? Are they readable? Do they make sense? If not, rename them.

Editors like VSCode make it easy to rename variables and functions across multiple files - but check the section of backwards compatibility.

## Comments
If you have to write very long comments to explain how your code works, it maybe because your code is not very clean. Ideally, code should be self-explanatory.

In the beginning of each function should be a doc-string that explains what the function does, and defines the input and output arguments. If you follow the following convention (google docstrings), you can build documentation for Python functions automatically using `sphinx` and `readthedocs` (see Documentation).

```python
def my_function(data,
        anatomical_struct='cerebellum',
        labels=None):
    """ Say what the function does followed by a blank line.

    Args:
        data (np.array):
             n_vert x n_col data
        anatomical_struct (string):
            Anatomical Structure for the Meta-data. default: 'Cerebellum'
        labels (list):
            Numerical values in data indicating the labels. default: np.unique(data)
    Returns:
        gifti (GiftiImage):
            Label gifti image
        info (dict):
            Dictionary with meta-data
    """
```

Comments within the code should be used sparingly. Structure different steps in the code with blank lines.

## Data Structures
So far we have talked about the *Micro-structure* of code - how to name things, how to comment. The most important difference between good and bad code often lies in its *Marco-structure*. Marco-structure of data analysis project consists of Data Structures, and the processes between them.

Bad code writing often starts with bad decision about the underlying data structures. If you have a lot of code that wrangles your data from one structure into the next, you probably made some bad decisions about how to store and organize your data.

An important technique when planning your analysis code is to draw a diagram of how your code goes from the raw data to final figures and statistical results. The figure shows an simple example from a behavioral experiment. Data structures (and the associated files) are shown in gray, workflows (see below) are shown in red.


![Data flow](/assets/coding_datastructs.png)

 In this simple example, the raw data consist of three type of data structures:

* a `participants.tsv` file with one row per participant with `participant_id`, `sex`, and `age` information + any other information that is relevant for the analysis (other questionnaires, group assignments, etc).
* a `<exp>_<subj_id>.dat` file with one row per trial, saved separately for each subject.
* a `<exp>_<subj_id>_run<num>.mov` file that holds a raw kinematic data for each trial.


The first step is a workflow that loads all the raw data and produces an overall analysis file with a row per trial (and all subject combine). This file then is used to produce summary tables with on entry per subject and condition - one for the RT results, one for the inter-press-intervals. These two are stored in separate files, as they required a different number of rows. Finally, each figure panel and statistical analysis is generated. This data flow structure is very clean and transparent. The advantages are:

* We have the absolutely smallest amount of data files - no data structure is duplicated
* Workflows only use the the data structures that they absolutely need
* There is only one replicable way to go from data structure to data structure
* The data can be easily be shared at 3 levels
    - the tabular data underlying each figure and stats reported
    - the preprocessed trial-based data
    - the raw data
* Each level is sufficient to reproduce the final results that depend on them
* It is clear how each data structure was generated - when an error is detected it is clear which part of the analysis is effected

It is likely that when you draw the diagram for your project, it will be much more complicated. **Your goal is to reduce the complexity of this tree as much as possible**. Common problems and solutions are:

| Problem | Solution |
| --------| ---------|
| Dead or depreciated data structures clutter the analysis code and data directories | clean up |
| Alternative data structures for overlapping purposes | Generate one common data structure |
| You data structures are nested to combine scalar data and large binary files | Keep separate data structures separate |
| Error trials are removed early in the analysis, so you need to go back to the raw data to analyze them | Include all trials in the second stage, and label them with a 'error' column |
| Your processes overwrite data structures | Avoid loops in the graph and make sure to include necessary later steps in the initial process that generates the data structure |
| Workflows depends on a lot of different data structures | Data structures should include *all* necessary data for subsequent analyses |
| Workflows need to load large data structures to extract a small amount of information | Data structures should include *only* the data are necessary for subsequent analysis |

Important data structures are stored on disk as files. In general, you want to choose file formats that are established in the lab and the community.
* Data frames stored in .tsv files. Generally, it is preferable to store the data frame in a long format (one column per dependent variable, and condition coded in another), not a wide format (a separate column for each condition).
* Nifti, Gifti, Cifti files for Neuroimaging data
* BIDS-compatibility where practical
* For custom binary file formats, .pickle and .mat files do not travel well between analysis platforms. Use HDF5 instead if necessary.

## Workflows, Functions and Factorization




To summarize:

* Functions should do one thing
* They should do this thing well
* They should be the only thing in your code that does this thing
* Refactorize

## Classes




## Computational Efficiency



## Backwards Compatibility


## Read more

* Clean Code: A Handbook of Agile Software Craftsmanship by Robert C. Martin
