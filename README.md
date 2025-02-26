# netflix_queries

![netflix_pnf](https://github.com/Firdousrahmani/netflix_queries/blob/main/%7BAD3D3A23-A3E3-485F-8566-638D79BE3D33%7D.png)


### ** Netflix Content Analysis Using SQL**    

  

#### **Project Overview**  

This project focuses on analyzing Netflix's content library using SQL to extract meaningful insights. By examining movies and TV shows, the objective is to understand content distribution,
genre popularity, country-wise production trends, and actor/director contributions.  

#### **Objectives**  
- Analyze the distribution of Movies vs. TV Shows  
- Identify the most common ratings and top content-producing countries  
- Determine the longest movies and TV shows on Netflix  
- Examine trends in content added over the last five years  
- Categorize content based on specific keywords  
- Gain insights into popular actors, directors, and genres  

This project demonstrates the ability to apply SQL for data analysis and uncover valuable trends from real-world datasets.



* 1) Count the number of Movies Vs Tv Shows.

select type,
count(*) as total_count
from netflix
group by type;

-- 2) Find the most common rating for movies and TV shows

            ----> 1st method

select
    type,
       mode() within group (order by rating) as most_common_rating
	   from netflix
	 group by type; 

	         ----> 2nd method
			 
select type, rating
from
(select 
      type,
	  rating,
	  count(*) as total_count,
	  rank() over (partition by
	  type order by count(*) desc) as rank
from netflix
group by type, rating) ranked
 where rank = 1;
 

-- 3) List all movies released in a specific year (e.g. 2020)

select *
from netflix
where type = 'Movie'
      and
      release_year = 2020;


-- 4) Find the top 5 countries with the most content on netflix.


 with count_of_country as (
  select

  trim(unnest(string_to_array(country, ','))) as countries
     from netflix
 )
 select countries, count(*) as total_content
 from count_of_country
 group by countries
 order by total_content desc
 limit 5;
 

-- 5)identify the longest movie .

    select title, duration from netflix
    where
        type = 'Movie'
        and
        duration = (select MAX(duration) from netflix) 
        order by duration desc
        limit 1;

   -- identify the longest  tv  show.
    
	select title,
        MAX(CAST(SPLIT_PART(duration, ' ', 1)as INTEGER)) as seasons from netflix
		
		WHERE duration ILIKE '%SEASON%'  
        Group by title
		order by seasons desc
		limit 1;


-- 6) Find content added in the last 5 years
  
  
  select * from netflix
  where
       TO_DATE(date_added, 'Month DD, YYYY') >= current_date - Interval '5 years';


-- 7) Find all the Movies/ TV shows by director 'Rajiv Chilaka'


select title, director 
from netflix
where director ilike '%Rajiv Chilaka%';


-- 8) List all TV shows with more than 5 seasons

	-----1st method-----
		
        select type, title, duration
          from netflix
       where type = 'TV Show'
             and 
	         Split_part(duration, ' ', 1)::numeric > 5;

   ------2nd Method----------------

	  select title,
	 
            CAST(SPLIT_PART(duration, ' ', 1)as INTEGER) as seasons from netflix
		
		     WHERE duration ILIKE '%SEASON%'  
             and CAST(SPLIT_PART(duration, ' ', 1)as INTEGER) > 5
		     order by seasons desc;


-- 9) Count the number of content items in each genre 

  select 
      unnest(string_to_array(listed_in, ',')) as  genre ,
      count(show_id) as total_count
  from netflix
  group by 1
  order by 2 desc;


10) Find each year and the average number of content released in India on Netflix.
 Return top 5 years with highest average content release 

  select 
      EXTRACT(YEAR from to_date(date_added, 'Month DD, YYYY')) as Year,
	  count(*) as yearly_count ,
	  Round(
        count(*)::numeric/(select count(*) from netflix where country = 'India')::numeric * 100, 2)
        as avg_content_per_year
 
    from netflix
     where country ILIKE '%India%'
     group by  Year 
	 ORDER BY avg_content_per_year desc
	 limit 5;
	  



11) List all movies that are Documentaries

   select title, listed_in from netflix
   where
      listed_in ILIKE '%Documentaries%';


12) Find all content  in which a  director is not mentioned

      select * from netflix
      where 
          director is NULL;

-- 13) Find how many movies actor 'Salman Khan' appeared in the last 10 years

    select * from netflix
    where 
	   casts ilike '%Salman Khan%'
	   AND 
	   release_year > extract(YEAR FROM CURRENT_DATE) - 10;

	
-- 14) Find the top 10 actors who have appeared in the highest number of movies produced in INDIA.


  select 
  
  unnest(string_to_array(casts, ',')) as top_actors,
     count(*) as film_done
	 from netflix
	 where country ILIKE '%India%'
     group by 1
	 order by film_done desc
	 limit 10;
   

-- 15) Categories the content based on the presence of the keywords 'kill' and 'violence' in the description field.
-- Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

With new_table
AS
(select *,
      case
	  when
      description ilike '%kill%' 
      or
	  description ilike '%violence%' THEN 'Bad_Content'
      else 'Good_content'
  End category
From Netflix 
)
SELECT 
    category,
	COUNT(*) as total_content
from new_table
Group by 1
