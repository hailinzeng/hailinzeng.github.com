---
layout: page
title: Resume
group: navigation
---
{% include JB/setup %}

### Hailin Zeng ###
- Software Engineer @ ATEC. Beijing, China. (Remote working)
- Email: <img src="{{ site.url }}/email.png">
- Phone:[]()
- Blog:[hailinzeng.github.io](http://hailinzeng.github.io)

### Skill ###
- 5 years Tech experience, fluent English
- C/C++ Engineer (5 years) , Java Engineer (3 years)
- `C++`, `Lex&Yacc`, `LLVM`, `Compiler`, `Shell`, `Linux`, `Java`, `MyBatis`, `Python`, `Flask`, `MySQL`, `jQuery`, `BootStrap`

### Work Experience ###
- `Dec, 2014 - Present`, ATEC Technologies Inc. San Francisco Bay Area, R&D
- `Jul, 2012 - Dec, 2014`, 58.com Inc. Beijing, Service Platform Architecture Team, R&D

#### *ATEC: A Tool for Equivalence Checking* ####
- `Jul, 2009 – Apr, 2011` (Intern) ; `Dec, 2014 – Present` (Full-Time)
- Formal Verification Tool to hunt bugs in Integrate Circuit Design.
- Responsibility:
* 1) Write lexical/syntax analyzer by Flex&Bison, able to handle SUN’s PicoJava II.
* 2) Transformation into SSA form, doing simple simplification and optimization, extract transition function.
* 3) Symbolic Unroll the transition function, create constraints and assertions.
* 4) Check constraints/assertions validity by constraint solver.

#### *Miner: C++ test case generation* ####
- `Feb, 2017 - Present`
- Formal Verification Tool to generate test case of interest.
- Responsibility:
* 1) Write LLVM optimization pass, tested with llvm test-suite.

#### *DLM: Distributed Lock Manager* ####
- `Aug, 2012 – Feb, 2013`
- Distributed Lock Manager based on the Paxos consensus algorithm.
- Responsibility:
* 1) Fully in charge, except Leader election algorithm, util class (log, hash, and thread pool), and java client.
* 2) The server use epoll + thread pool model for handling request. Client use epoll for async communication, and consistent hashing for load-balancing.
* 3) Designed the Distributed Lock by starting a commit proposal with a unique session id, queued in leader server, grant if previous lock request released or expired(heartbeat lost). Both Require and Release are keeping consensus agreement between servers by Paxos.

#### *USP: Unified Supervising Platform* ####
- `Mar, 2013 – Oct, 2014`
- Unified platform to monitor server machines and running services.
- Responsibility:
* 1) Implement DAT pattern matching algorithm for checking webpage containing a certain keyword.
* 2) Implement Nginx module and Lua script to collect http logs. (later switched to kernel modules to collect from Switcher)
* 3) Implement Storm topologies for top N http error URLs, http 404 count in last 5 min.
* 4) Implement Hadoop Jobs analyzing http logs, timeout logs in Web Framework, generating daily reports.
* 5) Implement contacts info, monitor configuration pages, from front-end javaScript to back-end database.
* 6) Implement simple distributed tracing system according to the idea of Google dapper.

#### *Raven: CMS for InfoQ China editor* ####
- `Mar, 2015 - Aug, 2015` (Part-Time)
- Simple CMS for editors in InfoQ China
- Responsibility:
* 1) Fix bugs in the old system, written by python flask framework, deployed on SAE.
* 2) Refactoring by Spring + MySql + Bootstrap + jQuery. Finish the draft version, and then hand it over.

#### *DSAP: Data Platform Integration* #####
- `Oct, 2015 - Nov, 2015`
- Apache Hadoop Ecosystem Data Platform in 58.com Inc.
- Responsibility:
* 1) Integrate Storm and Oozie in DSAP.

#### *Talos: Distributed Storage* ####
- `Apr, 2014 – Jun, 2014`
- A distributed storage system based on the idea of GFS.
- Responsibility:
* 1) Implement the binlog functionality, that is pull binlog from master dataserver, redo the post/delete operation in slave dataserver.

#### *Bastion: Record SSH Sessions for Security Audit* ####
- `Dec, 2013 – Jan, 2014`
- Uniq entrance to login online machine. User operations are saved for audit.
- Responsibility:
* 1) Capture the input/output of Linux Script command, save it to MySQL.

### Education Background ###
- `Sep, 2009 – Jan, 2012`, Master's Degree, Computer Science and Theory, Jilin University, Second Best
- `Sep, 2005 – Jan, 2009`, Bachelor's Degree, Computer Science and Technology, Jilin University, Top 30%

### Executive Summary ###
- Personality: Curious in the essence behind. Open-minded in learning new things. ISTJ personality.

<!-- ### Portfolios ### -->
