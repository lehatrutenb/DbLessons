// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Table users {
  id integer [primary key]
  username varchar
  adress varchar // optional - if user wants to create order - he must have it
  role_id integer
}

// user-buyer courier partnerWorker support admin
// давайте на начальных этапах выдадим partnerWorker идентичные права
// ??? далее это будет несложно видоизменить добавив табличку партнёры-роли
Table roles {
  id integer [primary key]
  role varchar
}

Table usersForPartners {
  user_id integer
  partner_id integer
}

// точки предоставляющие свои услуги нашему сервису
Table partner {
  id integer [primary key]
  name varchar
}

Table partnerPoint {
  id integer [primary key]
  partner_id integer // знаем какого конкретного партнёра точка
  address varchar // не уверен насколько хорошо адресс указывать как строку - KISS
  ApiAddress varchar // where to send reqs for dishes
  creditionals varchar // where to pay for orders
}

// мега сорян за такой спам полями - не знаю как сделать лучше
// но точно понимаю, что в разные дни обычно разное время работы
// к тому же праздники и тд
Table partnerPointTimetable {
  partnerPoint_id integer // не primary key а такая структура тк думаю так будет легче метрики и тд строить
  mondayWorkTimeStart timestamp // не уверен с типом
  mondayWorkTimeEnd timestamp
  tuesdayWorkTimeStart timestamp // не уверен с типом
  tuesdayWorkTimeEnd timestamp
  wednesdayWorkTimeStart timestamp // не уверен с типом
  wednesdayWorkTimeEnd timestamp
  thursdayWorkTimeStart timestamp // не уверен с типом
  thursdayWorkTimeEnd timestamp
  fridayWorkTimeStart timestamp // не уверен с типом
  fridayWorkTimeEnd timestamp
  saturdayWorkTimeStart timestamp // не уверен с типом
  saturdayWorkTimeEnd timestamp
  sundayWorkTimeStart timestamp // не уверен с типом
  sundayWorkTimeEnd timestamp
}

// не хочется рассматривать продукты которые только через время
// дойдут до самой точки партнёра - YAGNI
// предположительно часто меняется, но не так часто как данные относительно заказов
Table productsForPartnerPoint {
  product_id integer
  partnerPoint_id integer
  amount integer
  price float // у разных точек может быть разная цена и это вроде бы часто встречается в жизни
}

Table productsForPartner {
  product_id integer
  partner_id integer
}

// товар-продукт-выставленный слот для достваки привязан к 
// партнёру из предположения, что нет идентичных продуктов
// и у каждого продукта есть партнёр, который его готов выдать
Table product {
  id integer [primary key]
  productDescription_id  integer
}

Table productDescription {
  id integer [primary key]
  name varchar
  shortText varchar
  longText varchar
}

// хорошо бы ещё разное ккачество картинок учесть - но KISS
Table picsForProductDescription {
  productDescription_id integer
  url varchar
}

// а теперь забьём на небольшие детали и вернёмся к заказам

Table couriers {
  id integer [primary key]
  workNow bool
  workedInMonth integer // to calc payments
  docs varchar // path to another db with some docs for his employment
}

Table courierCords { // approx values not to to lots of useless work - for example last req end
  courier_id integer
  latitude float
  longitude float
}

Table orders {
  id integer [primary key]
  courier_id integer
  partnerPoint_id integer
  user_id integer
  state_id integer
  created_at timestamp
}

// states: created, waitToBeDone, waitForCourier, inPath, Done, canceled
Table orderStates {
  id integer [primary key]
  state varchar
}


Table productsForOrder {
  order_id integer
  product_id integer
}

Ref: users.role_id > roles.id
Ref: users.id > usersForPartners.user_id
Ref: partner.id > usersForPartners.partner_id
Ref: productDescription.id < picsForProductDescription.productDescription_id
Ref: product.productDescription_id - productDescription.id
Ref: productsForPartnerPoint.product_id < product.id
Ref: productsForPartnerPoint.partnerPoint_id > partnerPoint.id
Ref: productsForPartner.product_id < product.id
Ref: productsForPartner.partner_id > partner.id
Ref: partnerPointTimetable.partnerPoint_id - partnerPoint.id
Ref: couriers.id - courierCords.courier_id
Ref: couriers.id - orders.courier_id
Ref: users.id - orders.user_id
Ref: orders.state_id - orderStates.id
Ref: orders.id - productsForOrder.order_id
Ref: orders.partnerPoint_id - partnerPoint.id
Ref: productsForOrder.product_id - product.id