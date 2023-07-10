#### [Case Study: How Does A Bike-Share Navigate Speedy Success?](https://d3c33hcgiwev3.cloudfront.net/aacF81H_TsWnBfNR_x7FIg_36299b28fa0c4a5aba836111daad12f1_DAC8-Case-Study-1.pdf?Expires=1689120000&Signature=Js-MRPXHfBlfHFkV5J9dG3~LanLPccZd0uC4hdoigTLtf~f3~YXUigUZuUGQ6mQdOr2qqi-naqxH-oD125Cdk~C7VTeG3PasE~iYmXzIBTgVNeSSTVm6WVwz9Sz8J3iPGtjCcvmmH0mfylMcBCmiJO8e5G0m0anSociswwEKMUo_&Key-Pair-Id=APKAJLTNE6QMUY6HBC5A)

Trong Case Study này tôi vào vai 1 Junior Data Analyst tại 1 công ty giả định - Cyclist, với nhiệm vụ phân tích dữ liệu 5 tháng đầu năm 2023, hỗ trợ trả lời các business questions cho chiến dịch marketing sắp tới của công ty

Tôi sẽ thực hiện theo các bước của quá trình phân tích dữ liệu: Ask, Prepare, Process, Analyze và Share

--- 

## I. Ask

### Business Task

Đưa ra các chiến lược tiếp thị để chuyển đổi người đi xe bình thường thành thành viên

### Câu hỏi phân tích

1. Các thành viên hàng năm và người đi xe đạp bình thường sử dụng xe đạp Cyclistic khác nhau như thế nào?

2. Tại sao những tay đua bình thường lại mua tư cách thành viên hàng năm của Cyclistic?

3. Làm thế nào Cyclistic có thể sử dụng phương tiện kỹ thuật số để tác động đến những tay đua bình thường trở thành thành viên?

Moreno đã giao cho tôi câu hỏi đầu tiên để trả lời: Các thành viên hàng năm và những người đi xe đạp bình thường sử dụng xe đạp Cyclistic khác nhau như thế nào?

---

## II. Prepare

### Resource

Tôi sử dụng dữ liệu lịch sử của Cyclistc để thưc hiện phân tích và xác định xu hướng trong 5 năm đầu năm 2023

Dữ liệu được đây từ [divvy_tripdata](https://divvy-tripdata.s3.amazonaws.com/index.html). Được cung cấp bởi Motivate International theo [giấy phép](https://ride.divvybikes.com/data-license-agreement) này

---

## III. Process

### 1. Combine Data

Sử dụng dữ liệu từ 01/2023 đến 05/2023

- [divvy_tripdata_01_2023](https://divvy-tripdata.s3.amazonaws.com/202301-divvy-tripdata.zip)

- [divvy_tripdata_02_2023](https://divvy-tripdata.s3.amazonaws.com/202302-divvy-tripdata.zip)

- [divvy_tripdata_03_2023](https://divvy-tripdata.s3.amazonaws.com/202303-divvy-tripdata.zip)

- [divvy_tripdata_04_2023](https://divvy-tripdata.s3.amazonaws.com/202304-divvy-tripdata.zip)

- [divvy_tripdata_05_2023](https://divvy-tripdata.s3.amazonaws.com/202305-divvy-tripdata.zip)

#### Kết nối đến SQL Server trong R language

Tải Packages kết nối với SQL
```
install.packages("odbc")

install.packages("DBI")

install.packages("tidiverse")
```
Trong đó:

- Package "odbc" cho phép kết nối với cơ sở dữ liệu

- Package "DBI" cho phép tương tác với cơ sở dữ liệu


Sử dụng thư viện của 2 package

``` {r}
library(odbc) 

library(DBI)

library(ggplot2)
```

Kết nối với SQL Server

```{r}
con = odbc::dbConnect(odbc(), Driver = "SQL Server", 
                       Server = "GNOUD481902\\SQLEXPRESS", 
                       Database = "Case_Study_1", 
                       Trusted_Connection = "True")
```

Kết hợp dữ liệu của 5 bảng thành 1 bảng duy nhất

```{sql connection = con}
IF NOT EXISTS (SELECT 1 FROM sys.tables WHERE name = 'divvy_tripdata_2023')
BEGIN
	SELECT * INTO divvy_tripdata_2023 
	FROM (
		SELECT * FROM divvy_tripdata_01_2023
		UNION ALL
		SELECT * FROM divvy_tripdata_02_2023
		UNION ALL
		SELECT * FROM divvy_tripdata_03_2023
		UNION ALL
		SELECT * FROM divvy_tripdata_04_2023
		UNION ALL
		SELECT * FROM divvy_tripdata_05_2023 ) AS row_data
END

-- Đếm số lượng records trong bảng divvy_tripdata_2023
SELECT COUNT(*) AS count
FROM divvy_tripdata_2023
```

Như vậy, bảng 'divvy_tripdata_2023' đã được tạo với **1,607,841 records**


### 2. Data Exploration

Bảng bên dưới hiển thị tất cả tên trường và kiểu dữ liệu của bảng 'divvy_tripdata_2023'
![](https://scontent.fhan14-2.fna.fbcdn.net/v/t1.15752-9/358472608_254637580627001_2310393048757323259_n.png?_nc_cat=106&ccb=1-7&_nc_sid=ae9488&_nc_ohc=qknQ-AkgjgkAX8TIDlu&_nc_ht=scontent.fhan14-2.fna&oh=03_AdRU8vFuHgAC-HDrLR1kpkaEbmewTcMvuAwYO7YGyAf1_w&oe=64D26DE7)

Bảng dưới đây cho biết số lượng giá trị **NULL** trong mỗi cột

```{sql, connection = con}
SELECT
	COUNT(CASE WHEN ride_id IS NULL THEN 1 ELSE NULL END) AS ride_id,
	COUNT(CASE WHEN rideable_type IS NULL THEN 1 ELSE NULL END) AS rideable_type,
	COUNT(CASE WHEN started_at IS NULL THEN 1 ELSE NULL END) AS started_at,
	COUNT(CASE WHEN ended_at IS NULL THEN 1 ELSE NULL END) AS ended_at,
	COUNT(CASE WHEN start_station_name IS NULL THEN 1 ELSE NULL END) AS start_station_name,
	COUNT(CASE WHEN start_station_id IS NULL THEN 1 ELSE NULL END) AS start_station_id,
	COUNT(CASE WHEN end_station_name IS NULL THEN 1 ELSE NULL END) AS end_station_name,
	COUNT(CASE WHEN end_station_id IS NULL THEN 1 ELSE NULL END) AS end_station_id,
	COUNT(CASE WHEN start_lat IS NULL THEN 1 ELSE NULL END) AS start_lat,
	COUNT(CASE WHEN start_lng IS NULL THEN 1 ELSE NULL END) AS start_lng,
	COUNT(CASE WHEN end_lat IS NULL THEN 1 ELSE NULL END) AS end_lat,
	COUNT(CASE WHEN end_lng IS NULL THEN 1 ELSE NULL END) AS end_lng,
	COUNT(CASE WHEN member_casual IS NULL THEN 1 ELSE NULL END) AS member_casual
FROM divvy_tripdata_2023
```

**Nhận xét:**

  - Cột *start_station_name* có **241,158** giá trị **NULL** nên cần được loại bỏ
  - Cột *start_station_id* có **241,290** giá trị **NULL** nên cần được loại bỏ
  - Cột *end_station_name* có **256,913** giá trị **NULL** nên cần được loại bỏ
  - Cột *end_station_id* có **257,054** giá trị **NULL** nên cần được loại bỏ
  - Cột *end_lat* và *end_lng* đều có **1,571** giá trị **NULL** nên cần được loại bỏ
  
Vì cột *rise_id* là 1 **Primary Key** nên sẽ không có giá trị bị trùng lặp


### 3. Data Cleaning

Định dạng hiện tại của bảng divvy_tripdata_2023 là yyyy-mm-dd hh:mm:ss, để phân tích sâu hơn tôi sẽ tạo 4 cột mới với từng kiểu giá trị tương ứng lần lượt là  *Hour*, *Day of Week*, *Month* và *Year*

Tạo 1 cột mới mang tên *rise_length* để xác định độ dài thời gian mỗi chuyến đi (Định dạng phút)

Tất cả các records chứa giá trị **NULL** đều bị loại bỏ

```{sql connection = con}
IF NOT EXISTS (SELECT 1 FROM sys.tables WHERE name = 'pre_clean_combined_data')
BEGIN
	SELECT * INTO pre_clean_combined_data
	FROM (
		SELECT ride_id, rideable_type,
		  CAST(DATENAME(hh, started_at) AS INT) AS hour,
			DATENAME(dw, started_at) AS day_of_week,
			DAY(started_at) AS day,
			MONTH(started_at) AS month,
			YEAR(started_at) AS year,
			-- Ép kiểu dữ liệu kết quả của hàm DateDiff với kết quả làm tròn của phép chia đến 2 chữ số thập phân
			ROUND(CAST(DATEDIFF(s, started_at, ended_at) AS float)/60, 2) AS ride_length_minute,
			start_station_name, end_station_name, start_lat, start_lng, 
			end_lat, end_lng, member_casual
		FROM divvy_tripdata_2023
		WHERE start_station_name IS NOT NULL AND
			start_station_id IS NOT NULL AND
			end_station_name IS NOT NULL AND
			end_station_id IS NOT NULL AND
			end_lat IS NOT NULL AND
			end_lng IS NOT NULL
	) AS cleaned
END

-- Sắp xếp ride_length_minute theo chiều tăng dần để kiểm tra 
SELECT * FROM pre_clean_combined_data
ORDER BY ride_length_minute ASC
```

**Nhận xét:**

- Kết quả trả về trong cột ride_length_minute có 8 giá trị mang giá trị < 0
- Nguyên nhân có thể do lỗi người dùng hoặc do lỗi hệ thống khi chèn dữ liệu vào Database


```{sql connection = con}
-- Cần loại bỏ những records mà tại đó giá trị ride_length_minute < 0

IF NOT EXISTS (SELECT 1 FROM sys.tables WHERE name = 'clean_combined_data')
BEGIN
	SELECT * INTO clean_combined_data
	FROM (
		SELECT * FROM pre_clean_combined_data
		wHERE ride_length_minute >= 0 ) AS completely_clean
END

-- Đếm số lượng records trong bảng clean_combined_data sau khi đã làm sạch
SELECT COUNT(*) AS count
FROM clean_combined_data
```

Như vậy, sau bước làm sạch dữ liệu sẽ thu được bảng mới mang tên "clean_combined_data" với **1,285,707 records**

---

## IV. Analyze And Share

### 1. Phân tích thống kê

```{r}
query = "SELECT
	          member_casual,ROUND(AVG(ride_length_minute), 2) AS avg_ride_length
         FROM clean_combined_data
         GROUP BY member_casual"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization

ggplot(data = result) +
  geom_bar(stat = "identity", mapping = aes(x = member_casual, y = avg_ride_length, fill = member_casual)) +
  geom_text(aes(x = member_casual, y = avg_ride_length, label = avg_ride_length), vjust = -0.25)
```


### 2. Xác định số lượng thành viên hàng năm và những người đi xe đạp bình thường

```{r}
query = "SELECT member_casual, count(member_casual) AS total_riders
            FROM clean_combined_data
            GROUP BY member_casual"

riders_rs = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization

ggplot(data = riders_rs) +
  geom_bar(stat = "identity", mapping = aes(x = member_casual, y = total_riders, fill = member_casual)) +
  geom_text(aes(x = member_casual, y = total_riders, label = total_riders), vjust = -0.25)
```


### 3. Xác định số lượng thành viên hàng năm và những người đi xe đạp bình thường theo kiểu xe mà họ sử dụng

```{r}
query = "SELECT 
	          member_casual, rideable_type,
	          COUNT(*) AS total_trips
         FROM clean_combined_data
         GROUP BY member_casual, rideable_type"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization

ggplot(result, aes(x = member_casual, y = total_trips, fill = rideable_type)) +
  geom_col(position = "dodge") +
  geom_text(aes(label = total_trips), position = position_dodge(width = 0.9), vjust = -0.25)
```


### 4. Xác định tổng thời gian di chuyển mỗi chuyến đi của thành viên hàng năm và những người đi xe đạp bình thường theo tháng, theo ngày trong tuần và theo giờ

```{r}
# Truy vấn dữ liệu về tổng thời gian di chuyển của các riders theo ngày trong tháng
query = "SELECT 
	          member_casual, month,
          	ROUND(SUM(ride_length_minute), 2) AS total_trips
         FROM clean_combined_data
         GROUP BY month, member_casual
         ORDER BY member_casual, month"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = month, y = total_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In First 5 months of 2023", subtitle = "Per Month")
```


**Nhận xét:** Với biểu đồ trực quan "Total Trips In First 5 months of 2023", cả thành viên hàng năm và những người đi xe đạp bình thường đều thể hiện hành vi tương tự nhau, với ít việc chuyến đi vào mùa xuân và tăng dần vào các tháng mùa hè


```{r }
# Truy vấn dữ liệu về tổng thời gian di chuyển của các riders theo ngày trong tuần
query = "SELECT member_casual, day_of_week,
            CASE day_of_week
                WHEN 'Monday' THEN 2
                WHEN 'Tuesday' THEN 3
                WHEN 'Wednesday' THEN 4
                WHEN 'Thursday' THEN 5
                WHEN 'Friday' THEN 6
                WHEN 'Saturday' THEN 7
                WHEN 'Sunday' THEN 8
            END AS day_order,
	          ROUND(SUM(ride_length_minute), 2) AS total_trips
         FROM clean_combined_data
         GROUP BY day_of_week, member_casual
         ORDER BY member_casual, day_order"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = day_order, y = total_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In Day of Week", subtitle = "Per Day of Week")
```


**Nhận xét:** Với biểu đồ trực quan "Total Trips In Day of Week", các thành viên hàng năm và những người đi xe đạp bình thường có hành vi di chuyển trái ngược nhau. Với các thành viên hàng năm họ có thời gian di chuyển vào các ngày đầu tuần và có xu hướng giảm dần từ thứ 4 trở đi, còn đối với những người đi xe đạp bình thường họ có thời gian di chuyển ở các ngày trong tuần và có xu hướng tăng mạnh vào 2 ngày cuối tuần


```{r}
# Truy vấn dữ liệu về tổng thời gian di chuyển của các riders theo giờ
query = "SELECT 
	          member_casual, hour,
          	ROUND(SUM(ride_length_minute), 2) AS total_trips
         FROM clean_combined_data
         GROUP BY hour, member_casual
         ORDER BY member_casual, hour"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = hour, y = total_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In Hour", subtitle = "Per Hour")
```

**Nhận xét:** Với biểu đồ trực quan "Total Trips In Hour", các thành viên hàng năm và những người đi xe đạp bình thường cũng đều thể hiện hành vi tương tự nhau, họ gần như không di chuyển trong khoảng thời gian từ 0h đến 5h sáng, tăng thời gian di chuyển sau 5h và di chuyển nhất trong khoảng 15h đến 17h


### 5. Xác định thời gian di chuyển trung bình mỗi chuyến đi của các thành viên hàng năm và những người đi xe đạp bình thường theo tháng, theo ngày trong tuần và theo giờ

```{r}
# Truy vấn dữ liệu về thời gian di chuyển trung bình của các riders theo tháng
query = "SELECT 
	          member_casual, month,
          	ROUND(AVG(ride_length_minute), 2) AS AVG_trips
         FROM clean_combined_data
         GROUP BY month, member_casual
         ORDER BY member_casual, month"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = month, y = AVG_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In First 5 months of 2023", subtitle = "Per Month")
```

```{r}
# Truy vấn dữ liệu về thời gian di chuyển trung bình của các riders theo ngày trong tuần
query = "SELECT member_casual, day_of_week,
            CASE day_of_week
                WHEN 'Monday' THEN 2
                WHEN 'Tuesday' THEN 3
                WHEN 'Wednesday' THEN 4
                WHEN 'Thursday' THEN 5
                WHEN 'Friday' THEN 6
                WHEN 'Saturday' THEN 7
                WHEN 'Sunday' THEN 8
            END AS day_order,
	          ROUND(AVG(ride_length_minute), 2) AS AVG_trips
         FROM clean_combined_data
         GROUP BY day_of_week, member_casual
         ORDER BY member_casual, day_order"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = day_order, y = AVG_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In Day of Week", subtitle = "Per Day of Week")
```

```{r}
# Truy vấn dữ liệu về thời gian di chuyển trung bình của các riders theo giờ
query = "SELECT 
	          member_casual, hour,
          	ROUND(AVG(ride_length_minute), 2) AS AVG_trips
         FROM clean_combined_data
         GROUP BY hour, member_casual
         ORDER BY member_casual, hour"

result = dbGetQuery(con, query)

# Xử lý dữ liệu và tạo visualization
ggplot(data = result, aes(x = hour, y = AVG_trips, group = as.factor(member_casual), color = member_casual)) +
  geom_line() + geom_point() +
  labs(title = "Total Trips In Hour", subtitle = "Per Hour")
```

**Nhận xét:** Qua 3 biểu đồ trực quan trên, nhận thấy rằng thời gian di chuyển của các thành viên hàng năm có xu hướng không đổi theo giờ, ngày trong tuần hoặc tháng. Trong khi đó những người đi xe đạp bình thường có xu hướng di chuyển nhiều hơn gấp 2 lần, họ có xu hướng di chuyển nhiều hơn từ khung giờ 8h đến 14h, vào ngày cuối tuần và trong những tháng hè

---

### Tài liệu tham khảo
[Kaggle](https://www.kaggle.com/code/mmaguire/google-data-analytics-capstone-case-study-1#Introduction:)
