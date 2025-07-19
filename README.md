# utl-insights-on-sas-macros-and-macro-dosubl
Insights on sas macros and macro dosubl
    %let pgm=utl-insights-on-sas-macros-and-macro-dosubl;

    %stop_submission;

    Insights on sas macros and macro dosubl

    CONTENTS

        1 More flexible macro arguments (example simplifying sql inserts)
          By
            Bartosz Jablonski
            yabwon@gmail.com

        2 Create macro function mean
          Inspired by
          Bartosz Jablonski
          yabwon@gmail.com
          Beter example is the  %GenerateOneLiners() macro in Barts baseplus package

        3 Passing mutiple macro variables from back to the calling scope.

    github
    https://tinyurl.com/nabe6tvt
    https://github.com/rogerjdeangelis/utl-insights-on-sas-macros-and-macro-dosubl

    Bartosz, Base Plus Package
    https://tinyurl.com/5n748mya
    https://github.com/SASPAC/baseplus/blob/main/baseplus.md#generateoneliners-macro-


    Feature (of parmbuff and syspbuff macros)

    Complex Code Support       Handles commas/parentheses natively
    Return Code Useful         Accessible to caller
    Parameter Flexibility      Any input via &syspbuff
    Quoting Safety             Avoids over-quoting


    1 MORE FLEXIBLE MACRO ARGUMENTS
    ===============================

    PROBLEM: SIMPLIFY SQL CREATE TABLE INSERT

    %macro sqlins/parmbuff;
        insert into rows values &syspbuff;
    %mend sqlins;

    proc sql;
      create
        table rows (x numeric, y numeric);
    %sqlins(1,2);
    %sqlins(3,4);
    %sqlins(5,6);
    %sqlins(7,8);
    ;quit;

    /**************************************************************************************************************************/
    /*   Obs    X    Y                                                                                                        */
    /*                                                                                                                        */
    /*  1     1    2                                                                                                          */
    /*  2     3    4                                                                                                          */
    /*  3     5    6                                                                                                          */
    /*  4     7    8                                                                                                          */
    */ /***********************************************************************************************************************/

    2 CREATE MACRO FUNCTION MEAN
    =============================

    /*--- just for development and testing ---*/
    %symdel x pbuff mean avg /nowarn;
    %deletesasmacn;

    %macro func/parmbuff;
        %let pbuff=&syspbuff;
        %dosubl(
          data _null_;
            x=&pbuff;
            call symputx('x',x);
          run;quit;
        )
        &x
    %mend func;

    %deletesasmacn;

    %let avg = %func(mean(1,2,3));
    %put &=avg;

    /*--- just for development and testing ---*/
    %deletesasmacn;

    %put %func(mean(1,2,3));

    %put The mean of 1,2,3 is %func(mean(1,2,3));

    /**************************************************************************************************************************/
    /* The mean of 1,2,3 is 2                                                                                               */
    /**************************************************************************************************************************/

    LOG

    MLOGIC(DOSUBL):  %LET (variable name is RC)
    SYMBOLGEN:  Macro variable ARG resolves to data _null_; x=(mean(1,2,3)); call symputx('x',x); run;quit;
    NOTE: DATA statement used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              324.31k
          OS Memory           23540.00k
          Timestamp           07/19/2022 01:15:38 PM
          Step Count                        162  Switch Count  0


    MLOGIC(DOSUBL):  Ending execution.
    SYMBOLGEN:  Macro variable X resolves to 2
    MLOGIC(FUNC):  Ending execution.
    2
    713   %put The mean of 1,2,3 is %func(mean(1,2,3));
    MLOGIC(FUNC):  Beginning execution.
    MLOGIC(FUNC):  %LET (variable name is PBUFF)
    SYMBOLGEN:  Macro variable SYSPBUFF resolves to (mean(1,2,3))
    SYMBOLGEN:  Macro variable PBUFF resolves to (mean(1,2,3))
    SYMBOLGEN:  Macro variable ARG resolves to data _null_; x=(mean(1,2,3)); call symputx('x',x); run;quit;
    NOTE: DATA statement used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              324.31k
          OS Memory           23540.00k
          Timestamp           07/19/2022 01:15:38 PM
          Step Count                        163  Switch Count  0


    MLOGIC(DOSUBL):  Ending execution.
    SYMBOLGEN:  Macro variable X resolves to 2
    MLOGIC(FUNC):  Ending execution.
    The mean of 1,2,3 is     2



    2 ANOTHER WAY TO EXPORT MUTIPLE MACRO VARIABLES BACK TO THE CALLING ENVIRONMENT WITHOUT GLOBAL STATEMENT.
    =========================================================================================================

    FAILS
    =====

    %symdel name sex/nowarn;

    %macro parent();

      data _null_;

        set sashelp.class(obs=1);
        call symputx('name',name);
        call symputx('sex',sex);

        /*---- leave off run statement. Place run in calling program scope ----*/

    %mend parent;

    %put &=name;
    %put &=sex;

    LOG

    1079  %put &=name;
    WARNING: Apparent symbolic reference NAME not resolved.
    name
    1080  %put &=sex;
    WARNING: Apparent symbolic reference SEX not resolved.
    sex


    WORKS  (USEFUL IF YOU HAVE A LARGE NUMBER OF VARIABLES?)
    =========================================================
    run;quit;
    %parent;
    %put &=name;
    %put &=sex;

    LOG

    1153  %put &=name;
    SYMBOLGEN:  Macro variable NAME resolves to Alfred
    NAME=Alfred
    1154  %put &=sex;
    SYMBOLGEN:  Macro variable SEX resolves to M
    SEX=M

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
