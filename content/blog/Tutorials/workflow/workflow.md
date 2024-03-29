---
title: "Workflow Structure"
author: "Bijean Ghafouri"
lastmod: 2020-10-05
output: 
  html_document:
  theme: cosmo
  highlight: tango
---



*Programs must be written for people to read, and only incidentally for machines to execute.* 

*― Harold Abelson, Structure and Interpretation of Computer Programs*

How you [organize](https://hrdag.org/2016/06/14/the-task-is-a-quantum-of-workflow/) your data analysis project is crucial for many reasons. Without a pre-defined structure to your projects, you are likely to forget years or even months from now what you have done looking back at projects. This may be the case when you get back on a dormant project, or receive major revisions in an R&R six months after submission. Projects that involve collaboration between multiple members should also have a unified structure understand of how their analysis is structured to avoid any mistake. 


We call this your [workflow](https://hrdag.org/tech-notes/harmful.html). Although many will have seen this in the context of their coursework, social science students are rarely taught this. Your project should be divided between **tasks**. Tasks are individual parts of a larger project that are each self-contained and self-documented. The goal of dividing your project in tasks is to make each step of your project as clear as possible. Also, this allows you to facilitate testing and error handling on a higher level. 
\bigskip


### Tasks structure
The first task is to **import** your data. This tasks has at minimum three directories: `input`, `output` and `src` (source). 

* `input` contains raw files for task to read
* `output` contains files ready to go
*  `src`  contains scripts to read input and write to output

Let's see what a basic folder structure looks like. These folder names may change based on field or personal preference, but here they are mostly standard. 

```
  
│
└─── docs
└─── input
└─── output
└─── src
```

The `input` never get modified. You will only drop raw data files into it. This is the whole point of having scripts in `src`, to read and modify the input. Once transformed, we can place the new files in `output`. The reason why you should never modify input files in this directory is because you should always keep your raw files. This is key in reproducibility since we need to trace back your analysis from the very start of your project, i.e. your raw files. Furthermore, the objective of `src` is not to clean and manipulate data on a deep level, but rather to create a structured, rectangular datasets ready for further analyses, dumped into `output`. In subsequent tasks, you should take in the files from `output` as inputs to clean and analyze specifically for the goal of this part. You can imagine this as a chain-rule for a structured flow of your data. 


### Structuring your task in `R`
To create these folder, you can run the following commands. This should be at the top of any script. 

```r
# ------------ packages  ----------
require(pacman)
pacman::p_load(here)

# ---------- workflow -------------
# set directory
path <- here::here()
setwd(path)

# set up file paths 
dir.create(paste0(path,"/","input",sep=""),showWarnings = T)
dir.create(paste0(path,"/","output",sep=""),showWarnings = T)
dir.create(paste0(path,"/","src",sep=""),showWarnings = T)
pathin = paste0(path,"/","input",sep="")
pathout = paste0(path,"/","ouput",sep="")
pathsrc = paste0(path,"/","scr",sep="")
```

This will create three folders for each component of this tasks. You may notice that I set the directory with `{here}`. This package is extremely useful for others that may want to reproduce your workflow on their local machines. Given that [everyone](https://i.imgur.com/jKWxztR.png) has different directory structures, this [package](https://github.com/jennybc/here_here#readme) allows you to limit the exposure of your local setup. 


The `input` folder will contain **all** data files you need for this project (all subsequent tasks). Then, you will read them in your script, contained in the `src` folder, to then export to the `output` folder (often in a .csv format). Following tasks will read in data files from `output` that they each need. 


I have the following structure for a single analysis task that comes directly from the [HRDA Group](https://hrdag.org/). You can have many analysis tasks, depending on the nature of your project. 

```
├── import
│   ├── Makefile
│   ├── input
│   ├── output
│   └── src
├── clean
│   ├── Makefile
│   ├── input
│   ├── output
│   └── src
├── model
│   ├── Makefile
│   ├── input
│   ├── output
│   └── src
└── write
    ├── Makefile
    ├── input
    ├── output
    └── src
```

The goal of the following structure is create a linear flow to how your data is transformed. For example, your data from into the `import` directory raw. The `clean` directory will take in the data from the output folder of `import` and process it. The `model` directory will then itself take the output data from `clean` and process it. In this step, you will likely create a `tables` and `figs` folder in the output folder. Finally, `write` will follow the same flow. For specific file names, I recommend following these [best practices](https://speakerdeck.com/jennybc/how-to-name-files)


Remember, this structure is for one task only. Depending on the nature of your project, or personal preference, you may need only one task or more. 

### Self-Documentation
Many will dislike the use of documentation in the classic format of a .txt file that you update regularly as you move forward with your project. You probably experienced this where, at the end of the day, your code almost never keeps up with your documentation. This is why it is strongly recommended to **self-document** your code within your scripts. This will allow you to document your code within your script as-you-go. Obviously, you will continue to keep track of what you do in your documentation. The difference is that your documentation will be higher-level instructions, while your self-documentation will follow your script closely. In the case where your documentation is out-of-date, you will be able to rely on your script to reconstruct the steps of your project. 


### Optional directories
You may wish to incorporate personal workflow items into this system.
First, it is very well possible that you use notebooks or markdown files to test your code as you write. Although this might be a very good solution for testing, it is recommended that you always have a final script that contains all your chunks in the `src` folder. If you use notebooks, put these .rmd or .ipynb files in a `note` directory. 

You may have noticed that the import task structure contains a `docs` directory. This is for any documentation that comes with data you are using. This will often be the case with survey data, or data from other public sources.  


### Makefile and `{drake}` 
You may notice that each task contains a makefile. This file is to automate the execution of all the scripts from the `src` folder. Although in other programming languages you might need to manually create a makefile, in `R` you can use the `{drake}` [package](https://milesmcbain.xyz/posts/the-drake-post/) (more info [here](https://www.youtube.com/watch?v=4vu8h_Zh8Wg)). 

There a numerous tools to create and automate workflow styles. Many use targets, like `{drake}`. Here are a few: 

* [ProjectTemplate](http://projecttemplate.net/)
* [workflowr](https://jdblischak.github.io/workflowr/)
* [rrtools](https://github.com/benmarwick/rrtools)
* [ordely](https://github.com/vimc/orderly)
* [fnmate](https://github.com/MilesMcBain/fnmate)
* [dflow](https://github.com/milesmcbain/dflow)
* [drake](https://docs.ropensci.org/drake/)
* [represtool](https://pirategrunt.com/represtools/)
* [starters](https://itsalocke.com/starters/)


The `{drake}` package makes your code focus on functions, rather than simple scripts. Although it has the same intention as makefiles, the package's logic is embedded within a dataframe, following `R`'s ecosystem. 




### Notebooks
Notebooks combines the code, text and visualization in one document. This may help with reproducibility because each chunk is immediately followed by its output. So what are the downsides? The main issue with notebooks is the lack of structure in running chunks. Although they are sequentially ordered, you are most probably running them in all sorts of ways to fix bugs or add features. This lack of consistency is likely to produce more problems than not. For more on problems associated with notebooks, see this [talk](https://www.youtube.com/watch?v=7jiPeIFXb6U) by Joel Grus.

Notebooks remain useful for the courses, exploratory data analysis, reports (where more text than code) and dashboards. 

### Project as an `R` Package
As per McBain's [blog post](https://milesmcbain.xyz/posts/an-okay-idea/), although creating an `R` package is used by many, there are some pitfalls associated with it. Although it is a great way to automate your project with testing and solid structure, there remains the little payoff, and the signal to noise ratio is simply much too low. Another issue is the fact that not everyone is comfortable with the package environment, even if they are a domain expert. This defeats the purpose of reproducibility since the public's access becomes limited. 




### Alternatives to project structure
We will often see directories and project structures that follow a flow based on the content of the folders rather than the order of the project itself. For example, projects will be organized with the following folders: `data`, `analysis`, `tables-figures`. Although this does fits the purpose of reproducibility, it does not flow with the logic of the project. You should have different folders for different **tasks**. In the example above, many tasks can be included in the `analysis` folder, like cleaning and modeling. Some may point out that this is just a discrepancy in the placement of folders in different directories, which is essentially the case. However, the problem lies within how someone who has never worked on this project will be able to understand its logic. 

Finally, you can find great posts [here](https://chrisvoncsefalvay.com/2018/08/09/structuring-r-projects/),  [here](https://the-turing-way.netlify.app/welcome) and [here](https://edwinth.github.io/ADSwR/index.html) to help you structure your projects.

At the end of the day, the goal for a clear project structure is to convey what exactly you are doing to everyone else that may have access to it. 



## Patrick Ball's Principles of Data Processing
This is from his great [talk](https://www.youtube.com/watch?v=ZSunU9GQdcI). Basically, he gives his take on 'where to put your stuff'.

His main argument is that the way documentation is understood makes us take it for granted, when it is actually central to our code. How do we write code and organize projects to know what you did a few months from now? It is also meant to organize your work to facilitate collaboration with colleagues where they will be able to understand *exactly* what you are doing. 

He identifies four goals for statistical workflow: 

* **Transparency**: possible to review every task

* Auditbility: possible to test the result of every task, where the test is often done in a different computer language and by different analyst. 

* Reproducibility: any scientist with the same tools and the same code could recalculate the results and get the same answers. 

* Scalabilty: 'more than 2' input files, updates to your input files, analysts who are going to work on the project, results in the report you will publish, different analyses which derive from the inputs, languages needed in the analysis, etc. 


### Design Tactics
* Think like a pipeline for your flow
* Project flow with makefiles
* Everything in code: all the changes to the data are written in an executable file. If it's not in code, how are you going to remember it? You can't depend on yourself to remember something that needs to be done. Not in documentation, because code evolves much faster than documentation. You will thus not know if the documentation follows exactly what is happening.
* Simpler is better, but don't take this to the extreme. 
* Explicit is better than implicit, but don't take this to the extreme. 
* Exceptions are bad but sometimes necessary, but don't take this to the extreme. 


* Use Unix, bash, ssh. 
* Get conformable with the terminal. Your project will be available everywhere.
* Separate data from logic. 

### The Problems with documentation
Documentation for code is never complete, on point, up to date. You will never know if the documentation is reliable because the probability of a mismatch is too big given how we behave when coding. We should not trust ourselves, because too many mistakes happen. 

The goal of documentation, making clear what you are going, is great and incredibly important. However, it is the way we satisfy this goal that leads to deception.

For things that do not change much, like input data, exported data and common libraries, classic documentation is appropriate. However, it is for code that most likely changes extremely often that this will not do. In fact, a good project is self-documenting. 

On version control, you should keep track of the *process* of doing the project: completeness, todos, bugs, receiving data, shipping results. Do not document the content of a project in version control, because you can understand the content simply by reading the content. It is useless to repeat yourself twice. 


### Code Flow Structure

* **Import**: Read data from `input` with `src` files and write to `output`
  * This largely consists in reading all types of different data and writing to a rectangular .csv file.

* **Clean**: The data from `output` in the **import** task goes into to `input` directory of the **clean** task. 
  * Do you notice the linear flow here? 

... and so on.


### Self-Documentation, symbolic links and metadata. 
The way you get your tasks done is with structure, not documentation. You need to tie your tasks together in a coherent and consistent manner in order to facilitate the integration of others and you're eventual reintegration to your project.

Symlinks show the relationship among tasks.
Task names describe what happens in a task.
Nothing should be done by hand -- put down the mouse. 

### Symlinks


### Makefiles
This is allow you to use the command line without specific execution code. All you need to call is the makefile, and for all your projects. 
