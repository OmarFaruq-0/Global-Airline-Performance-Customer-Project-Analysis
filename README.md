# Global-Airline-Performance-Customer-Project-Analysis
```sql






DROP TABLE IF EXISTS airlines;
CREATE TABLE airlines 
			(
			airline_code Varchar (6) PRIMARY KEY,
			name VARCHAR (30)	
			);

DROP TABLE IF EXISTS airports;
CREATE TABLE airports
			(
             iata VARCHAR (5) PRIMARY KEY,
			 airport_name VARCHAR (25),
			 city VARCHAR (14),
			 country VARCHAR (12),
			 lan DECIMAL (8,4),
			 lon DECIMAL (8,4)
			);



DROP TABLE IF EXISTS booking_status;
CREATE TABLE booking_status 
			(
			booking_id INT PRIMARY KEY,
			passenger_id INT, 
			passenger_name VARCHAR (30),
			gender VARCHAR (10),
			nationality VARCHAR(15),
			frequent_flyer_no VARCHAR(20), 
			flight_instance_id INT,
			flight_number VARCHAR(15), 
			airline_code VARCHAR(15), 
			origin VARCHAR(10), 
			destination VARCHAR(10), 
			departure_scheduled TIMESTAMP, 
			arrival_scheduled TIMESTAMP, 
			booking_date TIMESTAMP, 
			travel_date DATE, 
			seat_class VARCHAR(12), 
			seat_no VARCHAR(10), 
			fare INT, 
			taxes INT,
			fees INT, 
			total_price INT, 
			payment_method VARCHAR(16), 
			booking_channel VARCHAR(17), 
			booking_status VARCHAR(16) , 
			refunded_amount INT, 
			checked_in VARCHAR(11), 
			baggage_kg INT, 
			meal_pref VARCHAR(11), 
			fare_basis VARCHAR(8), 
			ticket_number VARCHAR(15) 
			);



DROP TABLE IF EXISTS flight_instances;
CREATE TABLE flight_instances (
                              flight_instance_id INT Primary KEY, 
							  flight_id INT, 
							  flight_number VARCHAR(15) , 
							  airline_code VARCHAR(15), 
							  origin VARCHAR(13), 
							  destination VARCHAR(13), 
							  departure_scheduled TIMESTAMP, 
							  arrival_scheduled TIMESTAMP, 
							  aircraft_type VARCHAR(15), 
							  distance_km INT, 
							  capacity INT, 
							  base_fare_economy INT, 
							  base_fare_business INT, 
							  base_fare_first INT
							  );

DROP TABLE IF EXISTS flight_masters;
CREATE TABLE flight_masters ( 
                            flight_id INT Primary KEY , 
							flight_number VARCHAR(15), 
							airline_code VARCHAR(15), 
							origin VARCHAR(13),  
							destination VARCHAR(13), 
							aircraft_type VARCHAR(15) , 
							distance_km INT, 
							capacity INT,
							base_fare_economy INT,
							base_far_business INT, 
							base_fare_first INT
							);



DROP TABLE IF EXISTS passengers;
CREATE TABLE passengers (
						passenger_id INT PRIMARY KEY,
						full_name VARCHAR(20), 
						gender VARCHAR(10), 
						date_of_birth DATE, 
						nationality VARCHAR(15), 
						frequent_flyer_no VARCHAR(20), 
						email VARCHAR(38), 
						phone VARCHAR(14)
						);




                          -- Foreign Keys Constraints --

ALTER TABLE booking_status
ADD CONSTRAINT fk_passengers
FOREIGN KEY (passenger_id)
REFERENCES passengers(passenger_id);


ALTER TABLE booking_status
ADD CONSTRAINT fk_airlines
FOREIGN KEY (airline_code)
REFERENCES airlines(airline_code);


ALTER TABLE booking_status
ADD CONSTRAINT fk_flight_instances
FOREIGN KEY (flight_instance_id)
REFERENCES flight_instances(flight_instance_id);


ALTER TABLE booking_status
ADD CONSTRAINT fk_airports
FOREIGN KEY (origin)
REFERENCES airports(iata);


ALTER TABLE flight_instances
ADD CONSTRAINT fk_flight_masters
FOREIGN KEY (flight_id)
REFERENCES flight_masters (flight_id);


ALTER TABLE flight_masters
ADD CONSTRAINT fk_airlines
FOREIGN KEY (airline_code)
REFERENCES airlines(airline_code);


ALTER TABLE flight_masters
ADD CONSTRAINT fk_airports
FOREIGN KEY (origin)
REFERENCES airports(iata);

                   -- All tables --

SELECT * FROM passengers;
SELECT * FROM booking_status;
SELECT * FROM airports;
SELECT * FROM airlines;
SELECT * FROM flight_masters;
SELECT * FROM flight_instances;


                      -- Airline Performance and Market Share --

--Task 01:  Rank all airlines by:  Number of passengers flown, Average ticket pricE

-- (i)Total revenue generated 

SELECT AL.name, SUM (BS.total_price)  FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY AL.name;


-- (ii) Number of passenger flown from each airline                                          
SELECT AL.name, COUNT(BS.passenger_id) FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY AL.name;



--(iii) Average ticket price						
SELECT AL.name, SUM(BS.total_price) / COUNT(BS.passenger_id) AS Avg_price_per_ticket FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY AL.name;


-- Task 02: Compute market share of each airline by revenue
SELECT AL.name,
SUM(BS.total_price) AS airline_revenue,
ROUND(
SUM ((BS.total_price)* 100)/ SUM(SUM(BS.total_price)) over(), 2
) 
AS revenue_in_percentage
FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY AL.name;


-- Task 03: Find which airline operates the most profitable routes. 
SELECT  AL.name AS airline_name,
FM.origin, FM.destination, SUM(BS.total_price) AS total_revenue FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY AL.name,FM.origin, FM.destination
ORDER BY total_revenue DESC
LIMIT 10;



                   -- All tables --

SELECT * FROM passengers;
SELECT * FROM booking_status;
SELECT * FROM airports;
SELECT * FROM airlines;
SELECT * FROM flight_masters;
SELECT * FROM flight_instances;  



                                 -- Route & Flight Efficiency --


-- Task 05: Identify top 10 most profitable routes (Origin–Destination).
SELECT  
FM.origin, FM.destination,
SUM(BS.total_price) AS total_revenue 
FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY FM.origin, FM.destination
ORDER BY total_revenue DESC
LIMIT 10;


--Task 06: Calculate load factor per flight = (booked seats ÷ capacity) 
SELECT FM.flight_id, FM.flight_number,
ROUND(
COUNT(BS.seat_no)*100 / SUM(SUM(FM.capacity)) OVER(),2
) AS load_factor_per_flight
FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY FM.flight_id, FM.flight_number;


--Task 07: Detect 10 underperforming routes (low occupancy or revenue).
SELECT  
FM.origin, FM.destination,
SUM(BS.total_price) AS total_revenue 
FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
WHERE booking_status = 'Confirmed'
GROUP BY FM.origin, FM.destination
ORDER BY total_revenue ASC
LIMIT 10;


--Task 08: Find average fare difference between economy/business/first classes.
WITH class_average AS(                                                              
SELECT
	seat_class,
	AVG(total_price) AS avg_price

FROM
	booking_status
	WHERE booking_status= 'Confirmed'
GROUP BY seat_class
),

overall_avg AS(
	SELECT
	AVG(total_price) as overall_avg_price
	FROM booking_status
	WHERE booking_status ='Confirmed'
)
SELECT 
	ca.seat_class,
	ca.avg_price,
	ol.overall_avg_price,
	ROUND
		(ca.avg_price - ol.overall_avg_price,2) as average_fare_difference
FROM
	class_average ca, 
	overall_avg ol;


                   -- All tables --

SELECT * FROM passengers;
SELECT * FROM booking_status;
SELECT * FROM airports;
SELECT * FROM airlines;
SELECT * FROM flight_masters;
SELECT * FROM flight_instances;  


                 -- Customer Demographics & Behavior--

--Task 09: Which countries’ passengers book the most flights?
SELECT P.nationality, COUNT(booking_id) AS total_passengers FROM 
	passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
	WHERE booking_status = 'Confirmed'
GROUP BY P.nationality
ORDER BY total_passengers DESC;

--Task 10: Average baggage weight and meal preference trends by nationality.
SELECT nationality,meal_pref, SUM(BS.baggage_kg) / COUNT(DISTINCT P.nationality) AS avg_baggage_wieght FROM 
	passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
	WHERE booking_status = 'Confirmed'
GROUP BY P.nationality, BS.meal_pref
ORDER BY P.nationality;


--Task 11: Top frequent flyers (count bookings per frequent_flyer_no). 
SELECT 
	P.frequent_flyer_no, COUNT(booking_id) AS booking_ids_per_frequent_flyer_no FROM 
	passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
	WHERE booking_status = 'Confirmed'
	and P.frequent_flyer_no is not NULL
GROUP BY P.frequent_flyer_no
ORDER BY booking_ids_per_frequent_flyer_no DESC; 


--Task 12:Gender distribution across different airlines. 
SELECT 
	AL.name, P.gender, COUNT(P.passenger_id) FROM 
	passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
	WHERE booking_status = 'Confirmed'
	and P.frequent_flyer_no is not NULL
GROUP BY AL.name,  P.gender
ORDER BY AL.name DESC; 


                   -- All tables --

SELECT * FROM passengers;
SELECT * FROM booking_status;
SELECT * FROM airports;
SELECT * FROM airlines;
SELECT * FROM flight_masters;
SELECT * FROM flight_instances;  


                      -- Revenue, Refund, and Booking Analysis --

					  
--Task 13: Monthly revenue trend for each airline.
SELECT AL.name, DATE_TRUNC('month', BS.booking_date) AS per_revenue, Sum(BS.total_price) AS revenue
	 FROM 
	passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
	WHERE booking_status = 'Confirmed'

GROUP BY AL.name, DATE_TRUNC('month', BS.booking_date)
ORDER BY AL.name DESC;


--Task 14: % of canceled / no-show / refunded bookings per airline. 
SELECT AL.name as airline_name,BS.booking_status, COUNT(*) AS status_count,
ROUND(
COUNT(*)*100/ SUM(COUNT(*)) OVER(),2
) 
FROM 
booking_status BS
JOIN
flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
flight_masters FM
ON 
FI.flight_id = FM.flight_id
JOIN 
airlines AL
ON 
FM.airline_code = AL.airline_code
GROUP BY AL.name, BS.booking_status;


--Task 15: Calculate average revenue per flight and per passenger. 
SELECT ROUND(SUM(BS.total_price) / COUNT(DISTINCT FM.flight_id),2) as avg_revenue_per_flight,

Round(SUM(BS.total_price) / COUNT(DISTINCT P.passenger_id),2) as avg_revenue_per_passenger
FROM
passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code


--Task 16 Find correlation between payment method and booking channel success rate. 
SELECT  
	BS.payment_method,
	BS.booking_channel,
	COUNT(*) AS total_status,
	COUNT(*) FILTER (WHERE BS.booking_status = 'Confirmed') as confirmed_booking,
ROUND(
COUNT(*) FILTER (WHERE BS.booking_status = 'Confirmed')*100 / COUNT(*),2
) AS success_rate_percentage
FROM 
passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
GROUP BY BS.booking_channel,  BS.payment_method
ORDER BY success_rate_percentage DESC;



                   -- All tables --

SELECT * FROM passengers;
SELECT * FROM booking_status;
SELECT * FROM airports;
SELECT * FROM airlines;
SELECT * FROM flight_masters;
SELECT * FROM flight_instances;                    
                    
					
					-- Operational Insights --
					  
--Task 17: Flight utilization: average capacity vs. actual bookings by airline.
SELECT AL.name AS airline_name, AVG(FM.capacity) AS average_capacity,
      COUNT(BS.booking_status) as total_booking
FROM 
passengers P
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
where BS.booking_status = 'Confirmed'
GROUP BY airline_name;


--Task 18:Identify busiest airports by departures and arrivals. 
SELECT AP_org.airport_name,COUNT(FI.flight_instance_id) as arrival,
       COUNT(FI.flight_instance_id) as departures
FROM
passengers P 
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
JOIN
airports AP_org
ON FM.origin = AP_org.iata
JOIN
airports AP_des
ON FM.destination = AP_des.iata
GROUP BY  AP_org.airport_name
Order by arrival DESC;


--Task 19: Delays analysis (if actual times exist later) — otherwise estimate efficiency by flight 
SELECT 
			FM.flight_id, ROUND(COUNT(P.passenger_id)*100 / SUM(FM.capacity),2) AS Flight_Efficencey,
	        SUM(BS.total_price) as revenue
FROM
passengers P 
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
JOIN
airports AP_org
ON FM.origin = AP_org.iata
		WHERE BS.booking_status = 'Confirmed'
	    Group BY FM.flight_id
					



-- Task 20: Refund impact: total refunded amount as % of total bookings. 
WITH refunded_booking AS(
SELECT COUNT (booking_status) as refunded
FROM booking_status
WHERE booking_status = 'Refunded'

),

all_booking_status AS(
SELECT COUNT(booking_status) as all_booking
FROM booking_status
)

SELECT ROUND(RB.refunded*100 / AB.all_booking,2) as refund_percentage
FROM
refunded_booking RB
CROSS JOIN
all_booking_status AB;

                                



-- Airline Revenue Summary-- 
CREATE OR REPLACE VIEW airline_revenue_summary AS
SELECT 
	AL.name as airline_name, 
	SUM(BS.total_price) as total_revenue,
	COUNT(BS.booking_status) as total_booking_status,
	SUM (
		CASE WHEN BS.booking_status = 'Refunded' THEN 1 ELSE 0 END
		) AS refunded,
    ROUND(SUM(CASE WHEN BS.booking_status = 'Refunded' THEN 1 ELSE 0 END) * 100,2) / COUNT(BS.booking_status) 
	as refunded_percentage
FROM
passengers P 
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
JOIN
airports AP_org
ON FM.origin = AP_org.iata
GROUP BY airline_name;	



-- Route Performance Summary --
CREATE OR REPLACE VIEW route_performance_Summary AS
SELECT 

	AP_org.iata as origin_airport,
	AP_des.iata as destination_airport,
		COUNT(P.passenger_id) as highest_density_route,
		ROUND(COUNT(P.passenger_id)*100.0 / SUM(FM.capacity), 2) as load_percentage
	     
FROM
passengers P 
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
JOIN
airports AP_org
ON FM.origin = AP_org.iata
JOIN
airports AP_des
ON FM.destination = AP_des.iata
WHERE BS.booking_status = 'Confirmed'
GROUP BY origin_airport, destination_airport
ORDER BY COUNT(P.passenger_id) DESC;



--customer_behavior--
CREATE OR REPLACE VIEW customer_behavior AS 

SELECT
	BS.seat_class, BS.meal_pref, COUNT(P.passenger_id) AS passenger_id,  
	SUM(BS.total_price) AS total_spend_by_each_category
FROM
passengers P 
JOIN
	booking_status BS
ON P.passenger_id = BS.passenger_id
JOIN
	flight_instances FI
ON BS.flight_instance_id = FI.flight_instance_id
JOIN
	flight_masters FM
ON FI.flight_id = FM.flight_id
JOIN 
	airlines AL
ON FM.airline_code = AL.airline_code
JOIN
airports AP_org
ON FM.origin = AP_org.iata
JOIN
airports AP_des
ON FM.destination = AP_des.iata
WHERE BS.booking_status = 'Confirmed'
GROUP BY BS.seat_class, BS.meal_pref
ORDER BY SUM(BS.total_price) 
DESC;



--Booking Eifficiency--

CREATE OR REPLACE VIEW booking_efficiency AS
WITH status_count AS (
SELECT P.nationality as nationality, COUNT(booking_status) AS total_booking_status, 
SUM (CASE WHEN BS.booking_status = 'Confirmed' THEN 1 ELSE 0 END) AS confirmed_status,
SUM (CASE WHEN BS.booking_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_status,
SUM (CASE WHEN BS.booking_status = 'Refunded' THEN 1 ELSE 0 END) AS refunded_status,
SUM (CASE WHEN BS.booking_status = 'No_Show' THEN 1 ELSE 0 END) AS no_show_status
FROM 
passengers P
join
booking_status BS
ON P.passenger_id = BS.passenger_id
GROUP BY P.nationality
)

SELECT 
nationality,
ROUND((confirmed_status)* 100 / total_booking_status,2) AS confirmed_percentage,
ROUND ((cancelled_status)* 100 / total_booking_status,2) AS cancelled_percentage,
ROUND((refunded_status)* 100 / total_booking_status,2) AS refunded_percentage,
ROUND((no_show_status)* 100 / total_booking_status,2) AS no_show_percentage 
FROM 
status_count;

--Smmaries--
SELECT * FROM airline_revenue_summary;
SELECT * FROM route_performance_Summary;
SELECT * FROM customer_behavior;



```
SELECT * FROM booking_efficiency;

