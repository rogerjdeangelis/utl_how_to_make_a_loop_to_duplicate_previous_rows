# utl_how_to_make_a_loop_to_duplicate_previous_rows
How to make a loop to duplicate previous rows.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    How to make a loop to duplicate previous rows

    Same result in SAS nad WPS

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
