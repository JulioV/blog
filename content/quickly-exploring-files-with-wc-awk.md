---
date: "2020-04-18"
tags: ["bash", "productivity"]
title: "Quickly exploring CSV files with wc and awk"
---

This week I was extracting high-intensity activity episodes from the Fitbit data of 150 people. The first thing I wanted to know after processing all participants was how many people had at least 1 episode. I am using [RAPIDS](https://rapidspitt.readthedocs.io/en/latest/) to process the data, which means that the activity episodes for each participant are stored one per line in CSV files in individual folders. As I was looking for a quick and short solution, I went for Bash instead of Python or R.

Then, the problem is reduced to three steps: list all files in a subfolder with names that match a pattern, count the lines on each file, and filter those files with at least 2 lines (all files have at least the header row). For that, we can use `wc` and `awk`.

```bash
wc $(find . -name 'p*_fitbit_mvpa_episodes.csv') | awk '{if (($1 > 1) && ($4 ~ /^\.\/data/)) { print }}' | wc -l
```

The first part `wc $(find . -name 'p*_fitbit_mvpa_episodes.csv')` executes the `wc` command on the output of the `find` command, which retrieves all files in the current directory and any subdirectories with a name that matches the regular expression between quotes. The [default output](https://www.mkssoftware.com/docs/man1/wc.1.asp) of the `wc` command has four columns for each file: line count, word count, byte count, and its path. These are piped into `awk '{if (($1 > 1) && ($4 ~ /^\.\/data/)) { print }}'` which filters and prints those lines where the value of the first column `$1` (line count) is bigger than one and the fourth column `$4` (file path) starts with `./data`. The first part of the filter gets all the files with at least one activity episode (header + episode line), and the second part excludes the total count that `wc` appends. Finally, to obtain the *number* of files with at least one activity episode, I piped the previous list to `wc` with the `-l` flag that counts the number of lines (files) that `awk` printed. It turns out that out of 150 participants, only 20 have high-intensity activity episodes (this lead us to discover a problem with the data I was working with that is a matter for another post).

As an extra bit of information useful for our collaborators, I wanted to know the average number of episodes across all participants. For this I followed a similar process but instead of the second `wc -l`, I piped the output to `awk` where it is possible to keep a counter and sum of the values of each line, obtaining the average for the first column (line count) as follows:

```bash
wc $(find . -name 'p*_fitbit_mvpa_episodes.csv') | awk '{if (($1 > 1) && ($4 ~ /^\.\/data/)) { print $1}}' | awk '{ total += $1; count++ } END { print total/count }' 
```

We have an average of 17.3 episodes across 20 people. 