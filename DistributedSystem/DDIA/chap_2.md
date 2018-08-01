### Relational database vs document database
1. document database is more closed to application model. 

  The application needs to transform table data to data model before it can consume. But in document database, most of the time, the document is ready for use by application.


2. one to many, many to one, many to many relationship.

  For one to many relationship, document model can handle it naturally because document itself is tree like structure. For example each "resume" document will have "first name", "last name", "education", "address" etc. 

  The problem comes when we want to change the name of "education(univercity)". Because we need to change all records whose "education" field satisfy certain critieria.

  But in relational database, "education" field will be modeled as many to one relationship and thus normalized. We can easily search and update one raw to make the change.

3. Join

  Relational database is optimized for Joins. Document database's support for join is weak.

4. document database can handle greater scalability, including high write throughput

(need to understand the reason). There are also difference in fault-tolerance (chap 5) and concurrency (chap 7).

5. schema flexibility.

  alter table (adding a new column with NULL value) is fast in most of relational database, except for MySQL. But update data in table is very slow and both operations require downtime.

  document database can change add a field to document easily.

5. locality

  Document has performance advantage of storage locality because all data stored in nearly same place.

  But the performance goes down when document become huge and application only use a small part of document each time.
