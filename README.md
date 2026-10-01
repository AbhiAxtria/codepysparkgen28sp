MATCH (jg:JobGroup)-[r:HAS_JOB]->(j:Job)-[js:CONTAINS]->(s:Step)-[sp:EXECUTES]->(rs:Resource) where jg.name = "AUTO-RED Jobs" RETURN jg,j,r,s,js,sp,rs LIMIT 50;
 
MATCH (jg:JobGroup)-[r:HAS_JOB]->(j:Job)-[js:CONTAINS]->(s:Step)-[sp:EXECUTES]->(rs:Resource)-[inv:INVOKES]->(sql:SqlFile) where jg.name = "AUTO-RED Jobs" RETURN jg,j,r,s,js,sp,rs,inv,sql LIMIT 100;
 
