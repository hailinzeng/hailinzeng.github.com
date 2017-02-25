---
layout: page
title: Resume
group: navigation
---
{% include JB/setup %}

### Hailin Zeng ###
- Software Engineer @ ATEC. Beijing, China. (Remote working)
- mail: <img src="{{ site.url }}/email.png">
- tel:[]()
- blog:[hailinzeng.github.io](http://hailinzeng.github.io)

### Self Assessment ###
- Skills: Skilled in C/C++, Java, Shell, Familiar with Flex & Bison. Understand Python.
- Personality: Curious in the essence behind. Open-minded in learning new things. ISTJ personality.

### Work Experience ###
- `Dec, 2014 - Present`, ATEC Technologies Inc. San Francisco Bay Area, R&D
- `Sep, 2014 - Dec, 2014`, 58.com Inc. Beijing, Database Platform Engineer, R&D
- `July, 2012 - Sep, 2014`, 58.com Inc. Beijing, Service Platform Architecture Team (for inner infrastructure services), R&D

### Education ###
- `Sep, 2009 – Jan, 2012`, Master's Degree, Computer Science and Theory, Jilin University, Second Best
- `Sep, 2005 – Jan, 2009`, Bacheler's Degree, Computer Science and Technology, Jilin University, Top 30%

### Projects ###

#### *Formal Equivalence Checking* ####
- Duration: Jul, 2009 – Apr, 2011 (during undergraduate years), Dec, 2014 – Present (fulltime)
- Description: Formal Equivalence Checking [(wikipedia)](http://en.wikipedia.org/wiki/Formal_equivalence_checking) for Integrated Circuit. Customers are chip vendors (eg, MTK, huawei, ZTE).
- Responsibility:
* 1) Write lexical/syntax analyzer by Flex&Basion, able to handle SUN’s PicoJava II.
* 2) Transformation into SSA form, doing simple simplification and optimization, extract transition function.
* 3) Symbolic Unroll the transition function, create constraints and assertions.
* 4) Check constraints/assertions validity by constraint solver.

#### *Distributed Lock Manager* ####
- Duration: Arg, 2012 – Feb, 2013
- Description: Distributed lock manager based on the Paxos consensus algorithm implement by C++.
- Responsibility:
* 1) Fully in charge, except Leader election algorithm, util class (log, hash, and thread pool), and java client.
* 2) The server use epoll + thread pool model for handing request. Client use epoll for async communication, and consistent hashing for load-balancing.
* 3) Designed the Distributed Lock by starting a commit proposal with a unique session id, queued in leader server, grant if previous lock request released or expired(heartbeat lost). Both Require and Release are keeping consensus agreement between servers by Paxos.

#### *Unified Supervising Platform* ####
- Duration: Mar, 2013 – Oct, 2014
- Description: Part of USP project ([velocity talks](http://velocity.oreilly.com.cn/2013/index.php?func=session&id=16)), Monitoring machine info, service running status, and business logs across several hundred online machines.
- Responsibility:
* 1) Implement DAT pattern matching algorithm for checking webpage containing a certain keyword.
* 2) Implement Nginx module and Lua script to collect http logs. (latter switched to kernel modules to collect from Switcher)
* 3) Implement Storm topologies for top N http error URLs, http 404 count in last 5 min.
* 4) Implement Hadoop Jobs analyzing http logs, timeout logs in Web Framework, generating daily reports.
* 5) Implement contacts info, monitor configuration pages, from front-end javaScript to back-end database.
* 6) Implement simple distributed tracing system according to the idea of Google dapper.

#### *InfoQ Raven* ####
- Duration: Mar, 2015 - Aug, 2015 (part-time)
- Description: Simple CMS for editors in InfoQ
- Responsibility:
* 1) Fix bugs in old system, written by python flask framework, deployed on SAE.
* 2) Refactoring by Spring + MySql + Bootstrap + jQuery. Finish the draft version, and then hand it over.

#### *DSAP Integration* #####
- Duration: Oct, 2015 - Nov, 2015
- Description: Data Platform in 58.com Inc.
- Responsibility: Integrate Storm and Oozie in DSAP.

#### *Distributed Storage* ####
- Duration: April, 2014 – June, 2014
- Description: A distributed storage system based on the idea of GFS.
- Responsibility: Implement the binlog functionality, that is pull binlog from master dataserver, redo the post/delete operation in slave dataserver.

#### *Bastion Host Audit* ####
- Duration: Dec, 2013 – Jan, 2014
- Description: Capturing and logging user operations on bastion host for audit.
- Responsibility: Capture the input/output of Linux Script command, save it to MySQL.
