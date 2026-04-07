# Instagram Thrift & Handmade Store - ER Diagram
```
title Instagram Thrift Creator Store ERD
users [icon: user, color: blue] {
  id serial pk
  fname string
  lname string null
  email string unique not null
  phone char(15) unique not null
  password varchar(256) null
  auth enum('local', 'google') not null

  createdAt timestamp
  updatedAt timestamp
}

// In a one-to-many, the FK lives on the many side. Only. That's enough.
addresses [icon: map-pin, color: blue] {
  address_id serial pk
  user_id fk 
  addressLane string not null
  area string not null
  city string not null
  state string not null
  country string not null
  pincode int not null
  type enum('home', 'shipping', 'billing')
  
  createdAt timestamp
  updatedAt timestamp
}

// All the common things lie here
products [icon: box, color: yellow] {
  product_id serial pk
  name string not null
  price int not null
  category text not null
  size text
  color text
  description text
  type enum('thrifted', 'handmade')

  createdAt timestamp
  updatedAt timestamp
}

thrifted_items [icon: shirt, color: green] {
  thrifted_item_id serial pk
  product_id fk
  condition enum('new', 'like_new', 'good', 'worn')
  is_sold boolean

  createdAt timestamp
  updatedAt timestamp
}

handmade_items [icon: tree, color: green] {
  handmade_item_id serial pk
  product_id fk
  stock_quantity int

  createdAt timestamp
  updatedAt timestamp
}

product_photos [icon: camera, color: green] {
  photo_id serial pk
  product_id int fk
  photo_url string
  position int

  createdAt timestamp
  updatedAt timestamp
}

orders [icon: boxes, color: orange] {
  order_id serial pk
  orderCreatedAt timestamp
  orderUpdatedAt timestamp
  address_id fk
  user_id fk

  createdAt timestamp
  updatedAt timestamp
}

order_items [icon: package, color: orange] {
  item_id serial pk
  order_id fk
  product_id fk 
  quantity int
  price_at_purchase int

  createdAt timestamp
  updatedAt timestamp
}

payments [icon: payment, color: red] {
  payment_id serial pk
  order_id fk
  payment_type enum('COD', 'UPI', 'CARD')
  status enum('pending', 'paid', 'failed')

  createdAt timestamp
  updatedAt timestamp
}

shipping [icon: truck, color: purple] {
  shipping_id serial pk
  order_id fk
  deliveryPartner string
  tracking_url string
  status enum('PENDING', 'SHIPPED', 'OTD', 'DELIVERED')
  
  createdAt timestamp
  updatedAt timestamp
}

shipping_status_history [icon: clock, color: purple] {
  history_id serial pk
  shipping_id fk
  status enum('PENDING', 'SHIPPED', 'OTD', 'DELIVERED')
  changed_at timestamp not null
}


// one user can have multiple orders
users.id < orders.user_id

// one user can have multiple addresses
users.id < addresses.user_id

// one order can have multiple order items
orders.order_id < order_items.order_id

// one order can have multiple payment attempts
orders.order_id < payments.order_id

// one order can have multiple shipments
orders.order_id < shipping.order_id

// one shipment can have many shipment status history
shipping.shipping_id < shipping_status_history.shipping_id


// one product can have one handmade items
products.product_id - handmade_items.product_id

// one product can have one thrifted items
products.product_id - thrifted_items.product_id


// Multiple products photos can be of a single product
product_photos.product_id > products.product_id

// Multiple products can be ordered for a particular product
order_items.product_id > products.product_id

// Multiple orders can be from same address
orders.address_id > addresses.address_id
```