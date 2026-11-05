## Goal
I want to use Obsidian for all note-taking going forward in my work. 

I really love it's flexibility and clean interface. Additionally, because it is all markdown files, I can easily feed it to an AI model to use as context for work.

## Organization
Let's organize thusly:

- `career/`
	- Keep track of your career goals and accomplishments here.
	- Add a "hype doc", project stories, etc. 
	- Anything here should be backed up to a personal account.
	- You can even use AI to keep this updated based on content from other docs!
- `docs/`
	- A folder for any miscellaneous documents.
	- This is a good place to put learnings and ideas, back up old docs, etc.
- `meetings/`
	- Meeting notes. Can organize by month/year using filename prefixes, or folders.
	- Use templates to quickly start a new doc.
- `projects/<name>/`
	- Project notes and docs. Each project gets a subfolder.
	- Project docs may benefit from their own common template (TBD).
- `templates/`
	- For document templates
	- Eg: meeting notes, project docs, etc
- `AGENTS.md`
	- We're gonna want to use AI! Give it context here.
- `SCRATCH.md`
	- I always need a scratch pad!
- `WORK_LOG.md`
	- My typical work log doc. See [[#Work Log Format]] for details.

## Vault Backup
Obsidian does not provide cloud backup natively, nor does it allow for multi-device usage. All files remain local to your computer.

For backup, git is the obvious choice. However, given that I would be using this at my workplace, there are a few options:

- Use a git repo on my company's account
	- No issues of me saving work-related information.
	- But I will need them to provision this repo.
- Backup to a repo on my personal git account
	- Not a great idea as it contains work-related data.
	- Yet I _would_ like to save some information related to my work history, for job hunting and career progression.

Current thinking:

- Backup to git repo in company account.
- Copy select personal work history-related files to a **private** personal account. Eg:
	- Project stories to use in behavioral interviews
	- Learnings, successes, failures
	- Anything else that would be useful to me but not divulge company info

## Drawbacks
Obsidian has a few drawbacks. Nothing is perfect.

- **Backup:** 
	- Need to roll our own solution here. Git should work easily.
- **Multi-device use:** 
	- Honestly not a big deal. I've rarely needed to read or modify notes on the go.
	- Worst case I have Claude write a mobile app for reading my vault files from github!
- **Sharing:**
	- For any docs that require sharing or collaboration, I'll need to use something else (eg: Google Docs, Notion, etc.)
	- No way around this. I'll just have to link to the doc from Obsidian.
	- I could always write the first draft in Obsidian, and link to the Google Doc at the top.

## Work Log Format
I have always used a work log document at work to stay organized. Most recently I have structured it like so:

```
Backlog:
	- A list of tasks, as checkboxes
	- These can be organized by project, effort, etc
	- Trim the backlog periodically (eg: monthly)
	  
Week mm/dd - mm/dd
	- Separate sections for each day, Mon - Fri
	- For each day, write the tasks you want to complete
	- Mark tasks complete as you go
	- Tasks can be copied from the backlog, or from the previous day
	- Be sure to prioritize! Without reflection, you can find yourself
	  working on low-impact tasks.
```

## General Tips
- Keep it simple! Don't overcomplicate things.
	- If you're not sure of how to organize a folder, then don't! Just start with a file.
- Take time to organize and reflect:
	- Organize before the day / week starts
		- Groom your work log
		- Clear out the email queue
	- Reflect at the end of the day / week
		- What did you accomplish? Where could you improve?
		- Write it down! Especially for future career (behavioral) stories.
		- What could you do, or change, to improve?
- 