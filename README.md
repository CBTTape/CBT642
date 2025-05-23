# CBT642
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 642 is from Rich Hobt and contains two program packages.  *   FILE 642
//*           AFCLOGR1 is a program to find any strings you want,   *   FILE 642
//*           in a SYSPLEX OPERLOG.  AFCSMP1 provides an interface  *   FILE 642
//*           to the API provided by SMP/E and can be used to       *   FILE 642
//*           produce customized SMP/E reports that can be fed      *   FILE 642
//*           into REXX or (especially) SAS for further processing. *   FILE 642
//*                                                                 *   FILE 642
//*     AFCSMP1 is presented as an IEBUPDTE unloaded pds            *   FILE 642
//*     (actually in PDSLOAD format, to preserve the ISPF           *   FILE 642
//*     statistics.  The PDSLOAD program is provided here           *   FILE 642
//*     in this pds, and a job (member $PDSLOAD) is also            *   FILE 642
//*     provided to create a separate pds for AFCSMP1,              *   FILE 642
//*     with the install information and $README instructions.      *   FILE 642
//*                                                                 *   FILE 642
//*     A description of AFCLOGR1 follows, and is followed by a     *   FILE 642
//*           description of AFCSMP1.                               *   FILE 642
//*                                                                 *   FILE 642
//*                  --->  AFCLOGR1  <---                           *   FILE 642
//*                                                                 *   FILE 642
//*     AFCLOGR1 is a program I developed to scan through an        *   FILE 642
//*     OPERLOG datastream, printing out selected records in        *   FILE 642
//*     hardcopy SYSLOG format.  I find it convenient to use for    *   FILE 642
//*     a number of purposes:                                       *   FILE 642
//*                                                                 *   FILE 642
//*       Finding something in the log when I don't have the        *   FILE 642
//*       time or patience to keep hitting PF5 in SDSF to look      *   FILE 642
//*       for it.                                                   *   FILE 642
//*                                                                 *   FILE 642
//*       Filtering out all the garbage messages in a given time    *   FILE 642
//*       span, when the thing I am looking for may be "hidden"     *   FILE 642
//*       amongst many other messages.                              *   FILE 642
//*                                                                 *   FILE 642
//*       Finding more than one keyword or message ID.              *   FILE 642
//*                                                                 *   FILE 642
//*       Filtering based on jobname, jobid, or system name.        *   FILE 642
//*                                                                 *   FILE 642
//*       Running a daily job to scan for "interesting" messages    *   FILE 642
//*       since the last time I ran the job.  I have an             *   FILE 642
//*       automated process for building and submitting this job    *   FILE 642
//*       when I log on in the morning.                             *   FILE 642
//*                                                                 *   FILE 642
//*     I'm sure you can find other uses for it as well.            *   FILE 642
//*                                                                 *   FILE 642
//*     One note about the program design - I could have added      *   FILE 642
//*     SYSIN parameters to specify the date and time, then         *   FILE 642
//*     dynamically allocate and read the logstream, but I          *   FILE 642
//*     decided against it.  Too much like reinventing the          *   FILE 642
//*     wheel.  The subsystem JCL already has that capability,      *   FILE 642
//*     and it just didn't seem worth the effort.                   *   FILE 642
//*                                                                 *   FILE 642
//*     Our logstream name is SYSPLEX.OPERLOG - if yours is         *   FILE 642
//*     different, you'll have to change the name in the            *   FILE 642
//*     AFCLOGSC proc, or whatever JCL you use to run the           *   FILE 642
//*     program.                                                    *   FILE 642
//*                                                                 *   FILE 642
//*     This program has been run on z/OS 1.3 - 1.9, but I see      *   FILE 642
//*     no reason it can't run on later levels.  HOWEVER, due to    *   FILE 642
//*     the heavy use of relative addressing, it will only run      *   FILE 642
//*     on a box with the relative addressing instructions          *   FILE 642
//*     available.  I'm not sure at which architectural level       *   FILE 642
//*     they were introduced, but I'm pretty sure they've been      *   FILE 642
//*     around for a while.                                         *   FILE 642
//*                                                                 *   FILE 642
//*     UPDATES:                                                    *   FILE 642
//*                                                                 *   FILE 642
//*     2008/12/31                                                  *   FILE 642
//*     Fixed a bug where the program would get into a tight        *   FILE 642
//*     loop.  This was due to me assuming that the OPERLOG         *   FILE 642
//*     LRECL was somehow fixed at 4100.  When I tried running      *   FILE 642
//*     it at my new job, I discovered that this wasn't so.  Now    *   FILE 642
//*     the program will check the LRECL against the MDB length,    *   FILE 642
//*     and ABEND with a U200 if it isn't big enough.  If this      *   FILE 642
//*     happens, just increase the LRECL in the LOGIN JCL.          *   FILE 642
//*                                                                 *   FILE 642
//*     2006/09/25                                                  *   FILE 642
//*     This version has a couple of small enhancements -           *   FILE 642
//*     multiple SYSID statements honored, and FIND=QUIT.  It       *   FILE 642
//*     was also "adjusted" so as not to use a base register for    *   FILE 642
//*     the code - it's all relative now.  I haven't seen a         *   FILE 642
//*     program written this way before, and I couldn't resist      *   FILE 642
//*     experimenting with it.  There are undoubtedly better        *   FILE 642
//*     ways to do it, but this at least has the small virtue of    *   FILE 642
//*     actually working.                                           *   FILE 642
//*                                                                 *   FILE 642
//*     ----------------------------------------------------------  *   FILE 642
//*                                                                 *   FILE 642
//*     SYSIN statements:                                           *   FILE 642
//*                                                                 *   FILE 642
//*     JOBNAME=XXXXXXXX                                            *   FILE 642
//*       Limit the search to records produced by this jobname.     *   FILE 642
//*                                                                 *   FILE 642
//*     JOBID=JOBXXXXX                                              *   FILE 642
//*       Limit the search to records produced by this jobid.       *   FILE 642
//*                                                                 *   FILE 642
//*     JOBNAME and JOBID are not guaranteed to find every          *   FILE 642
//*     message related to the specified jobname and/or jobid.      *   FILE 642
//*     Not sure why this is so, but sometimes the messages just    *   FILE 642
//*     don't seem to have this data in the right fields.           *   FILE 642
//*                                                                 *   FILE 642
//*     SYSID=XXXXXXXX                                              *   FILE 642
//*       Limit the search to records produced from this system     *   FILE 642
//*       (or systems - up to 16, each specified on a separate      *   FILE 642
//*       SYSID= card).                                             *   FILE 642
//*                                                                 *   FILE 642
//*     Note that the following text and msgid keywords refer to    *   FILE 642
//*     "selected" records.  These are records "selected" by any    *   FILE 642
//*     of the preceeding keywords.  If none of these are           *   FILE 642
//*     specified, "selected" becomes all operlog records in the    *   FILE 642
//*     JCL-controlled timespan.                                    *   FILE 642
//*                                                                 *   FILE 642
//*     TEXT='TEXT YOU ARE LOOKING FOR'                             *   FILE 642
//*                                                                 *   FILE 642
//*       The first character after the "=" is the string           *   FILE 642
//*       delimiter and is required.  This can be any character,    *   FILE 642
//*       but must be matched at the end.  Maximum string length    *   FILE 642
//*       is 127.  The text of all selected messages (including     *   FILE 642
//*       multi record) will be scanned for this text.              *   FILE 642
//*                                                                 *   FILE 642
//*     MSGID='MSGID YOU ARE LOOKING FOR'                           *   FILE 642
//*                                                                 *   FILE 642
//*       The first character after the "=" is the string           *   FILE 642
//*       delimiter and is required.  This can be any character,    *   FILE 642
//*       but must be matched at the end.  Maximum string length    *   FILE 642
//*       is 127.  Only the first 3 columns of the 1st line of      *   FILE 642
//*       each selected message will be scanned for the             *   FILE 642
//*       beginning of this text, so it is faster than the TEXT=    *   FILE 642
//*       keyword.                                                  *   FILE 642
//*                                                                 *   FILE 642
//*     FIND=EXCLUDE                                                *   FILE 642
//*                                                                 *   FILE 642
//*       Entered as shown.  The result of this keyword is that     *   FILE 642
//*       if any of the text or msgids are found, the record        *   FILE 642
//*       will not be printed.  Use this to print everything        *   FILE 642
//*       except the matches.                                       *   FILE 642
//*                                                                 *   FILE 642
//*     FIND=QUIT                                                   *   FILE 642
//*                                                                 *   FILE 642
//*       Entered as shown.  Use this if you are scanning           *   FILE 642
//*       through a long time span for the 1st occurrance of        *   FILE 642
//*       something, and dont want to waste the time searching      *   FILE 642
//*       through the rest of the log after you found it.           *   FILE 642
//*       Causes execution to end after the 1st hit.                *   FILE 642
//*                                                                 *   FILE 642
//*     The order of the keywords does not matter.  A record is     *   FILE 642
//*     printed if it matches any of the text strings and the       *   FILE 642
//*     jobname, jobid, and/or sysid criteria (or not printed,      *   FILE 642
//*     if "FIND=EXCLUDE").                                         *   FILE 642
//*                                                                 *   FILE 642
//*     If you just want to print out all the log records in a      *   FILE 642
//*     given timespan, leave out the sysin parameters (or only     *   FILE 642
//*     include comments).                                          *   FILE 642
//*                                                                 *   FILE 642
//*     A "*" in column 1 indicates a comment record.               *   FILE 642
//*                                                                 *   FILE 642
//*     ----------------------------------------------------------  *   FILE 642
//*                                                                 *   FILE 642
//*     The members in this dataset are:                            *   FILE 642
//*                                                                 *   FILE 642
//*     $README  - I guess you already know about this one.         *   FILE 642
//*     AFCLOGR1 - The assembler source for the log scanner.  AFC   *   FILE 642
//*                stands for 'Airborne Freight Corporation', by    *   FILE 642
//*                the way - my employer when I wrote the           *   FILE 642
//*                original version.                                *   FILE 642
//*     AFCLOGSC - JCL proc to run the program.                     *   FILE 642
//*     ASMINFO  - A macro, used by INR to build a                  *   FILE 642
//*                human-readable program header.                   *   FILE 642
//*     ASMJCL   - JCL to assemble and link the program.            *   FILE 642
//*     CLEAR    - A macro, used by AFCLOGR1 to clear a storage     *   FILE 642
//*                field to blanks.                                 *   FILE 642
//*     INR      - A macro, used by AFCLOGR1 for entry              *   FILE 642
//*                housekeeping.  No base reg.                      *   FILE 642
//*     EXECJCL  - JCL to run the program (uses proc AFCLOGSC)      *   FILE 642
//*     OUTR     - A macro, used by AFCLOGR1 for exit housekeeping. *   FILE 642
//*                                                                 *   FILE 642
//*     This program may be used, modified, and/or shared by        *   FILE 642
//*     anyone.  Just don't sell it, and please give me some        *   FILE 642
//*     credit.                                                     *   FILE 642
//*                                                                 *   FILE 642
//*                                                                 *   FILE 642
//*                  --->  AFCSMP1  <---                            *   FILE 642
//*                                                                 *   FILE 642
//*     AFCSMP1 is a program I developed to access the API          *   FILE 642
//*     provided by SMP/E.  It uses control cards to build the      *   FILE 642
//*     SMP/E request (DD GIMIN), then formats and outputs the      *   FILE 642
//*     results (DD GIMOUT).  Status and error messages are         *   FILE 642
//*     reported via DD GIMPRINT.  It is mainly useful for          *   FILE 642
//*     extracting data to be further processed by REXX, SAS, or    *   FILE 642
//*     whatever.  It is especially useful in SAS, as it can be     *   FILE 642
//*     invoked directly from within the code.                      *   FILE 642
//*                                                                 *   FILE 642
//*     This program has been run on z/OS 1.3, 1.4, 1.7. and 1.8.   *   FILE 642
//*                                                                 *   FILE 642
//*     This program may be used, modified, and/or shared by        *   FILE 642
//*     anyone.  Just don't sell it, and please give me some        *   FILE 642
//*     credit.                                                     *   FILE 642
//*                                                                 *   FILE 642
//*     Richard Hobt                                                *   FILE 642
//*     Arizona Department of Public Safety                         *   FILE 642
//*     2310 N. 20th                                                *   FILE 642
//*     Phoenix, AZ.  85009                                         *   FILE 642
//*     (602) 223-2519                                              *   FILE 642
//*     RHobt@azdps.gov                                             *   FILE 642
//*                                                                 *   FILE 642
//*     Dec. 31, 2008                                               *   FILE 642
//*                                                                 *   FILE 642
```
