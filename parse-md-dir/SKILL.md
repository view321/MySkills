---
name: parse-md-dir
description: Workflow to translate a directory with .md files into code
---

# The workflow

## Get the contents of the STACK.md file located in the directory specified by user. This file will give you the information about the preffered technology stack.

## Scan the contents of the files in the folders

## Make an API Reference

### If an API Reference already exists
Update the API reference according to the .md files

### If this is a fresh start
You should make a markdown file describing each file, class, function, variable, etc. This file will be the source of truth. If API Refernce already exists, modify it so it stays up to date.

## Implement the code

### If the directory with the project exists

Spawn one subagent per one file. Give each subagent the full API reference and the markdown file corresponding to the code file the subagent will be working with. The purpose of each subagent is to compare the existing implementation against the API Reference and to fix the file to stay up to date with the API Reference and the corresponding markdown file. Example subagent prompt:

"You are a helpful coding assistant, who updates the existing codebase. Your source of truth is the API Reference along with a markdown file describing a real file with code. Compare the existing code file with the API Reference and the .md description. Then update the code file to match those two files. The API reference is located at: (location of API Reference). The markdown file is located at: (location of the .md file). The code file you are updating is located at: (location of the file with code). Do not touch any of the other code files."

### If this is a fresh start

- Create a directory where you will implement the codebase
- Scaffold the project. Do not write implementations, only do scaffolding.
- Spawn one subagent per one file. Give each subagent the full API reference and the markdown file corresponding to the code file the subagent will be implementing. The purpose of each subagent is to make the file with code correspond to the API Reference . Example subagent prompt:

"You are a helpful coding assistant, who builds a codebase from the ground up. Your source of truth is the API Reference along with a markdown file describing a real file with code. Translate the markdown file you are given into real code. The API reference is located at: (location of API Reference). The markdown file is located at: (location of the .md file). The code file you are working with is located at: (location of the file with code). Do not touch any of the other code files."

## Run the tests

If any of the tests fail, fix the issue and run the tests again to make sure the issue is gone