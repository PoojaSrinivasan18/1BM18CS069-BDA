cqlsh> create keyspace "Employee" with replication={
   ... 'class':'SimpleStrategy','replication_factor':1};
cqlsh> describe keyspaces

"Employee"  system_auth         system_schema  system_views         
system      system_distributed  system_traces  system_virtual_schema

cqlsh> USE "Employee";

cqlsh:Employee> create table employee_info( Emp_Id int PRIMARY KEY, Emp_Name text, Designation text, Date_Of_joining timestamp, Salary int, Dept_Name text);
cqlsh:Employee> describe employee_info;

CREATE TABLE "Employee".employee_info (
    emp_id int PRIMARY KEY,
    date_of_joining timestamp,
    dept_name text,
    designation text,
    emp_name text,
    salary int
) WITH additional_write_policy = '99p'
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND cdc = false
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '16', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND default_time_to_live = 0
    AND extensions = {}
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair = 'BLOCKING'
    AND speculative_retry = '99p';

cqlsh:Employee> BEGIN BATCH
            ... INSERT INTO employee_info(Emp_Id,Emp_Name,Designation,Date_Of_joining,Salary,Dept_Name) VALUES(121,'Rose','Software Developer','2021-03-16',80000,'IT')
            ... INSERT INTO employee_info(Emp_Id,Emp_Name,Designation,Date_Of_joining,Salary,Dept_Name) VALUES(122,'Jane','Software Tester','2020-04-16',70000,'IT')
            ... INSERT INTO employee_info(Emp_Id,Emp_Name,Designation,Date_Of_joining,Salary,Dept_Name) VALUES(123,'John','Manager','2020-05-25',65000,'Sales')
            ... APPLY BATCH;
cqlsh:Employee> SELECT * FROM employee_info;

 emp_id | date_of_joining                 | dept_name | designation        | emp_name | salary
--------+---------------------------------+-----------+--------------------+----------+--------
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |        IT | Software Developer |     Rose |  80000

(3 rows)

cqlsh:Employee> UPDATE employee_info SET Emp_Name='Rosy', Dept_Name='Software'  WHERE Emp_Id=121;
cqlsh:Employee> SELECT * FROM employee_info;

 emp_id | date_of_joining                 | dept_name | designation        | emp_name | salary
--------+---------------------------------+-----------+--------------------+----------+--------
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |  Software | Software Developer |     Rosy |  80000

(3 rows)

cqlsh:Employee> SELECT * from employee_info ORDER BY Salary;
InvalidRequest: Error from server: code=2200 [Invalid query] message="ORDER BY is only supported when the partition key is restricted by an EQ or an IN."

cqlsh:Employee> ALTER TABLE employee_info
            ... ADD projects set<text>;
cqlsh:Employee> SELECT * FROM employee_info;

 emp_id | date_of_joining                 | dept_name | designation        | emp_name | projects | salary
--------+---------------------------------+-----------+--------------------+----------+----------+--------
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John |     null |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |     null |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |  Software | Software Developer |     Rosy |     null |  80000

(3 rows)


cqlsh:Employee> UPDATE employee_info SET projects={'sales improvement proj','ad management sys'} WHERE Emp_ID=123;
cqlsh:Employee> UPDATE employee_info SET projects={'company website','Employee management app'} WHERE Emp_ID=121; 
cqlsh:Employee> UPDATE employee_info SET projects={'company website testing'} WHERE Emp_ID=122;                  
cqlsh:Employee> SELECT * FROM employee_info;
 emp_id | date_of_joining                 | dept_name | designation        | emp_name | projects                                        | salary
--------+---------------------------------+-----------+--------------------+----------+-------------------------------------------------+--------
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John | {'ad management sys', 'sales improvement proj'} |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |                     {'company website testing'} |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |  Software | Software Developer |     Rosy |  {'Employee management app', 'company website'} |  80000

(3 rows)


cqlsh:Employee> BEGIN BATCH
            ... INSERT INTO employee_info(Emp_Id,Emp_Name,Designation,Date_Of_joining,Salary,Dept_Name,projects) VALUES(124,'Joe','Intern','2021-03-20',25000,'IT',{'LMS'}) USING TTL 15
            ... APPLY BATCH;

cqlsh:Employee> SELECT * FROM employee_info;
 emp_id | date_of_joining                 | dept_name | designation        | emp_name | projects                                        | salary
--------+---------------------------------+-----------+--------------------+----------+-------------------------------------------------+--------
    124 | 2021-03-20 00:00:00.000000+0000 |        IT |             Intern |      Joe |                                         {'LMS'} |  25000
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John | {'ad management sys', 'sales improvement proj'} |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |                     {'company website testing'} |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |  Software | Software Developer |     Rosy |  {'Employee management app', 'company website'} |  80000

(4 rows)

cqlsh:Employee> SELECT * FROM employee_info;
 emp_id | date_of_joining                 | dept_name | designation        | emp_name | projects                                        | salary
--------+---------------------------------+-----------+--------------------+----------+-------------------------------------------------+--------
    123 | 2020-05-25 00:00:00.000000+0000 |     Sales |            Manager |     John | {'ad management sys', 'sales improvement proj'} |  65000
    122 | 2020-04-16 00:00:00.000000+0000 |        IT |    Software Tester |     Jane |                     {'company website testing'} |  70000
    121 | 2021-03-16 00:00:00.000000+0000 |  Software | Software Developer |     Rosy |  {'Employee management app', 'company website'} |  80000

(3 rows)
