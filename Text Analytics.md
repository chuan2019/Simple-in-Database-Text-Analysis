<center>
	<font size="20" color="blue"><strong>Project Report: Simple In-Database Text Analytics</a></strong></font><br><br>
	<font size="10" color="blue"><font style="font-style:italic;">Chuan Zhang</font> (2014)</font><br><br>
</center>
<hr>

<p font size="10">
   In this independent simple text analytics project, I demonstrates how to implement some basic <strong>relational algebra</strong>
   and <strong>matrix multiplication</strong> with SQL queries; how to measure the <strong>similarity between two documents</strong> based on
   <font style = "font-style:italic;">term frequency</font>; and how to search keywords.
</p>

<h3> Problem 1: Inspecting the Reuters Dataset; Basic Relational Algebra</h3>

(a) <strong>select</strong>: Write a query that is equivalent to the relational algebra expression:

   <center><strong><code>&sigma;<sub>96_txt_acq</sub>(frequency)</code></strong></center><br>

   <pre>
      <font color="red">sqlite3 reuters.db</font>
      sqlite> -- inspecting data model
      sqlite> <font color="red">.schema</font>
      <font color="blue">
      CREATE TABLE Frequency (
      docid VARCHAR(255),
      term VARCHAR(255),
      count int,
      PRIMARY KEY(docid, term));
      </font>
      sqlite> -- count the number of the records in the database satisfying the relational algebra expression
      sqlite> <font color="red"><strong>SELECT COUNT(*) FROM Frequency WHERE docid == "96_txt_acq";</strong></font>
      <font color="blue">40</font>
      sqlite> -- extract all the records in the database satisfying the relational algebra expression
      sqlite> <font color="red"><strong>SELECT * FROM Frequency WHERE docid == "96_txt_acq";</strong></font>
      <font color="blue">
      96_txt_acq|100|1
      96_txt_acq|110000|1
      96_txt_acq|17|1
      96_txt_acq|19|1
      ...
      96_txt_acq|washington|1
      96_txt_acq|york|1
      </font>
   </pre>

(b) <strong>select project</strong>: Write a SQL statement that is equivalent to the following relational algebra expression:

      <center><strong><code>
      &Pi;<sub>term</sub>(&sigma;<sub>docid=10398_txt_earn and count=1</sub>(frequency))
      </code></strong></center>

   <pre>
      sqlite> -- count the number of the terms appearing in the document "10398_txt_earn" only once
      sqlite> <font color="red"><strong>SELECT COUNT(term) FROM Frequency WHERE docid == "10398_txt_earn" AND count == 1;</strong></font>
      110
      sqlite> -- extract the terms appearing in the document "10398_txt_earn" only once
      sqlite> <font color="red"><strong>SELECT term FROM Frequency WHERE docid == "10398_txt_earn" AND count == 1;</strong></font>
      <font color="blue">
      100
      11340
      12
      ...
      will
      without
      world
      </font>
   </pre>

(c) <strong>union</strong>: Write a SQL statement that is equivalent to the following relational algebra expression.

   <center><strong><code>
      &Pi;<sub>term</sub>(&sigma;<sub>docid=10398_txt_earn and count=1</sub>(frequency))
      U
      &Pi;<sub>term</sub>(&sigma;<sub>docid=925_txt_trade and count=1</sub>(frequency))
   </code></strong></center>

   <pre>
      sqlite> <font color="red"><strong>SELECT term FROM Frequency WHERE docid =="10398_txt_earn" AND count == 1</strong></font>
         ...> <font color="red"><strong>UNION </strong></font>
         ...> <font color="red"><strong>SELECT term FROM Frequency WHERE docid == "925_txt_trade" AND count == 1;</strong></font>
      <font color="blue">
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
      </font>
      sqlite> <font color="red"><strong>SELECT COUNT(*) FROM (SELECT term FROM Frequency</strong></font>
         ...> <font color="red"><strong>WHERE docid =="10398_txt_earn" AND count == 1</strong></font>
         ...> <font color="red"><strong>UNION</strong></font>
         ...> <font color="red"><strong>SELECT term FROM Frequency WHERE docid =="925_txt_trade" AND count == 1);</strong></font>
      <font color="blue">
      324
      </font>
   </pre>

(d) <strong>count</strong>: Write a SQL statement to count the number of documents containing the word "parliament".

   <pre>
      sqlite> <font color="red"><strong>SELECT COUNT(DISTINCT docid) FROM Frequency WHERE term == "parliament";</strong></font>
      <font color="blue">
      15
      </font>
   </pre>

(e) <strong>big documents</strong>: Write a SQL statement to find all documents that have more than 300 total terms, including duplicate terms.

   <pre>
      sqlite> <font color="red"><strong>SELECT COUNT(DISTINCT docid) FROM</strong></font>
         ...> <font color="red"><strong>(SELECT docid, SUM(count) AS sum FROM Frequency</strong></font>
         ...> <font color="red"><strong>GROUP BY docid HAVING sum > 300);</strong></font>
      <font color="blue">
      107
      </font>
   </pre>

(f) <strong>two words</strong>: Write a SQL statement to count the number of unique documents that contain both the word
   '<code>transactions</code>' and the word '<code>world</code>'.
   <pre>
      sqlite> <font color="red"><strong>SELECT COUNT(DISTINCT docid) FROM (SELECT docid FROM Frequency</strong></font>
         ...> <font color="red"><strong>WHERE term == "transactions" INTERSECT SELECT docid</strong></font>
         ...> <font color="red"><strong>FROM Frequency WHERE term == "world");</strong></font>
      <font color="blue">
      3
      </font>
   </pre>


<h3>Problem 2: Matrix Multiplication in SQL</h3>

(g) <strong>multiply</strong>: Express A*B as a SQL query, where the two matrices are given in the database <code>matrix.db</code> as follows.

   <pre>
      A =
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

   <pre>
      sqlite> <font color="red"><strong>SELECT SUM(prod) FROM (</strong></font>
         ...> <font color="red"><strong>       SELECT a.value * b.value AS prod FROM A a, B b</strong></font>
         ...> <font color="red"><strong>       WHERE a.col_num == b.row_num AND a.row_num == 2 AND b.col_num == 3</strong></font>
         ...> <font color="red"><strong>);</strong></font>
      <font color="blue">
      2874
      </font>
   </pre>
OR:
   <pre>
      sqlite> <font color="red"><strong>SELECT SUM(prod) FROM (</strong></font>
         ...> <font color="red"><strong>       SELECT a.value * b.value AS prod FROM A a</strong></font>
         ...> <font color="red"><strong>          JOIN B b ON a.col_num == b.row_num AND a.row_num == 2</strong></font>
         ...> <font color="red"><strong>          AND b.col_num == 3</strong></font>
         ...> <font color="red"><strong>);</strong></font>
      <font color="blue">
      2874
      </font>
   </pre>


<h3>Problem 3: Working with a Term-Document Matrix</h3>

(h) <strong>similarity matrix</strong>: Write a query to compute the similarity matrix <code>DD<sup>T</sup></code>.
      The query could take some time to run if you compute the entire result. But notice that you don't need
      to compute the similarity of both <code>(doc1, doc2)</code> and <code>(doc2, doc1)</code> -- they are the same, since similarity
      is symmetric. If you wish, you can avoid this wasted work by adding a condition of the form <code>a.docid <
      b.docid</code> to your query. (But the query still won't return immediately if you try to compute every result
      -- don't expect otherwise.)

   <pre>
      sqlite> -- Find the common terms shared in two documents
      
      sqlite> <font color="red"><strong>SELECT a.* FROM Frequency a, Frequency b</strong></font>
         ...> <font color="red"><strong>WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"</strong></font>
         ...> <font color="red"><strong>AND a.term == b.term;</strong></font>
      <font color="blue">
      10080_txt_crude | april | 1
      10080_txt_crude | ended | 1
      10080_txt_crude | inc   | 1
      10080_txt_crude | mln   | 2
      10080_txt_crude | net   | 1
      10080_txt_crude | profit| 1
      10080_txt_crude | reuter| 1
      10080_txt_crude | six   | 1
      </font>
      sqlite> <font color="red"><strong>SELECT b.* FROM Frequency a, Frequency b</strong></font>
         ...> <font color="red"><strong>WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"</strong></font>
         ...> <font color="red"><strong>AND a.term == b.term;</strong></font>
      <font color="blue">
      17035_txt_earn  | april | 2
      17035_txt_earn  | ended | 1
      17035_txt_earn  | inc   | 1
      17035_txt_earn  | mln   | 3
      17035_txt_earn  | net   | 3
      17035_txt_earn  | profit| 4
      17035_txt_earn  | reuter| 1
      17035_txt_earn  | six   | 1
      </font>
   </pre>

   Based on these observation, we have that the similarity between the two documents,
   <code>"10080_txt_crude"</code> and <code>"17035_txt_earn"</code>, can be measured as follows:

   <pre>
      sqlite> <font color="red"><strong>SELECT SUM(a.count * b.count) FROM Frequency a, Frequency b</strong></font>
         ...> <font color="red"><strong>WHERE a.docid == "10080_txt_crude" AND b.docid == "17035_txt_earn"</strong></font>
         ...> <font color="red"><strong>AND a.term == b.term;</strong></font>
      <font color="blue">
      19
      </font>
   </pre>

(i) <strong>keyword search</strong>: Find the best matching document to the keyword query
"washington taxes treasury". You can add this set of keywords to the document corpus with
a union of scalar queries:
   <pre>
      SELECT * FROM frequency
      UNION
      SELECT 'q' as docid, 'washington' as term, 1 as count
      UNION
      SELECT 'q' as docid, 'taxes' as term, 1 as count
      UNION
      SELECT 'q' as docid, 'treasury' as term, 1 as count
   </pre>

   Then, compute the similarity matrix again, but filter for only
similarities involving the "query document": <code>docid = 'q'</code>. Consider 
creating a view of this new corpus to simplify things.

   <pre>
      sqlite> <font color="red"><strong>CREATE VIEW Keywords AS</strong></font>
         ...> <font color="red"><strong>SELECT * FROM Frequency</strong></font>
         ...> <font color="red"><strong>UNION</strong></font>
         ...> <font color="red"><strong>SELECT 'q' as docid, 'washington' as term, 1 as count</strong></font>
         ...> <font color="red"><strong>UNION</strong></font>
         ...> <font color="red"><strong>SELECT 'q' as docid, 'taxes' as term, 1 as count</strong></font>
         ...> <font color="red"><strong>UNION</strong></font>
         ...> <font color="red"><strong>SELECT 'q' as docid, 'treasury' as term, 1 as count;</strong></font>
      sqlite> <font color="red"><strong>SELECT * FROM (</strong></font>
         ...> <font color="red"><strong>       SELECT b.docid AS docid, SUM(a.count*b.count) AS count</strong></font>
         ...> <font color="red"><strong>       FROM Keywords a, Keywords b</strong></font>
         ...> <font color="red"><strong>       WHERE a.docid == 'q' AND a.term == b.term</strong></font>
         ...> <font color="red"><strong>       GROUP BY b.docid)</strong></font>
         ...> <font color="red"><strong>WHERE  count == (SELECT MAX(count) FROM (</strong></font>
         ...> <font color="red"><strong>       SELECT SUM(a.count*b.count) AS count</strong></font>
         ...> <font color="red"><strong>       FROM Keywords a, Keywords b</strong></font>
         ...> <font color="red"><strong>       WHERE a.docid == 'q' AND a.term == b.term</strong></font>
         ...> <font color="red"><strong>       GROUP BY b.docid));</strong></font>
      <font color="blue">
      16094_txt_trade | 6
      16357_txt_trade | 6
      19775_txt_interest | 6
      </font>
      -- So the highest score in the similarity list is 6.
   </pre>
