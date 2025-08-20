# YOURSTORE
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار المتقدمة</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: rtl; /* Changed to RTL */
      padding: 30px;
    }
    select, input, textarea {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
      box-sizing: border-box;
      background-color: #333;
      color: white;
    }
    textarea {
        resize: vertical;
        min-height: 60px;
    }
    input:disabled {
      background-color: #555;
      cursor: not-allowed;
      color: #aaa;
    }
    .section {
      background-color: #252526;
      padding: 15px;
      border-radius: 8px;
      margin-bottom: 20px;
      border: 1px solid #333;
    }
    label {
        font-weight: bold;
        color: #00e676;
    }
    button {
      background-color: #007acc;
      color: white;
      padding: 12px 22px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #005a9e;
    }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }

    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      background-color: #252526;
      border: 1px solid #333;
      border-radius: 8px;
      padding: 15px;
      margin-bottom: 15px;
    }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 15px;
      border-top: 2px solid #00e676;
      margin-top: 15px;
    }
    .result-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
        gap: 15px;
        margin-top: 10px;
    }
    .result-grid div { display: flex; flex-direction: column; }
    .result-grid label { font-size: 0.9em; color: #00e676; margin-bottom: 3px; }
    .result-grid input { padding: 6px; }
    .item-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #444; padding-bottom: 10px; margin-bottom: 10px; }
    .item-header h4 { margin: 0; font-size: 1.2em; }
    .delete-btn {
        background-color: #c0392b;
        padding: 6px 12px;
        font-size: 14px;
        margin: 0;
    }
  </style>
</head>
<body>

<div class="section">
    <label for="customerName">اسم العميل:</label>
    <input type="text" id="customerName" placeholder="مثال: جون دو" />
    <label for="customerPhone">رقم هاتف العميل:</label>
    <input type="text" id="customerPhone" placeholder="مثال: 99455082" />
</div>

<div class="section">
  <label for="mainCategory">الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">النوع الفرعي:</label>
  <select id="subType"></select>
</div>

<div class="section">
  <label for="quantity">الكمية المبدئية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<button onclick="addItem()">➕ إضافة عنصر</button>
<button onclick="clearAllResults()" class="btn-danger">🗑️ مسح الكل</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 حفظ كملف Word</button>

<div class="results" id="results"></div>

<script>
const SHIPPING_RATE = 48;

const productData = {
    "نوافذ": {
        "نافذة زجاج مزدوج إطار مزدوج ثابت": { price: 34, cbm: 0.13 },
        "نافذة زجاج مزدوج إطار مزدوج فتحة واحدة": { price: 34, fixed_component_cost: 39, cbm: 0.13 },
        "نافذة زجاج مزدوج إطار مزدوج فتحتين": { price: 34, fixed_component_cost: 58, cbm: 0.13 },
        "نافذة زجاج مزدوج إطار مفرد ثابت": { price: 26, cbm: 0.07 },
        "نافذة زجاج مزدوج إطار مفرد فتحة واحدة": { price: 26, fixed_component_cost: 20, cbm: 0.07 },
        "نافذة زجاج مزدوج إطار مفرد فتحتين": { price: 26, fixed_component_cost: 32, cbm: 0.07 },
        "نافذة زجاج مفرد إطار مفرد ثابت": { price: 20, cbm: 0.07 },
        "نافذة زجاج مفرد إطار مفرد فتحة واحدة": { price: 20, fixed_component_cost: 13, cbm: 0.07 },
        "نافذة زجاج مفرد إطار مفرد فتحتين": { price: 20, fixed_component_cost: 17, cbm: 0.07 },
        "نوافذ سحاب": { price: 41, fixed_component_cost: 10, cbm: 0.13 },
        "نوافذ كهربائية": { price: 102, cbm: 0.13 },
        "سكاي لايت بدون محرك": { price: 56, cbm: 0.13 },
        "سكاي لايت مع محرك": { price: 145, cbm: 0.13 },
        "ستائر جدارية ثقيلة": { price: 56, cbm: 0.15 },
        "ستائر جدارية خفيفة": { price: 45, cbm: 0.15 },
    },
    "أبواب": {
        "باب مدخل - زينك": { price: 66, cbm: 0.20 },
        "باب مدخل - ستانلس ستيل": { price: 120, cbm: 0.20 },
        "باب مدخل - ألمنيوم مصبوب": { price: 168, cbm: 0.20 },
        "باب WPC": { price: 45, cbm: 0.11 },
        "باب WPC - مع خشب": { price: 50, cbm: 0.11 },
        "باب WPC - بحشوة عازلة للصوت": { price: 60, cbm: 0.11 },
        "باب WPC - بإطار ألمنيوم": { price: 67, cbm: 0.11 },
        "باب ألمنيوم": { price: 65, cbm: 0.11 },
        "باب ألمنيوم - مع خشب": { price: 75, cbm: 0.11 },
        "باب ألمنيوم - كامل": { price: 85, cbm: 0.11 },
        "باب ألمنيوم - مخفي": { price: 110, cbm: 0.11 },
        "باب ألمنيوم - خارجي": { price: 61, cbm: 0.11 },
        "باب حمام - نوع جديد": { price: 55, cbm: 0.11 },
        "باب حمام - نوع قديم": { price: 45, cbm: 0.11 },
        "باب حمام - زجاج مخفي": { price: 65, cbm: 0.11 },
    },
    "أبواب سحاب": {
        "باب سحاب داخلي - زجاج": { price: 38, cbm: 0.15 },
        "باب سحاب داخلي - صلب": { price: 41, cbm: 0.15 },
        "باب سحاب خارجي - لوح واحد يفتح": { price: 55, cbm: 0.15 },
        "باب سحاب خارجي - لوحين يفتحان": { price: 58, cbm: 0.15 },
        "باب سحاب WPC": { price: 61, cbm: 0.15 },
    },
    "أبواب قابلة للطي": {
        "باب قابل للطي داخلي": { price: 39, cbm: 0.15 },
        "باب قابل للطي خارجي": { price: 56, cbm: 0.15 },
    },
    "شتر خارجي": {
        "شتر رولينج": { price: 28, cbm: 0.20 },
    },
    "بوابات حدائق": {
        "بوابة حديقة ألمنيوم مصبوب": { price: 91, cbm: 0.20 },
    },
    "حواجز": {
        "حاجز درج": { price: 43, cbm: 0.05 },
        "حاجز حمام": { price: 32, cbm: 0.05 },
    }
};

let resultsList = [];

function findProductData(subTypeName) {
    for (const category in productData) {
        if (productData[category][subTypeName]) {
            return { ...productData[category][subTypeName], name: subTypeName };
        }
    }
    return null;
}

function initializeApp() {
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- اختر الفئة --</option>`;
    Object.keys(productData).forEach(cat => mainCat.innerHTML += `<option value="${cat}">${cat}</option>`);
    loadSubTypes();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => subType.innerHTML += `<option value="${sub}">${sub}</option>`);
    }
}

function addItem() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    if (!mainCatVal || !subVal) { alert("الرجاء اختيار الفئة والنوع الفرعي."); return; }
    
    const data = findProductData(subVal);
    const newItem = {
        id: Date.now(),
        name: data.name,
        qty: parseInt(document.getElementById("quantity").value) || 1,
        h: 1,
        w: 1,
        basePrice: data.price || 0,
        fixedCost: data.fixed_component_cost || 0,
        cbm: data.cbm || 0,
        shippingRate: SHIPPING_RATE,
        itemNotes: "",
        unitPrice: 0,
        totalPrice: 0
    };
    
    resultsList.push(newItem);
    recalculateAndUpdate();
}

function updateItemValue(id, field, value) {
    const item = resultsList.find(i => i.id === id);
    if (!item) return;

    if (field === 'name' || field === 'itemNotes') {
        item[field] = value;
    } else {
        item[field] = parseFloat(value) || 0;
    }
    recalculateAndUpdate();
}

function deleteItem(id) {
    resultsList = resultsList.filter(item => item.id !== id);
    recalculateAndUpdate();
}

function recalculateAndUpdate() {
    resultsList.forEach(item => {
        const area = item.h * item.w;
        const materialCost = item.basePrice * area;
        const shippingCost = area * item.cbm * item.shippingRate;
        
        item.unitPrice = materialCost + item.fixedCost + shippingCost;
        item.totalPrice = item.unitPrice * item.qty;
    });
    renderResults();
}

function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    let subtotal = 0;

    resultsList.forEach(item => {
        subtotal += item.totalPrice;
        const itemHtml = `
            <div class="result-item" id="item-${item.id}">
                <div class="item-header">
                    <input type="text" value="${item.name}" onchange="updateItemValue(${item.id}, 'name', this.value)" style="flex-grow: 1; font-size: 1.2em; font-weight: bold;">
                    <button class="delete-btn" onclick="deleteItem(${item.id})">حذف 🗑️</button>
                </div>
                <div class="result-grid">
                    <div>
                        <label for="qty-${item.id}">الكمية</label>
                        <input type="number" id="qty-${item.id}" value="${item.qty}" onchange="updateItemValue(${item.id}, 'qty', this.value)">
                    </div>
                    <div>
                        <label for="h-${item.id}">الارتفاع</label>
                        <input type="number" step="0.01" id="h-${item.id}" value="${item.h}" onchange="updateItemValue(${item.id}, 'h', this.value)">
                    </div>
                    <div>
                        <label for="w-${item.id}">العرض</label>
                        <input type="number" step="0.01" id="w-${item.id}" value="${item.w}" onchange="updateItemValue(${item.id}, 'w', this.value)">
                    </div>
                    <div>
                        <label for="basePrice-${item.id}">السعر الأساسي/م²</label>
                        <input type="number" step="0.01" id="basePrice-${item.id}" value="${item.basePrice}" onchange="updateItemValue(${item.id}, 'basePrice', this.value)">
                    </div>
                    <div>
                        <label for="fixedCost-${item.id}">تكلفة ثابتة</label>
                        <input type="number" step="0.01" id="fixedCost-${item.id}" value="${item.fixedCost}" onchange="updateItemValue(${item.id}, 'fixedCost', this.value)">
                    </div>
                     <div>
                        <label for="cbm-${item.id}">CBM/م²</label>
                        <input type="number" step="0.01" id="cbm-${item.id}" value="${item.cbm}" onchange="updateItemValue(${item.id}, 'cbm', this.value)">
                    </div>
                    <div>
                        <label for="shippingRate-${item.id}">سعر الشحن</label>
                        <input type="number" step="0.01" id="shippingRate-${item.id}" value="${item.shippingRate}" onchange="updateItemValue(${item.id}, 'shippingRate', this.value)">
                    </div>
                </div>
                <div style="margin-top: 15px;">
                     <label for="itemNotes-${item.id}">ملاحظات على العنصر</label>
                     <textarea id="itemNotes-${item.id}" onchange="updateItemValue(${item.id}, 'itemNotes', this.value)">${item.itemNotes}</textarea>
                </div>
                <div style="font-weight: bold; margin-top: 10px; text-align: left; font-size: 1.1em;">
                    سعر الوحدة: ${item.unitPrice.toFixed(2)} ر.ع. | الإجمالي: ${item.totalPrice.toFixed(2)} ر.ع.
                </div>
            </div>
        `;
        container.innerHTML += itemHtml;
    });

    if (resultsList.length > 0) {
        const commission = subtotal * 0.04;
        const total = subtotal + commission;
        const summaryHtml = `
            <div class="summary">
                <div style="display: flex; justify-content: space-between; padding: 5px 0; font-size: 1.2em; color: white;">
                    <span>المجموع الفرعي:</span>
                    <span style="font-weight: bold;">${subtotal.toFixed(2)} ر.ع.</span>
                </div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0; font-size: 1.2em; border-bottom: 1px solid #555; margin-bottom: 10px; color: white;">
                    <span>عمولة المكتب (4%):</span>
                    <span style="font-weight: bold;">${commission.toFixed(2)} ر.ع.</span>
                </div>
                <div style="background-color: #00e676; color: #1e1e1e; padding: 15px; border-radius: 5px; text-align: center; margin-top: 10px;">
                    <span style="font-size: 1.3em; font-weight: bold;">💰 المجموع الكلي</span>
                    <div style="font-size: 2.0em; font-weight: bold; margin-top: 5px;">${total.toFixed(2)} ر.ع.</div>
                </div>
            </div>`;
        container.innerHTML += summaryHtml;
    }
}

function clearAllResults() {
    resultsList = [];
    recalculateAndUpdate();
    document.getElementById("customerName").value = "";
    document.getElementById("customerPhone").value = "";
}

function formatNumber(num) {
    return parseFloat(num.toFixed(2));
}

function saveAsWord(){
    if (resultsList.length === 0) {
        alert("لا توجد نتائج للحفظ.");
        return;
    }

    const customerName = document.getElementById('customerName').value || 'عميل';
    const customerPhone = document.getElementById('customerPhone').value || 'غير متوفر';
    
    const headerHtml = `
        <div style="width: 100%; font-family: Arial, sans-serif; text-align: center; margin-bottom: 30px; border-bottom: 2px solid #4472C4; padding-bottom: 20px;">
            <div style="font-size: 26px; font-weight: bold; color: #4472C4; margin-bottom: 15px;">
                شركة أمواج المحيط الأزرق للخدمات ش.م.م
            </div>
            <table width="100%" style="font-family: Arial, sans-serif; font-size: 15px; color: #4472C4; border-collapse: collapse;">
                <tr>
                    <td style="text-align: left; width: 33%;">هاتف: 77224511 – 90998810</td>
                    <td style="text-align: center; width: 34%;">س.ت. : 1595256</td>
                    <td style="text-align: right; width: 33%;">عُمان – مسقط</td>
                </tr>
            </table>
        </div>`;

    const customerHtml = `
        <div style="background-color: #f2f2f2; border: 1px solid #ddd; color: #333; padding: 14px; font-family: Arial, sans-serif; font-size: 17px; margin-bottom: 25px; border-radius: 5px; text-align: right;">
            <b>عرض سعر للسيد/ة: </b> ${customerName} - ${customerPhone}
        </div>`;

    let tableHtml = `<table border="1" width="100%" style="border-collapse: collapse; font-family: Arial, sans-serif; text-align: center; font-size: 14px;">
                        <thead>
                            <tr style="background-color: #4472C4; color: white;">
                                <th style="padding: 10px;">#</th>
                                <th style="padding: 10px;">النوع / الوصف</th>
                                <th style="padding: 10px;">الارتفاع</th>
                                <th style="padding: 10px;">العرض</th>
                                <th style="padding: 10px;">الكمية</th>
                                <th style="padding: 10px;">ملاحظات</th>
                                <th style="padding: 10px;">سعر الوحدة</th>
                                <th style="padding: 10px;">الإجمالي</th>
                            </tr>
                        </thead>
                        <tbody>`;
    
    let subtotal = 0;

    resultsList.forEach((r, index) => {
        subtotal += r.totalPrice;
        
        tableHtml += `<tr>
                        <td style="padding: 10px; font-weight: bold;">${index + 1}</td>
                        <td style="padding: 10px; font-weight: bold;">${r.name}</td>
                        <td style="padding: 10px;">${formatNumber(r.h)}م</td>
                        <td style="padding: 10px;">${formatNumber(r.w)}م</td>
                        <td style="padding: 10px;">${r.qty}</td>
                        <td style="padding: 10px; text-align: right; white-space: pre-wrap;">${r.itemNotes || '-'}</td>
                        <td style="padding: 10px; background-color: #f2f2f2;">${formatNumber(r.unitPrice)}</td>
                        <td style="padding: 10px; background-color: #f2f2f2; font-weight: bold;">${formatNumber(r.totalPrice)}</td>
                      </tr>`;
    });
    tableHtml += `</tbody></table>`;
    
    const commission = subtotal * 0.04;
    const grandTotal = subtotal + commission;

    const summaryTable = `
        <table width="45%" align="left" style="border-collapse: collapse; font-family: Arial, sans-serif; margin-top: 25px; font-size: 15px;">
            <tbody>
                <tr>
                    <td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(subtotal)} ر.ع.</td>
                    <td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">المجموع الفرعي</td>
                </tr>
                <tr>
                    <td style="padding: 14px; text-align: right; font-weight: bold; background-color: #f2f2f2; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(commission)} ر.ع.</td>
                    <td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">عمولة المكتب (4%)</td>
                </tr>
                <tr style="background-color: #4472C4; color: white;">
                    <td style="padding: 16px; text-align: right; font-weight: bold; border: 1px solid #4472C4; font-size: 1.9em; vertical-align: middle;">${formatNumber(grandTotal)} ر.ع.</td>
                    <td style="padding: 16px; text-align: right; font-weight: bold; border: 1px solid #4472C4; font-size: 1.5em;">المجموع الكلي</td>
                </tr>
            </tbody>
        </table>`;
    
    const footerNotesHtml = `
        <div style="clear: both; font-family: Arial, sans-serif; margin-top: 50px; text-align: center; border-top: 2px solid #4472C4; padding-top: 15px; font-size: 16px; color: #333;">
            <p style="margin: 5px 0;"><b>السعر يشمل التوصيل</b></p>
            <p style="margin: 5px 0;"><b>السعر لا يشمل التركيب</b></p>
        </div>`;

    const finalHtml = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
                        <head><meta charset='utf-8'><title>عرض سعر لـ - ${customerName}</title></head>
                        <body dir="rtl" style="padding: 20px;">` 
                      + headerHtml + customerHtml + tableHtml + summaryTable + footerNotesHtml
                      + `</body></html>`;

    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(finalHtml);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = `عرض سعر لـ - ${customerName}.doc`;
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

initializeApp();
</script>

</body>
</html>
