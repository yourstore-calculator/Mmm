# Mmm
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      background-color: #f7f7f7;
      color: #333;
      padding: 20px;
      direction: rtl;
    }

    .container {
      background: #fff;
      max-width: 800px;
      margin: auto;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
    }

    h1 {
      text-align: center;
      margin-bottom: 20px;
    }

    label, select, input {
      display: block;
      width: 100%;
      margin-bottom: 15px;
    }

    button {
      padding: 10px 20px;
      background-color: #444;
      color: #fff;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin-right: 10px;
    }

    button:hover {
      background-color: #222;
    }

    #results {
      margin-top: 20px;
      padding: 15px;
      background: #eee;
      border-radius: 8px;
      min-height: 60px;
    }

    .inline {
      display: flex;
      gap: 10px;
    }
  </style>
</head>
<body>

<div class="container">
  <img src="logo.png" alt="شعار الشركة" style="max-width: 150px; display: block; margin: auto; margin-bottom: 20px;">

  <h1>حاسبة الأسعار</h1>

  <label>نوع المنتج</label>
  <select id="productType">
    <option value="">اختر</option>
    <option value="نافذة">نافذة</option>
    <option value="باب">باب</option>
  </select>

  <label>النوع</label>
  <select id="itemType"></select>

  <label>الطول (متر)</label>
  <input type="number" id="length" step="0.01" placeholder="مثلاً 2.2">

  <label>العرض (متر)</label>
  <input type="number" id="width" step="0.01" placeholder="مثلاً 1">

  <label>الكمية</label>
  <input type="number" id="quantity" step="1" value="1" min="1">

  <label>هل ترغب بإضافة ستارة؟</label>
  <select id="addCurtain">
    <option value="لا">لا</option>
    <option value="نعم">نعم</option>
  </select>

  <div id="curtainSize" style="display: none;">
    <label>مقاس الستارة (طول × عرض)</label>
    <div class="inline">
      <input type="number" id="curtainLength" placeholder="الطول (متر)">
      <input type="number" id="curtainWidth" placeholder="العرض (متر)">
    </div>
  </div>

  <label>هل ترغب بإضافة شبك؟</label>
  <select id="addMesh">
    <option value="لا">لا</option>
    <option value="باب">شبك باب - 39 ريال/م</option>
    <option value="فولدنج">شبك فولدنج - 18 ريال/م</option>
    <option value="سلايد">شبك سلايد - 14 ريال/م</option>
  </select>

  <button onclick="calculate()">احسب</button>
  <button onclick="clearResults()">🗑️ مسح النتائج</button>
  <button onclick="saveAsWord()">💾 حفظ النتائج كملف Word</button>

  <div id="results"></div>
</div>

<script>
const prices = {
  "نافذة": {
    "دبل جلاس دبل فريم - ثابت": { price: 34, cbm: 0.13 },
    "دبل جلاس دبل فريم - حركة": { price: 73, cbm: 0.13 },
    "دبل جلاس دبل فريم - حركتين": { price: 92, cbm: 0.13 },
    "دبل جلاس سنجل فريم - ثابت": { price: 26, cbm: 0.07 },
    "دبل جلاس سنجل فريم - حركة": { price: 46, cbm: 0.07 },
    "دبل جلاس سنجل فريم - حركتين": { price: 58, cbm: 0.07 },
    "سنجل جلاس سنجل فريم - ثابت": { price: 20, cbm: 0.07 },
    "سنجل جلاس سنجل فريم - حركة": { price: 43, cbm: 0.07 },
    "سنجل جلاس سنجل فريم - حركتين": { price: 47, cbm: 0.07 },
    "سلايدنج - RMP": { price: 10, cbm: 0.13, extra: true },
    "كهربائية": { price: 102, cbm: 0.13 },
    "سكاي لايت بدون مكينة": { price: 56, cbm: 0.13 },
    "سكاي لايت مع مكينة": { price: 145, cbm: 0.13 },
    "كارتن وول ثقيل": { price: 56, cbm: 0.13 },
    "كارتن وول خفيف": { price: 45, cbm: 0.13 }
  },
  "باب": {
    "باب سحب داخلي زجاج": { price: 38, cbm: 0.13 },
    "باب سحب داخلي متين": { price: 41, cbm: 0.13 },
    "باب سحب خارجي مفتوح": { price: 55, cbm: 0.13 },
    "باب سحب خارجي جزئين": { price: 58, cbm: 0.13 },
    "باب سحب WPC": { price: 61, cbm: 0.11 },
    "باب فولدنج داخلي": { price: 39, cbm: 0.13 },
    "باب فولدنج خارجي": { price: 56, cbm: 0.13 },
    "باب مدخل زينك": { price: 76, cbm: 0.2 },
    "باب مدخل ستيل": { price: 130, cbm: 0.2 },
    "باب مدخل كاست المنيوم": { price: 178, cbm: 0.2 },
    "باب حديقة": { price: 91, cbm: 0.2 },
    "باب WPC": { price: 67, cbm: 0.11 },
    "باب المنيوم": { price: 65, cbm: 0.11 },
    "باب المنيوم مع خشب": { price: 75, cbm: 0.11 },
    "باب المنيوم مخفي": { price: 110, cbm: 0.11 },
    "باب دورة مياه": { price: 55, cbm: 0.11 }
  }
};

const meshPrices = {
  "باب": 39,
  "فولدنج": 18,
  "سلايد": 14
};

document.getElementById("productType").addEventListener("change", function () {
  const type = this.value;
  const itemType = document.getElementById("itemType");
  itemType.innerHTML = "";

  if (!prices[type]) return;

  for (const key in prices[type]) {
    const option = document.createElement("option");
    option.value = key;
    option.textContent = key;
    itemType.appendChild(option);
  }

  if (type === "باب") {
    document.getElementById("length").value = 2.2;
    document.getElementById("width").value = 1;
  } else {
    document.getElementById("length").value = "";
    document.getElementById("width").value = "";
  }
});

document.getElementById("addCurtain").addEventListener("change", function () {
  document.getElementById("curtainSize").style.display = this.value === "نعم" ? "block" : "none";
});

function calculate() {
  const type = document.getElementById("productType").value;
  const item = document.getElementById("itemType").value;
  const length = parseFloat(document.getElementById("length").value);
  const width = parseFloat(document.getElementById("width").value);
  const quantity = parseInt(document.getElementById("quantity").value);
  const addCurtain = document.getElementById("addCurtain").value;
  const addMesh = document.getElementById("addMesh").value;

  if (!type || !item || isNaN(length) || isNaN(width) || isNaN(quantity)) return;

  const area = length * width;
  const price = prices[type][item].price;
  const cbm = prices[type][item].cbm;
  const extra = prices[type][item].extra;

  let cost = extra ? (area * 92) + 10 : area * price;
  let shipping = cbm * area * 48;
  let total = (cost + shipping) * quantity;

  if (addCurtain === "نعم") {
    const cl = parseFloat(document.getElementById("curtainLength").value);
    const cw = parseFloat(document.getElementById("curtainWidth").value);
    if (!isNaN(cl) && !isNaN(cw)) {
      const curtainArea = cl * cw;
      total += curtainArea * 26;
    }
  }

  if (addMesh !== "لا") {
    const meshArea = area * 0.5;
    total += meshArea * meshPrices[addMesh];
  }

  document.getElementById("results").innerHTML += `<div>🔹 ${item} × ${quantity}: ${total.toFixed(2)} ريال</div>`;
}

function clearResults() {
  document.getElementById("results").innerHTML = "";
}

function saveAsWord() {
  const content = document.getElementById("results").innerHTML;
  const header = "<html><body dir='rtl'><h2>النتائج</h2>";
  const footer = "</body></html>";
  const sourceHTML = header + content + footer;
  const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
  const link = document.createElement("a");
  link.href = source;
  link.download = 'النتائج.doc';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>

</body>
</html>
