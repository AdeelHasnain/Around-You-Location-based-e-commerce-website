Here’s the database schema extracted in a non-tabular form (pure descriptive structure, nothing omitted, no extra code explanations):

⸻

Database Schema (AroundYou Application)

Prerequisites
	•	PostgreSQL with PostGIS extension enabled.

Enums
	1.	user_type → values: consumer, merchant, admin
	2.	order_status → values: pending, confirmed, ready_for_pickup, out_for_delivery, completed, cancelled
	3.	payment_status → values: paid, unpaid, refunded
	4.	delivery_type → values: pickup, shop_delivery
	5.	discount_type → values: percentage, fixed_amount, buy_x_get_y

⸻

Table: users
	•	user_id (UUID, PK, default random)
	•	email (varchar 255, unique)
	•	phone_number (varchar 20, unique)
	•	password_hash (varchar 255, required)
	•	user_type (enum user_type, required)
	•	first_name (varchar 100)
	•	last_name (varchar 100)
	•	profile_picture_url (varchar 255)
	•	last_known_location (Geography Point, SRID 4326)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Index on last_known_location (GIST)

⸻

Table: shops
	•	shop_id (UUID, PK, default random)
	•	merchant_user_id (UUID, FK → users.user_id, cascade delete)
	•	shop_name (varchar 255, required)
	•	description (text)
	•	address (varchar 255)
	•	location (Geography Point, SRID 4326)
	•	delivery_areas (Geography MultiPolygon, SRID 4326)
	•	phone_number (varchar 20)
	•	email (varchar 255)
	•	category (varchar 100)
	•	opening_hours (JSONB)
	•	status (varchar 20, must be one of open, closed, holiday)
	•	logo_url (varchar 255)
	•	cover_image_url (varchar 255)
	•	free_delivery_threshold (numeric 10,2, default 600.00, required)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Unique constraint on (merchant_user_id, shop_name)
	•	Indexes on location (GIST), delivery_areas (GIST), and category

⸻

Table: products
	•	product_id (UUID, PK, default random)
	•	shop_id (UUID, FK → shops.shop_id, cascade delete)
	•	product_name (varchar 255, required)
	•	description (text)
	•	price (numeric 10,2, required)
	•	currency (varchar 10, default PKR, required)
	•	category (varchar 100)
	•	sub_category (varchar 100)
	•	stock_quantity (integer, default 0, required)
	•	image_urls (array of text)
	•	is_available (boolean, default true, required)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Indexes on shop_id and category

⸻

Table: orders
	•	order_id (UUID, PK, default random)
	•	consumer_id (UUID, FK → users.user_id, restrict delete)
	•	shop_id (UUID, FK → shops.shop_id, restrict delete)
	•	order_date (timestamptz, default now)
	•	status (enum order_status, default pending)
	•	total_amount (numeric 10,2, required)
	•	delivery_fee (numeric 10,2)
	•	final_amount (numeric 10,2)
	•	payment_status (enum payment_status, default unpaid)
	•	payment_method (varchar 50)
	•	delivery_type (enum delivery_type, required)
	•	pickup_time (timestamptz)
	•	delivery_address (JSONB)
	•	delivery_location (Geography Point, SRID 4326)
	•	notes (text)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Indexes on consumer_id, shop_id, status, and delivery_location (GIST)

⸻

Table: order_items
	•	order_item_id (UUID, PK, default random)
	•	order_id (UUID, FK → orders.order_id, cascade delete)
	•	product_id (UUID, FK → products.product_id, restrict delete)
	•	quantity (integer, must be >0, required)
	•	unit_price (numeric 10,2, required)
	•	total_price (numeric 10,2, generated always as quantity * unit_price)
	•	Indexes on order_id and product_id

⸻

Table: reviews
	•	review_id (UUID, PK, default random)
	•	consumer_id (UUID, FK → users.user_id, cascade delete)
	•	shop_id (UUID, FK → shops.shop_id, cascade delete, nullable)
	•	product_id (UUID, FK → products.product_id, cascade delete, nullable)
	•	rating (integer, must be between 1 and 5, required)
	•	comment (text)
	•	created_at (timestamptz, default now)
	•	Constraint: either shop_id or product_id must be present (at least one not null)
	•	Indexes on shop_id and product_id

⸻

Table: deals
	•	deal_id (UUID, PK, default random)
	•	shop_id (UUID, FK → shops.shop_id, cascade delete)
	•	title (varchar 255, required)
	•	description (text)
	•	discount_type (enum discount_type, required)
	•	discount_value (numeric 10,2, required)
	•	start_date (timestamptz, required)
	•	end_date (timestamptz, required)
	•	min_order_amount (numeric 10,2)
	•	usage_limit (integer)
	•	is_active (boolean, default true, required)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Indexes on shop_id and is_active

⸻

Table: delivery_fee_tiers
	•	tier_id (UUID, PK, default random)
	•	min_distance_m (integer, required)
	•	max_distance_m (integer, nullable)
	•	fee_amount (numeric 10,2, required)
	•	per_extra_distance_m (integer)
	•	per_extra_fee (numeric 10,2)
	•	is_active (boolean, default true, required)
	•	created_at (timestamptz, default now)
	•	updated_at (timestamptz, default now)
	•	Index on is_active

⸻

Triggers
	•	Function set_updated_at() updates updated_at column automatically on update.
	•	Triggers attached to: users, shops, products, orders, deals, delivery_fee_tiers.

• Indexing: Sp a tial indexes (GiST indexes) will b e cre a ted on the loc a tion a nd delivery_are as columns in the Shops t a ble, a nd last_known_loc a tion in Users t a ble, a nd delivery_loc a tion in Orders t a ble to optimize geosp a tial queries. • Multi-Shop M n a a gement: The merch a nt_user_id in the Shops t a ble directly supports one-to-m a a ny rela tionship, allowing a single user to own nd m n a a a ge multiple shops. Queries will b e structured to retrieve all shops associa ted with a given merch a nt user. • Delivery Fee Logic: The b a ckend will query the Delivery_Fee_Tiers t a ble to determine the a ppropria te delivery fee b ased on the dist nce c a alcula ted b y Google M a ps Dist a nce M a trix API. The free_delivery_threshold in the Shops t a ble will b e checked g a ainst the order tot al to w aive the delivery fee if a pplic a ble.