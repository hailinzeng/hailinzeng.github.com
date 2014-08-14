---
layout: page
title: Resume
group: navigation
---
{% include JB/setup %}

## Hailin Zeng
- Senior Software Engineer @ 58.com Inc. Beijing, China
- mail:[](mailto:)
- tel:[](tel:)
- blog:[http://hailinzeng.github.io](http://hailinzeng.github.io)

### Self Assessment ###
- Skills: Skilled in C/C++ (2.5 years experience), Java (1.5 year experience), Shell, Familar with Flex & Bison. Understand Python.
- Personality: Curious in the essence behind. Open mind in learning new things. ISTJ personality.

### Work Experience ###
- July 03, 2012 - Present 	 58.com(China's Craigslist), Service Platform Architecture Team (team responsable for inner infrastructure services), R&D

### Education ###
- Sep, 2009 – Jan, 2012, Master's Degree, Computer Science and Theory, Jilin University, Second Best
- Sep, 2005 – Jan, 2009, Bacheler's Degree, Computer Science and Technology, Jilin University, Top 30%

### Projects ###

#### *Distributed Storage* ####
- Duration: April, 2014 – June, 2014
- Description: A distributed storage system based on the idea of GFS.
- Responsibility: Implement the binlog functionality, that is pull binlog from master dataserver, redo the post/delete operation in slave dataserver.

#### *Bastion Host Audit* ####
- Duration: Dec, 2013 – Jan, 2014
- Description: Capturing and logging user operations on bastion host for audit.
- Responsibility: Capture the input/output of linux Script command, save it to MySQL.

#### *Unified Supervising Platform* ####
- Duration: Mar, 2013 – Present
- Description: Part of USP project ([velocity talks](http://velocity.oreilly.com.cn/2013/index.php?func=session&id=16)), Monitoring machine info, service running status, and business logs across serveral hundred online machines.
- Responsibility: 
* 1) Implement DAT pattern matching algorithm for checking webpage containing a certain keyword.
* 2) Implement Nginx module and Lua script to collect http logs. (latter switched to kernel modules to collect from Switcher)
* 3) Implement Storm topologies for top N http error URLs, http 404 count in last 5 min.
* 4) Implement Hadoop Jobs analyzing http logs, timeout logs in Web Framework, generating daily reports.
* 5) Implement contacts info, monitor configuration pages, from front-end javascript to back-end database. 
* 6) Curring working on distributed tracing system according to the idea of Google's dapper.

#### *Distributed Lock Manager* ####
- Duration: Arg, 2012 – Feb, 2013
- Description: Distributed lock manager based on the Paxos consensus algorithm implement by C++.
- Responsibility: 
* 1) Fully in charge, except Leader election algorithm, util class (log, hash, and thread pool), and java client.
* 2) The server use epoll + thread pool model for handing request. Client use epoll for async communication, and consistent hashing for load-balancing.
* 3) Designed the Distributed Lock by starting a commit proposal with a uniq session id, queued in leader server, grant if previous lock request released or expired(heartbeat lost). Both Require and Release are keeping consensus agreement between servers by Paxos.

#### *Formal Equivalence Checking* ####
- Duration: Jul, 2009 – Apr, 2011
- Description: Formal Equivalence Checking [(wikipedia)](http://en.wikipedia.org/wiki/Formal_equivalence_checking) of Verilog combinational circuits.
- Responsibility: 
* 1) Write lexical/syntax analyzer by Flex&Basion, able to handle SUN’s PicoJava II.
* 2) Transformation into SSA form, doing simple simplification and optimazation.
* 3) Unroll the function call, represent outputs in a formula constraint by input variables.
* 3) Buiding the equivalence logic, check it validity by constraint solver [STP](https://github.com/stp/stp).
