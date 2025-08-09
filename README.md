# utl-count-ids-by-group-but-drop-any-later-record-that-contain-an-earlier-id
    %let pgm=utl-count-ids-by-group-but-drop-any-later-record-that-contain-an-earlier-id;

    %stop_submission;

    Count ids by group but drop any later record that contain an earlier id

    PROBLEM
    =======
        INPUT                                          OUTPUT
         ID        DATE                                   DATE       CNT

        1234    2020-02-07                             2020-02-07     3
        4567    2020-02-07                             2020-02-14     2
        9870    2020-02-07

        1234    2020-02-14  drop because
                            1234 occurs earlier
        7654    2020-02-14
        3421    2020-02-14

    CONTENTS

        1 sas sql with partiioning
        2 r sql with partitioning
        3 python with partitioning
        4 nice sort freq by
          Nat Wooding <nathani@VERIZON.NET>

    github
    https://tinyurl.com/y8d693v2
    https://github.com/rogerjdeangelis/utl-count-ids-by-group-but-drop-any-later-record-that-contain-an-earlier-id

    sas communities
    https://tinyurl.com/ye9b45en
    https://communities.sas.com/t5/SAS-Programming/SAS-cumulative-count-by-unique-ID-and-date/td-p/653043

    /**************************************************************************************************************************/
    /* INPUT                        |  PROCESS                                               | OUTPUT                         */
    /* =====                        |  =======                                               | ======                         */
    /*  ID     DATE                 | 1 SAS SQL WITH PARTIIONING                             | SAS                            */
    /*                              | ==========================                             |    DATE       CNT              */
    /* 1234 2020-02-07              |                                                        |                                */
    /* 4567 2020-02-07              | proc sql;                                              | 2020-02-07     3               */
    /* 9870 2020-02-07              |   select                                               | 2020-02-14     2               */
    /*                              |     date                                               |                                */
    /* 1234 2020-02-14 drop         |    ,count(*) as cnt                                    |                                */
    /*                              |   from                                                 |                                */
    /* 7654 2020-02-14              |     %sqlpartitionx(sd1.have,by=id)                     |                                */
    /* 3421 2020-02-14              |   where                                                |                                */
    /*                              |     partition=1                                        |                                */
    /* options                      |   group                                                |                                */
    /*  validvarname=upcase;        |     by date                                            |                                */
    /* libname sd1 "d:/sd1";        | ;quit;                                                 |                                */
    /* data sd1.have;               |                                                        |                                */
    /*   input id date $10.;        |-----------------------------------------------------------------------------------------*/
    /* cards4;                      | 2 R SQL WITH PARTITIONING                              | R                              */
    /* 1234 2020-02-07              | =========================                              |         date cnt               */
    /* 4567 2020-02-07              |                                                        |                                */
    /* 9870 2020-02-07              | proc datasets lib=sd1 nolist nodetails;                | 1 2020-02-07   3               */
    /* 1234 2020-02-14              |  delete want;                                          | 2 2020-02-14   2               */
    /* 7654 2020-02-14              | run;quit;                                              |                                */
    /* 3421 2020-02-14              |                                                        | SAS                            */
    /* ;;;;                         | %utl_rbeginx;                                          |    DATE       CNT              */
    /* run;quit;                    | parmcards4;                                            |                                */
    /*                              | library(haven)                                         | 2020-02-07     3               */
    /*                              | library(sqldf)                                         | 2020-02-14     2               */
    /*                              | source("c:/oto/fn_tosas9x.R")                          |                                */
    /*                              | options(sqldf.dll = "d:/dll/sqlean.dll")               |                                */
    /*                              | have<-read_sas("d:/sd1/have.sas7bdat")                 |                                */
    /*                              | print(have)                                            |                                */
    /*                              | want<-sqldf('                                          |                                */
    /*                              |  select                                                |                                */
    /*                              |    date                                                |                                */
    /*                              |   ,count(date) as cnt                                  |                                */
    /*                              |  from                                                  |                                */
    /*                              |     (select                                            |                                */
    /*                              |         *                                              |                                */
    /*                              |        ,row_number()                                   |                                */
    /*                              |      OVER                                              |                                */
    /*                              |         (PARTITION BY ID) as partition                 |                                */
    /*                              |      from                                              |                                */
    /*                              |         have )                                         |                                */
    /*                              |  where                                                 |                                */
    /*                              |     partition=1                                        |                                */
    /*                              |  group                                                 |                                */
    /*                              |     by date                                            |                                */
    /*                              | ')                                                     |                                */
    /*                              | want                                                   |                                */
    /*                              | fn_tosas9x(                                            |                                */
    /*                              |       inp    = want                                    |                                */
    /*                              |      ,outlib ="d:/sd1/"                                |                                */
    /*                              |      ,outdsn ="want"                                   |                                */
    /*                              |      )                                                 |                                */
    /*                              | ;;;;                                                   |                                */
    /*                              | %utl_rendx;                                            |                                */
    /*                              |                                                        |                                */
    /*                              | proc print data=sd1.want;                              |                                */
    /*                              | run;quit;                                              |                                */
    /*                              |                                                        |                                */
    /*                              |-----------------------------------------------------------------------------------------*/
    /*                              |                                                        |                                */
    /*                              | 3 PYTHON WITH PARTITIONING                             | python                         */
    /*                              | ==========================                             |           DATE  cnt            */
    /*                              |                                                        |                                */
    /*                              | proc datasets lib=sd1 nolist nodetails;                | 0  2020-02-07    3             */
    /*                              |  delete pywant;                                        | 1  2020-02-14    2             */
    /*                              | run;quit;                                              |                                */
    /*                              |                                                        | SAS                            */
    /*                              | %utl_pybeginx;                                         |    DATE       CNT              */
    /*                              | parmcards4;                                            |                                */
    /*                              | exec(open('c:/oto/fn_pythonx.py').read());             | 2020-02-07     3               */
    /*                              | have,meta = \                                          | 2020-02-14     2               */
    /*                              |  ps.read_sas7bdat('d:/sd1/have.sas7bdat');             |                                */
    /*                              | want=pdsql('''                                         |                                */
    /*                              |   select                                               |                                */
    /*                              |     date                                               |                                */
    /*                              |    ,count(date) as cnt                                 |                                */
    /*                              |   from                                                 |                                */
    /*                              |      (select                                           |                                */
    /*                              |          *                                             |                                */
    /*                              |         ,row_number()                                  |                                */
    /*                              |       over                                             |                                */
    /*                              |          (partition by id) as partition                |                                */
    /*                              |       from                                             |                                */
    /*                              |          have )                                        |                                */
    /*                              |   where                                                |                                */
    /*                              |      partition=1                                       |                                */
    /*                              |   group                                                |                                */
    /*                              |      by date                                           |                                */
    /*                              |    ''');                                               |                                */
    /*                              | print(want);                                           |                                */
    /*                              | fn_tosas9x(want                                        |                                */
    /*                              |   ,outlib='d:/sd1/'                                    |                                */
    /*                              |   ,outdsn='pywant'                                     |                                */
    /*                              |   ,timeest=3);                                         |                                */
    /*                              | ;;;;                                                   |                                */
    /*                              | %utl_pyendx;                                           |                                */
    /*                              |                                                        |                                */
    /*                              | proc print data=sd1.pywant;                            |                                */
    /*                              | run;quit;                                              |                                */
    /*                              |                                                        |                                */
    /*                              |-----------------------------------------------------------------------------------------*/
    /*                              |                                                        |                                */
    /*                              | 4 SAS SORT FREQ                                        | SAS                            */
    /*                              | ==========================                             |    DATE       CNT              */
    /*                              |                                                        |                                */
    /*                              | Proc sort data =sd1.have nodupkey;                     | 2020-02-07     3               */
    /*                              |  by id;                                                | 2020-02-14     2               */
    /*                              | run;quit;                                              |                                */
    /*                              |                                                        |                                */
    /*                              | proc freq noprint;                                     |                                */
    /*                              | table date /                                           |                                */
    /*                              |  Out= want (keep = date count);                        |                                */
    /*                              | run;                                                   |                                */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options
     validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
      input id date $10.;
    cards4;
    1234 2020-02-07
    4567 2020-02-07
    9870 2020-02-07
    1234 2020-02-14
    7654 2020-02-14
    3421 2020-02-14
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*  ID        DATE                                                                                                        */
    /*                                                                                                                        */
    /* 1234    2020-02-07                                                                                                     */
    /* 4567    2020-02-07                                                                                                     */
    /* 9870    2020-02-07                                                                                                     */
    /* 1234    2020-02-14                                                                                                     */
    /* 7654    2020-02-14                                                                                                     */
    /* 3421    2020-02-14                                                                                                     */
    /**************************************************************************************************************************/

    /*                             _
    / |  ___  __ _ ___   ___  __ _| |
    | | / __|/ _` / __| / __|/ _` | |
    | | \__ \ (_| \__ \ \__ \ (_| | |
    |_| |___/\__,_|___/ |___/\__, |_|
                                |_|
    */

    proc sql;
      select
        date
       ,count(*) as cnt
      from
        %sqlpartitionx(sd1.have,by=id)
      where
        partition=1
      group
        by date
    ;quit;

    /**************************************************************************************************************************/
    /* DATE             cnt                                                                                                   */
    /* 2020-02-07         3                                                                                                   */
    /* 2020-02-14         2                                                                                                   */
    /**************************************************************************************************************************/

    /*___                     _
    |___ \   _ __   ___  __ _| |
      __) | | `__| / __|/ _` | |
     / __/  | |    \__ \ (_| | |
    |_____| |_|    |___/\__, |_|
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
    want<-sqldf('
     select
       date
      ,count(date) as cnt
     from
        (select
            *
           ,row_number()
         OVER
            (PARTITION BY ID) as partition
         from
            have )
     where
        partition=1
     group
        by date
    ')
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
    /*  r                 | sas                                                                                               */
    /*          DATE cnt  |    DATE       CNT                                                                                 */
    /*                    |                                                                                                   */
    /*  1 2020-02-07   3  | 2020-02-07     3                                                                                  */
    /*  2 2020-02-14   2  | 2020-02-14     2                                                                                  */
    /**************************************************************************************************************************/

    /*____               _   _                             _
    |___ /   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) | | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
            |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_pythonx.py').read());
    have,meta = \
     ps.read_sas7bdat('d:/sd1/have.sas7bdat');
    want=pdsql('''
      select
        date
       ,count(date) as cnt
      from
         (select
             *
            ,row_number()
          over
             (partition by id) as partition
          from
             have )
      where
         partition=1
      group
         by date
       ''');
    print(want);
    fn_tosas9x(want
      ,outlib='d:/sd1/'
      ,outdsn='pywant'
      ,timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;


    /**************************************************************************************************************************/
    /*  python            | sas                                                                                               */
    /*          DATE cnt  |    DATE       CNT                                                                                 */
    /*                    |                                                                                                   */
    /*  0 2020-02-07   3  | 2020-02-07     3                                                                                  */
    /*  1 2020-02-14   2  | 2020-02-14     2                                                                                  */
    /**************************************************************************************************************************/

    /*  _                                    _      __
    | || |    ___  __ _ ___   ___  ___  _ __| |_   / _|_ __ ___  __ _
    | || |_  / __|/ _` / __| / __|/ _ \| `__| __| | |_| `__/ _ \/ _` |
    |__   _| \__ \ (_| \__ \ \__ \ (_) | |  | |_  |  _| | |  __/ (_| |
       |_|   |___/\__,_|___/ |___/\___/|_|   \__| |_| |_|  \___|\__, |
                                                                   |_|
    */

    Proc sort data =sd1.have nodupkey;
     by id;
    run;quit;

    proc freq noprint;
    table date /
     Out= want (keep = date count);
    run;

    /**************************************************************************************************************************/
    /*|    DATE       COUNT                                                                                                   */
    /*|                                                                                                                       */
    /*| 2020-02-07      3                                                                                                     */
    /*| 2020-02-14      2                                                                                                     */
    /**************************************************************************************************************************/


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

