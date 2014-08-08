Simple-in-Database-Text-Analysis
================================

SQL queries for implementing <strong>relational algebra</strong> expressions, <strong>sparse matrix multiplication</strong> (usually used for huge sparse similarity matrices), and simple <strong>in-database text analysis</strong>.

For details please refer to the file: TextAnalytics.html

Database:

reuters.db: a single table with three fields,  <code>docid</code> is a document identifier corresponding to a particular file of text, <code>term</code> is an English word,
and <code>count</code> is the number of the occurrences of the term within the document indicated by <code>docid</code>
    <pre>
        frequency(docid, term, count)
    </pre>

 matrix.db: two matrices A and B represented as follows
    <pre>
        A(row_num, col_num, value)
        B(row_num, col_num, value)
    </pre>