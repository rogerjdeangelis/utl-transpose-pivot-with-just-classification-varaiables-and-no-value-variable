# utl-transpose-pivot-with-just-classification-varaiables-and-no-value-variable
Transpose pivot with just classification varaiables and no value variable
    %let pgm=utl-transpose-pivot-with-just-classification-varaiables-and-no-value-variable;

    %stop_submission;

    Transpose pivot with just classification varaiables and no value variable

    SOLUTIONS  (all ofthese create sas tables includin procs freq report and tabulate)
               (many of these soltions can be generalized to higher dimenions and mutiple statistics)

        1 proc corresp
        2 sas sql (with and w/o sql arrays)
        3 proc report
        4 sql r with sql arrays
        5 sql python without sql arrays
        6 excel spss/pspp matlab/octave postreSQL sqlite3
          not shown,  use sqlite3 with very similar code as r and python (
        7 proc freq
        8 proc tabulate
        9 data step then transpose
          Kurt Bresmer

          https://tinyurl.com/cukhvaxe
          https://github.com/rogerjdeangelis/utl-transpose-pivot-with-just-classification-varaiables-and-no-value-variable

    Kurt Bresmer
    https://communities.sas.com/t5/user/viewprofilepage/user-id/11562

    https://communities.sas.com/t5/SAS-Programming/Transpose/m-p/761563#M241007

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    */ /**************************************************************************************************************************/
    */ /* SD1.HAVE total obs=4
    */ /*
    */ /*  Obs    ID    FRUIT
    */ /*
    */ /*   1     1     apple
    */ /*   2     1     orange
    */ /*   3     2     apple
    */ /*   4     3     apple
    */ /**************************************************************************************************************************/

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input id $ fruit $;
    datalines;
    1 apple
    1 orange
    2 apple
    3 apple
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* SD1.HAVE total obs=4                                                                                                   */
    /*                                                                                                                        */
    /*  Obs    ID    FRUIT                                                                                                    */
    /*                                                                                                                        */
    /*   1     1     apple                                                                                                    */
    /*   2     1     orange                                                                                                   */
    /*   3     2     apple                                                                                                    */
    /*   4     3     apple                                                                                                    */
    /**************************************************************************************************************************/

    /*
    / |  _ __  _ __ ___   ___    ___ ___  _ __ _ __ ___  ___ _ __
    | | | `_ \| `__/ _ \ / __|  / __/ _ \| `__| `__/ _ \/ __| `_ \
    | | | |_) | | | (_) | (__  | (_| (_) | |  | | |  __/\__ \ |_) |
    |_| | .__/|_|  \___/ \___|  \___\___/|_|  |_|  \___||___/ .__/
        |_|                                                 |_|
    */

    ods exclude all;
    Ods Output Observed=want;
    proc corresp data=sd1.have dim=1 observed missing all print=both;
    tables id, fruit;
    run;quit;
    ods select all;

    /**************************************************************************************************************************/
    /* WANT total obs=4                                                                                                       */
    /*                                                                                                                        */
    /*  LABEL    APPLE    ORANGE    SUM                                                                                       */
    /*                                                                                                                        */
    /*   1         1         1       2                                                                                        */
    /*   2         1         0       1                                                                                        */
    /*   3         1         0       1                                                                                        */
    /*   Sum       3         1       4                                                                                        */
    /**************************************************************************************************************************/

    /*___                                     _
    |___ \   _ __  _ __ ___   ___   ___  __ _| |
      __) | | `_ \| `__/ _ \ / __| / __|/ _` | |
     / __/  | |_) | | | (_) | (__  \__ \ (_| | |
    |_____| | .__/|_|  \___/ \___| |___/\__, |_|
            |_|                            |_|
    */

    /*---WITHOUT ARRAYS ----*/

    proc sql;
      select
       id
      ,max(case when fruit="apple"  then 1 else 0 end) as apple
      ,max(case when fruit="orange" then 1 else 0 end) as orange
     from
       sd1.have
     group
       by id
    ;quit;


    /*---WITH ARRAYS    ----*/

    %array(_fruits,values=%utl_concat(sd1.have,var=fruit,unique=Y));

    %put &=_fruitsn;  /* 2      */
    %put &=_fruits1;  /* apple  */
    %put &=_fruits2;  /* orange */

    proc sql;
      create
        table want
      select
       id
      ,%do_over(_fruits,phrase=%str(
       max(case when fruit="?" then 1 else 0 end) as ?)
        ,between=comma)
     from
       sd1.have
     group
       by id
    ;quit;

    /*____                                                    _
    |___ /   _ __  _ __ ___   ___   _ __ ___ _ __   ___  _ __| |_
      |_ \  | `_ \| `__/ _ \ / __| | `__/ _ \ `_ \ / _ \| `__| __|
     ___) | | |_) | | | (_) | (__  | | |  __/ |_) | (_) | |  | |_
    |____/  | .__/|_|  \___/ \___| |_|  \___| .__/ \___/|_|   \__|
            |_|                             |_|
    */

    /*---- NOTE NO HARDCODING ----*/

    options missing='0';
    proc report data=sd1.have out=want(drop=_BREAK_);
     cols id fruit;
    define id / group;
    define fruit /n across ;
    run;quit;

    /*---- rename cols

    %array(_fruits,values=%utl_concat(sd1.have,var=fruit,unique=Y));

    %put &=_fruitsn;  /* 2      */
    %put &=_fruits1;  /* apple  */
    %put &=_fruits2;  /* orange */

    %array(_cols,values=%utl_varlist(want,keep=_C:));

    %put &=_colsn;  /*   2  */
    %put &=_cols1;  /* _C2_ */
    %put &=_cols2;  /* _C3_ */

    proc datasets lib=work ;
     modify want;
     rename %do_over(_cols _fruits,phrase=?_cols=?_fruits);
    run;quit;

    /**************************************************************************************************************************/
    /* ID    APPLE    ORANGE   |   Variable    Type    Len                                                                    */
    /*                         |                                                                                              */
    /* 1       1         1     |   ID          Char      8                                                                    */
    /* 2       1         0     |   APPLE       Num       8                                                                    */
    /* 3       1         0     |   ORANGE      Num       8                                                                    */
    /**************************************************************************************************************************/

    /*  _               _
    | || |    ___  __ _| |  _ __
    | || |_  / __|/ _` | | | `__|
    |__   _| \__ \ (_| | | | |
       |_|   |___/\__, |_| |_|
                     |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    have<-read_sas("d:/sd1/have.sas7bdat")
    print(have)
    unq_fruit <- unique(have$FRUIT)
    phrases <- paste(sprintf(
        ",max(case when fruit='%1$s' then 1 else 0 end) as %1$s"
         ,unq_fruit
        ),collapse=' ')
    str(phrases)
    phrases
    want<-sqldf(paste(
      'select ID'
      ,phrases
      ,'from have group by id'
    ))
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /* R                  | SAS                                                                                               */
    /*   ID apple orange  | ROWNAMES    ID    APPLE    ORANGE                                                                 */
    /*                    |                                                                                                   */
    /* 1  1     1      1  |     1       1       1         1                                                                   */
    /* 2  2     1      0  |     2       2       1         0                                                                   */
    /* 3  3     1      0  |     3       3       1         0                                                                   */
    /**************************************************************************************************************************/

    /*___              _               _   _
    | ___|   ___  __ _| |  _ __  _   _| |_| |__   ___  _ __
    |___ \  / __|/ _` | | | `_ \| | | | __| `_ \ / _ \| `_ \
     ___) | \__ \ (_| | | | |_) | |_| | |_| | | | (_) | | | |
    |____/  |___/\__, |_| | .__/ \__, |\__|_| |_|\___/|_| |_|
                    |_|   |_|    |___/
    */

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_pythonx.py').read());
    have,meta = ps.read_sas7bdat('d:/sd1/have.sas7bdat');
    want=pdsql('''
        select
         id
        ,max(case when fruit="apple"  then 1 else 0 end) as apple
        ,max(case when fruit="orange" then 1 else 0 end) as orange
       from
         have
       group
         by id
       ''')
    print(want);
    fn_tosas9x(want,outlib='d:/sd1/',outdsn='pywant',timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;

    /**************************************************************************************************************************/
    /* R                  | SAS                                                                                               */
    /*   ID APPLE ORANGE  | ROWNAMES    ID    APPLE    ORANGE                                                                 */
    /*                    |                                                                                                   */
    /* 0  1     1      1  |     1       1       1         1                                                                   */
    /* 1  2     1      0  |     2       2       1         0                                                                   */
    /* 2  3     1      0  |     3       3       1         0                                                                   */
    /**************************************************************************************************************************/

    /*__                             __
     / /_    _ __  _ __ ___   ___   / _|_ __ ___  __ _
    | `_ \  | `_ \| `__/ _ \ / __| | |_| `__/ _ \/ _` |
    | (_) | | |_) | | | (_) | (__  |  _| | |  __/ (_| |
     \___/  | .__/|_|  \___/ \___| |_| |_|  \___|\__, |
            |_|                                     |_|
    */


    %utlfkil(d:/txt/frq.txt);

    ods _all_ close;
    ods listing file="d:/txt/frq.txt";
    options  FORMCHAR='|';
    ods noproctitle;
    proc freq data=sd1.have;
    tables id*fruit/nopercent
      norow nocol nocum;
    run;quit;
    options  FORMCHAR='|----|+|---+=|-/\<>*' ;
    ods listing;

    /*----
    d:/txt/frq.txt

    ID|APPLE|ORANGE

    Table of ID by FRUIT

    ID        FRUIT

    Frequency|apple   |orange  |  Total

    1        |      1 |      1 |      2

    2        |      1 |      0 |      1

    3        |      1 |      0 |      1

    Total           3        1        4
    ----*/

    proc datasets lib=work nodetails nolist;
     delete want;
    run;quit;

    /*---- delete all sasmac# where # is a number will not de  ----*/
    %deleteSasmacN();

    %symdel inp  / nowarn;
    run;quit;

    /*---- you get warning that &nob is not resolved but it is ----*/
    options missing='.';
    data _null_;
      infile "d:/txt/frq.txt";
      input;
      if _infile_=:"Freq" then do;
         _infile_=translate(_infile_,' ','|');
         call symputx('inp',_infile_);
         rc=dosubl(resolve("
           data want;
            infile 'd:/txt/frq.txt' dlm='|'  missover;
            input &inp;
            if not missing (input(total,best.));
           run;quit;
         "));
         stop;
      end;
    run;quit;


    /**************************************************************************************************************************/
    /* Note frequency is the class variablr ID                                                                                */
    /*                                                                                                                        */
    /* WORK.WANT total obs=3                                                                                                  */
    /*                                                                                                                        */
    /*  FREQUENCY    APPLE    ORANGE    TOTAL                                                                                 */
    /*                                                                                                                        */
    /*      1          1         1        2                                                                                   */
    /*      2          1         0        1                                                                                   */
    /*      3          1         0        1                                                                                   */
    /**************************************************************************************************************************/

    /*___    _        _           _       _
     ( _ )  | |_ __ _| |__  _   _| | __ _| |_ ___
     / _ \  | __/ _` | `_ \| | | | |/ _` | __/ _ \
    | (_) | | || (_| | |_) | |_| | | (_| | ||  __/
     \___/   \__\__,_|_.__/ \__,_|_|\__,_|\__\___|

    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input id $ fruit $;
    datalines;
    1 apple
    1 orange
    2 apple
    3 apple
    ;;;;
    run;quit;


    title;
    ods listing close;
    ods html close;
    ods listing
      file="d:/txt/tab.txt";

    options
       missing=0
       formchar="|"
       ls=255;

    proc tabulate
      data=sd1.have;
      class id fruit;
       table
          id="",fruit=""*n="" /rts=5 box="ID" ;
    run;quit;
    ods listing;

    /*----
    d:/txt/tab.txt

    ,ID ,   apple    ,   orange   ,

    ,1  ,        1.00,        1.00,

    ,2  ,        1.00,           0,

    ,3  ,        1.00,           0,
    ----*/

    %utl_close;

    proc datasets lib=work nodetails nolist;
     delete want;
    run;quit;

    /*---- delete all sasmac# where # is a number will not delete sasmacr  ----*/
    %deleteSasmacN;

    %symdel inp / nowarn;

    data _null_;
       infile 'd:/txt/tab.txt' firstobs=2 obs=2;
       input;
       call symputx('inp',translate(_infile_,' ','|'));
       rc=dosubl(resolve("
         data want ;
           infile 'd:/txt/tab.txt' dlm='|' missover firstobs=4;
           input &inp ;
           if not missing(ID);
         run;quit;
       "));
    run;quit;


    /*___                        _       _            _              _
     / _ \   ___  __ _ ___    __| | __ _| |_ __ _ ___| |_ ___ _ __  | |_ _ __ __ _ _ __  ___ _ __   ___  ___  ___
    | (_) | / __|/ _` / __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \ | __| `__/ _` | `_ \/ __| `_ \ / _ \/ __|/ _ \
     \__, | \__ \ (_| \__ \ | (_| | (_| | || (_| \__ \ ||  __/ |_) || |_| | | (_| | | | \__ \ |_) | (_) \__ \  __/
       /_/  |___/\__,_|___/  \__,_|\__,_|\__\__,_|___/\__\___| .__/  \__|_|  \__,_|_| |_|___/ .__/ \___/|___/\___|
                                                             |_|                            |_|
    */


    data have;
    input id $ fruit $;
    datalines;
    1 apple
    1 orange
    2 apple
    3 apple
    ;

    data pretrans;
    set have;
    val = 1;
    run;

    proc transpose
      data=pretrans
      out=want (drop=_name_)
    ;
    by id;
    id fruit;
    var val;
    run;

    /**************************************************************************************************************************/
    /* WORK.WANT total obs=3                                                                                                  */
    /*                                                                                                                        */
    /*   ID    APPLE    ORANGE                                                                                                */
    /*                                                                                                                        */
    /*   1       1         1                                                                                                  */
    /*   2       1         0                                                                                                  */
    /*   3       1         0                                                                                                  */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
