# Labs - Ansible for z/OS 

## Lab 1: Connect to z/OS 

If you don't already have a 3270 terminal emulator installed, install one. C3270 is a popular choice. It's a command-line wrapper around the open source x3270 emulator. 

- [Install c3270 on Linux](https://pkgs.org/download/c3270)
- [Install c3270 on OSX with Homebrew](https://formulae.brew.sh/formula/x3270)
- [Install c3270 on OSX with MacPorts](https://ports.macports.org/port/c3270/)
- [Install c3270 on Windows](https://x3270.miraheze.org/wiki/Downloads)

Using the terminal emulator, connect to the lab z/OS instance and disconnect again. 

## Lab 2: Ping z/OS from Control Node 

Run the ping.yml playbook with a successful result. 

## Lab 3: Locate IBM z/OS documentation online 

Using a Web browser, find the IBM documentation entitled "z/OSMF core services" for z/OS version 3.2.0.

## Lab 4: Log on and off z/OS 

Log on to TSO and get into ISPF. Visit a few of the ISPF menu options. Return to the Primary Option Menu. Exit from ISPF and delete your log file. Logoff from TSO. Disconnect from z/OS. 

## Lab 5: Use ISPF to create a QSAM data set

Use ISPF option 3.2, "A" to create a QSAM (sequential) data set on z/OS. Give the data set the following attributes: 

- name: as you wish
- data set organization: physical sequential
- record format: fixed, blocked 
- logical record length: 250 
- block size: 5000

Verify the attributes of the data set visually using ISPF option 3.2, "blank" to view data set information. 

## Lab 6: Use Ansible module zos_data_set to create a QSAM data set 

Using sample code and/or documentation as a guide, develop a playbook to create the same QSAM data set as you did in Lab 4, with the same name. Change the logical record length to 300 and the block size to 6000. Use the appropriate parameters to replace the existing data set of the same name without a separate "delete" task. 

Run the playbook and verify the result visually by logging on to TSO and viewing the data set attributes via ISPF option 3.2. 

## Lab 7: Use JCL to create a QSAM data set 

Upload the sample JCL from jcl/CRQSAM1 to the z/OS instance. The target for the file transfer is your JCL library on z/OS, which is named <userid>.JCL, with member name CRQSAM1. Use the ISPF Editor to modify the JCL so it has your userid instead of the default one. 

We have not covered the ISPF Editor yet, so if you need help with this please assist each other and/or ask the instructor. We don't want you to get stalled on that step because it isn't the point of the lab exercise. 

Look at file ansible/create_qsam.yml and the JCL, and find the correspondence between the parameters in the Ansible task and the parameters on the DD statements in the JCL. This is the main learning point of the lab exercise. 

When the JCL is set up with your userid, use the SUBMIT command on the Command line of the ISPF Editor. You can abbreviate it to SUB. This will submit the JCL to JES for processing. Assuming you set your userid in the right place on the JOB statement, you'll see messages indicating when your job starts and when it completes, and the condition code. 

Navigate in ISPF to SDSF, which is option 13 on the Primary Option Menu, and then option 14 on the next menu, in the lab z/OS system. (This can be changed per installation, and your company may have placed SDSF elsewhere in the ISPF menu hierarchy.)

Navigate to the Output queue by typing "o" in the Command field, and locate your job in the queue. Type a "?" next to it, and the system will display a list of spooler output data sets. You can view each of them by typing an "s" next to the name. Briefly examine these data sets to get a sense of the information they provide. We'll go into them in more detail later. 

## Lab 8: Use Ansible to run the same job from the Control Node 

Create a playbook that uses the zos_job_submit module to submit the same job as in Lab 6 from your z/OS JCL library. Examine the results returned from the run and notice that they include the spooler data sets that you saw in Lab 6.

## Lab 9: Use a TSO command to create a QSAM data set 

Log on to TSO/ISPF. Exit ISPF to get to the TSO ready prompt. Enter a command to create a QSAM data set. Specify any name and other attributes you wish. 

Use any combination of IBM documentation, Internet searches, LLM assistants, and/or course materials to discover the appropriate TSO command and its arguments. 

Verify the result visually by viewing the data set attributes via ISPF option 3.2. (From the TSO ready prompt, enter "ispf" to start ISPF again; no need to log off and on again.)

## Lab 10: Use Ansible module zos_tso_command to create a QSAM data set 

Develop a playbook that uses zos_tso_command to submit the TSO command that creates a QSAM data set. Specify any attributes you wish and verify the result as you did for Lab 6. 

## Lab 11: Identify the four JCL statements

Group discussion. Looking at jcl/CRQSAM2 in the course repo, identify examples of each of the four JCL statements - JOB, EXEC, DD, and comment. Determine how many steps there are in the job. Explain the effects of the DISP= parameter values on the DD statements. 

## Lab 12: Break down the parts of the JOB statement

Group discussion. Referring to jcl/CRQSAM2 again, examine the JOB statement. 

- What is the job accounting information on this JOB statement?
- Why does the job name begin with the TSO prefix (same as userid)? 
- What is the effect of the &SYSUID parameter? 
- What is the effect of the MSGLEVEL parameter? 
- Why would you specify a TIME parameter on a JOB statement?

Find the IBM documentation that explains the parameters of the JOB statement. 

## Lab 13: Examine system output from a job 

Upload the provided sample JCL in file jcl/GENER1 to your JCL library on the z/OS instance. From the ISPF Primary Option Menu, navigate to View (usually option 1). Open GENER1 and submit it. 

Navigate to SDSF (System Display and Search Facility). Go to the Output queue and scroll down to find your job. Type a question mark (?) next to the job name and press Enter to get a list of output spooler data sets from the job. 

You will see JESMSGLG, JESJCL, JESYSMSG, SYSPRINT, and SYSUT2.

Type an "s" next to the name of one of the data sets and press Enter to view the contents of the data set. 

View the JESMSGLG data set. It tells you the job was assigned to your userid, the start and end times of the job, and the condition code (also known as return code, and shown as RC in this data set). 

JESMSGLG also shows the execution time of the job. This job is very short and may run so quickly that a time of zero is shown. 

Notice the output shows how many cards were read, how many records (lines) were printed, and how many cards were punched. This is an example of the brain-in-a-jar aspect of the system. z/OS still thinks it's working with punched cards and printers. 

Look at the JESJCL data set. This JCL doesn't execute any JCL procedures and has no symbolics other than SYSUID, so there isn't much to see in this case. 

The JESYSMSG data set tells you a lot about what happened during job execution. It shows the condition code - this time it's called "condition code" rather than "return code," but it's the same value. 

This dataset shows all the data sets that were allocated and what happened to them (their "disposition"). 

This job runs one of the standard system utility programs, IEBGENER. The program can copy and modify data sets - conceptually similar to sed on *nix systems. The simplest use case for it is to copy one data set to another, and that's what the sample job does. 

The SYSUT2 data set is the target for IEBGENER. After it copies and optionally modifies data from SYSUT1, it writes the result to SYSUT2. In this case, it just copies the contents of SYSUT1 into SYSUT2. 

# Lab 14: Use Ansible to run the sample job and save SYSUT2 

Now that you've seen what the output looks like, go to your Ansible control node and develop a playbook that submits the same job, GENER1, and copies the contents of the SYSUT2 dataset to a local file. 

You now have two playbooks that submit jobs and capture the jobs' spooler output data sets - one for CRQSAM1 and one for GENER1. If you haven't thought of it already, break out tasks for these steps and create a single playbook that submits a job from JCL resident on z/OS and captures the spooler output. 

These changes do not alter the behavior of the playbooks, but only their internal structure. Software engineers call this kind of modification a "refactoring." Infrastructure engineers who practice "infrastructure as code" apply the same software engineering principles to their scripts and configuration files as programmers do to their source code. 

## Labs 15-20 

Use your mad skills for locating IBM documentation, locating Ansible documentation, doing Internet searches, collaborating with your classmates, finding useful samples in the course repo, and/or prompting your favorite LLM coding assistant to find the information you require to complete these labs. You can work individually, in pairs, or in small groups as you prefer.

## Lab 15: Use JCL to create a variable-blocked QSAM data set 

Create JCL for a job that conditionally deletes and then creates a QSAM data set containing variable-length records with a maximum length of 100 bytes and a block size that can contain 40 maximum-length records.

Use your mad skills in locating IBM documentation, locating Ansible documentation, doing Internet searches, collaborating with your classmates, and/or prompting your favorite LLM coding assistant to find the information you require.

## Lab 16: Use Ansible to create a variable-blocked QSAM data set 

Do the same task as in Lab 15 but using Ansible with the zos_data_set module. 

## Lab 17: Use JCL to create a program source library

A program source library on z/OS is a Partitioned Data Set (PDS) whose members all have a fixed record length of 80 bytes, corresponding to the number of columns on an ancient punched card. 

Write JCL for a job that conditionally deletes and then creates this type of data set.

## Lab 18: Use Ansible to create a program source library

Do the same task as in Lab 17 but using Ansible with the zos_data_set module. 

## Lab 19: Use JCL to create a program object library

z/OS supports two types of object code libraries. 

A Load Library is a PDS whose members are compiled and linked programs that are not quite ready to be loaded into memory for execution. z/OS prepares them and loads them when they are called for in a job. This is an older format, supported for backward compatibility.

A Program Library is a PDSE whose members are executables ready to be loaded into memory. This is the current format. 

Write JCL to conditionally delete and then create a Program Library.

## Lab 20: Use Ansible to create a program object library

Do the same task as in Lab 19 but using Ansible with the zos_data_set module. 

## Lab 21: Use JCL to define a GDG 

Based on the example shown in the presentation, write JCL to execute the IDCAMS utility to DELETE and DEFINE a Generation Data Group (GDG). Working on the z/OS instance, run the job and see that it did in fact create the GDG.

Make sure the step is idempotent. It shouldn't fail when the GDG already exists.

## Lab 22: Use Ansible to define a GDG 

Using Ansible documentation and any other resources at your disposal, write a playbook that performs the same functions as the JCL you wrote in Lab 21. 

Make sure the task is idempotent. It shouldn't fail when the GDG already exists.

Hint: Use module zos_data_set from the ibm_zos_core collection. 

## Lab 23: Use JCL to define a Model DSCB for GDSs in your GDG 

Either (a) create a new job or (b) add a second step to the job you created in Lab 21 that creates a model DSCB with the DCB attributes you want applied to all the Generation Data Sets that will belong to your Generation Data Group. 

Make sure the step is idempotent. It shouldn't fail when the model DSCB already exists.

Hint: Execute the do-nothing utility program, IEFBR14, and define a DD statement that specifies the attributes of the model DSCB. 

## Lab 24: Use Ansible to define a Model DSCB for GDSs in your GDG

Add a task to the playbook you developed in Lab 22 to create the same model DSCB as you created with JCL in Lab 23. 

Make sure the task is idempotent. It shouldn't fail when the model DSCB already exists.

Hint: A model DSCB is just a catalog entry for a data set that has no space allocation. You define it the same way you define any other data set. 

## Lab 25: Retrieve a concatenation of selected generations of a GDG

With your GDG as the source, develop an Ansible playbook that retrieves a concatenation of two Generation Data Sets. 

You know how to do each piece of this. The exercise is to put the pieces together and make whatever customizations to the JCL and the playbook that are necessary. 

You will need to write JCL that concatenates two GDSs into a single data set on the z/OS system. Then you will need Ansible tasks to submit that JCL and fetch the concatenated data set, and finally to display the content of the data set.

You can write and test your JCL on the z/OS platform if you wish, but for the lab we would like you to store the JCL on the Control Node and pass it to z/OS in a task. In everyday work you will not switch frequently between working on the Control Node and working directly on the z/OS Managed Node.

The work will involve modules zos_job_submit, zos_copy, and zos_fetch from the ibm_zos_core collection.

## Lab 25a BONUS: Concatenate and retrieve data sets using zos_mvs_raw 

Develop a playbook that does the same thing as your solution for Lab 25, but using module zos_mvs_raw to avoid having to write JCL to do the concatenation. 

## Lab 26: Use Ansible to define a VSAM KSDS

Using Ansible documentation, IBM documentation, examples from the slide deck, samples in the course repo, Internet searches, your favorite neighborhood LLM helper, and/or collaboration with your classmates, develop an Ansible playbook that defines a VSAM KSDS cluster and queries the catalog information for it.

## Lab 27: Use Ansible to execute DFSORT/ICEMAN 

Using module zos_mvs_raw, develop an Ansible playbook to execute ICEMAN (DFSORT) with a task that sorts the two input data sets as one (concatenated) and lists the records in descending order by city population. Also remove duplicate records.

Input record format:

01-14 State name
15-34 City name 
35-36 State postal abbreviation
37-43 City population 

Output record format:

01-02 State postal abbreviation 
03-22 City name 
23-30 City population 

## Lab 28: Set up data sets for an application developer 

A new software developer has joined your company. She will be working as a software engineer on a team that supports a mainframe-based product. She will need certain data sets created under her z/OS userid:

<userid>.COBOL.SOURCE	
<userid>.COBOL.COPYLIB	
<userid>.REXX.SCRIPTS	
<userid>.DEV.JCL		
<userid>.QMF.SCRIPTS	
<userid>.DEV.PROGLIB

All these are PDSE data sets (LIBRARY type). TEST.PROGLIB is a program library containing executables. The others are all source libraries and all contain 80-byte card-image records – same attributes.

Develop an Ansible playbook to create these data sets on the z/OS instance. You will have to use your own userid, as we don't have authorization to create data sets under any other high level qualifier. 

For all except the program library, add a member after allocating the PDSE.

Data Set            Member Name     Contents

COBOL.SOURCE        README          COBOL SOURCE CODE GOES HERE
COBOL.COPYLIB       README          COBOL COPYBOOKS GO HERE 
REXX.SCRIPTS        README          REXX SCRIPTS GO HERE 
QMF.SCRIPTS         README          SQL SCRIPTS FOR DB2 GO HERE 
DEV.JCL             JOBCARD         (a JOB card with appropriate settings for the environment)

Make good use of Ansible features such as loops and variables, and factor out repetitive tasks, to keep the playbook clean and understandable. That is the main point of the exercise - the technical details are already familiar to you by now.

Note that the older format for library-type data sets, the PDS, is supported for backward compatibility. It's preferable to use the PDSE format for these libraries.

## Lab 29: Collect z/OS System Information 

Develop a playbook that performs the following tasks:

- Query the z/OS node to retrieve static information from subsets ipl, cpu, iodf, and sys. 
- Display the results.
- Find all the active jobs that are started tasks. Hint: job_class will be "STC".
- Display the results. 
- List the names of the members in the SYS.PARMLIB data set.
- Display the results.

This lab exercise doesn't provide step by step instructions. It only states the desired outcome. By this point you know how to find the IBM z/OS documentation, the Ansible documentation, IBM's Ansible examples on Github, the source code for the ibm_zos_core collection on Github, the samples provided in the course repo, reusable patterns in your own playbooks, LLM assistants, class mates you can ask, Internet searches, and other resources to discover the details of how to accomplish the outcome. You know how to log on to the z/OS instance directly and try out TSO commands, look at SDSF output, find data set names under a given high-level qualifier, and other actions to help yourself understand what to query for in your Ansible playbook, and how the results are structured. 



