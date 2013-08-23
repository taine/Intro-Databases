Intro-Databases
===============
DTD Exercises

Question 1

Write a DTD for the XML data set. 

<!ELEMENT countries (country*)>
<!ELEMENT country (city*, language*)>
<!ATTLIST  country name CDATA #REQUIRED
                                population CDATA #IMPLIED
                                area CDATA #IMPLIED>

<!ELEMENT city (name, population)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT population (#PCDATA)>
<!ELEMENT language (#PCDATA)>
<!ATTLIST language percentage CDATA #REQUIRED>


Question 2   

<!ELEMENT Course_Catalog (Department*)>
<!ELEMENT Department (Title, Chair, Course*)>
<!ATTLIST Department Code CDATA #REQUIRED>
<!ELEMENT Chair (Professor)>
<!ELEMENT Course (Title, Description*, Instructors, Prerequisites*)>
<!ATTLIST Course Number CDATA #REQUIRED   Enrollment CDATA #IMPLIED>
<!ELEMENT Title (#PCDATA)>
<!ELEMENT Description (#PCDATA)>
<!ELEMENT Instructors (Professor | Lecturer)*>
<!ELEMENT Professor (First_Name, Middle_Initial?, Last_Name)>
<!ELEMENT Lecturer (First_Name, Middle_Initial?, Last_Name)>
<!ELEMENT First_Name (#PCDATA)>
<!ELEMENT Middle_Initial (#PCDATA)>
<!ELEMENT Last_Name (#PCDATA)>
<!ELEMENT Prerequisites (Prereq*)>
<!ELEMENT Prereq (#PCDATA)>

Question 3

<!ELEMENT Course_Catalog (Department*)>
<!ELEMENT Department (Title, Course*,(Lecturer | Professor)*)>
<!ATTLIST Department Code CDATA #REQUIRED
                                      Chair IDREF #REQUIRED>
<!ELEMENT Course (Title, Description*)>
<!ELEMENT Title (#PCDATA)>
<!ELEMENT Description (#PCDATA | Courseref)*>
<!ELEMENT Courseref (#PCDATA)>
<!ATTLIST Courseref Number CDATA #IMPLIED>
<!ATTLIST Course Number CDATA #REQUIRED
	         Prerequisites CDATA #IMPLIED  
	         Instructors  IDREFS #IMPLIED           
                 Enrollment CDATA #IMPLIED>
<!ELEMENT Professor (First_Name, Middle_Initial?, Last_Name)>
<!ATTLIST Professor InstrID ID #REQUIRED>
<!ELEMENT Lecturer (First_Name, Middle_Initial?, Last_Name)>
<!ATTLIST Lecturer InstrID ID #REQUIRED>
<!ELEMENT First_Name (#PCDATA)>
<!ELEMENT Middle_Initial (#PCDATA)>
<!ELEMENT Last_Name (#PCDATA)>

SQL Movie-Rating Query Exercises (core set)

Question 1

Find the titles of all movies directed by Steven Spielberg.

select title
from Movie
where director = 'Steven Spielberg';

Question 2

Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order. 

select distinct year
from Movie, Rating
where Movie.mID = Rating.mID and stars > 3 
order by year;

Question 3

Find the titles of all movies that have no ratings. 

select title from movie
except
select title from movie, rating where Movie.mID = Rating.mID

Question 4

Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date. 

select name 
from Reviewer, Rating
where  Reviewer.rID = Rating.rID and ratingDate is null;

Question 5

Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars. 

select name, title, stars, ratingDate
from Rating, Movie, Reviewer
where Rating.rID = Reviewer.rID
and Rating.mID = Movie.mID
order by name, title, stars;

Question 6

For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie. 

select distinct name, title
from Movie, Reviewer, (select R1.mID, R1.rID from Rating R1, Rating R2, Movie where R1.rID = R2.rID and R1.mID = R2.mID and R1.ratingDate > R2.ratingDate and R1.stars > R2.stars ) Rate
where Movie.mID = Rate.mID and Reviewer.rID = Rate.rID

Question 7

For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title. 

select title, max(stars)
from Movie join Rating using(mID)
group by mID
order by title

Question 8

List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order. 

select title, avg(stars)
from Movie join Rating using(mID)
group by title
order by avg(stars) desc


Question 9

Find the names of all reviewers who have contributed three or more ratings. (As an extra challenge, try writing the query without HAVING or without COUNT.) 

select name
from Reviewer join Rating using(rID)
group by name
having count(stars) > 2


SQL Movie-Rating Query Exercises (challenge-level)


Question 1

For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title. 

Select title, (max(stars)- min(stars)) 'rating spread'
from movie, rating using(mID)
group by title
order by (max(stars)- min(stars)) desc, title


Question 2

Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.) 

select avg(A) - avg(B) 
from (select avg(stars) A
from Rating, Movie using(mID)
where year < 1980
group by title) J,  (select avg(stars) B
from Rating, Movie using(mID)
where year > 1980
group by title) S


Question 3

Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title. (As an extra challenge, try writing the query both with and without COUNT.) 

select Movie.Title, d.director from 
(select title, director
from movie
group by director
having count(director) >1) D, movie using(director)


Question 4

Find the movie(s) with the highest average rating. Return the movie title(s) and average rating. (Hint: This query is more difficult to write in SQLite than other systems; you might think of it as finding the highest average rating and then choosing the movie(s) with that average rating.) 

select title, K.B
from (select avg(stars) B, title
from Rating, Movie using(mID)
group by title) K
where K.B in (select max(B) from (select avg(stars) B, title
from Rating, Movie using(mID)
group by title))


Question 5

Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating. (Hint: This query may be more difficult to write in SQLite than other systems; you might think of it as finding the lowest average rating and then choosing the movie(s) with that average rating.) 

select title, K.B
from (select avg(stars) B, title
from Rating, Movie using(mID)
group by title) K
where K.B in (select min(B) from (select avg(stars) B, title
from Rating, Movie using(mID)
group by title))


Question 6

For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL.

select director, title, max(B)
from
(select max(stars) B, title, director
from Rating, Movie using(mID)
where director <> '<null>'
group by title)
group by director



SQL Movie-Rating Query Exercises (extra practice)


SQL Social-Network Query Exercises (core set)


Question 1

Find the names of all students who are friends with someone named Gabriel.

select name
from Highschooler, Friend, (select ID
from Highschooler
where name = 'Gabriel') G
 where Highschooler.ID = ID1 and G.ID = ID2 


Question 2

For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like. 


Select distinct L.name, L.grade, Highschooler.name, Highschooler.grade
from Highschooler, Likes, 
(select name, grade, ID
from Highschooler, likes
where highschooler.ID = Likes.ID1) L
where Highschooler.ID =Likes.ID2 and L.ID = Likes.ID1 and (L.grade - Highschooler.grade) >= 2

Question 3

For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order. 

select a, b, c, d
from
(select h.name a, h.grade b , h1.name c, h1.grade d, H1.ID e, H.ID f
from Highschooler h, Highschooler H1, Likes L
where h.id = l.id1 and h1.id= L.id2 
Intersect
select h.name a, h.grade b, h1.name c, h1.grade d, H1.ID e, H.ID f
from Highschooler h, Highschooler H1, Likes L
where h1.id = l.id1 and h.id= L.id2
order by a )
where a <= c

Question 4

Find names and grades of students who only have friends in the same grade. Return the result sorted by grade, then by name within each grade. 

select H1.name an, H1.grade ag
from highschooler H1, Highschooler H2, Friend
where H1.ID = Friend.ID1 and H2.ID = Friend.ID2 and h2.grade = h1.grade
except
select H1.name an, H1.grade ag
from highschooler H1, Highschooler H2, Friend
where H1.ID = Friend.ID1 and H2.ID = Friend.ID2 and h2.grade <> h1.grade
order by ag

Question 5

Find the name and grade of all students who are liked by more than one other student. 

select H1.name, H1.grade
from Highschooler H, Highschooler H1, Likes L
where h.ID = L.ID1 and h1.ID =L.ID2 
group by H1.name
having count(*) >1

SQL Social-Network Query Exercises (challenge-level)

Question 1

Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade. 


select name, grade
from Highschooler
except
select name, grade
from highschooler h, likes l
where h.id = l.id1 or h.id = l.id2
order by grade, name

Question 2

For each student A who likes a student B where the two are not friends, find if they have a friend C in common (who can introduce them!). For all such trios, return the name and grade of A, B, and C. 

Select s.name, s.grade, s1.name, s1.grade, s2.name, s2.grade
from
(select a as liker, b as liked
from
(select h.id as a, h1.ID as b
from highschooler h, highschooler h1, likes L
where h.id = l.id1 and h1.id = l.id2
except
select a, b 
from
(select h.id a, h1.ID b
from highschooler h, highschooler h1, likes L
where h.id = l.id1 and h1.id = l.id2
intersect 
select h.id a, h1.ID b
from highschooler h, highschooler h1, friend f
where h.id = f.id1 and h1.id = f.id2))), Friend f1, friend f2, Highschooler s, Highschooler s1, Highschooler s2
where liker = s.id and liked = s1.id and liked = f1.id1 and liker = f2.id1 and s2.id = f2.id2 and s2.id =f1.id2

Question 3

Find the difference between the number of students in the school and the number of different first names. 

select count(*) from (select *
from highschooler
except
select *
from highschooler
group by name)

Question 4

What is the average number of friends per student? (Your result should be just one number.) 

select avg(jello)
from
(select count(h1.id) as jello
from Highschooler h, Highschooler h1, friend f
where f.id1 = h.id and f.id2 = h1.id
group by h.id)
 
Question 5

Find the number of students who are either friends with Cassandra or are friends of friends of Cassandra. Do not count Cassandra, even though technically she is a friend of a friend. 

select count(distinct skinnykid) + count(distinct fatkid)
from
(select skinny.id as SkinnyKid, fat.id as FatKid
from highschooler cass, highschooler Skinny, highschooler Fat, Friend real, friend super
where cass.id = 1709  and cass.id = real.id1 and skinny.id = real.id2 and skinny.id = super.id1 and fat.id = super.id2 and fat.id <>1709)

Question 6

Find the name and grade of the student(s) with the greatest number of friends.

Select hero, HeroLevel
from(select Heroes.name Hero, Heroes.grade HeroLevel, sidekick.name
from highschooler Heroes, highschooler Sidekick, Friend real
where Heroes.id = real.id1 and real.id2 = sidekick.id
group by heroes.id
having count(sidekick.name) = 4)

SQL Social-Network Query Exercises (extra practice)

Question 1

For every situation where student A likes student B, but student B likes a different student C, return the names and grades of A, B, and C. 


Question 2

Find those students for whom all of their friends are in different grades from themselves. Return the students' names and grades. 

Question 3

For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades. 





SQL Social Data 

Highschooler
ID	name	grade
1510	Jordan	9
1689	Gabriel	9
1381	Tiffany	9
1709	Cassandra	9
1101	Haley	10
1782	Andrew	10
1468	Kris	10
1641	Brittany	10
1247	Alexis	11
1316	Austin	11
1911	Gabriel	11
1501	Jessica	11
1304	Jordan	12
1025	John	12
1934	Kyle	12
1661	Logan	12

Friend
ID1	ID2
1510	1381
1510	1689
1689	1709
1381	1247
1709	1247
1689	1782
1782	1468
1782	1316
1782	1304
1468	1101
1468	1641
1101	1641
1247	1911
1247	1501
1911	1501
1501	1934
1316	1934
1934	1304
1304	1661
1661	1025
1381	1510
1689	1510
1709	1689
1247	1381
1247	1709
1782	1689
1468	1782
1316	1782
1304	1782
1101	1468
1641	1468
1641	1101
1911	1247
1501	1247
1501	1911
1934	1501
1934	1316
1304	1934
1661	1304
1025	1661

Likes
ID1	ID2
1689	1709
1709	1689
1782	1709
1911	1247
1247	1468
1641	1468
1316	1304
1501	1934
1934	1501
1025	1101


SQL Movie Data

Movie
mID	title	year	director
101	Gone with the Wind	1939	Victor Fleming
102	Star Wars	1977	George Lucas
103	The Sound of Music	1965	Robert Wise
104	E.T.	1982	Steven Spielberg
105	Titanic	1997	James Cameron
106	Snow White	1937	<null>
107	Avatar	2009	James Cameron
108	Raiders of the Lost Ark	1981	Steven Spielberg

Reviewer
rID	name
201	Sarah Martinez
202	Daniel Lewis
203	Brittany Harris
204	Mike Anderson
205	Chris Jackson
206	Elizabeth Thomas
207	James Cameron
208	Ashley White

Rating
rID	mID	stars	ratingDate
201	101	2	2011-01-22
201	101	4	2011-01-27
202	106	4	<null>
203	103	2	2011-01-20
203	108	4	2011-01-12
203	108	2	2011-01-30
204	101	3	2011-01-09
205	103	3	2011-01-27
205	104	2	2011-01-22
205	108	4	<null>
206	107	3	2011-01-15
206	106	5	2011-01-19
207	107	5	2011-01-20
208	104	3	2011-01-02




SQL Movie-Rating Modification Exercises


Question 1

Add the reviewer Roger Ebert to your database, with an rID of 209.

Insert into Reviewer
Values (209, "Roger Ebert")


Question 2

Insert 5-star ratings by James Cameron for all movies in the database. Leave the review date as NULL.

Insert into Rating 
 select 207, mID, 5, null
 from Movie



Question 3

For all movies that have an average rating of 4 stars or higher, add 25 to the release year. (Update the existing tuples; don't insert new tuples.)

update movie
set year = year + 25
where mID in (select mID
from Rating, movie using(mID)
group by mID
having avg(stars) >= 4)


Question 4

Remove all ratings where the movie's year is before 1970 or after 2000, and the rating is fewer than 4 stars.

Delete From Rating
Where stars < 4 and mID in(select mID from movie, rating using(mID) where year < 1970 or year > 2000) 



SQL Social-Network Modification Exercises


Question 1

It's time for the seniors to graduate. Remove all 12th graders from Highschooler.

delete from Highschooler
where grade = 12

Question 2

If two students A and B are friends, and A likes B but not vice-versa, remove the Likes tuple.

delete from Likes
where id1 in (select f.id1
from friend f, likes l
where f.ID1 = l.ID1 and f.ID2 = l.ID2 and f.id2 not in(select l1.id1 from friend f, likes l1 where f.id2 = l1.id1 and f.id1 = l1.id2))


Question 3

For all cases where A is friends with B, and B is friends with C, add a new friendship for the pair A and C. Do not add duplicate friendships, friendships that already exist, or friendships with oneself.

insert into friend
select f1.id1,f2.id2 from friend f1,friend f2 
where f1.id2=f2.id1 and f1.id1<>f2.id2 except 
select * from friend



XML Course-Catalog XPath and XQuery Exercises (core set)


Question 1

Return all Title elements (of both departments and courses). 

doc("courses.xml")  //Title


Question 2

Return last names of all department chairs. 

doc("courses.xml") /Course_Catalog/Department/Chair/Professor/Last_Name


Question 3

Return titles of courses with enrollment greater than 500. 

doc("courses.xml") /Course_Catalog/Department/Course[@Enrollment > 500]/Title


Question 4

Return titles of departments that have some course that takes "CS106B" as a prerequisite. 

for $b in doc("courses.xml") /Course_Catalog/Department
where $b/Course/Prerequisites/Prereq = "CS106B"
return $b/Title


Question 5

Return last names of all professors or lecturers who use a middle initial. Don't worry about eliminating duplicates. 

doc("courses.xml")//(Professor|Lecturer)[Middle_Initial]/Last_Name


Question 6

Return the count of courses that have a cross-listed course (i.e., that have "Cross-listed" in their description). 

count(doc("courses.xml")/Course_Catalog/Department/Course[contains(Description, "Cross-listed")])


Question 7

Return the average enrollment of all courses in the CS department.

 avg(doc("courses.xml") /Course_Catalog/Department[@Code = "CS"]/Course/@Enrollment)


Question 8

Return last names of instructors teaching at least one course that has "system" in its description and enrollment greater than 100. 

doc("courses.xml")/Course_Catalog/Department/Course[@Enrollment > 100 and contains(Description, "system")]/Instructors/(Professor | Lecturer)/Last_Name




XML Course-Catalog XPath and XQuery Exercises (challenge-level)


Question 1

Return the title of the course with the largest enrollment. 


Question 2

Return course numbers of courses that have the same title as some other course. (Hint: You might want to use the "preceding" and "following" navigation axes for this query, which were not covered in the video or our demo script; they match any preceding or following node, not just siblings.) 


Question 3

Return the number (count) of courses that have no lecturers as instructors. 


Question 4

Return titles of courses taught by the chair of a department. For this question, you may assume that all professors have distinct last names. 


Question 5

Return titles of courses taught by a professor with the last name "Ng" but not by a professor with the last name "Thrun". 


Question 6

Return course numbers of courses that have a course taught by Eric Roberts as a prerequisite. 


Question 7

Create a summary of CS classes: List all CS department courses in order of enrollment. For each course include only its Enrollment (as an attribute) and its Title (as a subelement). 


Question 8

Return a "Professors" element that contains as subelements a listing of all professors in all departments, sorted by last name with each professor appearing once. The "Professor" subelements should have the same structure as in the original data. For this question, you may assume that all professors have distinct last names. Watch out -- the presence/absence of middle initials may require some special handling. 


Question 9

Expanding on the previous question, create an inverted course listing: Return an "Inverted_Course_Catalog" element that contains as subelements professors together with the courses they teach, sorted by last name. You may still assume that all professors have distinct last names. The "Professor" subelements should have the same structure as in the original data, with an additional single "Courses" subelement under Professor, containing a further "Course" subelement for each course number taught by that professor. Professors who do not teach any courses should have no Courses subelement at all. 



XML Course-Catalog XPath and XQuery Exercises (extra practice)


Question 1

Return the course number of the course that is cross-listed as "LING180". 


Question 2

Return course numbers of courses taught by an instructor with first name "Daphne" or "Julie". 


Question 3

Return titles of courses that have both a lecturer and a professor as instructors. Return each title only once. 


XML World-Countries XPath and XQuery Exercises (core set)


Question 1

Return the area of Mongolia. 

doc("countries.xml") /countries/country[@name = 'Mongolia']/data(@area)


Question 2

Return the names of all cities that have the same name as the country in which they are located. 


doc("countries.xml") //city[name = parent::*/@name]/name


Question 3

Return the average population of Russian-speaking countries. 

avg(doc("countries.xml")/countries/country[language= 'Russian']/(@population))


Question 4

Return the names of all countries where over 50% of the population speaks German. (Hint: Depending on your solution, you may want to use ".", which refers to the "current element" within an XPath expression.) 

doc("countries.xml")/countries/country[language[@percentage >50 and .='German']]/data(@name)


Question 5

Return the name of the country with the highest population. (Hint: You may need to explicitly cast population numbers as integers with xs:int() to get the correct answer.) 

doc("countries.xml")//country[(@population) >= max(//country/data(@population))]/data(@name)



XML World-Countries XPath and XQuery Exercises (challenge-level)


Question 1

Return the names of all countries that have at least three cities with population greater than 3 million.




XML Course-Catalog XSLT Exercises (core set)


Question 1

Return a list of department titles. 


 <?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        
                 <xsl:template match="Department/Title">
                <xsl:copy-of select="." />
                </xsl:template>

                  <xsl:template match="text()" />

             </xsl:stylesheet>



Question 2

Return a list of department elements with no attributes and two subelements each: the department title and the entire Chair subelement structure. 

<?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:template match="@*|node()">
        <xsl:apply-templates select="@*|node()"/>
</xsl:template>

<xsl:template match="Department">
   <xsl:copy>
      <xsl:apply-templates />
   </xsl:copy>
</xsl:template>

<xsl:template match="Department/Title">
   <xsl:copy-of select="." />
</xsl:template>

<xsl:template match="Chair">
   <xsl:copy-of select="." />
</xsl:template>
</xsl:stylesheet>



XML Course-Catalog XSLT Exercises (challenge-level)


Question 1

Create a summarized version of the EE part of the course catalog. For each course in EE, return a Course element, with its Number and Title as attributes, its Description as a subelement, and the last name of each instructor as an Instructor subelement. Discard all information about department titles, chairs, enrollment, and prerequisites, as well as all courses in departments other than EE. (Note: To specify quotes within an already-quoted XPath expression, use &quot;.) 

 <?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        
                 <xsl:template match="Department/Title">
                <xsl:copy-of select="." />
                </xsl:template>

                  <xsl:template match="text()" />

             </xsl:stylesheet>


XML Course-Catalog XSLT Exercises (extra practice)


Question 1

Return all courses with enrollment greater than 500. Retain the structure of Course elements from the original data.

Question 2

Delete from the data all courses with enrollment greater than 60, or with no enrollment listed. Otherwise the structure of the data should be the same. 



XML World-Countries XSLT Exercises (core set)


Question 1

Return all countries with population between 9 and 10 million. Retain the structure of country elements from the original data. 

<?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
       
<xsl:template match="//country[@population > 9000000]">
<xsl:copy-of select="."/>
</xsl:template>

<xsl:template match="//country[@population > 10000000]"/>

<xsl:template match="text()" />
    </xsl:stylesheet>


Question 2

Find all country names containing the string "stan"; return each one within a "Stan" element. (Note: To specify quotes within an already-quoted XPath expression, use &quot;.) 

 <?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
       
<xsl:template match="countries/country"> <xsl:if test="contains(@name,'stan')" > <Stan><xsl:value-of select="@name" /></Stan> </xsl:if> </xsl:template>
    </xsl:stylesheet>


XML World-Countries XSLT Exercises (challenge-level)

Question 1

Create a table using HTML constructs that lists all countries that have more than 3 languages. Each row should contain the country name in bold, population, area, and number of languages. Sort the rows in descending order of number of languages. No header is needed for the table, but use <table border="1"> to make it format nicely, should you choose to check your result in a browser. (Hint: You may find the data-type and order attributes of <xsl:sort> to be useful.) 


Question 2

Create an alternate version of the countries database: for each country, include its name and population as sublements, and the number of languages and number of cities as attributes (called "languages" and "cities" respectively). 


XML World-Countries XSLT Exercises (extra practice)

Remove from the data all countries with area greater than 40,000 and all countries with no cities listed. Otherwise the structure of the data should be the same. 


SQL Social-Network Triggers Exercises (core set)

Question 1

Write a trigger that makes new students named 'Friendly' automatically like everyone else in their grade. That is, after the trigger runs, we should have ('Friendly', A) in the Likes table for every other Highschooler A in the same grade as 'Friendly'.

create trigger R
after insert on Highschooler
for each row
when New.name = "Friendly"
begin
insert into Likes (ID1,ID2)
select New.ID, ID from Highschooler
where grade = New.grade and New.ID <> ID;
end;


Question 2

Write one or more triggers to manage the grade attribute of new Highschoolers. If the inserted tuple has a value less than 9 or greater than 12, change the value to NULL. On the other hand, if the inserted tuple has a null value for grade, change it to 9.

create trigger R1
after insert on Highschooler
when new.grade < 9 or new.grade>12 or new.grade is null
begin
update Highschooler set grade = NULL where (id=new.id and new.grade>12);
update Highschooler set grade = NULL where (id=new.id and new.grade<9);
update Highschooler set grade = 9 where (id=new.id and new.grade is null);
end;


Question 3

Write a trigger that automatically deletes students when they graduate, i.e., when their grade is updated to exceed 12.

create trigger graduate
after update of grade on Highschooler
for each row
when (new.grade >12)
begin
delete From HighSchooler 
where Highschooler.ID=new.ID;
end;


SQL Social-Network Triggers Exercises (challenge-level)

Question 1

Write one or more triggers to maintain symmetry in friend relationships. Specifically, if (A,B) is deleted from Friend, then (B,A) should be deleted too. If (A,B) is inserted into Friend then (B,A) should be inserted too. Don't worry about updates to the Friend table.

Create trigger FU2
before Delete on Friend
for each row 
begin
Delete from Friend where old.id1 =iD2 and old.id2 = id1;
end;
|
Create trigger FreindU2
after insert on Friend
for each row 
begin
insert into Friend values (new.ID2, new.ID1);
end;

Question 2

Write a trigger that automatically deletes students when they graduate, i.e., when their grade is updated to exceed 12. In addition, write a trigger so when a student is moved ahead one grade, then so are all of his or her friends.

create trigger grad
after update on highschooler
for each row
when New.grade = Old.grade + 1
begin
update Highschooler set grade = grade + 1 where ID in (select ID2 from Friend where ID1=Old.ID);
end;
|
create trigger graduated
after update on highschooler
for each row
when New.grade > 12
begin
delete from Highschooler where grade >12;
end;


Question 3

Write a trigger to enforce the following behavior: If A liked B but is updated to A liking C instead, and B and C were friends, make B and C no longer friends. Don't forget to delete the friendship in both directions, and make sure the trigger only runs when the "liked" (ID2) person is changed but the "liking" (ID1) person is not changed.

create trigger ruinnedfriendship
after update on likes
for each row 
when old.ID2 <> new.ID2 and new.ID1 = old.ID1
begin
delete from Friend where (id1 = old.id2 and id2 = new.id2) or (id2 = old.id2 and id1 = new.id2);
end;


SQL Movie-Rating View Modification Exercises (core set)

Question 1

Write an instead-of trigger that enables updates to the title attribute of view LateRating.

CREATE TRIGGER LateRatingUpdate
INSTEAD OF UPDATE OF title ON LateRating
FOR EACH ROW
BEGIN
UPDATE Movie
SET title = New.title
WHERE new.mID = Old.mID
AND title = Old.title;
END;


Question 2

Write an instead-of trigger that enables updates to the stars attribute of view LateRating. 

CREATE TRIGGER updateStars
INSTEAD OF update of Stars ON LateRating
FOR EACH ROW
BEGIN
UPDATE Rating
SET Stars = New.Stars
WHERE mID = New.mID and ratingDate = New.ratingDate;
END;


Question 3

Write an instead-of trigger that enables updates to the mID attribute of view LateRating. 

create trigger UpdateLate
instead of update on LateRating
for each row
begin
Update Rating
set mID = New.mID
where mID in (select mID from Rating where mID = Old.mID);
Update Movie
set mID = New.mID
where mID in (select mID from Movie where mID = Old.mID);
end;


Question 4

Write an instead-of trigger that enables deletions from view HighlyRated. 

create trigger updateHighlyRated
instead of delete on HighlyRated
for each row
begin
delete from Rating
where mID = old.mID and stars > 3;
end;


Question 5

Write an instead-of trigger that enables deletions from view HighlyRated. 

create trigger updateHighlyRated
instead of delete on HighlyRated
for each row
begin
Update Rating
set stars = 3
where mID = old.mID and stars > 3;
end;


SQL Movie-Rating View Modification Exercises (challenge-level)

Question 1

Write a single instead-of trigger that enables simultaneous updates to attributes mID, title, and/or stars in view LateRating. Combine the view-update policies of the questions 1-3 in the core set, with the exception that mID may now be updated. Make sure the ratingDate attribute of view LateRating has not also been updated -- if it has been updated, don't make any changes.

CREATE TRIGGER LateRatingUpdate
INSTEAD OF UPDATE ON LateRating
FOR EACH ROW
when New.ratingDate=Old.ratingDate
BEGIN

UPDATE Movie  SET title = New.title WHERE mID=Old.mID;
Update Movie set mID = New.mID where  mID = Old.mID;


UPDATE Rating SET Stars = New.Stars WHERE mID=Old.mID and old.ratingDate = ratingDate;
Update Rating set mID = New.mID where mID = Old.mID ;



end;



Question 2

Write an instead-of trigger that enables insertions into view HighlyRated. 

create trigger B
instead of insert on HighlyRated
for each row
Begin
Insert into Rating ( rID, mID, stars, ratingDate )
select '201',movie.mID,'5',null FROM Movie WHERE New.mID = mID AND New.title = title  ;
end;


Question 3

Write an instead-of trigger that enables insertions into view NoRating. 

create trigger c
instead of insert on NoRating
for each row
WHEN exists (SELECT mID, title
FROM Movie
WHERE New.mID <> mID AND New.title <> title)
begin
Delete From Rating
 WHERE New.mID = mID;
end;


SQL Movie-Rating View Modification Exercises (extra practice)

Question 1

Write an instead-of trigger that enables deletions from view NoRating. 

Question 2

Write an instead-of trigger that enables deletions from view NoRating. 

























