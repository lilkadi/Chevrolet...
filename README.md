<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>مكتب الشفرليت العالمي</title>
<style>
body{margin:0;font-family:Tahoma;background:#f5f5f5;color:#000}
header{background:#000;color:#fff;padding:10px 15px;display:flex;justify-content:center;align-items:center;position:sticky;top:0;z-index:1000}
button{cursor:pointer}

/* زر البحث بالأسود */
#searchButton{
  width:95%;
  margin:15px auto;
  display:block;
  padding:12px;
  font-size:16px;
  border-radius:8px;
  border:1px solid #000;
  background:#000; /* الخلفية سوداء */
  color:#fff;      /* النص أبيض */
  cursor:pointer;
  text-align:center;
}

/* باقي الموقع */
.product-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:15px;padding:15px;}
.product{background:#fff;border:1px solid #ccc;border-radius:10px;padding:10px;text-align:center;box-shadow:0 2px 6px rgba(0,0,0,0.1);}
.product img{width:100%;height:120px;object-fit:cover;border-radius:8px;margin-bottom:8px;}
.product h4{font-size:18px;margin:5px 0;}
.product .price{font-weight:bold;color:#ff6600;margin-bottom:5px;}
.quantity-buttons{display:flex;justify-content:center;align-items:center;gap:5px;margin:5px 0;}
.quantity-buttons button{width:30px;height:30px;border:none;border-radius:5px;cursor:pointer;font-size:16px;}
.quantity-buttons .increase{background:yellow;color:#000;}
.quantity-buttons .decrease{background:red;color:#fff;}
.add-cart{background:#000;color:#fff;padding:5px 10px;border:none;border-radius:8px;cursor:pointer;margin-top:5px;}
.whatsapp-btn{background:#25D366;color:#fff;border:none;padding:5px 10px;border-radius:8px;cursor:pointer;margin-top:5px;display:block;width:100%;}

#cartButton{position:fixed;bottom:20px;right:20px;background:yellow;color:#000;border-radius:50%;padding:15px;font-size:18px;display:flex;align-items:center;justify-content:center;z-index:1000;}
#cartCount{background:red;color:#fff;border-radius:50%;padding:3px 8px;font-size:14px;position:absolute;top:-5px;right:-5px;}
#cartBox{position:fixed;bottom:70px;right:20px;background:#fff;border:2px solid #000;border-radius:10px;padding:15px;width:300px;max-height:400px;overflow-y:auto;box-shadow:0 2px 6px rgba(0,0,0,0.2);z-index:999;display:none;transition: transform 0.3s;}
.cart-item{padding:10px 0;display:flex;flex-direction:column;}
.cart-item button{background:red;color:#fff;border:none;padding:5px 10px;border-radius:5px;cursor:pointer;margin-top:5px;}
#loginModal, #checkoutModal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);justify-content:center;align-items:center;z-index:1001;}
.modal-content{background:#fff;padding:20px;border-radius:10px;width:300px;text-align:center;}
.modal-content input{width:90%;margin:5px 0;padding:8px;}
.modal-content button{margin:5px 0;padding:8px;width:95%;}
#adminBox{display:none;margin:10px;padding:10px;border:2px solid #000;border-radius:10px;background:#f9f9f9;}
</style>
</head>
<body>

<header>
<button onclick="showLogin()">تسجيل الدخول</button>
</header>

<!-- زر البحث -->
<button id="searchButton" onclick="toggleSearch()">🔍 ابحث عن قطعة...</button>
<input type="text" id="searchInput" placeholder="ابحث عن قطعة..." style="display:none;width:95%;margin:0 auto;display:block;padding:10px;font-size:16px;border-radius:8px;border:1px solid #000;">

<div id="loginModal">
<div class="modal-content">
<h3>تسجيل الدخول</h3>
<input type="text" id="username" placeholder="اسم المستخدم">
<input type="text" id="code" placeholder="رمز الدخول">
<input type="password" id="password" placeholder="كلمة المرور">
<button onclick="login()">تسجيل الدخول</button>
<button onclick="closeLogin()">إغلاق</button>
</div>
</div>

<div id="adminBox">
<h3>📂 إدارة الموقع</h3>
<button onclick="addProductPrompt()">➕ إضافة قطعة جديدة مع صورة</button>
<hr>
<input type="file" id="fileInput">
<button onclick="loadFile()">📁 رفع XLSX</button>
</div>

<div id="cartButton" onclick="toggleCart()">🛒<span id="cartCount">0</span></div>

<div id="cartBox">
<h3>🛒 سلة المشتريات</h3>
<div id="cartItems"></div>
<hr>
<b>المجموع الكلي: <span id="total">0</span> $</b>
<button onclick="openCheckout()">إتمام الطلب</button>
</div>

<div id="checkoutModal">
<div class="modal-content">
<h3>تفاصيل الطلب</h3>
<input type="text" id="customerName" placeholder="اسم الزبون">
<input type="text" id="customerPhone" placeholder="رقم الهاتف">
<input type="text" id="customerAddress" placeholder="العنوان">
<button onclick="sendOrder()">إرسال الطلب</button>
<button onclick="closeCheckout()">إلغاء</button>
</div>
</div>

<div class="product-grid" id="products"></div>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script>
let products=[], cart=[], isAdmin=false;
const adminUser="admin";
const adminCode="1";
const adminPass="1234";
const myWhatsApp="07872159504";
const productsContainer=document.getElementById("products");

if(localStorage.getItem("products")) products=JSON.parse(localStorage.getItem("products"));

function showLogin(){document.getElementById("loginModal").style.display="flex";}
function closeLogin(){document.getElementById("loginModal").style.display="none";}
function login(){
  const user=document.getElementById("username").value;
  const code=document.getElementById("code").value;
  const pass=document.getElementById("password").value;
  if(user===adminUser && code===adminCode && pass===adminPass){
    alert("تم تسجيل الدخول كمستخدم مسجل!");
    isAdmin=true; document.getElementById("adminBox").style.display="block";
  } else {
    alert("تم تسجيل الدخول كزائر عادي!");
    isAdmin=false; document.getElementById("adminBox").style.display="none";
  }
  closeLogin(); renderProducts();
}

function toggleSearch(){
  const input=document.getElementById("searchInput");
  input.style.display=(input.style.display==="block")?"none":"block";
  if(input.style.display==="block") input.focus();
}

function loadFile(){
  if(!isAdmin) return alert("يجب تسجيل الدخول للمستخدم المسجل");
  const file=document.getElementById("fileInput").files[0];
  if(!file) return alert("اختر ملف XLSX");
  const reader=new FileReader();
  reader.onload=function(e){
    const data=new Uint8Array(e.target.result);
    const workbook=XLSX.read(data,{type:'array'});
    const sheet=workbook.Sheets[workbook.SheetNames[0]];
    const json=XLSX.utils.sheet_to_json(sheet);
    json.forEach(row=>{products.push({name: row["اسم المادة"]||"بدون اسم", price: row["البيع"]||0, image:""});});
    localStorage.setItem("products", JSON.stringify(products));
    renderProducts();
    alert("تم تحميل البيانات وحفظها!");
  };
  reader.readAsArrayBuffer(file);
}

function addProductPrompt(){
  if(!isAdmin) return alert("غير مسموح لك");
  const name=prompt("اسم القطعة:"); if(!name) return;
  const price=parseFloat(prompt("سعر القطعة بالدولار:")); if(isNaN(price)) return alert("السعر غير صالح");
  const fileInput=document.createElement("input"); fileInput.type="file"; fileInput.accept="image/*";
  fileInput.onchange=e=>{
    const file=e.target.files[0]; if(file){
      const reader=new FileReader();
      reader.onload=ev=>{
        products.push({name, price, image:ev.target.result});
        localStorage.setItem("products", JSON.stringify(products));
        renderProducts();
      }; reader.readAsDataURL(file);
    } else {
      products.push({name, price, image:""});
      localStorage.setItem("products", JSON.stringify(products));
      renderProducts();
    }
  };
  fileInput.click();
}

function renderProducts(){
  productsContainer.innerHTML="";
  const term=document.getElementById("searchInput").value.toLowerCase();
  products.forEach((p,i)=>{
    if(term && !p.name.toLowerCase().includes(term)) return;
    const div=document.createElement("div"); div.className="product";
    const h4=document.createElement("h4"); h4.textContent=p.name;
    div.appendChild(h4);
    if(p.image){
      const img=document.createElement("img"); img.src=p.image; img.alt=p.name;
      div.appendChild(img);
    }
    const priceDiv=document.createElement("div"); priceDiv.className="price"; priceDiv.textContent=p.price+" $";
    div.appendChild(priceDiv);
    const qDiv=document.createElement("div"); qDiv.className="quantity-buttons";
    const dec=document.createElement("button"); dec.textContent="-"; dec.className="decrease";
    const qtySpan=document.createElement("span"); qtySpan.id="qty"+i; qtySpan.textContent="1";
    const inc=document.createElement("button"); inc.textContent="+"; inc.className="increase";
    dec.onclick=()=>changeQuantity(i,-1); inc.onclick=()=>changeQuantity(i,1);
    qDiv.appendChild(dec); qDiv.appendChild(qtySpan); qDiv.appendChild(inc);
    div.appendChild(qDiv);
    const addBtn=document.createElement("button"); addBtn.className="add-cart"; addBtn.textContent="➕ أضف للسلة";
    addBtn.onclick=()=>addToCart(p,parseInt(qtySpan.textContent));
    div.appendChild(addBtn);
    const waBtn=document.createElement("button"); waBtn.className="whatsapp-btn"; waBtn.textContent="واتساب";
    waBtn.onclick=()=>openWhatsApp(p.name,p.price);
    div.appendChild(waBtn);
    productsContainer.appendChild(div);
  });
}

function changeQuantity(i,change){
  const qty=document.getElementById("qty"+i);
  let q=parseInt(qty.textContent)+change;
  if(q<1) q=1; if(q>10) q=10; qty.textContent=q;
}

function addToCart(p,qty){
  let existing=cart.find(c=>c.name===p.name);
  if(existing){existing.qty+=qty; if(existing.qty>10) existing.qty=10; existing.total=existing.qty*p.price;}
  else{if(qty>10) qty=10; cart.push({name:p.name, qty, total:qty*p.price});}
  renderCart();
}

function renderCart(){
  const cartItems=document.getElementById("cartItems"); cartItems.innerHTML=""; let sum=0;
  cart.forEach((c,i)=>{
    sum+=c.total;
    const item=document.createElement("div"); item.className="cart-item";
    const name=document.createElement("b"); name.textContent=c.name;
    const counter=document.createElement("div"); counter.className="counter";
    const dec=document.createElement("button"); dec.textContent="-"; dec.onclick=()=>updateQty(i,-1);
    const qty=document.createElement("span"); qty.textContent=c.qty;
    const inc=document.createElement("button"); inc.textContent="+"; inc.onclick=()=>updateQty(i,1);
    counter.appendChild(dec); counter.appendChild(qty); counter.appendChild(inc);
    const price=document.createElement("div"); price.textContent="السعر: "+c.total+" $";
    const remove=document.createElement("button"); remove.textContent="حذف"; remove.onclick=()=>removeItem(i);
    item.appendChild(name); item.appendChild(counter); item.appendChild(price); item.appendChild(remove);
    cartItems.appendChild(item);
  });
  document.getElementById("total").textContent=sum;
  document.getElementById("cartCount").textContent=cart.reduce((a,b)=>a+b.qty,0);
}

function updateQty(i,change){
  cart[i].qty+=change;
  if(cart[i].qty<1) cart.splice(i,1);
  else if(cart[i].qty>10) cart[i].qty=10;
  cart[i].total=cart[i].qty*products.find(p=>p.name===cart[i].name).price;
  renderCart();
}

function removeItem(i){cart.splice(i,1); renderCart();}
function toggleCart(){const c=document.getElementById("cartBox"); c.style.display=(c.style.display==="block")?"none":"block";}
function openCheckout(){document.getElementById("checkoutModal").style.display="flex";}
function closeCheckout(){document.getElementById("checkoutModal").style.display="none";}
function sendOrder(){
  const name=document.getElementById("customerName").value;
  const phone=document.getElementById("customerPhone").value;
  const address=document.getElementById("customerAddress").value;
  if(!name||!phone||!address) return alert("أدخل الاسم ورقم الهاتف والعنوان");
  let text=`📦 طلبية جديدة\nاسم: ${name}\nهاتف: ${phone}\nالعنوان: ${address}\n\n`;
  cart.forEach(c=>{text+=`- ${c.name} | كمية: ${c.qty} | السعر: ${c.total} $\n`;});
  text+=`\nالمجموع: ${document.getElementById("total").innerText} $`;
  window.open(`https://wa.me/964${myWhatsApp.slice(1)}?text=${encodeURIComponent(text)}`);
  cart=[]; renderCart(); closeCheckout(); document.getElementById("cartBox").style.display="none";
}

function openWhatsApp(name, price){
  let text=`📦 استفسار عن القطعة: ${name} | السعر: ${price} $`;
  window.open(`https://wa.me/964${myWhatsApp.slice(1)}?text=${encodeURIComponent(text)}`);
}

document.getElementById("searchInput").addEventListener("input", renderProducts);

renderProducts();
</script>
</body>
</html>
