---
permalink: /csis_604_summer_2017/linux_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

This is not a course about the linux command line, but we will need enough knowledge of many things in order to study distributed systems. Some of you may already know enough command line, but for many of you this is new. I've selected a relatively straightforward and brief introduction for everyone. If you need more help, there are several links at the end of this tutorial for you to dig into. If you are a command line wizard, feel free to skip the tutorial and answer the questions below.

## Tutorial
<a href="https://www.learnenough.com/command-line-tutorial">https://www.learnenough.com/command-line-tutorial</a>

## Questions
All of the questions below are pulled verbatum from the link above and full attribution, copyright, and credit is given to the original authors. I only paste them here, so you can easily access them in one place. 

### Chapter 1
1. By referring to Figure 5, identify the prompt, command, options, arguments, and cursor in each line of Figure 6.

2. Most modern terminal programs have the ability to create multiple tabs (Figure 7), which are useful for organizing a set of related terminal windows. By examining the menu items for your terminal program (Figure 8), figure out how to create a new tab. Extra credit: Learn the keyboard shortcut for creating a new tab. (Learning keyboard shortcuts for your system is an excellent habit to cultivate.)

### Chapter 2
1. By copying and pasting the text from the HTML version of Figure 17, use echo to make a file called sonnet_1_complete.txt containing the full (original) text of Shakespeare’s first sonnet. Hint: You may recall getting stuck when echo was followed by an unmatched double quote (Section 1.2 and Box 4), as in echo ", but in fact this construction allows you to print out a multi-line block of text. Just remember to put a closing quote at the end, and then redirect to a file with the appropriate name. Check that the contents are correct using cat (Figure 14).

2. Type the sequence of commands needed to create an empty file called foo, rename it to bar, and copy it to baz.

3. What is the command to list only the files starting with the letter “b”? Hint: Use a wildcard.

4. Remove both bar and baz using a single call to rm. Hint: If those are the only two files in the current directory that start with the letter “b”, you can use the wildcard pattern from the previous exercise.

### Chapter 3
1. The history command prints the history of commands in a particular terminal shell (subject to some limit, which is typically large). Pipe history to less to examine your command history. What was your 17th command?

2. By piping the output of history to wc, count how many commands you’ve executed so far.

3. One use of history is to grep your commands to find useful ones you’ve used before, with each command preceded by the corresponding number in the command history. By piping the output of history to grep, determine the number for the last occurrence of curl.

4. In Box 9, we learned about !! (“bang bang”) to execute the previous command. Similarly, !n executes command number n, so that, e.g., !17 executes the 17th command in the command history. Use the result from the previous exercise to re-run the last occurrence of curl.

5. What do the O and L options in Listing 10 mean? Hint: Pipe the output of curl -h to  less and search first for the string -O and then for the string -L.

### Chapter 4
1. Starting in your home directory, execute a single command-line command to make a directory foo, change into it, create a file bar with content “baz”, print out bar’s contents, and then cd back to the directory you came from. Hint: Combine the commands as described in Box 12.

2. What happens when you run the previous command again? How many of the commands executed? Why?

3. Explain why the command rm -rf / is unbelievably dangerous, and why you should never type it into a terminal window, not even as a joke.

4. How can the previous command be made even more dangerous? Hint: Refer to Box 11. (This command is so dangerous you shouldn’t even think it, much less type it.)
