# Instagram Thrift & Handmade Store - ER Diagram
```

// ─────────────────────────────────────────────
// USERS & ADDRESSES
// ─────────────────────────────────────────────

users [icon: user, color: blue] {
  id serial pk
  fname string
  lname string null
  email string unique not null
  phone char(15) unique not null
  password varchar(256)

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

// ─────────────────────────────────────────────
// PRODUCTS
// ─────────────────────────────────────────────
 
// All the common attributes live here
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

// Unique, one-of-a-kind thrifted items
thrifted_items [icon: shirt, color: green] {
  thrifted_item_id serial pk
  product_id fk
  condition enum('new', 'like_new', 'good', 'worn')
  is_sold boolean
}

// Handmade items that can be restocked
handmade_items [icon: tree, color: green] {
  handmade_item_id serial pk
  product_id fk
  stock_quantity int
}

// Multiple photos per product listing
product_photos [icon: camera, color: green] {
  photo_id serial pk
  product_id int fk
  photo_url string
  position int

  createdAt timestamp
  updatedAt timestamp
}

// ─────────────────────────────────────────────
// ORDERS
// ─────────────────────────────────────────────

orders [icon: boxes, color: orange] {
  order_id serial pk
  orderCreatedAt timestamp
  orderUpdatedAt timestamp
  address_id fk
  user_id fk
}

// one order can have multiple products
order_items [icon: package, color: orange] {
  item_id serial pk
  order_id fk
  product_id fk 
  quantity int
  price_at_purchase int
}

// ─────────────────────────────────────────────
// PAYMENTS
// ─────────────────────────────────────────────

payments [icon: payment, color: red] {
  payment_id serial pk
  order_id fk
  payment_type enum('COD', 'UPI', 'CARD')
  status enum('pending', 'paid', 'failed')
  paymentTransferedAt timestamp
}

// ─────────────────────────────────────────────
// SHIPPING
// ─────────────────────────────────────────────

shipping [icon: truck, color: purple] {
  shipping_id serial pk
  order_id fk
  deliveryPartner string
  tracking_url string
  status enum('PENDING', 'SHIPPED', 'OTD', 'DELIVERED')
  pending_at timestamp null
  shipped_at timestamp null
  otd_at timestamp null
  delivered_at timestamp null
}

// ─────────────────────────────────────────────
// RELATIONSHIPS
// ─────────────────────────────────────────────

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

// one product can have one handmade items
products.product_id - handmade_items.product_id

// one product can have one thrifted items
products.product_id - thrifted_items.product_id

// Multiple products photos can be of a single product
product_photos.product_id > products.product_id

// Multiple products can be ordered for a particlar product
order_items.product_id > products.product_id

// Multiple orders can be from same address
orders.address_id > addresses.address_id
```