DBMS_REDEFINITION

Ensure Table Can Be Redefined:

DECLARE v_can_redf PLS_INTEGER; BEGIN v_can_redf :=DBMS_REDEFINITION.CAN_REDEF_TABLE(USERNAME, 'YOUR_TABLE_NAME', DBMS_REDEFINITION.CONS_USE_ROWID); IF v_can_redf = 0 THEN DBMS_OUTPUT.PUT_LINE('Table can be redefined'); ELSE DBMS_OUTPUT.PUT_LINE('Table cannot be redefined'); END IF; END;/

Create an Interim Table: 

This table should ideally be a replica of the original table (but might contain structural changes we want to apply).

1. Create an empty table
2. Create table with two years of data
    1. timestamp between 1628596800 and snapshot timestamp
    2. move data between snapshot timestamp and newly created data later.

CREATE TABLE interim_table AS SELECT * FROM your_table WHERE 1=0;

Start the Redefinition Process:

BEGIN DBMS_REDEFINITION.START_REDEF_TABLE(USERNAME, 'YOUR_TABLE_NAME', 'INTERIM_TABLE');END; /

Apply DML Changes If Needed 

Making changes to the interim table directly is generally not recommended.

Synchronize the Interim Table: 

To ensure the interim table captures all changes made to the original table after starting the redefinition:

BEGIN DBMS_REDEFINITION.SYNC_INTERIM_TABLE(USERNAME, 'YOUR_TABLE_NAME', 'INTERIM_TABLE'); END; /

Copy Dependent Objects: 

Transfer indexes, triggers, grants, and constraints from the original table to the interim table.

DECLARE v_errors PLS_INTEGER; BEGIN v_errors :=DBMS_REDEFINITION.COPY_TABLE_DEPENDENTS(USERNAME, 'YOUR_TABLE_NAME', 'INTERIM_TABLE', DBMS_REDEFINITION.CONS_ORIG_PARAMS); DBMS_OUTPUT.PUT_LINE('Errors: ' ||TO_CHAR(v_errors)); END; /

Finish the Redefinition:

BEGIN DBMS_REDEFINITION.FINISH_REDEF_TABLE(USERNAME, 'YOUR_TABLE_NAME', 'INTERIM_TABLE'); END; /

Cleanup: 

After we have confident in the redefinition's success, drop the interim table (which now contains the old data/structure).

DROP TABLE interim_table;

NOTE:

* Backup Data: Before starting the redefinition.
* Review Locks:
    * The redefinition process will take a short lock on the table during the final swap.
    * Usually how long will it be?
* Table synchronization:
    * Can we keep interim_table after finish the redefinition and then migrate the old data?
    * In this case we will do a empty table swap and then migrate last two years’ data.




-- Define and create the interim table (must be done before the script execution).
CREATE TABLE SV_SESSION_INTERIM AS 
SELECT * FROM SV_SESSION 
WHERE ss_updated_time >= ADD_MONTHS(SYSDATE, -24)
AND ss_updated_time<{timestamp};

-- 1. Start the table redefinition:
BEGIN
   DBMS_REDEFINITION.START_REDEF_TABLE('RW_OWNER', 'SV_SESSION', 'SV_SESSION_INTERIM');
END;
/

-- 2. Create a Procedure for Synchronization:
CREATE OR REPLACE PROCEDURE sync_interim_table AS 
BEGIN
   DBMS_REDEFINITION.SYNC_INTERIM_TABLE('RW_OWNER', 'SV_SESSION', 'SV_SESSION_INTERIM');
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error encountered during synchronization: ' || SQLERRM);
      RAISE;
END;
/
SHOW ERRORS;

-- 3. Schedule the Procedure using DBMS_SCHEDULER:
DECLARE
  job_created BOOLEAN := FALSE;
BEGIN
   FOR c IN (SELECT 1 FROM DBA_SCHEDULER_JOBS WHERE JOB_NAME = 'SYNC_JOB') LOOP
      job_created := TRUE;
   END LOOP;
   
   IF NOT job_created THEN
      DBMS_SCHEDULER.create_job (
         job_name        => 'SYNC_JOB',
         job_type        => 'PLSQL_BLOCK',
         job_action      => 'BEGIN sync_interim_table; END;',
         start_date      => SYSTIMESTAMP,
         repeat_interval => 'FREQ=MINUTELY; INTERVAL=10', 
         enabled         => TRUE
      );
   END IF;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error encountered during job creation: ' || SQLERRM);
      RAISE;
END;
/

-- 4. (Optionally) perform any transformations or operations on the interim table here.

-- 5. Manual synchronization can be performed if needed (or rely on the scheduled job).
-- BEGIN
--    sync_interim_table;
-- END;
-- /

-- 6. Finish the table redefinition (this would be done manually after validation):
-- BEGIN
--    DBMS_REDEFINITION.FINISH_REDEF_TABLE('RW_OWNER', 'SV_SESSION', 'SV_SESSION_INTERIM');
-- END;
-- /

-- 7. Cleanup (stop and remove the synchronization job after redefinition is finished):
-- BEGIN
--    DBMS_SCHEDULER.stop_job('SYNC_JOB');
--    DBMS_SCHEDULER.drop_job('SYNC_JOB');
-- END;
-- /


