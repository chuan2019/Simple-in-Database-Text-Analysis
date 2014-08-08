
<!-- saved from url=(0091)file:///C:/Users/Chuan/Documents/GitHub/Simple-in-Database-Text-Analysis/TextAnalytics.html -->
<html><head><meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Independent Data Science Course Project: Text Analytics</title>
</head><body bgcolor="white"><font face="Times" size="4">

<center>
	<font size="6" color="blue"><strong>Project Report: Simple In-Database Text Analytics</strong></font><br><br>
	<font size="4" color="blue"><font style="font-style:italic;">Chuan Zhang</font> (2014)</font><br><br>
</center>
<hr>

<p font="" size="4">
   In this independent simple text analytics project, I demonstrates how to implement some basic <strong>relational algebra</strong>
   and <strong>matrix multiplication</strong> with SQL queries; how to measure the <strong>similarity between two documents</strong> based on
   <font style="font-style:italic;">term frequency</font>; and how to search keywords.
</p>

<h3> Problem 1: Inspecting the Reuters Dataset; Basic Relational Algebra</h3>

(a) <strong>select</strong>: Write a query that is equivalent to the relational algebra expression:

   <center><strong><code>&#963;<sub>96_txt_acq</sub>(frequency)</code></strong></center><br>

   <pre>      sqlite3 reuters.db
      sqlite&gt; -- inspecting data model
      sqlite&gt; .schema
      CREATE TABLE Frequency (
      docid VARCHAR(255),
      term VARCHAR(255),
      count int,
      PRIMARY KEY(docid, term));
      sqlite&gt; -- count the number of the records in the database satisfying the relational algebra expression
      sqlite&gt; SELECT COUNT(*) FROM Frequency WHERE docid == "96_txt_acq";
      40
      sqlite&gt; -- extract all the records in the database satisfying the relational algebra expression
      sqlite&gt; SELECT * FROM Frequency WHERE docid == "96_txt_acq";
      96_txt_acq|100|1
      96_txt_acq|110000|1
      96_txt_acq|17|1
      96_txt_acq|19|1
      ...
      96_txt_acq|washington|1
      96_txt_acq|york|1
   </pre><br>

(b) <strong>select project</strong>: Write a SQL statement that is equivalent to the following relational algebra expression:

      <center><strong><code>&#928;<sub>term</sub>(&#963;<sub>docid=10398_txt_earn and count=1</sub>(frequency))</code></strong></center>

   <pre>      sqlite&gt; -- count the number of the terms appearing in the document "10398_txt_earn" only once
      sqlite&gt; SELECT COUNT(term) FROM Frequency WHERE docid == "10398_txt_earn" AND count == 1;
      110
      sqlite&gt; -- extract the terms appearing in the document "10398_txt_earn" only once
      sqlite&gt; SELECT term FROM Frequency WHERE docid == "10398_txt_earn" AND count == 1;
      100
      11340
      12
      ...
      will
      without
      world
   </pre>

(c) <strong>union</strong>: Write a SQL statement that is equivalent to the following relational algebra expression.

   <center><strong><code>
      &#928;<sub>term</sub>(&#963;<sub>docid=10398_txt_earn and count=1</sub>(frequency))
      U
      &#928;<sub>term</sub>(&#963;<sub>docid=925_txt_trade and count=1</sub>(frequency))
   </code></strong></center>

   <pre>      sqlite&gt; SELECT term FROM Frequency WHERE docid =="10398_txt_earn" AND count == 1
         ...&gt; UNION
         ...&gt; SELECT term FROM Frequency WHERE docid == "925_txt_trade" and count == 1;
      100
      11340
      12
      1204
      13
      ...
      without
      won
      world
      worldwide
      year
      sqlite&gt; SELECT COUNT(*) FROM (SELECT term FROM Frequency WHERE docid =="10398_txt_earn" AND count == 1
         ...&gt; UNION
         ...&gt; SELECT term FROM Frequency WHERE docid =="925_txt_trade" AND count == 1);
      324
   </pre>

(d) <strong>count</strong>: Write a SQL statement to count the number of documents containing the word "parliament".

   <pre>      sqlite&gt; select count(distinct docid) from frequency where term == "parliament";
      15
   </pre>

(e) <strong>big documents</strong>: Write a SQL statement to find all documents that have more than 300 total terms, including duplicate terms.

   <pre>      sqlite&gt; select count(distinct docid) from (select docid, sum(count) as sum from frequency group by docid having sum &gt; 300);
      107
   </pre>

(f) <strong>two words</strong>: Write a SQL statement to count the number of unique documents that contain both the word
   '<code>transactions</code>' and the word '<code>world</code>'.
   <pre>      sqlite&gt; select count(distinct docid) from (select docid from frequency
         ...&gt; WHERE term == "transactions" INTERSECT SELECT docid
         ...&gt; FROM Frequency WHERE term == "world");
      3
   </pre>


<h3>Problem 2: Matrix Multiplication in SQL</h3>

(g) <strong>multiply</strong>: Express A*B as a SQL query, where the two matrices are given in the database <code>matrix.db</code> as follows.

   <pre>      A =
      {{ 0, 0, 0,55,78},
       {19, 0,21, 3,81},
       { 0,48,50, 1, 0},
       { 0, 0,33, 0,67},
       {95, 0, 0, 0,31}}

      B =
      {{ 0,73, 0, 0,42},
       { 0, 0,82, 0, 0},
       {83,13, 0,57, 0},
       {48,85,18,24, 0},
       {98, 7, 0, 0, 3}}
   </pre>

   <pre>      sqlite&gt; SELECT SUM(prod) FROM (
         ...&gt;        SELECT a.value * b.value AS prod FROM A a, B b
         ...&gt;        WHERE a.col_num == b.row_num AND a.row_num == 2 AND b.col_num == 3
         ...&gt; );
      2874
   </pre>
OR:
   <pre>      sqlite&gt; SELECT SUM(prod) FROM (
         ...&gt;        SELECT a.value * b.value AS prod FROM A a
         ...&gt;           JOIN B b ON a.col_num == b.row_num AND a.row_num == 2
         ...&gt;           AND b.col_num == 3
         ...&gt; );
      2874
   </pre>


<h3>Problem 3: Working with a Term-Document Matrix</h3>

(h) <strong>similarity matrix</strong>: Write a query to compute the similarity matrix <code>DD<sup>T</sup>.
      The query could take some time to run if you compute the entire result. But notice that you don't need
      to compute the similarity of both (doc1, doc2) and (doc2, doc1) -- they are the same, since similarity
      is symmetric. If you wish, you can avoid this wasted work by adding a condition of the form a.docid &lt;
      b.docid to your query. (But the query still won't return immediately if you try to compute every result
      -- don't expect otherwise.)

   <pre>      sqlite&gt; -- Find the common terms shared in two documents
      sqlite&gt; SELECT a.* FROM Frequency a, Frequency b
         ...&gt; WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"
         ...&gt; AND a.term == b.term;
      10080_txt_crude | april | 1
      10080_txt_crude | ended | 1
      10080_txt_crude | inc   | 1
      10080_txt_crude | mln   | 2
      10080_txt_crude | net   | 1
      10080_txt_crude | profit| 1
      10080_txt_crude | reuter| 1
      10080_txt_crude | six   | 1
      sqlite&gt; SELECT b.* FROM Frequency a, Frequency b
         ...&gt; WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"
         ...&gt; AND a.term == b.term;
      17035_txt_earn  | april | 2
      17035_txt_earn  | ended | 1
      17035_txt_earn  | inc   | 1
      17035_txt_earn  | mln   | 3
      17035_txt_earn  | net   | 3
      17035_txt_earn  | profit| 4
      17035_txt_earn  | reuter| 1
      17035_txt_earn  | six   | 1
   </pre>

   Based on these observation, we have that the similarity between the two documents,
   "10080_txt_crude" and "17035_txt_earn", can be measured as follows:

   <pre>      sqlite&gt; SELECT sum(a.count * b.count) FROM Frequency a, Frequency b
         ...&gt; WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"
         ...&gt; AND a.term == b.term;
      19
   </pre>

(i) <strong>keyword search</strong>: Find the best matching document to the keyword query
"washington taxes treasury". You can add this set of keywords to the document corpus with
a union of scalar queries:
   <pre>      SELECT * FROM frequency
      UNION
      SELECT 'q' as docid, 'washington' as term, 1 as count
      UNION
      SELECT 'q' as docid, 'taxes' as term, 1 as count
      UNION
      SELECT 'q' as docid, 'treasury' as term, 1 as count
   </pre>

   Then, compute the similarity matrix again, but filter for only
similarities involving the "query document": docid = 'q'. Consider 
creating a view of this new corpus to simplify things.

   <pre>      sqlite&gt; CREATE VIEW Keywords AS
         ...&gt; SELECT * FROM Frequency
         ...&gt; UNION
         ...&gt; SELECT 'q' as docid, 'washington' as term, 1 as count
         ...&gt; UNION
         ...&gt; SELECT 'q' as docid, 'taxes' as term, 1 as count
         ...&gt; UNION
         ...&gt; SELECT 'q' as docid, 'treasury' as term, 1 as count;
      sqlite&gt; SELECT * FROM (
         ...&gt;        SELECT b.docid AS docid, SUM(a.count*b.count) AS count
         ...&gt;        FROM Keywords a, Keywords b
         ...&gt;        WHERE a.docid == 'q' AND a.term == b.term
         ...&gt;        GROUP BY b.docid)
         ...&gt; WHERE  count == (SELECT MAX(count) FROM (
         ...&gt;        SELECT SUM(a.count*b.count) AS count
         ...&gt;        FROM Keywords a, Keywords b
         ...&gt;        WHERE a.docid == 'q' AND a.term == b.term
         ...&gt;        GROUP BY b.docid));
      16094_txt_trade | 6
      16357_txt_trade | 6
      19775_txt_interest | 6
      -- So the highest score in the similarity list is 6.</pre></code></font></body></html>