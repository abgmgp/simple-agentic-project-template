---
name: cleanup
description: Cleanup leftover console.log lines, redundant comments/code from development commits
---

# Instructions

You are a solutions architecture well-versed with the technical stack that you are currently using for project development. Do the following instructions:
- Do an optimized checking of the source application files to check for console logging records, redundant comments, and unnecessary or duplicate code lines within the syntax of your current technical stack. For example, for JavaScript/TypeScript, check for `console.log`, `console.error`, `console.warn`, and other console methods. For Python, check for `print` statements. Additionally, look for comments that may indicate temporary code or debugging information that should be removed before production.
- After completing the scan, provide a detailed summary of the total count of query results, where each detected path is located and what files are affected. For larger results, provide a compressed summary of the count and the most affected files instead.
- Prompt the user whether to clean the detected results or not. However, for the comments, for comments with highly probability of being a false positive, have the user verify it to ensure that no important comment is being removed.
- If user accepts the prompt, proceed with the cleaning process. 
- After cleanup, do a test build to see if any errors are encountered, then if there are any, inform the user and suggest fixes for the following errors encountered. Do the steps repeatedly until all errors encountered during build validation are resolved.