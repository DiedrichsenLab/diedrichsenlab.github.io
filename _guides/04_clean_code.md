---
layout: page
title: Writing clean code
---

## Introduction
Some people like to code. Others do not. Often we write code as a thinking tool, so we can understand a specific problem better. But most of the time we write code to solve problems that we (and others) have already solved before. This can be a big time sink. As a lab, we want to minimize our total time spend on this, so we can focus on the interesting problems.

Clean code is easy to read and to understand. It easy to modify and extend. Clean code is easy to test and to debug. Clean code should feel natural and intuitive when you use it.  Most often, unfortunately, it is not. Too often, it is faster to write your own code to solve a problem than to understand somebody else's code, making sure it works as you think it works, and then use it. That's a sign that the original code was not good.

Writing clean code is therefore an essential skill. Like writing clear papers, there are a lot of skills, techniques, and habits that can all be learned. Mastering them it will save you (down the line) and others (immediately) a lot of time and frustration. It allows us as a lab to do more complicated things faster. Writing clean code requires clear thinking about the structure of the problem, about data organization, and about the scientific question itself. Writing clean code is not just a service to others, it will make your own science better.

## Spotting bad code

You are likely the least qualified person to judge the quality of your own code. It works, right? It solves the problem that you wanted to solve? You understand what it does and  how to call? So it must be good - no?

![WTF/minute](/assets/coding_wtfm.jpg)

The ultimate judge of your code quality is a new student to the lab who wants to build upon your project. It's the person that is trying to replicate your paper. It's the person that wants to use your analysis technique. So even if you already write pretty clean code - it can always be better. And that takes work and attention to detail.

Here are some signs of bad code:

* You find yourself writing the same kind of code over and over again
* You find yourself copy-pasting code from one project to another
* You have to look up the name, input or output arguments of a function you wrote
* You discover that you have written two functions that do very similar things
* You know that you have solved this problem before, but you cannot find the code or remember how you did it
* Sharing your code online upon publication fills you with dread (or you secretly hope nobody will ever look at it)
* Other people in the lab who do very related things do not use your code

One principle to improve your code is code review. Companies like Google and Facebook have a very strict code review process. Every line of code that is written is reviewed by at least one other person. In the lab, we do not have the resources to do this - at least not for all the code that is written. But in general, show you code to others. Ask them to use it. Improve your code based on their feedback. And if you see somebody write bad code, tell them and help them to do it better.

![Code review](https://imgs.xkcd.com/comics/code_quality_2x.png)

## Different types of software projects
Not all code requires the same level of scrutiny and care. I think we can distinguish the following levels of code projects:

| Level | Description | shared with | Version control | Documentation |
| --- | --- | --- | --- | --- |
| 1. Private  | Your dirty underwear | Nobody | - | - |
| 2. Project  | Code, scripts for a single project | A few people - public upon publication | Github | Readme |
| 3. Internal | Code that benefits multiple projects | Mostly lab and close collaborators | Branch control | Readme + Sphinx docs |
| 4. External | Software packages we support for the community | SUIT, RSA, PcmPy, nitools, etc | Code review, releases, PiPy packaging | Readthedocs |
| --- | --- | --- | --- | --- |

Many projects and repositories contain code at different levels. Keep them separate. A good practice is to place general tools (classes and functions) in modules at the root of the directory, so they can be easily imported. Place project-specific code in a `scripts` or `notebooks` directory. If you find that you want to use the same code in multiple scripts, it belongs in a module. Place your dirty underwear in a appropriately named directory - `scrapheap` or  `scratch`  comes to mind.

## Meaningful Names
Name things in a way that makes clear what they are. There are tons of rules on naming out there, but remember that the main point is to make the code easily readable. The better your names are, the less comments you need to write.

### Variables

Variable names should tell the reader what the variable is. `median_rt` is a better name than `mrt`. `condition` is better than `c`.

On the other hand, variable names that are too long do not make the code more readable. `number_of_trials_per_block` maybe very clear, it makes the lines of code very long. When using shorter forms like `n_trials`, make sure that you use these forms **consistently**. Do not mix `n_trials`, `num_conditions`, `number_of_blocks`, and `N_runs` in the same project.

Some guides say to avoid single-letter variable names. I would disagree: Local variables in loops can definitely be single letter - for example `i` and `j` for indices. `for i,sn in enumerate(subj_name):` is perfectly clear and makes the code within the loop more readable.
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
Classes describe *Objects*, who have both features (attributes) and behaviors (function). Class names should be nouns - and in the lab code we usually use CamelCase for them (where otherwise we use understroke). So `Nifti1Volume` is a good class name.

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

## Clean data structures
Bad code writing often starts with bad decision about the underlying data structures. If you have a lot of code that wrangles your data from one structure into the next, you probably made some bad decisions about how to store and organize your data.



## Functions and factorization
Functions should do one thing

They should do this thing well

They should be the only thing in your code that does this thing


## Classes

## Overall Software Structure
## Computational Efficiency
## Backwards Compatibility


## Read more

* Clean Code: A Handbook of Agile Software Craftsmanship by Robert C. Martin
