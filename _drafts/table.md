---
layout: post
title: "Matlab table & string"
author: Michał Szczepanik
tags: ["software", "tips & tricks", "matlab"]
---

In this post I am taking a closer look at tables and strings in matlab.

Matlab continuously evolves, and the two data types are fairly recent additions to the language: [table](https://www.mathworks.com/help/matlab/ref/table.html) was added in R2013b and [string](https://www.mathworks.com/help/matlab/ref/string.html) in R2016b. Table arrays store tabular data and can make its handling (sorting, filtering, joining etc.) much more convenient than "classical" cell arrays. In a way, they are similar to dataframes known from R or python / pandas, although more basic. String arrays are a proper string type, and in my opinion they are much more workable then their predecessors, character arrays and cellstrings.

To demonstrate how useful they can be together, I will use an example modelled after preparing an input for SPM batch (second level two sample t-test design). If you work in neuroimaging, you'll probably be familiar with SPM, but if not, you will be able to follow along just as well.


# Problem statement
What we have is a tab-separated file `participants.tsv` with a list of subjects (in accordance with the [BIDS specification](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#participants-file)):

```
participant_id age group
sub-01         34  read
sub-02         12  write
sub-03         33  read
```

We also have a set of intermediate files (first level output), which we organised in the following way (this time according to our whim, rather than any standard):

```
/work/sub-01/task-motor/con_0001.nii
/work/sub-01/task-nback/con_0001.nii
/work/sub-02/task-motor/con_0001.nii
/work/sub-02/task-nback/con_0001.nii
```

What we want to get is two list of files for the motor task: one for the read group, and one for the write group:

```
group1 = {
	/work/sub-01/task-motor/con_0001.nii,
	/work/sub-03/task-motor/con_0001.nii,
	...
	}
group2 = {
	/work/sub-02/task-motor/con_0001.nii,
	...
	}
```

# Solution

Let's start by reading the participant file:

```matlab
participants = readtable('/path/to/participants.tsv', ...
	'FileType', 'text', 'Delimiter', '\t', 'TextType', 'string')
```

Unfortunately, `readtable` knows `.txt`, `.dat` or `.csv` extensions, but not `.tsv`, so we have to specify both `FileType` and `Delimiter` (either `\t` or `tab` will work) in addition to the file name. Additionally, we use `TextType` to import text data as string arrays, rather than character vectors. This produces the following:

```
participants =

  3×3 table

    participant_id    age     group 
    ______________    ___    _______

       "sub-01"       34     "read" 
       "sub-02"       12     "write"
       "sub-03"       33     "read" 

```

Next, let's add another column with paths to the con files:

```matlab
participants.con_files = fullfile(...
	"/work", participants.participant_id, "task-motor", "con_0001.nii")
```

The `fullfile` function joins its arguments by adding a path separator (`/` or `\`, depending on system). One of the inputs we gave, `particiants.participant_id` is a text array (with as many rows as there are subjects), and consequently the output will be a text array of the same shape. Adding a column is as simple as using a dot notation with a new name. Convenient. This is what the table looks as a result:

```
participants =

  3×4 table

    participant_id    age     group                   con_files               
    ______________    ___    _______    ______________________________________

       "sub-01"       34     "read"     "/work/sub-01/task-motor/con_0001.nii"
       "sub-02"       12     "write"    "/work/sub-02/task-motor/con_0001.nii"
       "sub-03"       33     "read"     "/work/sub-03/task-motor/con_0001.nii"
```

We can next filter the table for each group in turn.

```matlab
read_con = participants{participants.group == "read", 'con_files'}
write_con = participants{participants.group == "write", 'con_files'}
```

Here, we used brace indexing to get a subset of rows (since the group column is a string array, we could use `==` to obtain a logical array of a matching shape) and a single column (by giving its name). In a typical matlab fashion, using round brackets would keep the type of the indexed object and return a table with selected rows and columns, while the curly brackets above return column content[^1] (in this case, a string array). This produces:

```
read_con = 

  2×1 string array

    "/work/sub-01/task-motor/con_0001.nii"
    "/work/sub-03/task-motor/con_0001.nii"


write_con = 

    "/work/sub-02/task-motor/con_0001.nii"
```

Finally, since SPM batch requires cellstrings rather than string arrays, we can do a simple conversion:

```matlab
group1 = cellstr(read_con)
group2 = cellstr(write_con)
```

```
group1 =

  2×1 cell array

    {'/work/sub-01/task-motor/con_0001.nii'}
    {'/work/sub-03/task-motor/con_0001.nii'}


group2 =

  1×1 cell array

    {'/work/sub-02/task-motor/con_0001.nii'}
```

# Further comments

Naturally, the example above was fairly basic, and there is more that can be done with strings and tables. Here are some things that I found particularly useful. 

You can join string arrays using `+` (eg. `"sub-" + participants.participant_id` if the "sub-" prefix was missing). If you are a fan of `sprintf`, you can use [compose](https://www.mathworks.com/help/matlab/ref/compose.html) as its "vectorised" alternative (eg. `compose("sub-%s", participants.participant_id)`). Should you wish to replace part of the string, you can use things like `replace(participants.participant_id, "sub-", "subject-"`.

For tables, in addition to the regular `size` function there are intuitive `height` and `width` functions. If you have another table (perhaps with behavioural scores) with the same participant IDs but different ordering, you can join them using [join](https://www.mathworks.com/help/matlab/ref/table.join.html). And finally, if you need to sort a table first by one column, and then (breaking ties) by another, you can do so using [sortrows](https://www.mathworks.com/help/matlab/ref/double.sortrows.html), as in `sortrows(participants, {'group', 'age'})` (which also allows you to specify ascending or descending order independently).

[^1]: Strictly speaking, curly brackets return an array concatenated from the content of selected rows and columns.  a single-column example I prefer thinking of it simply as column content. You can read more about table indexing in the docs: [Access Data in Tables](https://www.mathworks.com/help/matlab/matlab_prog/access-data-in-a-table.html)
