# Epic: Book Workspace Per User
### Objective:
As a Holochain.Docs user, for every book I can have a private workspace representing a collection of book pages representing all my contributed content and revisions. Each page is represented by a markdown file and each book is a book zome that is referenced to either a "private book" (localchain or private dht) or a "published book" (public book dht that is shared with at least one other user). The workspace should only store the minimum amount of data required to represent a revision to another prior version of a given page. 

### Related Stories:
* Story 1...
* etc....

### Accepted If:
1. < *Criteria or test case* >
2. etc...

### Release Only When:
1. < *R2PROD requirement* >
2. etc...

### References/Attachments:
* [description-absoluteurl](http://developer.holochain.org)
* [description-relativeurl](/thisimage.png)
* etc.... 

### story.toml
``` 
# this TOML schema is used to index the epic/story into the requirements management system
schema = "story.toml"
title = "[story|epic]:story or epic name" # should match title header in story/epic markdown file
itemid = "[s|e].yyyymmdd.###@usr" # see note.1
status = "status" # see note.2
added = yyyy-mm-dd
revised = yyyy-mm-dd
iteration = "YY.????" # see note.3
value = { e.points = #, e.rank = # } # see note.4
estimate = #.# # see note.5
[requestor]
reqname = "Requestor or Author Name"
reqemail = "Requestor or Author email"
[owners]
	[owner.1]
	ownername = "Owner1Name"
	owneremail = "OwnerEmail"
	[owner.2]
	ownername = "Owner1Name"
	owneremail = "OwnerEmail"
[notes]
	note.1 = """ \
		itemid: s=story, e=epic, use date when submitted or drafted \
		, ### is for sequence (start with 001), @usr = author user id (i.e. github) \
		"""
	note.2 = """ 
		status: where in the project lifecycle is this item \
			planning [draft, submitted, rejected, inreview, accepted, roadmap, parking ] \
			development [in-queue, in-dev, r2qa, in-qa, failed, passed, r2stage ] \
			release [ staged, r2master, in-ci, ci-failed, released, done ] \ 
			"""
	note.3 = """ \
		iteration: planned work iteration using YY.MMA, where YY = year and MM = month \
		and A = period in month development \
		YY.00A means unassigned iteration for year YY \
		"""
	note.4 = """ 
		value: collection of planning/ranking metrics used to manage and prioritize work items
		> e.points is integer between 1 and 10, 10 being most effort/difficulty \
		> e.rank is categorical rank of priority, \
			0 = undefined, \
			1 = low priority, \
			2 = routine priority, \
			3 = high priority, \
			4 = top priority, \
			5 = blocking/critical priority \
		"""
	note.5 = '"" estimate: in hours - smallest work hours are in half hour increments = 0.5 """
[relateditems]
	relitem.#### = { relitem.id="", relitem.name="", relitem.type="prereq|peer|child|postreq|related" }
[tasks]
	task.#### = { taskid = "",  taskname = "" } # task: #### represents the unique task # from repo
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMTIyMjYwNDEsLTcxMjU4OTA5Nl19
-->