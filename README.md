# utl_how_to_make_a_loop_to_duplicate_previous_rows
How to make a loop to duplicate previous rows.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    How to make a loop to duplicate previous rows

    Same result in SAS nad WPS
    
    New improved solutions on end
    Paul Dorfman <sashole@bellsouth.net>
    Keintz, Mark via listserv.uga.edu
    mkeintz@wharton.upenn.edu

    
    
    see github
    https://tinyurl.com/y8r3b2hf
    https://github.com/rogerjdeangelis/utl_how_to_make_a_loop_to_duplicate_previous_rows

    see
    https://tinyurl.com/yaxdtyge
    https://communities.sas.com/t5/Base-SAS-Programming/how-to-make-a-loop-to-duplicate-previous-rows/m-p/471287

    INPUT
    =====

    WORK.HAVE total obs=3

      ID    NAME

       1    Jam
       2    John
       3    Will

    EXAMPLE OUTPUT

     WORK.WANT total obs=6   |  RULE
                             |
      NEWID  ID    NAME      |
                             |
        1    1     Jam       |  * no previous row so output just once
                             |
        2    1     Jam       |  * output ALL previous and current records
        3    2     John      |
                             |
        4    1     Jam       |  * output ALL previous and current records
        5    2     John      |
        6    3     Will      |


    PROCESS
    =======

    * make fat;
    proc transpose data=have out=havXpo(drop=_name_);
    var name;
    id id;
    run;quit;

    * simple double loop to repeat n,n-1,n-2... for 1=1 to n;
    data want;
      retain newid 0;
      set havXpo;
      array nams[*] _character_;
      do i=1 to 3;
        do j=1 to i;
          put i= j=;
          id=substr(vname(nams[j]),2);
          name=nams[j];
          newid=newid+1;
          output;
          keep newid id name;
        end;
      end;
    run;quit;


    OUTPUT
    ======

     WORK.WANT total obs=6

       NEWID    ID    NAME

         1      1     Jam
         2      1     Jam
         3      2     John
         4      1     Jam
         5      2     John
         6      3     Will

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;


    data have;
    input id name $;
    cards4;
    1 Jam
    2 John
    3 Will
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    proc transpose data=wrk.have out=havXpo(drop=_name_);
    var name;
    id id;
    run;quit;
    data want;
      retain newid 0;
      set havXpo;
      array nams[*] $ _character_;
      do i=1 to 3;
        do j=1 to i;
          put i= j=;
          id=substr(vname(nams[j]),2);
          name=nams[j];
          newid=newid+1;
          output;
          keep newid id name;
        end;
      end;
    run;quit;
    proc print;
    run;quit;
    ');
    
    
    
    Elegant solution by Mark on the end

    Mark Keintz
    Mark Keintz's profile photo
    mkeintz@wharton.upenn.edu

    data have;
    input id name $;
    cards4;
    1 Jam
    2 John
    3 Will
    ;;;;
    run;quit;

    data vneed2 / view=vneed2;
      set have;
      position=_n_;
    run;

    data want2 (drop=group position);

      declare hash h (dataset:'vneed2');
        h.definekey('position');
        h.definedata(all:'Y');
        h.definedone();
      if 0 then set have nobs=nhave;

      do group=1 to nhave;
        do position=1 to group;
          h.find();
          output;
        end;
      end;
    run;


    *                                     _   _               _
     _ __   _____      __  _ __ ___   ___| |_| |__   ___   __| |___
    | '_ \ / _ \ \ /\ / / | '_ ` _ \ / _ \ __| '_ \ / _ \ / _` / __|
    | | | |  __/\ V  V /  | | | | | |  __/ |_| | | | (_) | (_| \__ \
    |_| |_|\___| \_/\_/   |_| |_| |_|\___|\__|_| |_|\___/ \__,_|___/

    ;


    Paul Dorfman <sashole@bellsouth.net>
    to SAS-L, me
    Roger,

    Having seen nice and clever solutions offered by yourself and Mark,
    I couldn't get rid of the feeling that there must be a way to attain
    the goal without re-reading data, be it from the data set itself,
    arrays, or a hash table. Finally, it has dawned on me that
    if one doesn't care about the order of the output records, it
    can be done as tersely as:

    data want ;
      set have nobs = n ;
      do _iorc_ = 0 to n - _n_ ;
        output ;
      end ;
    run ;

    for the simple reason that the 1st record will end up output NOBS times,
    the 2nd - (NOBS-1) times, and so forth. However, if the output order is important,
    it's easy to add an indexed variable RID:

    data want (index=(rid)) ;
      set have nobs = n ;
      rid = _n_ * (1 + _n_) / 2 ;
      do _iorc_ = 0 to n - _n_ ;
        rid + ifn (_iorc_, _n_ - 1 + _iorc_, 0) ;
        output ;
      end ;
    run ;

    and then, whenever WANT is to be read in the desired order, just add the BY statement:

    data ... ;
      set want ;
      by rid ;
      ...
    run ;

    Best regards



    Keintz, Mark via listserv.uga.edu
    to SAS-L
    Ok, I'll bite.   If RID needs only to be ordered, but not equally spaced:

    data want ;
      set have nobs = n ;
      do _iorc_ = 0 to n - _n_ ;
        rid=  _iorc_ + (_n_-1)/n;
        output ;
      end;
    run ;

    But if the strategy is to use RID as an index in order to support BY RID order,
    there will be increasing amounts of random-access input over a large
    file, whose size will be (NHAVE/2 + 0.5) times the size of HAVE.

    In which case, why not enforce order by using random-access on the original
    small data set?  Yes, it has the disadvantage of rereading observations but
    the incoming pages of data will not be  saturated with repeated observations,
    mandating more page reads.

    data want;
      do p=1 to _n_;
        set have point=p nobs=nhave;
        output;
      end;
      if _n_=nhave then stop;
    run;
