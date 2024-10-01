
### Using wedding database

USE Wedding_database;

### DROPPING buckets
DROP TEMPORARY TABLE IF EXISTS `buckets`;

/* Creting a buckets table to divide the data of each department in budget levels, although not perfect, this method seems good enough for us 
to make sensible decisions on what products to use on each wedding type */

###### Buckets Table ######
CREATE TEMPORARY TABLE `buckets`
SELECT 
#We are only interested in knowing the range of prices per department
  vendor_depart,
  #Min value
  ROUND(MIN(price_unit),1) AS min_value,
  #25th percentile
  ROUND((MAX(price_unit)*0.25),1) AS Q1,
  #50th percentile
  ROUND((MAX(price_unit)*0.50),1) AS Q2,
  #75th percentile
  ROUND((MAX(price_unit)*0.75),1) AS Q3,
  #Max value
  ROUND(MAX(price_unit),1) AS max_value
FROM Products
JOIN vendors
USING (vendor_id)
GROUP BY vendor_depart;

###DROPING T2
DROP TEMPORARY TABLE IF EXISTS T2;

###Creating the temporary table T2 that will serve as the data base to select the adequate products for each wedding type

CREATE TEMPORARY TABLE T2 
SELECT v.vendor_id, v.vendor_depart, v.vendor_sustainable, p.product_id, p.product_name, p.price_unit, 
	   p.unit_vol, CASE #Case statement to divide the data inside the buckets previously stated in the buckets table
																   WHEN price_unit >= b.min_value AND 
																		price_unit <  b.Q1
                                                                   THEN 1
                                                                   ELSE 0
                                                                   END AS inexpensive, #For inexpensive
																CASE 
																   WHEN price_unit >= b.Q1 AND 
																		price_unit <  b.Q2
                                                                   THEN 1
                                                                   ELSE 0
                                                                   END AS affordable, #For affordable
																CASE 
																   WHEN price_unit >= b.Q2 AND 
																		price_unit <  b.Q3
                                                                   THEN 1
                                                                   ELSE 0
                                                                   END AS moderate, #For moderate
																CASE 
																   WHEN price_unit >= b.Q3 AND 
																		price_unit <= b.max_value
                                                                   THEN 1
                                                                   ELSE 0
                                                                   END AS luxury #For lucury
FROM Products AS p
#Joining with the vendors and bucket tables
JOIN vendors AS v USING (vendor_id)
JOIN `buckets` AS b USING (vendor_depart)
#Filtering for the attire department
WHERE vendor_id LIKE 'att%' # defining the department
    AND product_id IN (
        SELECT product_id
        FROM products
        INNER JOIN dress USING (product_id) #dresses code
        WHERE product_id IN (
            SELECT product_id
            FROM dress
            WHERE fabric IN ('tulle', 'lace', 'satin', 'specialty', 'chiffon') #light, silky  and airy txtiles
                AND neckline IN ('v-neck', 'sweetheart', 'illusion') # romantic cuts 
                AND silhouette IN ('trumpet', 'a-line') #comfortable for outdoors venues
                AND sleeve NOT LIKE 'long'))
    OR product_id IN (
        SELECT product_id
        FROM products
        INNER JOIN attire USING (product_id) # for suits and tux
        WHERE product_id IN (
        SELECT product_id
        FROM attire
        WHERE tie LIKE 'necktie')) #just suits not tux since they dont match the venues 
	#Filtering for the catering department
	OR product_id IN ( 
		SELECT product_id
		FROM products
		INNER JOIN cuisine USING (product_id) #cuisine def
		WHERE product_id IN(
			SELECT product_id
			FROM cuisine
			WHERE mediterranean LIKE 1 AND middle_eastern LIKE 1) #preferred foods
			AND product_name IN ('buffet service','food stations', 'hors doeuvres') #type of services
            )
	#Filtering for the jewelry department
	OR product_id IN( 
		SELECT product_id
		FROM products
        INNER JOIN vendors USING (vendor_id)
		WHERE vendor_id LIKE 'jwl%' # defining the department
			AND product_id IN (
				SELECT product_id
				FROM products
				WHERE product_id IN(
					SELECT product_id
					FROM products
					WHERE product_name LIKE '%diamond%'))) #really nice rings 
	#Filtering for the venue department
	OR vendor_id IN ( 
		SELECT vendor_id
		FROM vendors
		WHERE vendor_id LIKE 'ven%' # defining department 
		AND vendor_rating > 45 # great experiences
		AND vendor_location IN( 'san ramon ','half moon bay ','pleasanton ','oakland ','san jose ','los gatos ')) #desired area
	#Filtering for the music department
    OR product_id IN (
		SELECT product_id
		FROM products
		INNER JOIN vendors USING (vendor_id)
		WHERE vendor_id LIKE 'dj%' # defining the department
			AND vendor_id IN (
				SELECT vendor_id
				FROM products
				WHERE vendor_location IN ( 'santa clara ','freemont ','livermore ','hayward ','san jose ','los gatos ', 'san francisco ', 'berkeley') #close locations
                AND unit_vol LIKE '6 hours') #avg time found)
			)
	#Filtering for the hair & makeup department
	OR product_id IN ( 
		SELECT product_id 
		FROM products 
		INNER JOIN vendors USING (vendor_id) 
		WHERE vendor_id LIKE 'hmu%' # defining department
			AND product_name IN ('airbrush', 'simple', 'half up', 'simple', 'traditional') # type of makeup and hair do
			AND unit_vol NOT LIKE '1 per kids') #no kids wedding :)
	#Filtering for the photo & video department
	OR product_id IN ( 
		SELECT product_id
		FROM products
		WHERE product_name IN ('photo and video') #together
			OR product_name LIKE 'photo' AND price_unit < 2500)
	#Filtering for the flowers department
	OR vendor_id LIKE 'flo%' 
    AND product_id IN (
        SELECT product_id
        FROM products
        INNER JOIN Flower_Season_Style USING (product_id) #Flowers style and season
        WHERE product_id IN (
            SELECT product_id
            FROM Flower_Season_Style
            WHERE flower_season IN ('fall', 'summer') #flower seasons
                AND flower_style IN ('romantic', 'vintage', 'rustic','classic','elegant'))) # styles of flowers 
	#Filtering for the invitations department
	OR vendor_id LIKE 'inv%' #Defining the vendor
    AND vendor_name LIKE 'TheKnot%' OR vendor_name LIKE 'Paper%'
    
    #Filtering for the rentals department
    OR vendor_id IN ('ren_01', 'ren_02', 'ren_21', 'ren_24');
    
    #Displaying the table
SELECT *
FROM T2;

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS inexpensive_small;

#Inexpensive-small temporary table with all the products that go into this type of wedding
CREATE TEMPORARY TABLE inexpensive_small
SELECT vendor_id, product_id, product_name, CASE 					   #Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_844', 'prod_250', 'prod_330', 'prod_189', 'prod_197',   #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement,  
                     'prod_796', 'prod_607', 'prod_448', 'prod_096',  'prod_409', 'prod_154',   #Grooms Simple H&M, Bride Simple H, Bride Traditional M, Diamond ring, Photo, Invitations,  
                     'prod_181', 'prod_149', 'prod_161', 'prod_182', 'prod_158', 'prod_067',   #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_017');
                     #Tables, Chairs, Venue
#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS inexpensive_medium;

#Inexpensive-medium temporary table with all the products that go into this type of wedding

CREATE TEMPORARY TABLE inexpensive_medium
SELECT vendor_id, product_id, product_name, CASE 						#Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_853', 'prod_248', 'prod_334', 'prod_189', 'prod_197', #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement, 
                     'prod_421', 'prod_550', 'prod_448', 'prod_085', 'prod_409', 'prod_154', #Grooms Simple H&M, Bride Simple H, Bride traditional M, Diamond ring, Photo, Invitations,  
                     'prod_181', 'prod_180', 'prod_174', 'prod_172', 'prod_159', 'prod_070', #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_017');									 #Tables, Chairs, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS inexpensive_large;

#Inexpensive-large temporary table with all the products that go into this type of wedding

CREATE TEMPORARY TABLE inexpensive_large
SELECT vendor_id, product_id, product_name, CASE 						#Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_854', 'prod_250', 'prod_330', 'prod_189', 'prod_197', #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement, 
                     'prod_467', 'prod_457', 'prod_555', 'prod_096', 'prod_410', 'prod_137', #Grooms Traditional H&M,Brid Half up H, Bride Traditional M, Diamond ring, Photo, Invitations,  
                     'prod_181', 'prod_168', 'prod_169', 'prod_182', 'prod_160', 'prod_070', #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_017');		 							 #Tables, Chairs, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS affordable_small;

#Affordable-small temporary table with all the products that go into this type of wedding

CREATE TEMPORARY TABLE affordable_small
SELECT vendor_id, product_id, product_name, CASE 						#Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_842', 'prod_252', 'prod_354', 'prod_189', 'prod_197', 			 #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement, 
                     'prod_748', 'prod_457', 'prod_553', 'prod_093', 'prod_388', 'prod_137', 'prod_191', #Grooms Traditional H&M, Bride Half up H, Bride Traditional M, Diamond ring, Photo & video, Invitations, Boutounneries  
                     'prod_181', 'prod_168', 'prod_169', 'prod_182', 'prod_176', 'prod_070', 			 #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_006');		 							 			 #Tables, Chairs, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS affordable_medium;

#Affordable-medium temporary table with all the products that go into this type of wedding

CREATE TEMPORARY TABLE affordable_medium
SELECT vendor_id, product_id, product_name, CASE 						#Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_842', 'prod_254', 'prod_348', 'prod_189', 'prod_197', 			 #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement, 
                     'prod_508', 'prod_419', 'prod_480', 'prod_111', 'prod_388', 'prod_137', 'prod_191', #Grooms Simple H&M, Bride half-up H&M, Bride Traditional M, Diamond ring, Photo & Video, Invitations, Boutounneries  
                     'prod_181', 'prod_168', 'prod_169', 'prod_182', 'prod_176', 'prod_070',		     #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_015');									 			 #Tables, Chairs, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS affordable_large;

#Affordable-large temporary table with all the products that go into this type of wedding

CREATE TEMPORARY TABLE affordable_large
SELECT vendor_id, product_id, product_name, CASE 						#Creating the budget_level column and filling it according to each product characteristic
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE #Creating the wedding_size column where every item can take place in ALL sizes of wedding excepting prod_037 
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_827', 'prod_842', 'prod_259', 'prod_353', 'prod_189', 'prod_197', 			 #Dress, Suit, Buffet, Music, Bouquet, Flower Arrangement, 
                     'prod_508', 'prod_507', 'prod_480', 'prod_114', 'prod_388', 'prod_137', 'prod_191', #Grooms Simple H&M, BrideSimple H&M, Bride Traditional M, Diamond ring, Photo & Video, Invitations, Boutounneries  
                     'prod_181', 'prod_168', 'prod_169', 'prod_182', 'prod_175', 'prod_070', 			 #RSVP, Table number, Wedding Program, Menu, Envelope, Glasswares,  
                     'prod_071', 'prod_083', 'prod_008');									 			 #Tables, Chairs, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS moderate_small;

#Moderate-small temporary table with all the products that go into this type of wedding
CREATE TEMPORARY TABLE moderate_small
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_004'
                                                                       THEN 'small'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_805', 'prod_854', 'prod_824', 'prod_273', 'prod_352', 'prod_189', 'prod_224', 'prod_191', #Dress, Groom Suit, Bridesmaid, Food station, Music, Bouquet, Flower Arregement, Boutounniers
                     'prod_421', 'prod_541', 'prod_417', 'prod_112', 'prod_120', 'prod_398', 'prod_153', 			 # Groom simple h&m, H&M bride airbrush, Bride H&M (halfup), Diamond ring, diamond bracelet, Photo & video, Invitations
					 'prod_151', 'prod_149', 'prod_161', 'prod_182', 'prod_158', 'prod_070',  						 # RSVP, Table number, Wedding Program, Menu, Envelope, Glassware
					 'prod_074', 'prod_083', 'prod_051', 'prod_004'  );												 #Tables, Chairs, Linen, Venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS moderate_medium;
                     
#Moderate-medium temporary table with all the products that go into this type of wedding 
CREATE TEMPORARY TABLE moderate_medium
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_015'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_806', 'prod_854', 'prod_824', 'prod_286', 'prod_352', 'prod_189', 'prod_224', 'prod_191', #Dress, Groom Suit, Bridesmaid, Food station, Music, Bouquet, Flower Arregement, Boutounniers
					 'prod_421', 'prod_417', 'prod_478', 'prod_112', 'prod_120', 'prod_398', 'prod_153', 			 #Groom simple h&m, H&M bride (airb), H&M (half), Diamond ring, diamond bracelt, Photo & video, Invitations 
					 'prod_151', 'prod_150', 'prod_161', 'prod_182', 'prod_158', 'prod_070',   			 			 #RSVP, Table number, Wedding Program, Menu, Envelope, glassware
                     'prod_074', 'prod_083', 'prod_051', 'prod_015' );						 			             #tables, chairs, linen, venue
          
#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS moderate_large;
                    
#Moderate-large temporary table with all the products that go into this type of wedding
CREATE TEMPORARY TABLE moderate_large
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_001'
                                                                       THEN 'large'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_806', 'prod_854', 'prod_824', 'prod_286', 'prod_352', 'prod_189', 'prod_224', 'prod_191', #Dress, Groom Suit, Bridesmaid, Food station, Music, Bouquet, Flower Arregement, Boutounniers
					 'prod_421', 'prod_417', 'prod_478', 'prod_112', 'prod_120', 'prod_398', 'prod_153', 			 #Groom simple h&m, H&M bride (airb), H&M (half), Diamond ring, diamond bracelt, Photo & video, Invitations 
					 'prod_151', 'prod_150', 'prod_161', 'prod_182', 'prod_158', 'prod_070',   			 			 #RSVP, Table number, Wedding Program, Menu, Envelope, glassware
                     'prod_074', 'prod_083', 'prod_051', 'prod_001' );						 			             #tables, chairs, linen, venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS luxury_small;

#Luxury-small temporary table with all the products that go into this type of wedding
CREATE TEMPORARY TABLE luxury_small
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_004'
                                                                       THEN 'small'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_807', 'prod_854', 'prod_824', 'prod_291', 'prod_261', 'prod_362', 'prod_189', 'prod_224', 'prod_191', 'prod_218',  #Dress,Groom Suit, Bridesmaids, Hors doeuvres, Buffet, Music, Flower arrengement, Bouquet, Boutounneries, Corsage
					 'prod_421', 'prod_595', 'prod_631', 'prod_113', 'prod_120', 'prod_399', 'prod_134',    								  #Groom simple h&m, H&M bride (trd), H&M (hair half), Diamond ring, diamond bracelt,Photo & video, Invitations
					 'prod_146', 'prod_149', 'prod_169', 'prod_182', 'prod_158', 'prod_070',  												  #RSVP, Table number, Wedding Program, Menu, Envelope, glassware
                     'prod_074', 'prod_083', 'prod_051', 'prod_004');															              #tables, chairs, linen, venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS luxury_medium;

#Luxury-medium temporary table with all the products that go into this type of wedding 
CREATE TEMPORARY TABLE luxury_medium
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'medium'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_807', 'prod_854', 'prod_824', 'prod_291', 'prod_261', 'prod_362', 'prod_189', 'prod_224', 'prod_191', 'prod_218',  #Dress,Groom Suit, Bridesmaids, Hors doeuvres, Buffet, Music, Flower arrengement, Bouquet, Boutounneries, Corsage
					 'prod_421', 'prod_595', 'prod_631', 'prod_113', 'prod_120', 'prod_399', 'prod_134',    								  #Groom simple h&m, H&M bride (trd), H&M (hair half), Diamond ring, diamond bracelt,Photo & video, Invitations
					 'prod_146', 'prod_149', 'prod_169', 'prod_182', 'prod_158', 'prod_070',  												  #RSVP, Table number, Wedding Program, Menu, Envelope, glassware
                     'prod_074', 'prod_083', 'prod_051', 'prod_037');																          #tables, chairs, linen, venue

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS luxury_large;

#Luxury-large temporary table with all the products that go into this type of wedding
CREATE TEMPORARY TABLE luxury_large
SELECT vendor_id, product_id, product_name, CASE 
											  WHEN inexpensive = 1
											  THEN 'inexpensive'
											  WHEN affordable = 1
											  THEN 'affordable'
											  WHEN moderate = 1
											  THEN 'moderate'
											  WHEN luxury = 1
											  THEN 'luxury'
											  END AS budget_level, CASE
																	   WHEN product_id = 'prod_037'
                                                                       THEN 'large'
                                                                       ELSE 'All'
                                                                       END AS wedding_size
FROM T2
#Filtering for only the products we chose for this specific type of wedding
WHERE product_id IN ('prod_807', 'prod_854', 'prod_824', 'prod_291', 'prod_261', 'prod_362', 'prod_189', 'prod_224', 'prod_191', 'prod_218',  #Dress,Groom Suit, Bridesmaids, Hors doeuvres, Buffet, Music, Flower arrengement, Bouquet, Boutounneries, Corsage
					 'prod_421', 'prod_595', 'prod_631', 'prod_113', 'prod_120', 'prod_399', 'prod_134',    								  #Groom simple h&m, H&M bride (trd), H&M (hair half), Diamond ring, diamond bracelt,Photo & video, Invitations
					 'prod_165', 'prod_180', 'prod_174', 'prod_182', 'prod_158', 'prod_070',  												  #RSVP, Table number, Wedding Program, Menu, Envelope, glassware
                     'prod_074', 'prod_083', 'prod_051', 'prod_004');															              #tables, chairs, linen, venue


#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS small_inexpensive_cost;


CREATE TEMPORARY TABLE small_inexpensive_cost
SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit)/2 # as small weddings are for 50 people we calculate half the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 50) # as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 5)# as small weddings are for 50 peopleand tables avg is 10 per table we just divided 
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS s_i_cost
FROM inexpensive_small 
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;

SELECT ROUND(SUM(s_i_cost), 2) AS total_cost_with_two_decimals
FROM small_inexpensive_cost;

DROP TEMPORARY TABLE IF EXISTS inexpensive_medium_cost;


CREATE TEMPORARY TABLE inexpensive_medium_cost
SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.5) # as medium weddings are up to 150 people we multiply by 150 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 1.5 the invitations
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 15) # as medium weddings are up to 150 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS i_e_cost
FROM inexpensive_medium 
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;

SELECT ROUND(SUM(i_e_cost), 2) AS total_cost_with_two_decimals
FROM inexpensive_medium_cost;

SELECT ROUND(SUM(l_l_cost ), 2) AS total_cost_with_two_decimals
FROM luxury_large_cost;

DROP TEMPORARY TABLE IF EXISTS inexpensive_medium_cost;

SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.51) # as large weddings are from 151 people up, we multiply by 151 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 151) # as large weddings are from 151 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS i_l_cost 
FROM inexpensive_large
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,	
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit)/2 # as small weddings are for 50 people we calculate half the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 50) # as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 5)# as small weddings are for 50 peopleand tables avg is 10 per table we just divided 
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS a_s_cost 
FROM affordable_small
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.5) # as medium weddings are up to 150 people we multiply by 150 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 1.5 the invitations
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 15) # as medium weddings are up to 150 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS a_m_cost 
FROM affordable_medium
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.51) # as large weddings are from 151 people up, we multiply by 151 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 151) # as large weddings are from 151 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS a_l_cost 
FROM affordable_large
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;



SELECT p.vendor_id, p.product_id, p. product_name,	
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit)/2 # as small weddings are for 50 people we calculate half the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 50) # as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 5)# as small weddings are for 50 peopleand tables avg is 10 per table we just divided 
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS m_s_cost 
FROM moderate_small
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;



SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.5) # as medium weddings are up to 150 people we multiply by 150 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 1.5 the invitations
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 15) # as medium weddings are up to 150 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS m_m_cost 
FROM moderate_medium
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.51) # as large weddings are from 151 people up, we multiply by 151 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 151) # as large weddings are from 151 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS m_l_cost  
FROM moderate_large
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,	
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit)/2 # as small weddings are for 50 people we calculate half the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 50) # as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 5)# as small weddings are for 50 peopleand tables avg is 10 per table we just divided 
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 50)# as small weddings are for 50 people we multiply by 50 
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS l_s_cost  
FROM luxury_small
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.5) # as medium weddings are up to 150 people we multiply by 150 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 1.5 the invitations
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 15) # as medium weddings are up to 150 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 150) # as medium weddings are up to 150 people we multiply by 150
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS l_m_cost 
FROM luxury_medium
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;


SELECT p.vendor_id, p.product_id, p. product_name,
	CASE 
        WHEN unit_vol LIKE '%per person%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '% hours' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per bride%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per groom%' THEN SUM(price_unit)
        WHEN unit_vol LIKE '%per service%' THEN SUM(price_unit) # single per event
        WHEN unit_vol LIKE '%per 100 invitations%' THEN SUM(price_unit*1.51) # as large weddings are from 151 people up, we multiply by 151 the invitations
        WHEN unit_vol LIKE '%per piece%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per table%' THEN SUM(price_unit * 151) # as large weddings are from 151 people and tables avg is 10 per table ... math
        WHEN unit_vol LIKE '%per chair%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        WHEN unit_vol LIKE '%per vendor%' AND p.vendor_id NOT LIKE 'ven%' THEN SUM(price_unit * 151) # as large weddings are from 151 people up, we multiply by 151
        ELSE SUM(price_unit) # include ring, bouquet, arrengments
    END AS l_l_cost 
FROM luxury_large
INNER JOIN products AS p USING (product_id) 
GROUP BY unit_vol, p.vendor_id, p.product_id, p. product_name;

/*
We assumed that the attendants to the wedding would be 50, 100 and 150 according to their respective wedding size small,
medium and large. We also assumed that the price for flowers and other products of this department would remain constant
disregarding the amount ordered, this will be changed for the final deliverable. Finally, our last assumption was that they
wouldn't need wedding bands (this was because we didn't have the options available in the data base). For special cases, 
there were some weddings that lacked products in that budget bracket, in those occasions we utilized products/vendors from other 
budget brackets to fill the need. We tried to pick sustainable products/vendors, when possible, as to comply with our natural
wedding theme, but in some occasions were sustainable options were not plausible non-sustainable options were picked.
*/

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS relevant_vendors;

#Creating a temporary table relevant_vendors by combining the results from the 12 temporary tables for each wedding type
CREATE TEMPORARY TABLE relevant_vendors
SELECT vendor_id, budget_level, wedding_size
FROM inexpensive_small
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM inexpensive_medium
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM inexpensive_large
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM affordable_small
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM affordable_medium
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM affordable_large 
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM moderate_small
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM moderate_medium
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM moderate_large
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM luxury_small
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM luxury_medium
#Using union distinct to combine the distinct results from the different tables
UNION DISTINCT 
SELECT vendor_id, budget_level, wedding_size
FROM luxury_large
ORDER BY vendor_id; 

#Displaying the table
SELECT *
FROM relevant_vendors;

/* 
Selecting vendors was a hard job since we carefully review each of them. Our first vendor selection was based on the 
ones whose products align with the client's vision board and the best quality products from the available desired options. 
Afterwards, we used the price_unit per department to divide by quartiles and segment them in 4 categories (inexpensive, 
affordable, moderate, luxury). Later, according to our sources (the knot articles), we selected the bridesmaids’ recommended 
quantities and determined the base items and additional accessories like bracelets or corsages per wedding size and budget. 
Finally, we assume the couple want to strictly stick to the vision board's attire. Therefore, the selections were based on 
suits not tuxedos and airy dresses like A cut with tulle, chiffon lace, or specialty fabric. Moreover, the flower selection 
was assumed by the seasonality of them like lilies for summer, roses for fall. Finally, deciding on the waterproof options 
in hair and makeup assures that even a light drizzle will just make the wedding even more memorable.  
 */

#Droping the table if it already exists
DROP TEMPORARY TABLE IF EXISTS vendor_options;

#Creating the temporary table vendor_options from the 12 different temporary tables for each wedding type
CREATE TEMPORARY TABLE vendor_options 

#Adding a row with the data from the inexpensive_small temporary table
SELECT
  'inexpensive_small' AS Wedding_type, 								    #Here I'm creating the column of wedding type to classify each wedding
  'Natural'           AS Wedding_theme,  								#Here I'm creating the column of wedding theme and assigning natural as the theme 
  '7298.13'             AS Est_cost,  									#Here I'm creating the column of imated cost and assigning the calculated price for each wedding type
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Suit, #Using the Max function so all the results are in the same row
  MAX(CASE WHEN product_id = 'prod_844' THEN 'prod_844' END) AS Dress,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_250' THEN 'prod_250' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_330' THEN 'prod_330' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_796' THEN 'prod_796' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_607' THEN 'prod_607' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_448' THEN 'prod_448' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_096' THEN 'prod_096' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_409' THEN 'prod_409' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_154' THEN 'prod_154' END) AS Invitations,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Boutounneries,#Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_149' THEN 'prod_149' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_161' THEN 'prod_161' END) AS Wedding_Program,
  MAX(CASE WHEN product_id in ('prod_182') THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_067' THEN 'prod_067' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,        #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_017' THEN 'prod_017' END) AS Venue
FROM inexpensive_small
GROUP BY Wedding_type
#Adding another row with the data from the inexpensive_large temporary table
UNION ALL
SELECT
'inexpensive_medium' AS Wedding_type,
'Natural'            AS Wedding_theme, 
'10345.18'              AS Est_cost,  									
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_853' THEN 'prod_853' END) AS Suit,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_248' THEN 'prod_248' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_334' THEN 'prod_334' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_550' THEN 'prod_550' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_448' THEN 'prod_448' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_085' THEN 'prod_085' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_409' THEN 'prod_409' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_154' THEN 'prod_154' END) AS Invitations,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Boutounneries,#Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_180' THEN 'prod_180' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_174' THEN 'prod_174' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_172' THEN 'prod_172' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_159' THEN 'prod_159' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,        #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_017' THEN 'prod_017' END) AS Venue
  FROM inexpensive_medium
  GROUP BY Wedding_type
  #Adding another row with the data from the inexpensive_large temporary table
  UNION ALL
SELECT
'inexpensive_large' AS Wedding_type,
'Natural'           AS Wedding_theme,
'11534.49'             AS Est_cost,  									
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_250' THEN 'prod_250' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_330' THEN 'prod_330' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_467' THEN 'prod_467' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_457' THEN 'prod_457' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_555' THEN 'prod_555' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_096' THEN 'prod_096' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_410' THEN 'prod_410' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_137' THEN 'prod_137' END) AS Invitations,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Boutounneries,#Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_168' THEN 'prod_168' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_160' THEN 'prod_160' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,       #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_017' THEN 'prod_017' END) AS Venue
  FROM inexpensive_large
  GROUP BY Wedding_type
    #Adding another row with the data from the affordable_small temporary table
  UNION ALL
SELECT
'affordable_small' AS Wedding_type,
'Natural'          AS Wedding_theme,
'19257.86'            AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_842' THEN 'prod_842' END) AS Suit,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_252' THEN 'prod_252' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_354' THEN 'prod_354' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_748' THEN 'prod_748' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_457' THEN 'prod_457' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_553' THEN 'prod_553' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_093' THEN 'prod_093' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_388' THEN 'prod_388' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_137' THEN 'prod_137' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, #Because the budget of this wedding increased Boutounneries became a viable opttion, thids will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_168' THEN 'prod_168' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_176' THEN 'prod_176' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,        #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_006' THEN 'prod_006' END) AS Venue
  FROM affordable_small
  GROUP BY Wedding_type
   #Adding another row with the data from the affordable_medium temporary table
  UNION ALL
SELECT
'affordable_medium' AS Wedding_type,
'Natural'           AS Wedding_theme,
'21938.58'             AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_842' THEN 'prod_842' END) AS Suit,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_254' THEN 'prod_254' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_348' THEN 'prod_348' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_508' THEN 'prod_508' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_419' THEN 'prod_419' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_480' THEN 'prod_480' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_111' THEN 'prod_111' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_388' THEN 'prod_388' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_137' THEN 'prod_137' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, #Because the budget of this wedding increased Boutounneries became a viable opttion, thids will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_168' THEN 'prod_168' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_176' THEN 'prod_176' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,        #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_015' THEN 'prod_015' END) AS Venue
  FROM affordable_medium
  GROUP BY Wedding_type
     #Adding another row with the data from the affordable_large temporary table
  UNION ALL
SELECT
'affordable_large' AS Wedding_type,
'Natural'          AS Wedding_theme,
'29309.19'            AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_827' THEN 'prod_827' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_842' THEN 'prod_842' END) AS Suit,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bridesmaid, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_259' THEN 'prod_259' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_353' THEN 'prod_353' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_197' THEN 'prod_197' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_508' THEN 'prod_508' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_507' THEN 'prod_507' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_480' THEN 'prod_480' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_114' THEN 'prod_114' END) AS Ring,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Bracelet,     #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_388' THEN 'prod_388' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_137' THEN 'prod_137' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, #Because the budget of this wedding increased Boutounneries became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_181' THEN 'prod_181' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_168' THEN 'prod_168' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_175' THEN 'prod_175' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_071' THEN 'prod_071' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Linen,        #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_008' THEN 'prod_008' END) AS Venue
  FROM affordable_large
  GROUP BY Wedding_type
  #Adding another row with the data from the moderate_small temporary table
  UNION ALL
SELECT
'moderate_small' AS Wedding_type,
'Natural'        AS Wedding_theme,
'34401.8'          AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_805' THEN 'prod_805' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_273' THEN 'prod_273' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_352' THEN 'prod_352' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_417' THEN 'prod_417' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_541' THEN 'prod_541' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_112' THEN 'prod_112' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_398' THEN 'prod_398' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_153' THEN 'prod_153' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_151' THEN 'prod_151' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_149' THEN 'prod_149' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_161' THEN 'prod_161' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_004' THEN 'prod_004' END) AS Venue
  FROM moderate_small
  GROUP BY Wedding_type
  #Adding another row with the data from the moderate_medium temporary table
  UNION ALL
SELECT
'moderate_medium' AS Wedding_type,
'Natural'         AS Wedding_theme,
'30944.78'           AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_806' THEN 'prod_806' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_286' THEN 'prod_286' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_352' THEN 'prod_352' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_417' THEN 'prod_417' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_478' THEN 'prod_478' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_112' THEN 'prod_112' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_398' THEN 'prod_398' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_153' THEN 'prod_153' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_151' THEN 'prod_151' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_150' THEN 'prod_150' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_161' THEN 'prod_161' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_015' THEN 'prod_015' END) AS Venue
  FROM moderate_medium
  GROUP BY Wedding_type
  #Adding another row with the data from the moderate_large temporary table
  UNION ALL
SELECT
'moderate_large' AS Wedding_type,
'Natural'        AS Wedding_theme,
'38910.95'          AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_806' THEN 'prod_806' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_286' THEN 'prod_286' END) AS Catering,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Catering_2, #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_352' THEN 'prod_352' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = '' THEN '' END) AS Corsage,    #Including this so the tables are all the same size
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_417' THEN 'prod_417' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_478' THEN 'prod_478' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_112' THEN 'prod_112' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_398' THEN 'prod_398' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_153' THEN 'prod_153' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_151' THEN 'prod_151' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_150' THEN 'prod_150' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_161' THEN 'prod_161' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_001' THEN 'prod_001' END) AS Venue
  FROM moderate_large
  GROUP BY Wedding_type
    #Adding another row with the data from the luxury_small temporary table
    UNION ALL
SELECT
'luxury_small' AS Wedding_type,
'Natural'      AS Wedding_theme,
'40013.81'        AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_807' THEN 'prod_807' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_291' THEN 'prod_291' END) AS Catering,
  MAX(CASE WHEN product_id = 'prod_261' THEN 'prod_261' END) AS Catering_2,
  MAX(CASE WHEN product_id = 'prod_362' THEN 'prod_362' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = 'prod_218' THEN 'prod_218' END) AS Corsage,
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_595' THEN 'prod_595' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_631' THEN 'prod_631' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_113' THEN 'prod_113' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_399' THEN 'prod_399' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_134' THEN 'prod_134' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_146' THEN 'prod_146' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_149' THEN 'prod_149' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_004' THEN 'prod_004' END) AS Venue
  FROM luxury_small
  GROUP BY Wedding_type
      #Adding another row with the data from the luxury_medium temporary table
      UNION ALL
SELECT
'luxury_medium'  AS Wedding_type,
'Natural'        AS Wedding_theme,
'50723.43'          AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_807' THEN 'prod_807' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_291' THEN 'prod_291' END) AS Catering,
  MAX(CASE WHEN product_id = 'prod_261' THEN 'prod_261' END) AS Catering_2,
  MAX(CASE WHEN product_id = 'prod_362' THEN 'prod_362' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = 'prod_218' THEN 'prod_218' END) AS Corsage,
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_595' THEN 'prod_595' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_631' THEN 'prod_631' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_113' THEN 'prod_113' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_399' THEN 'prod_399' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_134' THEN 'prod_134' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_146' THEN 'prod_146' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_149' THEN 'prod_149' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_169' THEN 'prod_169' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_037' THEN 'prod_037' END) AS Venue
  FROM luxury_medium
  GROUP BY Wedding_type
        #Adding another row with the data from the luxury_large temporary table
      UNION ALL
SELECT
'luxury_large' AS Wedding_type,
'Natural'      AS Wedding_theme,
'46814.3'        AS Est_cost, 
  MAX(CASE WHEN product_id = 'prod_807' THEN 'prod_807' END) AS Dress,
  MAX(CASE WHEN product_id = 'prod_854' THEN 'prod_854' END) AS Suit,
  MAX(CASE WHEN product_id = 'prod_824' THEN 'prod_824' END) AS Bridesmaid, #Because the budget of this wedding increased Bridemaid dress became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_291' THEN 'prod_291' END) AS Catering,
  MAX(CASE WHEN product_id = 'prod_261' THEN 'prod_261' END) AS Catering_2,
  MAX(CASE WHEN product_id = 'prod_362' THEN 'prod_362' END) AS Music,
  MAX(CASE WHEN product_id = 'prod_189' THEN 'prod_189' END) AS Bouquet,
  MAX(CASE WHEN product_id = 'prod_218' THEN 'prod_218' END) AS Corsage,
  MAX(CASE WHEN product_id = 'prod_224' THEN 'prod_224' END) AS Flower_Arrangement,
  MAX(CASE WHEN product_id = 'prod_421' THEN 'prod_421' END) AS Grooms_HnM,
  MAX(CASE WHEN product_id = 'prod_595' THEN 'prod_595' END) AS Brides_H,
  MAX(CASE WHEN product_id = 'prod_631' THEN 'prod_631' END) AS Brides_M,
  MAX(CASE WHEN product_id = 'prod_113' THEN 'prod_113' END) AS Ring,
  MAX(CASE WHEN product_id = 'prod_120' THEN 'prod_120' END) AS Bracelet,   #Because the budget of this wedding increased Bracelet became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_399' THEN 'prod_399' END) AS Photo_n_video,
  MAX(CASE WHEN product_id = 'prod_134' THEN 'prod_134' END) AS Invitations,
  MAX(CASE WHEN product_id = 'prod_191' THEN 'prod_191' END) AS Boutounneries, 
  MAX(CASE WHEN product_id = 'prod_165' THEN 'prod_165' END) AS RSVP,
  MAX(CASE WHEN product_id = 'prod_180' THEN 'prod_180' END) AS Table_number,
  MAX(CASE WHEN product_id = 'prod_174' THEN 'prod_174' END) AS Wedding_Program,
  MAX(CASE WHEN product_id = 'prod_182' THEN 'prod_182' END) AS Menu,
  MAX(CASE WHEN product_id = 'prod_158' THEN 'prod_158' END) AS Enevelope,
  MAX(CASE WHEN product_id = 'prod_070' THEN 'prod_070' END) AS Glassware,
  MAX(CASE WHEN product_id = 'prod_074' THEN 'prod_074' END) AS `Tables`,
  MAX(CASE WHEN product_id = 'prod_083' THEN 'prod_083' END) AS Chairs,
  MAX(CASE WHEN product_id = 'prod_051' THEN 'prod_051' END) AS Linen,     #Because the budget of this wedding increased Linen became a viable opttion, this will cause null values in the previous wedding types.
  MAX(CASE WHEN product_id = 'prod_004' THEN 'prod_004' END) AS Venue
  FROM luxury_large
  GROUP BY Wedding_type;
  
#Displaying the table
SELECT *
FROM vendor_options;
