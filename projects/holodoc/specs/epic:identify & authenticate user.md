
# Epic: Identify & Authenticate User
### Objective:
As a holodoc user, I want to authenticate my session with a unique identity so that any action in the system can be associated to my identity. My identity can be recognized globally by any other holodoc user and all transactions can be associated to my identity and cannot be spoofed or hijacked or compromised. If there is an existing holochain identity infrastructure (dpki) in place that addresses the preceding goals, then the ability to establish and use a holochain dpki based identity. If I lose credentials required to authenticate, there is a way to reset those credentials that is secure and does not compromise the integrity of my identity.

### Related Stories:
* Story 1...
* etc....

### Accepted If:
1. Can provide credentials and authenticate session with a unique identity via user interface.
2. Credentials can be reset if lost or compromised in a secure manner.
3. Session transactions are associated to my authenticated identity and cannot be spoofed or faked.
4. Known potential exploits to hijack or steal credentials or my authenticated session are tested for and properly mitigated.

### Release Only When:
1. All acceptance criteria are met.
2. Multiple users across at least two nodes on separate network broadcast domains and spanning the public internet are tested.
3. Basic compromise testing kits are run against web application and all credential reset mechanisms.
4. Authenticated session can be associated to a basic permission right for a basic logged or persisted application event or transaction.

### References/Attachments:
* [description-absoluteurl](http://dev.holochain.org)
* [description-relativeurl](/thisimage.png)
* etc.... 

### story.toml
``` 
# this TOML schema is used to index the epic/story into the requirements management system
schema = "story.toml"
title = "epic:Identify & Authenticate User"
itemid = "e.20190116.001@ddw" # see note.1
status = "draft" # see note.2
added = 2019-01-16
revised = 2019-01-16 00:00:00Z
iteration = "19.00A" # see note.3
value = { e.points = 7, e.rank = 4 } # see note.4
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
eyJoaXN0b3J5IjpbLTEyOTM3OTI2XX0=
-->