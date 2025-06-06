# harukaedu

Step:
1. Import Collection: /Auth
2. create scripts :
var jsonData = pm.response.json();
pm.environment.set("token", jsonData.token);
4. set up environment:
  name : Lokal
  Variable : token
  type: default
  current value: [automatically filled when hit api /auth]
5. hit collection /auth
6. import collection create booking
   set body :
   {
    "firstname" : "{{firstname}}", --> dynamic firstname
    "lastname" : "{{lastname}}", --> dynamic lastname
    "totalprice" : 111,
    "depositpaid" : true,
    "bookingdates" : {
        "checkin" : "{{checkin}}", -->  checkin date by today
        "checkout" : "{{checkout}}" -->  checkout +7 days
    },
    "additionalneeds" : "Breakfast"
}
  scripts :

pre-req:
// Generate random name
const randomNum = Math.floor(Math.random() * 1000);
pm.environment.set("firstname", `Jim${randomNum}`);
pm.environment.set("lastname", `Brown${randomNum}`);


// Fungsi bantu untuk format tanggal ke YYYY-MM-DD
function formatDate(date) {
    return date.toISOString().split('T')[0];
}

//  check-in today
let today = new Date();
let checkin = formatDate(today);

// 7 hari ke depan sebagai check-out
let checkoutDate = new Date();
checkoutDate.setDate(today.getDate() + 7);
let checkout = formatDate(checkoutDate);

// Set ke environment variables
pm.environment.set("checkin", checkin);
pm.environment.set("checkout", checkout);

post-res:
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response should contain bookingid", function () {
    var jsonData = pm.response.json();
    pm.environment.set("bookingId", jsonData.bookingid);
        pm.expect(jsonData).to.have.property("bookingid");
});

pm.test("Check firstname is Jim", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.booking.firstname).to.include("Jim");
});

pm.test("Check lastname is Brown", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.booking.lastname).to.include("Brown");
});  

pm.test("Check checkin is today", function () {
    var jsonData = pm.response.json();
    var today = pm.environment.get("checkin");
    pm.expect(jsonData.booking.bookingdates.checkin).to.eql(today);
});


pm.test("Check checkout is 7 days", function () {
    var jsonData = pm.response.json();
    var checkout = pm.environment.get("checkout");
    pm.expect(jsonData.booking.bookingdates.checkout).to.eql(checkout);
});

7. send
8. import booking by id /booking/1
9. hit  collection /booking/1
10. import bookingIds
  set:
  - all
  - filter by firstname and lastname
  - filter by checkin and checkout

11. hit collcetion bookingIds --> success
12. import collection Delete /booking/[id]
  Set:
  environment : Lokal
  Content-Type:application/json
Cookie:token={{token}} --> from \auth and environment
13. hit collection Delete /booking/[id] --> created



  


