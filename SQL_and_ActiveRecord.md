1) How many users are there? 

```
irb(main):002:0> User.count
```
As shown below with the related SQL command, there are 50 users:

```
   (0.2ms)  SELECT COUNT(*) FROM "users"
=> 50
```

2) What are the 5 most expensive items?

```
irb(main):016:0> Item.select(:id, :title).order(price: :desc).limit(5)
```

The result shows the 5 most-expensive items with the item's id and title:

```
Item Load (0.4ms)  SELECT  id, title FROM "items" ORDER BY "items"."price" DESC LIMIT ?  [["LIMIT", 5]]
=> #<ActiveRecord::Relation [
#<Item id: 25, title: "Small Cotton Gloves">, 
#<Item id: 83, title: "Small Wooden Computer">, 
#<Item id: 100, title: "Awesome Granite Pants">, 
#<Item id: 40, title: "Sleek Wooden Hat">, 
#<Item id: 60, title: "Ergonomic Steel Car">]>
```

3) What's the cheapest book?

```
irb(main):054:0> Item.select(:id, :title).where("category LIKE '%Books%'").order(price: :asc).limit(1)
```

```
Item Load (0.4ms)  SELECT  "items"."id", "items"."title" FROM "items" WHERE (category LIKE '%Books%') ORDER BY "items"."price" ASC LIMIT ?  [["LIMIT", 1]] 
=> #<ActiveRecord::Relation [#<Item id: 76, title: "Ergonomic Granite Chair">]>
```

4) Who lives at "6439 Zetta Hills, Willmouth, WY"? Do they have another address?

```
User.find(Address.select(:user_id).find_by(street: "6439 Zetta Hills").user_id)

OR

User.joins("INNER JOIN addresses ON addresses.user_id = users.id AND addresses.street = '6439 Zetta Hills'").first
```
RESULTS:

```
THE FIRST RUNS TWO QUERIES:
Address Load (0.1ms)  SELECT  "addresses"."user_id" FROM "addresses" WHERE "addresses"."street" = ? LIMIT ?  [["street", "6439 Zetta Hills"], ["LIMIT", 1]]
  
User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 40], ["LIMIT", 1]]
=> #<User:0x007ff7c46491f0 id: 40, first_name: "Corrine", last_name: "Little", email: "rubie_kovacek@grimes.net">

THE SECOND ONLY RUNS ONE:
User Load (0.3ms)  SELECT  "users".* FROM "users" INNER JOIN addresses ON addresses.user_id = users.id AND addresses.street = '6439 Zetta Hills' ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 1]]

=> #<User:0x007ff7c452c3d0 id: 40, first_name: "Corrine", last_name: "Little", email: "rubie_kovacek@grimes.net">
```

GET ALL OF CORRINE'S ADDRESSES:

```
Address.where(user_id: (User.select(:id).find(Address.select(:user_id).find_by(street: "6439 Zetta Hills").user_id)))
```

```
TRIPLE QUERY:
Address Load (0.1ms)  SELECT  "addresses"."user_id" FROM "addresses" WHERE "addresses"."street" = ? LIMIT ?  [["street", "6439 Zetta Hills"], ["LIMIT", 1]]

User Load (0.1ms)  SELECT  "users"."id" FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 40], ["LIMIT", 1]]

Address Load (0.1ms)  SELECT "addresses".* FROM "addresses" WHERE "addresses"."user_id" = 40

=> [#<Address:0x007ff7c0f70020 id: 43, user_id: 40, street: "6439 Zetta Hills", city: "Willmouth", state: "WY", zip: 15029>,
 #<Address:0x007ff7c0f6bb10 id: 44, user_id: 40, street: "54369 Wolff Forges", city: "Lake Bryon", state: "CA", zip: 31587>]
```

5) Correct Virginie Mitchell's address to "New York, NY, 10108".

```
Address.joins("INNER JOIN users ON users.id = addresses.user_id").where("users.first_name = 'Virginie' AND state = 'NY'").update(city: 'New York', zip: '10108')
```

```
Address Load (0.2ms)  SELECT "addresses".* FROM "addresses" INNER JOIN users ON users.id = addresses.user_id WHERE (users.first_name = 'Virginie' AND state = 'NY')
   (0.1ms)  begin transaction
  SQL (0.8ms)  UPDATE "addresses" SET "city" = ?, "zip" = ? WHERE "addresses"."id" = ?  [["city", "New York"], ["zip", 10108], ["id", 41]]
   (1.0ms)  commit transaction
=> [#<Address:0x007ff7c40f21c0 id: 41, user_id: 39, street: "12263 Jake Crossing", city: "New York", state: "NY", zip: 10108>]
```

6) How much would it cost to buy one of each tool?

```
Item.where("category LIKE '%Tools%'").distinct.sum("price")

   (0.3ms)  SELECT DISTINCT SUM("items"."price") FROM "items" WHERE (category LIKE '%Tools%')
=> 46477
```

7) How many total items did we sell?

```
Item.joins("INNER JOIN orders ON orders.item_id = items.id").sum("price * quantity")

   (0.4ms)  SELECT SUM(price * quantity) FROM "items" INNER JOIN orders ON orders.item_id = items.id
=> 10045128
```

8) How much was spent on books?

```
Item.joins("INNER JOIN orders ON orders.item_id = items.id").where("category LIKE '%Books%'").sum("price")

   (0.4ms)  SELECT SUM("items"."price") FROM "items" INNER JOIN orders ON orders.item_id = items.id WHERE (category LIKE '%Books%')
=> 180356
```

9) Simulate buying an item by inserting a User for yourself and an Order for that User.

```
User.create(first_name: 'Rob', last_name: 'Luckfield', email: 'someemail@hotmail.co.uk')

Order.create(user_id: User.where("last_name LIKE '%Luck%'").pluck(:id).first, item_id: '2', quantity: '10')
   
   
   (0.2ms)  SELECT "users"."id" FROM "users" WHERE (last_name LIKE '%Luck%')
   (0.1ms)  begin transaction
  SQL (0.3ms)  INSERT INTO "orders" ("user_id", "item_id", "quantity", "created_at") VALUES (?, ?, ?, ?)  [["user_id", 51], ["item_id", 2], ["quantity", 10], ["created_at", 2016-07-07 21:20:57 UTC]]
   (2.0ms)  commit transaction
=> #<Order:0x007ff7c404a150 id: 383, user_id: 51, item_id: 2, quantity: 10, created_at: Thu, 07 Jul 2016 21:20:57 UTC +00:00>
```