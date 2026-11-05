# c-utl-jnr-Jenner-Analytics-a-simple-flexible-macro-function-for-curated-lists-of-column-names
Jennner Analytics: a macro function to create a macro string containig a curated list of table column names
    /* --
    c:/utl/jnr-Jenner-Analytics-a-simple-flexible-macro-function-for-curated-lists-of-column-names.sas
    ---*/
    Jennner Analytics: a macro function to create a macro string containig a curated list of table column names

    Too long to post, see
    https://github.com/rogerjdeangelis/jnr-Jenner-Analytics-a-simple-flexible-macro-function-for-curated-lists-of-column-names

    SOAPBOX ON

    Jenner macros turn code generation into a powerful engine for **automation**,
    dynamically generating and executing workflows that combine Jenner, SQL, R, Python, PowerShell, and more.

    A single macro can generate Jenner code, code in another language,
    or a coordinated sequence spanning multiple languages—expanding
    what a workflow can accomplish without forcing every task into one language.

    Simple macro syntax, an extensive library of built-in macro functions,
    and access to functions through SYSFUNC make even sophisticated code generation
    straightforward. The result: less repetitive coding, more flexible workflows,
    and greater power to automate complex tasks.

    In addtion the resulting code  is often simpler, more flexible, and maintainable.

    In the macro below PROC SQL could be replaced with any language or SQL dialect.

    SOAPBOX OFF

    CONTENTS (A macro function to create a macro string containig a curated list of table column names)

       1  HOW IT WORKS (probably does not work with nliteral variable names)
          IMPORTANT (ORDER OF FILTER)
          Prxmatch filter comes first then keep then drop
       2  USAGE SIX TEST CASES
       3  INPUT
       4  PROCESS
       5  LIST LOG
       6  MACRO

    /******************************************************************************************************************/
    /* 1  HOW IT WORKS (probably does not work with nliteral variable names)                                          */
    /******************************************************************************************************************/

     THIS IS HOW IT WORKS (This is all the code. Note less Klingon macro triggers)

      /*--- create table with keep and drop applied        ---*/
      /*--- probably should not do both keep and drop      ---*/
      data __keepdrop__; /*--- reduced meta data use later ---*/
        set &libname..&memname(keep=&keep);
          drop &drop;
          stop;
      run;

      proc sql;
      select &function(name) into :varlist separated by &od
      from
      ( select name,varnum
        from sashelp.vcolumn
        where upcase(libname) = upcase("&libname")
        and upcase(memname) = upcase("&memname")
        and prxmatch(&prx, name) > 0    /*--- prxmatch first ---*/
        union corr                      /*--- union corr removes duplicate names        ---*/
        select name, varnum             /*--- __keepdrop__ has reduced column meta data ---*/
        from sashelp.vcolumn
        where upcase(libname) = upcase("work")
        and upcase(memname) = upcase("__keepdrop__") /*--- __keepdrop__ curated has meta data ---*/
      )
      order by varnum

    /******************************************************************************************************************/
    /* 2  USAGE SIX TEST CASES                                                                                        */
    /******************************************************************************************************************/

    1 Age Height Name Sex Weight
      %put %getvars(libname=jenhelp,memname=class);

    2 Height Name Weight
      %put %getvars(keep=name height weight);

    3 Height Sex Weight
                    /* FIRST        SECOND                      THIRD    */
      %put %getvars(prx="/weight/i",keep=name sex height weight,drop=name);

    4 "Name" "Sex" "Age" "Weight"
      %put %jnr_varlist(libname=sashelp,memname=class,keep=name age sex,function=quote,prx="/weight/i");

    5 name,sex,age,weight *** function lowcase ***
      %put %jnr_varlist(libname=sashelp,memname=class,prx='/weight/i',keep=name age sex,function=lowcase,od=",");

    6 Macro with data step and sql run at macro time
      data bmi;
       set jenhelp.class(
         obs=2         /*---     FIRST                     SECOND      THIRD  ---*/
         keep=%getvars(prx="/eight/i",keep=name sex height weight,drop=name));
         /*--- sex height weight ---*/
       bmi=703*weight/height**2;
      run;

      proc print data=bmi;
      run;

      Sex  Height  Weight            bmi

      M      61.2      95  17.8310158486
      F      63.8     105  18.1344031603

    /******************************************************************************************************************/
    /* 3  INPUT SASHELP.CLASS  or JENHELP.CLASS                                                                       */
    /******************************************************************************************************************/

    proc print data=jenhelp.class(obs=2);
    run;

    Obs  Name     Sex  Age  Height  Weight
      1  Amir     M     13    61.2      95
      2  Bethany  F     14    63.8     105

    /******************************************************************************************************************/
    /* 4  PROCESS                                                                                                     */
    /******************************************************************************************************************/

      data bmi;
       set jenhelp.class(
         obs=2         /*---     FIRST                     SECOND      THIRD  ---*/
         keep=%getvars(prx="/eight/i",keep=name sex height weight,drop=name));
         /*--- sex height weight ---*/
       bmi=703*weight/height**2;
      run;

      proc print data=bmi;
      run;

    /******************************************************************************************************************/
    /* 5  LIST LOG                                                                                                    */
    /******************************************************************************************************************/

    LIST:   04NOV2026:15:06:04
    ==========================

      Obs  Sex  Height  Weight            bmi

        1  M      61.2      95  17.8310158486
        2  F      63.8     105  18.1344031


    LOG:    04NOV2026:15:06:04
    ==========================

    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.110 (build v1.5.110+13530c7ccc.20261030T030012Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: DATA _null_

    autexec started.

    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:\wpswrkx.
    NOTE: DATA workx.class

    NOTE: Processing inline DATALINES (19 lines)

    NOTE: Read 19 rows from DATALINES.
    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_

    LOG:    04NOV2026:15:06:04
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_

    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
        1  data bmi;
        2    set jenhelp.class(
        3      obs=2         /*---     FIRST                     SECOND      THIRD  ---*/
        4      keep=%getvars(prx="/eight/i",keep=name sex height weight,drop=name));
        5      /*--- sex height weight ---*/
        6    bmi=703*weight/height**2;
        7   run;
        8
        9   proc print data=bmi;
       10   run;

    NOTE: Read 10 rows from c:/jnr/runsas_selection.sas.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_

    autoexec completed    04NOV2026:15:06:04

    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA work.__keepdrop__


    NOTE: Read 1 rows from sashelp.class.
    NOTE: The data set WORK.__KEEPDROP__ has 0 observations and 3 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: PROC PRINT data=__keepdrop__


    No observations in dataset.

    Completed
    NOTE: PROC SQL

    NOTE: Macro variable VARLIST = Height Sex Weight
    NOTE: PROC SQL statement used.
    Completed
    NOTE:VARLIST=Height Sex Weight
    NOTE: DATA work.bmi


    NOTE: Read 2 rows from jenhelp.class.
    NOTE: The data set WORK.BMI has 2 observations and 4 variables.
    NOTE: DATA elapsed:
      wall  0.01 seconds
      cpu   0.02 seconds
    NOTE: PROC PRINT data=bmi

    NOTE: PROC PRINT completed: 2 observations printed, 4 variables


    /******************************************************************************************************************/
    /* 6  MACRO                                                                                                       */
    /******************************************************************************************************************/

    /*--- Save in your autocall library ---*/

    filename ft15f001 "c:/otojnr/jnr_varlist.sas";
    parmcards4;
    %macro jnr_varlist(
        libname=jenhelp
       ,memname=class
       ,function=  /*--- many functions possible like lowcase or quote ---*/
       ,od=" "
       ,prx="/(?!)/"  /*-- "/^/" always match '/(?!)/' never match ---*/
       ,keep=_all_
       ,drop=
     ) / des="Populate global VARLIST matching column names";

    /*---

    IMPORTANT (ORDER OF FILTER)

    Prxmatch filter comes first then keep then drop

    Cases

    1 Age Height Name Sex Weight
      %put %getvars(libname=jenhelp,memname=class);

    2 Height Name Weight
      %put %getvars(keep=name height weight);

    3 Height Sex Weight
      %put %getvars(keep=name sex height weight,drop=name,prx="/weight/i");

    4 "Name" "Sex" "Age" "Weight"
      %put %jnr_varlist(libname=sashelp,memname=class,keep=name age sex,function=quote,prx="/weight/i");

    5 name,sex,age,weight *** function lowcase ***
      %put %jnr_varlist(libname=sashelp,memname=class,keep=name age sex,function=lowcase,prx='/weight/i',od=",");

    ---*/

    %global varlist;
    %let varlist=;

    %dosubl('
    data __keepdrop__;
        set &libname..&memname(keep=&keep);
        drop &drop;
        stop;
    run;
    proc print data=__keepdrop__;
    run;
    ');


    %dosubl('
    proc sql;
      select &function(name) into :varlist separated by &od
      from
      ( select name,varnum
        from sashelp.vcolumn
        where upcase(libname) = upcase("&libname")
        and upcase(memname) = upcase("&memname")
        and prxmatch(&prx, name) > 0
        union corr
        select name, varnum
        from sashelp.vcolumn
        where upcase(libname) = upcase("work")
        and upcase(memname) = upcase("__keepdrop__")
      )
      order by varnum
    ;quit;');

    %put NOTE: &=varlist;

    &varlist

    %mend jnr_varlist;
    ;;;;
    run;

    /*--- END --- */
