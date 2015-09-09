Quartz Basic
=========

- [One Example](#one-example)
	- [maven dependency](#maven-dependency)
	- [maven plugin to execute the program](#maven-plugin-to-execute-the-program)
	- [add startup code in main method](#add-startup-code-in-main-method)
	- [define one simple hello world jobs](#define-one-simple-hello-world-jobs)
	- [put the initial code for jobs and triggers](#put-the-initial-code-for-jobs-and-triggers)
	- [import statements](#import-statements)
	- [build and run](#build-and-run)
- [concepts](#concepts)

## One Example
### maven dependency
First thing should be adding dependencies to pom.xml:
```xml
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>${quartz.version}</version>
</dependency>
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz-jobs</artifactId>
  <version>${quartz.version}</version>
</dependency>
```
the current version should be 2.2, add property:
```xml
<quartz.version>2.2.1</quartz.version>
```

### maven plugin to execute the program
we still need one maven plugin to execute the whole program. Add this plugin:
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>${exec-maven.version}</version>
  <configuration>
     <mainClass>com.twabs.autosys.App</mainClass>
  </configuration>
</plugin>
```
the version should be 1.4. Add it as one property:
```xml
<exec-maven.version>1.4.0</exec-maven.version>
```

### add startup code in main method
quartz will need initialization in the main method:
```java
Scheduler scheduler;
try {
scheduler = StdSchedulerFactory.getDefaultScheduler();
  scheduler.start();

  // place to put jobs and triggers

  Thread.sleep(60 * 1000);

  scheduler.shutdown();
} catch (Exception e) {
e.printStackTrace();
}
```

### define one simple hello world jobs
In quartz, the job class will have to implement quartz job interface:
```java
package com.twabs.autosys.job;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class HelloJob implements Job {

	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		int count = 10;
		while (count-- > 0) {
			System.out.println("hello world");
			try {
				Thread.sleep(5 * 1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```

### put the initial code for jobs and triggers
After we have a job class, then we can put the following code in main:
```java
// define the job and tie it to our HelloJob class
JobDetail job = newJob(HelloJob.class)
    .withIdentity("job1", "group1")
    .build();

// Trigger the job to run now, and then repeat every 40 seconds
Trigger trigger = newTrigger()
    .withIdentity("trigger1", "group1")
    .startNow()
    .build();

// Tell quartz to schedule the job using our trigger
scheduler.scheduleJob(job, trigger);
```

### import statements
when putting the above code in, then we need to define the following imports:
```java
import static org.quartz.JobBuilder.*;
import static org.quartz.TriggerBuilder.*;
```

### build and run
Run this command to execute it:
```bash
mvn exec:java
```

## concepts
- Scheduler - the main API for interacting with the scheduler.
- Job - an interface to be implemented by components that you wish to have executed by the scheduler.
- JobDetail - used to define instances of Jobs.
- Trigger - a component that defines the schedule upon which a given Job will be executed.
- JobBuilder - used to define/build JobDetail instances, which define instances of Jobs.
- TriggerBuilder - used to define/build Trigger instances.
