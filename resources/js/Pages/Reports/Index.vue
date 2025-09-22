<style>
/* General DataTables Pagination Container Style */
.dataTables_wrapper .dataTables_paginate {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

/* Style the filter container */
#TransitionTable_filter {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin-bottom: 16px; /* Add spacing below the filter */
}

/* Style the label and input field inside the filter */
#TransitionTable_filter label {
  font-size: 17px;
  color: #000000; /* Match text color of the table header */
  display: flex;
  align-items: center;
}

/* Style the input field */
#TransitionTable_filter input[type="search"] {
  font-weight: 400;
  padding: 9px 15px;
  font-size: 14px;
  color: #000000cc;
  border: 1px solid rgb(209 213 219);
  border-radius: 5px;
  background: #fff;
  outline: none;
  transition: all 0.5s ease;
}
#TransitionTable_filter input[type="search"]:focus {
  outline: none; /* Removes the default outline */
  border: 1px solid #4b5563;
  box-shadow: none; /* Removes any focus box-shadow */
}

#TransitionTable_filter {
  float: left;
}

.dataTables_wrapper {
  margin-bottom: 10px;
}
</style>

<template>
  <Head title="Reports" />
  <Banner />

  <div class="flex flex-col items-center justify-start min-h-screen py-8 space-y-8 bg-gray-100 md:px-36 px-16">
    <Header />

    <!-- Page header + date filters -->
    <div class="w-full py-12 space-y-16">
      <div class="flex md:flex-row flex-col md:items-center items-start justify-between md:space-y-0 space-y-4">
        <div class="flex items-center justify-center space-x-4">
          <Link href="/"><img src="/images/back-arrow.png" class="w-14 h-14" /></Link>
          <p class="text-4xl font-bold tracking-wide text-black uppercase">Reports</p>
        </div>

        <div date-rangepicker class="flex items-center space-x-4">
          <div class="relative">
            <input v-model="startDate" type="date"
              class="text-xl font-normal tracking-wider text-blue-600 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
              placeholder="Start Date" />
          </div>
          <span class="text-xl font-bold tracking-wider text-blue-600">To</span>
          <div class="relative">
            <input v-model="endDate" type="date"
              class="text-xl font-normal tracking-wider text-blue-600 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
              placeholder="End Date" />
          </div>

          <!-- IMPORTANT: DO NOT PRESERVE STATE -->
          <button @click="filterData"
            class="px-6 py-3 text-xl font-normal tracking-wider text-white text-center bg-blue-600 rounded-lg custom-select hidden sm:inline-block">
            Filter
          </button>
          <Link href="/reports"
            class="px-6 py-3 text-xl font-normal tracking-wider text-white text-center bg-blue-600 rounded-lg custom-select hidden sm:inline-block">
            Reset
          </Link>
        </div>

        <div class="w-full flex justify-center items-center space-x-4 md:hidden">
          <button @click="filterData"
            class="px-6 py-3 text-xl font-normal tracking-wider text-white text-center bg-blue-600 rounded-lg custom-select">
            Filter
          </button>
          <Link href="/reports"
            class="px-6 py-3 text-xl font-normal tracking-wider text-white text-center bg-blue-600 rounded-lg custom-select">
            Reset
          </Link>
        </div>
      </div>
    </div>

    <!-- Stat boxes -->
    <div class="grid w-full md:grid-cols-6 grid-cols-3 gap-4">
      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#EC6116] w-full space-y-4 rounded-2xl bg-[#EC611666] shadow-lg hover:-translate-y-4 transition">
        <div class="flex flex-col items-center justify-center">
          <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Sales</h2>
          <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Amount</h2>
        </div>
        <p class="text-2xl font-bold text-black">{{ toMoney(totalSaleAmount) }} LKR</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#488D3F] w-full space-y-8 rounded-2xl bg-[#488D3F66] shadow-lg hover:-translate-y-4 transition">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Net Profit</h2>
        <p class="text-2xl font-bold text-black">{{ toMoney(netProfit) }} LKR</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#16D0EC] w-full space-y-4 rounded-2xl bg-[#16D0EC66] shadow-lg hover:-translate-y-4 transition">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Discount</h2>
        <p class="text-2xl font-bold text-black">
          {{ toMoney(Number(totalDiscountLkr) + Number(totalCustomDiscountLkr)) }} LKR
        </p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#F6F20E] w-full space-y-4 rounded-2xl bg-[#F6F20E66] shadow-lg hover:-translate-y-4 transition">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Avg. Transaction Value</h2>
        <p class="text-2xl font-bold text-black">{{ toMoney(averageTransactionValue) }} LKR</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#9E16EC] w-full space-y-4 rounded-2xl bg-[#9E16EC66] shadow-lg hover:-translate-y-4 transition">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Number of Transactions</h2>
        <p class="text-2xl font-bold text-black">{{ totalTransactions }}</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#EC16D7] w-full space-y-4 rounded-2xl bg-[#EC16D766] shadow-lg hover:-translate-y-4 transition">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Number of Customers</h2>
        <p class="text-2xl font-bold text-black">{{ totalCustomer }}</p>
      </div>
    </div>

    <!-- Inventory totals -->
    <div class="grid w-full md:grid-cols-3 grid-cols-2 gap-8">
      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#ffb224] w-full space-y-4 rounded-2xl bg-[#ffb224] shadow-lg">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Quantity In Stock:</h2>
        <p class="text-2xl font-bold text-black">{{ totalQty }} QTY</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#41ec16] w-full space-y-4 rounded-2xl bg-[#41ec16] shadow-lg">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Selling Price In Stock:</h2>
        <p class="text-2xl font-bold text-black">{{ totalSellingPrice.toFixed(2) }} LKR</p>
      </div>

      <div class="py-6 flex flex-col justify-center items-center border-2 border-[#3e41ff] w-full space-y-4 rounded-2xl bg-[#3e41ff] shadow-lg">
        <h2 class="text-xl font-extrabold tracking-wide text-black uppercase">Total Cost Price In Stock:</h2>
        <p class="text-2xl font-bold text-black">{{ totalRetailValue.toFixed(2) }} LKR</p>
      </div>
    </div>

    <!-- Charts -->
    <div class="flex md:flex-row flex-col items-center justify-center w-full h-full md:space-x-4 md:space-y-0 space-y-4">
      <div class="flex flex-col justify-between items-center md:w-1/3 w-full bg-white border-4 border-black rounded-xl h-[450px]">
        <div class="chart-container w-full p-4">
          <div class="w-full flex justify-between items-center pb-8">
            <h2 class="text-2xl font-medium tracking-wide text-slate-700 text-left">Top Employee Sales</h2>
            <button @click="downloadPDF" class="w-full mt-6 px-4 py-2 text-md font-normal tracking-wider text-white bg-orange-600 rounded-lg custom-select hover:bg-orange-700 hover:shadow-lg">Download PDF</button>
          </div>
          <div class="w-full h-full flex justify-center items-center">
            <Doughnut :data="chartData4" :options="chartOptions4" />
          </div>
        </div>
      </div>

      <div class="flex flex-col justify-between items-center md:w-1/3 w-full bg-white border-4 border-black rounded-xl h-[450px]">
        <div class="chart-container w-full p-4">
          <div class="w-full flex justify-between items-center md:pt-12">
            <h2 class="text-2xl font-medium tracking-wide text-slate-700 text-left">Product</h2>
            <button @click="downloadPDF2" class="w-full mt-6 px-4 py-2 text-md font-normal tracking-wider text-white bg-orange-600 rounded-lg custom-select hover:bg-orange-700 hover:shadow-lg">Download PDF</button>
          </div>
          <Pie :data="chartData" :options="chartOptions" />
        </div>
      </div>

      <div class="flex flex-col justify-between items-center md:w-1/3 w-full bg-white border-4 border-black rounded-xl h-[450px]">
        <div class="chart-container w-full p-4">
          <div class="w-full flex justify-between items-center md:pt-12">
            <h2 class="text-2xl font-medium tracking-wide text-slate-700 text-left">Top Sales By Payment Method</h2>
            <button @click="downloadPDF3" class="w-full mt-6 px-4 py-2 text-md font-normal tracking-wider text-white bg-orange-600 rounded-lg custom-select hover:bg-orange-700 hover:shadow-lg">Download PDF</button>
          </div>
          <Doughnut :data="chartData1" :options="chartOptions1" />
        </div>
      </div>
    </div>

    <!-- ─────────────────────────────────────────────────────────────── -->
    <!-- NEW: PRODUCT SALES (BY MONTH / RANGE) + "ALL PRODUCTS"          -->
    <!-- ─────────────────────────────────────────────────────────────── -->
    <div class="w-full bg-white border-4 border-black rounded-xl p-6">
      <div class="flex items-center justify-between pb-4">
        <h2 class="text-2xl font-semibold text-slate-700">Product Sales (by month / range)</h2>

        <div class="flex items-center gap-3 flex-wrap">
          <!-- Product select (All Products supported) -->
          <select v-model="selectedProductId"
                  class="text-base font-medium text-slate-700 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 p-2.5 min-w-[240px]">
            <option value="">All Products</option>
            <option v-for="p in products" :key="p.id" :value="p.id">
              {{ p.name }}
            </option>
          </select>

          <!-- Month filter (YYYY-MM) -->
          <input v-model="monthFilter" type="month"
                 class="text-base font-medium text-blue-600 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 p-2.5" />

          <span class="text-base font-bold tracking-wider text-blue-600">Or</span>

          <!-- Explicit range just for this widget -->
          <input v-model="prodStart" type="date"
                 class="text-base font-medium text-blue-600 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 p-2.5" />
          <span class="text-base font-bold tracking-wider text-blue-600">To</span>
          <input v-model="prodEnd" type="date"
                 class="text-base font-medium text-blue-600 bg-white border border-blue-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 p-2.5" />

          <button @click="clearProductFilters"
                  class="px-4 py-2 text-md font-semibold text-white bg-blue-600 rounded-lg hover:bg-blue-700 shadow-md">
            Clear
          </button>

          <button @click="downloadProductSalesPDF"
                  class="px-4 py-2 text-md font-semibold text-white bg-orange-600 rounded-lg hover:bg-orange-700 shadow-md">
            Download PDF
          </button>
        </div>
      </div>

      <!-- KPI pills -->
      <div class="flex flex-wrap gap-3 mb-4">
        <div class="py-2 px-4 border-2 border-green-600 rounded-xl bg-green-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Qty:
            <span class="text-base font-bold">{{ prodQtyTotal.toLocaleString() }}</span>
          </p>
        </div>
        <div class="py-2 px-4 border-2 border-blue-600 rounded-xl bg-blue-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Gross:
            <span class="text-base font-bold">
              {{ prodGrossTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
            </span>
          </p>
        </div>
        <div class="py-2 px-4 border-2 border-yellow-600 rounded-xl bg-yellow-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Discount (allocated):
            <span class="text-base font-bold">
              {{ prodDiscountTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
            </span>
          </p>
        </div>
        <div class="py-2 px-4 border-2 border-slate-600 rounded-xl bg-slate-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Net:
            <span class="text-base font-bold">
              {{ prodNetTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
            </span>
          </p>
        </div>
        <div class="py-2 px-4 border-2 border-purple-600 rounded-xl bg-purple-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Cost:
            <span class="text-base font-bold">
              {{ prodCostTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
            </span>
          </p>
        </div>
        <div class="py-2 px-4 border-2 border-red-600 rounded-xl bg-red-100 shadow-sm text-center">
          <p class="text-sm font-extrabold text-black uppercase">
            Profit:
            <span class="text-base font-bold">
              {{ prodProfitTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
            </span>
          </p>
        </div>
      </div>

      <!-- Items table -->
      <div class="overflow-x-auto overflow-y-auto max-h-[420px] border rounded-xl mt-2">
        <table id="productSalesTbl" class="w-full text-gray-800 bg-white border border-gray-300 rounded-lg shadow-md table-auto">
          <thead class="sticky top-0 z-10">
            <tr class="bg-gradient-to-r from-emerald-700 via-emerald-600 to-emerald-700 text-white text-[14px] border-b border-emerald-800">
              <th class="p-3 text-left font-semibold">#</th>
              <th class="p-3 text-left font-semibold">Date</th>
              <th class="p-3 text-left font-semibold">Order</th>
              <th class="p-3 text-left font-semibold">Customer</th>
              <th class="p-3 text-left font-semibold">Product</th>
              <th class="p-3 text-center font-semibold">Qty</th>
              <th class="p-3 text-right font-semibold">Unit Price</th>
              <th class="p-3 text-right font-semibold">Gross</th>
              <th class="p-3 text-right font-semibold">Alloc. Discount</th>
              <th class="p-3 text-right font-semibold">Net</th>
              <th class="p-3 text-right font-semibold">Cost</th>
              <th class="p-3 text-right font-semibold">Profit</th>
            </tr>
          </thead>

          <tbody class="text-[12px] font-medium">
            <tr v-for="(row, i) in filteredProductItems" :key="row.key" class="border-b transition duration-200 hover:bg-gray-100">
              <td class="p-3">{{ i + 1 }}</td>
              <td class="p-3 whitespace-nowrap">{{ formatDate(row.sale_date) }}</td>
              <td class="p-3">{{ row.order_label }}</td>
              <td class="p-3">{{ row.customer || 'N/A' }}</td>
              <td class="p-3">{{ row.product_name }}</td>
              <td class="p-3 text-center">{{ row.qty }}</td>
              <td class="p-3 text-right">{{ toMoney(row.unit_price) }}</td>
              <td class="p-3 text-right">{{ toMoney(row.gross) }}</td>
              <td class="p-3 text-right">{{ toMoney(row.alloc_discount) }}</td>
              <td class="p-3 text-right">{{ toMoney(row.net) }}</td>
              <td class="p-3 text-right">{{ toMoney(row.cost) }}</td>
              <td class="p-3 text-right">{{ toMoney(row.profit) }}</td>
            </tr>
          </tbody>

          <tfoot class="bg-gray-50 text-[12px] font-semibold">
            <tr>
              <td class="p-3 text-right" colspan="5">Totals:</td>
              <td class="p-3 text-center">{{ prodQtyTotal.toLocaleString() }}</td>
              <td class="p-3"></td>
              <td class="p-3 text-right">{{ prodGrossTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}</td>
              <td class="p-3 text-right">{{ prodDiscountTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}</td>
              <td class="p-3 text-right">{{ prodNetTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}</td>
              <td class="p-3 text-right">{{ prodCostTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}</td>
              <td class="p-3 text-right">{{ prodProfitTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}</td>
            </tr>
          </tfoot>
        </table>
      </div>
    </div>
    <!-- /NEW PRODUCT SALES SECTION -->

    <!-- Sales Table -->
    <div class="w-full bg-white border-4 border-black rounded-xl p-6">
      <h2 class="text-2xl font-semibold text-slate-700 text-center pb-4">Sales Table</h2>

      <div class="flex justify-between items-center pb-4">
        <div class="flex gap-4">
          <button @click="downloadSalesTablePDF"
                  class="px-4 py-2 text-md font-semibold text-white bg-orange-600 rounded-lg hover:bg-orange-700 shadow-md">
            Download PDF
          </button>
        </div>

        <div class="flex items-center gap-3">
          <div class="py-2 px-4 border-2 border-green-600 rounded-xl bg-green-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Total Qty:
              <span class="text-base font-bold">{{ salesTotalQty.toLocaleString() }}</span>
            </p>
          </div>
          <div class="py-2 px-4 border-2 border-blue-600 rounded-xl bg-blue-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Gross:
              <span class="text-base font-bold">
                {{ salesGrossTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </span>
            </p>
          </div>
          <div class="py-2 px-4 border-2 border-yellow-600 rounded-xl bg-yellow-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Discounts:
              <span class="text-base font-bold">
                {{ salesDiscountTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </span>
            </p>
          </div>
          <div class="py-2 px-4 border-2 border-red-600 rounded-xl bg-red-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Profit:
              <span class="text-base font-bold">
                {{ salesProfitTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </span>
            </p>
          </div>
        </div>
      </div>

      <div class="overflow-x-auto overflow-y-auto max-h-[480px] border rounded-xl mt-2">
        <table id="salesTbl" class="sales-table w-full text-gray-800 bg-white border border-gray-300 rounded-lg shadow-md">
          <colgroup>
            <col style="width:48px" />
            <col style="width:110px" />
            <col style="width:170px" />
            <col style="width:200px" />
            <col style="width:70px" />
            <col style="width:140px" />
            <col style="width:140px" />
            <col style="width:140px" />
            <col style="width:140px" />
          </colgroup>

          <thead class="sticky top-0 z-10">
            <tr class="bg-gradient-to-r from-slate-700 via-slate-600 to-slate-700 text-white text-[14px] border-b border-slate-800">
              <th class="p-3 text-left font-semibold">#</th>
              <th class="p-3 text-left font-semibold">Date</th>
              <th class="p-3 text-left font-semibold">Order Number</th>
              <th class="p-3 text-left font-semibold">Customer</th>
              <th class="p-3 text-center font-semibold">Qty</th>
              <th class="p-3 num font-semibold">Gross (LKR)</th>
              <th class="p-3 num font-semibold">Discounts</th>
              <th class="p-3 num font-semibold">Cost (LKR)</th>
              <th class="p-3 num font-semibold">Profit (LKR)</th>
            </tr>
          </thead>

          <tbody class="text-[12px] font-medium">
            <tr v-for="(s, i) in sales" :key="s.id ?? i" class="border-b transition duration-200 hover:bg-gray-100">
              <td class="p-3 text-center">{{ i + 1 }}</td>
              <td class="p-3 whitespace-nowrap">{{ formatDate(s.sale_date) }}</td>
              <td class="p-3">
                {{ s.order_id ? s.order_id : 'Service -' }} {{ s.service_name }}
              </td>
              <td class="p-3">{{ s.customer?.name ?? 'N/A' }}</td>
              <td class="p-3 text-center">{{ saleQty(s) }}</td>
              <td class="p-3 num">{{ toMoney(s.total_amount) }}</td>
              <td class="p-3 num">{{ discountDisplay(s) }}</td>
              <td class="p-3 num">{{ toMoney(s.total_cost || 0) }}</td>
              <td class="p-3 num">
                <ul>
                  <li class="font-semibold text-green-600">{{ toMoney(saleProfit(s)) }}</li>
                </ul>
              </td>
            </tr>
          </tbody>

          <tfoot class="bg-gray-50 text-[12px] font-semibold">
            <tr>
              <td class="p-3 text-right" colspan="4">Totals:</td>
              <td class="p-3 text-center">{{ salesTotalQty.toLocaleString() }}</td>
              <td class="p-3 num">
                {{ salesGrossTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}
              </td>
              <td class="p-3 num">
                {{ salesDiscountTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}
              </td>
              <td class="p-3 num">
                {{ salesCostTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}
              </td>
              <td class="p-3 num">
                {{ salesProfitTotal.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }}
              </td>
            </tr>
          </tfoot>
        </table>
      </div>
    </div>

    <!-- Top Products Stock Table -->
    <div class="w-full bg-white border-4 border-black rounded-xl p-6">
      <h2 class="text-2xl font-semibold text-slate-700 text-center pb-4">Top Products Stock Table</h2>

      <div class="flex justify-between items-center pb-4">
        <div class="flex gap-4">
          <button @click="downloadStockTablePDF" class="px-4 py-2 text-md font-semibold text-white bg-orange-600 rounded-lg hover:bg-orange-700 shadow-md">
            Download PDF
          </button>
        </div>

        <div class="flex items-center gap-3">
          <div class="py-2 px-4 border-2 border-green-600 rounded-xl bg-green-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Total Sales Qty:
              <span class="text-base font-bold">{{ totalSalesQty.toLocaleString() }}</span>
            </p>
          </div>
          <div class="py-2 px-4 border-2 border-blue-600 rounded-xl bg-blue-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Total Profit:
              <span class="text-base font-bold">
                {{ grandTotalProfit.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </span>
            </p>
          </div>
        </div>
      </div>

      <div class="overflow-x-auto max-h-[420px] border rounded-xl mt-2">
        <table id="stockQtyTbl" class="w-full text-gray-800 bg-white border border-gray-300 rounded-lg shadow-md table-auto">
          <thead>
            <tr class="bg-gradient-to-r from-blue-700 via-blue-600 to-blue-700 text-white text-[14px] border-b border-blue-800">
              <th class="p-3 text-left font-semibold">#</th>
              <th class="p-3 text-left font-semibold">Product</th>
              <th class="p-3 text-center font-semibold">Sales Qty</th>
              <th class="p-3 text-center font-semibold">Total Sales Value (LKR)</th>
              <th class="p-3 text-center font-semibold">Price (LKR)</th>
              <th class="p-3 text-center font-semibold">Discount</th>
              <th class="p-3 text-center font-semibold">Price After Discount</th>
              <th class="p-3 text-center font-semibold">Profit</th>
            </tr>
          </thead>

          <tbody class="text-[12px] font-medium">
            <tr v-for="(p, i) in products" :key="p.id ?? i" class="border-b transition duration-200 hover:bg-gray-100">
              <td class="p-3 text-center">{{ i + 1 }}</td>
              <td class="p-3 font-bold">{{ p.name || 'N/A' }}</td>
              <td class="p-3 text-center">{{ Number(p.sales_qty || 0) }}</td>
              <td class="p-3 text-center">
                {{ (Number(p.sales_qty || 0) * Number(p.selling_price || 0)).toFixed(2) }}
              </td>
              <td class="p-3 text-center">{{ Number(p.selling_price || 0).toFixed(2) }}</td>
              <td class="p-3 text-center">
                <span v-if="Number(p.discount || 0) <= 100">{{ Number(p.discount || 0) }}%</span>
                <span v-else>Rs. {{ Number(p.discount).toFixed(2) }}</span>
              </td>
              <td class="p-3 text-center">{{ priceAfterDiscount(p).toFixed(2) }}</td>
              <td class="p-3 text-center">{{ totalProfit(p).toFixed(2) }} LKR</td>
            </tr>
          </tbody>

          <tfoot class="bg-gray-50 text-[12px]  font-semibold">
            <tr>
              <td class="p-3 text-right" colspan="2">Totals:</td>
              <td class="p-3 text-right">{{ totalSalesQty.toLocaleString() }}</td>
              <td class="p-3 text-right"></td>
              <td class="p-3 text-right"></td>
              <td class="p-3 text-right"></td>
              <td class="p-3 text-right"></td>
              <td class="p-3 text-right">
                {{ grandTotalProfit.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </td>
            </tr>
          </tfoot>
        </table>
      </div>
    </div>

    <!-- EXPENSES SECTION -->
    <div class="w-full bg-white border-4 border-black rounded-xl p-6 mt-8">
      <h2 class="text-2xl font-semibold text-slate-700 text-center pb-4">Expenses</h2>

      <div class="flex justify-between items-center pb-4">
        <div class="flex gap-4">
          <button @click="downloadExpensesPDF"
                  class="px-4 py-2 text-md font-semibold text-white bg-orange-600 rounded-lg hover:bg-orange-700 shadow-md">
            Download PDF
          </button>
        </div>

        <div class="flex items-center gap-3">
          <div class="py-2 px-4 border-2 border-red-600 rounded-xl bg-red-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Total Expenses:
              <span class="text-base font-bold">
                {{ totalExpenseAmount.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) }} LKR
              </span>
            </p>
          </div>
          <div class="py-2 px-4 border-2 border-slate-600 rounded-xl bg-slate-100 shadow-sm text-center">
            <p class="text-sm font-extrabold text-black uppercase">
              Records:
              <span class="text-base font-bold">{{ totalExpenseCount }}</span>
            </p>
          </div>
        </div>
      </div>

      <div class="overflow-x-auto overflow-y-auto max-h-[420px] border rounded-xl mt-2">
        <table id="expenseTbl" class="w-full text-gray-800 bg-white border border-gray-300 rounded-lg shadow-md table-auto">
          <colgroup>
            <col style="width:60px" />
            <col style="width:220px" />
            <col style="width:160px" />
            <col style="width:140px" />
          </colgroup>

          <thead class="sticky top-0 z-10">
            <tr class="bg-gradient-to-r from-rose-700 via-rose-600 to-rose-700 text-white text-[14px] border-b border-rose-800">
              <th class="p-3 text-left font-semibold">#</th>
              <th class="p-3 text-left font-semibold">Title</th>
              <th class="p-3 text-right font-semibold">Amount (LKR)</th>
              <th class="p-3 text-left font-semibold">Expense Date</th>
            </tr>
          </thead>

          <tbody class="text-[12px] font-medium">
            <tr v-for="(e, i) in expenses" :key="e.id ?? i" class="border-b transition duration-200 hover:bg-gray-100">
              <td class="p-3">{{ i + 1 }}</td>
              <td class="p-3">{{ e.title || '—' }}</td>
              <td class="p-3 text-right">{{ toMoney(e.amount) }}</td>
              <td class="p-3">{{ formatDate(e.expense_date) }}</td>
            </tr>
          </tbody>

          <tfoot class="bg-gray-50 text-[12px] font-semibold">
            <tr>
              <td class="p-3 text-right" colspan="2">Total:</td>
              <td class="p-3 text-right">
                {{ totalExpenseAmount.toLocaleString(undefined,{minimumFractionDigits:2}) }}
              </td>
              <td class="p-3"></td>
            </tr>
          </tfoot>
        </table>
      </div>
    </div> <!-- /EXPENSES SECTION -->

  </div>

  <Footer />
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { Doughnut, Pie } from "vue-chartjs";
import { Link, router, Head } from "@inertiajs/vue3";
import Header from "@/Components/custom/Header.vue";
import Footer from "@/Components/custom/Footer.vue";
import Banner from "@/Components/Banner.vue";
import jsPDF from "jspdf";
import "jspdf-autotable";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  ArcElement,
  CategoryScale,
  LinearScale,
  BarElement,
} from "chart.js";

ChartJS.register(Title, Tooltip, Legend, ArcElement, CategoryScale, LinearScale, BarElement);

// Props expected from controller (updated names)
const props = defineProps({
  products: { type: Array, required: true },
  sales: { type: Array, required: true },

  totalSaleAmount: { type: Number, required: true },
  averageTransactionValue: { type: Number, required: true },
  netProfit: { type: Number, required: true },
  totalTransactions: { type: Number, required: true },

  // New clear totals (in LKR)
  totalDiscountLkr: { type: Number, required: true },        // product-level discount total
  totalCustomDiscountLkr: { type: Number, required: true },  // custom discount total after type conversion

  totalCustomer: { type: Number, required: true },
  startDate: { type: String, default: "" },
  endDate: { type: String, default: "" },
  categorySales: { type: Object, required: true },
  employeeSalesSummary: { type: Object, required: true },

  expenses: { type: Array, required: true },
  totalExpenseAmount: { type: Number, required: true },
  totalExpenseCount: { type: Number, required: true },
});

// State
const startDate = ref(props.startDate);
const endDate   = ref(props.endDate);
const products  = ref(props.products);
const sales     = ref(props.sales);

// NEW — expenses state
const expenses            = ref(props.expenses);
const totalExpenseAmount  = ref(props.totalExpenseAmount || 0);
const totalExpenseCount   = ref(props.totalExpenseCount || 0);

// Formatting helpers
const toMoney   = (n) => (Number(n || 0)).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 });
const formatDate= (d) => (d ? new Date(d).toLocaleDateString() : "");

// Totals (inventory cards)
const totalQty = computed(() => products.value.reduce((sum, p) => sum + (p.stock_quantity || 0), 0));
const totalSellingPrice = computed(() =>
  products.value.reduce((sum, p) => sum + (p.stock_quantity || 0) * (parseFloat(p.selling_price) || 0), 0)
);
const totalRetailValue = computed(() =>
  products.value.reduce((sum, p) => sum + (p.stock_quantity || 0) * (parseFloat(p.cost_price) || 0), 0)
);

// Price helpers for product table (product-level discount for product catalog display)
const priceAfterDiscount = (product) => {
  const price = Number(product.selling_price || 0);
  const discount = Number(product.discount || 0);
  return discount <= 100 ? price * (1 - discount / 100) : price - discount;
};
const profitPerUnit = (product) => priceAfterDiscount(product) - Number(product.cost_price || 0);
const totalProfit   = (product) => profitPerUnit(product) * Number(product.sales_qty || 0);

// Product table totals
const totalSalesQty    = computed(() => products.value.reduce((s, p) => s + Number(p.sales_qty || 0), 0));
const grandTotalProfit = computed(() => products.value.reduce((s, p) => s + totalProfit(p), 0));

// Date filter (IMPORTANT: preserveState false)
const filterData = () => {
  if (startDate.value && endDate.value && new Date(startDate.value) > new Date(endDate.value)) {
    alert("Start date cannot be greater than end date.");
    return;
  }
  router.get(
    route("reports.index"),
    { start_date: startDate.value, end_date: endDate.value },
    { preserveScroll: true, preserveState: false } // <<< key fix
  );
};

// ===== Charts data =====
const sortDescending = (data) =>
  Object.entries(data).sort((a, b) => b[1] - a[1]).reduce((acc, [k, v]) => ((acc[k] = v), acc), {});

const productQuantities = computed(() => {
  const quantities = {};
  props.sales.forEach((sale) => {
    sale.sale_items.forEach((item) => {
      const name = item.product && item.product.name ? item.product.name : "N/A";
      quantities[name] = (quantities[name] || 0) + item.quantity;
    });
  });
  return sortDescending(quantities);
});

const chartData = computed(() => ({
  labels: Object.keys(productQuantities.value),
  datasets: [{ data: Object.values(productQuantities.value),
    backgroundColor: ["#FF6384","#36A2EB","#FFCE56","#4BC0C0","#9966FF","#28a745","#ffc107","#17a2b8","#e83e8c","#fd7e14","#6610f2","#6f42c1","#dc3545","#adb5bd","#20c997","#ffc93c","#6a0572","#8ac926","#ff595e","#198754"] }],
}));
const chartOptions = { responsive: true, plugins: { legend: { display: true, position: "bottom" } } };

const paymentMethodTotals = computed(() => {
  const totals = {};
  props.sales.forEach((s) => {
    const m = s.payment_method;
    totals[m] = (totals[m] || 0) + parseFloat(s.total_amount);
  });
  return sortDescending(totals);
});
const chartData1 = computed(() => ({
  labels: Object.keys(paymentMethodTotals.value),
  datasets: [{ data: Object.values(paymentMethodTotals.value),
    backgroundColor: ["#FF6384","#36A2EB","#FFCE56","#4BC0C0","#9966FF","#28a745","#ffc107","#17a2b8","#e83e8c","#fd7e14","#6610f2","#6f42c1","#dc3545","#adb5bd","#20c997","#ffc93c","#6a0572","#8ac926","#ff595e","#198754"] }],
}));
const chartOptions1 = {
  responsive: true,
  plugins: { legend: { display: true, position: "bottom" },
    tooltip: { callbacks: { label: (c) => `LKR ${(+c.raw || 0).toLocaleString()}` } } },
};

const sortedEmployeeSales = computed(() =>
  Object.fromEntries(Object.entries(props.employeeSalesSummary).sort(([, a], [, b]) => b["Total Sales Amount"] - a["Total Sales Amount"]))
);
const chartData4 = computed(() => ({
  labels: Object.keys(sortedEmployeeSales.value),
  datasets: [{ data: Object.values(sortedEmployeeSales.value).map((e) => e["Total Sales Amount"]),
    backgroundColor: ["#6610f2","#36A2EB","#8ac926","#ff595e","#198754","#6f42c1","#dc3545","#adb5bd","#20c997","#28a745","#ffc107","#17a2b8","#e83e8c","#fd7e14","#FF6384","#FFCE56","#4BC0C0","#9966FF","#ffc93c"] }],
}));
const chartOptions4 = { responsive: true, plugins: { legend: { display: true, position: "bottom" },
  tooltip: { callbacks: { label: (c) => `LKR ${(+c.raw).toLocaleString()}` } } } };

// ===== Sales Table helpers & totals (respect custom_discount_type) =====
const saleQty    = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((n, it) => n + Number(it.quantity || 0), 0) : 0);

// Convert custom discount to LKR based on type for a row
const customDiscountLkr = (s) => {
  const gross = Number(s.total_amount || 0);
  const val   = Number(s.custom_discount || 0);
  const type  = s.custom_discount_type || 'fixed';
  return type === 'percent' ? (gross * val / 100) : val;
};

// display helper
const discountDisplay = (s) => {
  const parts = [];
  const gross = Number(s.total_amount || 0);
  const customVal   = Number(s.custom_discount || 0);
  const customType  = s.custom_discount_type || "fixed";
  if (customVal > 0) {
    if (customType === "percent") {
      const lkr = (gross * customVal) / 100;
      parts.push(`${customVal}% (${toMoney(lkr)} LKR)`);
    } else {
      parts.push(`${toMoney(customVal)} LKR`);
    }
  }
  return parts.join("  |  ");
};

// Net = gross - custom discount (LKR)
const netAmount    = (s) => Number(s.total_amount || 0) - customDiscountLkr(s);

// Totals
const salesTotalQty    = computed(() => sales.value.reduce((a, s) => a + saleQty(s), 0));
const salesGrossTotal  = computed(() => sales.value.reduce((a, s) => a + Number(s.total_amount || 0), 0));
const salesDiscountTotal = computed(() => sales.value.reduce((a, s) => a + customDiscountLkr(s), 0));
const salesNetTotal    = computed(() => sales.value.reduce((a, s) => a + netAmount(s), 0));
const salesCostTotal   = computed(() =>
  sales.value.reduce((a, s) => a + Number(s.total_cost || 0), 0)
);

// inside <script setup> or computed block
const salesProfitTotal = computed(() => sales.value.reduce((sum, s) => sum + saleProfit(s), 0));

// Date label for PDFs
const dateRangeLabel = computed(() => {
  const s = startDate.value ? new Date(startDate.value).toLocaleDateString() : "All";
  const e = endDate.value ? new Date(endDate.value).toLocaleDateString() : "All";
  return `${s} — ${e}`;
});

// per item helpers
const itemGrossTotal = (it) => (it.quantity || 0) * (it.product?.selling_price || 0);
const itemCostTotal  = (it) => (it.quantity || 0) * (it.product?.cost_price || 0);

// gross / cost / profit on sale
const saleGrossTotal = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((sum, it) => sum + itemGrossTotal(it), 0) : 0);
const saleCostTotal  = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((sum, it) => sum + itemCostTotal(it), 0) : 0);
const saleNetTotal   = (s) => saleGrossTotal(s) - (s.custom_discount || 0);
const saleProfit     = (s) => saleNetTotal(s) - saleCostTotal(s);

// ───────────────────────────────────────────────────────────────
// NEW: PRODUCT SALES (BY MONTH / RANGE) — STATE & COMPUTEDS
// ───────────────────────────────────────────────────────────────
const selectedProductId = ref("");
// default to current month YYYY-MM
const now = new Date();
const pad = (n) => (n < 10 ? `0${n}` : `${n}`);
const defaultMonth = `${now.getFullYear()}-${pad(now.getMonth() + 1)}`;
const monthFilter = ref(defaultMonth);
const prodStart = ref(""); // explicit range override (optional)
const prodEnd   = ref("");

// date window checker
const inProductDateWindow = (saleDateStr) => {
  const d = saleDateStr ? new Date(saleDateStr) : null;
  if (!d) return false;

  // If explicit range provided, use it
  if (prodStart.value && prodEnd.value) {
    const s = new Date(prodStart.value);
    const e = new Date(prodEnd.value);
    if (s > e) return false;
    return d >= s && d <= e;
  }

  // Else use monthFilter (YYYY-MM).
  if (monthFilter.value) {
    const [y, m] = monthFilter.value.split("-").map(Number);
    if (!y || !m) return true;
    const first = new Date(y, m - 1, 1);
    const next  = new Date(y, m, 1);
    return d >= first && d < next;
  }

  return true;
};

// flatten sale -> items; include ALL products if none selected
const filteredProductItems = computed(() => {
  const rows = [];
  for (const s of sales.value) {
    if (!inProductDateWindow(s.sale_date)) continue;

    const grossSale = saleGrossTotal(s); // for pro-rata allocation
    const discountSaleLkr = customDiscountLkr(s);

    for (const it of (s.sale_items || [])) {
      const prod = it?.product;
      if (!prod) continue;
      if (selectedProductId.value && String(prod.id) !== String(selectedProductId.value)) continue;

      const qty = Number(it.quantity || 0);
      const unitPrice = Number(prod.selling_price || 0);
      const gross = qty * unitPrice;
      const cost  = qty * Number(prod.cost_price || 0);

      // allocate sale-level discount pro-rata by item's share of sale gross
      const alloc = grossSale > 0 ? (discountSaleLkr * (gross / grossSale)) : 0;

      const net   = gross - alloc;
      const profit= net - cost;

      rows.push({
        key: `${s.id}-${prod.id}-${rows.length}`,
        sale_date: s.sale_date,
        order_label: s.order_id ? s.order_id : `Service - ${s.service_name || ""}`,
        customer: s.customer?.name || "",
        product_name: prod.name || "",
        qty,
        unit_price: unitPrice,
        gross,
        alloc_discount: alloc,
        net,
        cost,
        profit,
      });
    }
  }

  // sort by date asc for readability
  return rows.sort((a, b) => new Date(a.sale_date) - new Date(b.sale_date));
});

// totals for the section
const prodQtyTotal      = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.qty || 0), 0));
const prodGrossTotal    = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.gross || 0), 0));
const prodDiscountTotal = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.alloc_discount || 0), 0));
const prodNetTotal      = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.net || 0), 0));
const prodCostTotal     = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.cost || 0), 0));
const prodProfitTotal   = computed(() => filteredProductItems.value.reduce((t, r) => t + Number(r.profit || 0), 0));

const clearProductFilters = () => {
  selectedProductId.value = "";
  monthFilter.value = defaultMonth;
  prodStart.value = "";
  prodEnd.value = "";
};

// PDF for product sales
const downloadProductSalesPDF = () => {
  const doc = new jsPDF("l", "mm", "a4");
  const gen = new Date();

  const productName = selectedProductId.value
    ? (products.value.find(p => String(p.id) === String(selectedProductId.value))?.name || "—")
    : "All Products";

  const period = prodStart.value && prodEnd.value
    ? `${new Date(prodStart.value).toLocaleDateString()} — ${new Date(prodEnd.value).toLocaleDateString()}`
    : (monthFilter.value ? monthFilter.value : "All");

  doc.setFontSize(16);
  doc.text("Product Sales Report", 14, 12);
  doc.setFontSize(10);
  doc.text(`Product: ${productName} • Period: ${period} • Generated: ${gen.toLocaleString()}`, 14, 18);

  const rows = filteredProductItems.value.map((r, i) => ([
    i + 1,
    new Date(r.sale_date).toLocaleDateString(),
    r.order_label || "",
    r.customer || "",
    r.product_name || "",
    String(r.qty),
    (Number(r.unit_price)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    (Number(r.gross)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    (Number(r.alloc_discount)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    (Number(r.net)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    (Number(r.cost)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    (Number(r.profit)).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
  ]));

  doc.autoTable({
    head: [["#","Date","Order","Customer","Product","Qty","Unit Price","Gross","Alloc. Discount","Net","Cost","Profit"]],
    body: rows,
    startY: 24,
    theme: "striped",
    styles: { fontSize: 9, cellPadding: 2 },
    headStyles: { fillColor: [5, 150, 105], textColor: 255 },
    columnStyles: {
      0: { cellWidth: 10 },
      1: { cellWidth: 22 },
      2: { cellWidth: 34 },
      3: { cellWidth: 34 },
      4: { cellWidth: 34 },
      5: { cellWidth: 14, halign: "center" },
      6: { cellWidth: 24, halign: "right" },
      7: { cellWidth: 26, halign: "right" },
      8: { cellWidth: 26, halign: "right" },
      9: { cellWidth: 26, halign: "right" },
      10:{ cellWidth: 26, halign: "right" },
      11:{ cellWidth: 26, halign: "right" },
    },
    margin: { top: 18, left: 8, right: 8 },
  });

  const totalsRow = [
    { content: "Totals:", colSpan: 5, styles: { halign: "right", fontStyle: "bold" } },
    { content: prodQtyTotal.value.toLocaleString(), styles: { halign: "center", fontStyle: "bold" } },
    "",
    prodGrossTotal.value.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    prodDiscountTotal.value.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    prodNetTotal.value.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    prodCostTotal.value.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
    prodProfitTotal.value.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}),
  ];

  doc.autoTable({
    body: [totalsRow],
    startY: (doc.lastAutoTable?.finalY ?? 24) + 2,
    theme: "plain",
    styles: { fontSize: 10, fontStyle: "bold" },
    margin: { left: 8, right: 8 },
  });

  const safe = (s) => String(s).replace(/[^\dA-Za-z-]/g, "_");
  doc.save(`Product_Sales_${safe(productName)}_${safe(period)}.pdf`);
};

// ===== Chart PDFs =====
const downloadPDF = () => {
  const doc = new jsPDF();
  doc.text("Top Employee Sales", 14, 10);
  const rows = Object.entries(sortedEmployeeSales.value).map(([employee, entry]) => [employee, entry["Total Sales Amount"]]);
  doc.autoTable({ head: [["Employee", "Total Sales Amount"]], body: rows, startY: 20 });
  doc.save("EmployeeSales.pdf");
};
const downloadPDF2 = () => {
  const doc = new jsPDF();
  doc.text("Product Quantities", 14, 10);
  const rows = Object.entries(productQuantities.value).map(([product, qty]) => [product, qty]);
  doc.autoTable({ head: [["Product Name", "Quantity"]], body: rows, startY: 20 });
  doc.save("ProductQuantities.pdf");
};
const downloadPDF3 = () => {
  const doc = new jsPDF();
  doc.text("Payment Method Totals", 14, 10);
  const rows = Object.entries(paymentMethodTotals.value).map(([m, t]) => [m, `LKR ${t.toLocaleString()}`]);
  doc.autoTable({ head: [["Payment Method", "Total Amount"]], body: rows, startY: 20 });
  doc.save("PaymentMethodTotals.pdf");
};

 // ===== Sales table PDF =====
const downloadSalesTablePDF = () => {
  const doc = new jsPDF("l", "mm", "a4");
  const now = new Date();

  const toMoney = (n) => (Number(n || 0)).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 });
  const formatDate = (d) => (d ? new Date(d).toLocaleDateString() : "");
  const saleQty = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((total, item) => total + Number(item.quantity || 0), 0) : 0);
  const customDiscountLkr = (s) => {
    const gross = Number(s.total_amount || 0);
    const val = Number(s.custom_discount || 0);
    const type = s.custom_discount_type || 'fixed';
    return type === 'percent' ? (gross * val / 100) : val;
  };
  const discountDisplay = (s) => {
    const parts = [];
    const gross = Number(s.total_amount || 0);
    const customVal = Number(s.custom_discount || 0);
    const customType = s.custom_discount_type || "fixed";
    if (customVal > 0) {
      if (customType === "percent") {
        const lkr = (gross * customVal) / 100;
        parts.push(`${customVal}% (${toMoney(lkr)} LKR)`);
      } else {
        parts.push(`${toMoney(customVal)} LKR`);
      }
    }
    return parts.join("  |  ");
  };
  const saleProfit = (s) => {
    const saleGrossTotal = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((sum, it) => sum + ((it.quantity || 0) * (it.product?.selling_price || 0)), 0) : 0);
    const saleCostTotal = (s) => (Array.isArray(s.sale_items) ? s.sale_items.reduce((sum, it) => sum + ((it.quantity || 0) * (it.product?.cost_price || 0)), 0) : 0);
    const saleNetTotal = (s) => saleGrossTotal(s) - (s.custom_discount || 0);
    return saleNetTotal(s) - saleCostTotal(s);
  };

  const rowData = sales.value.map((s, i) => {
    const date = formatDate(s.sale_date);
    const orderNumber = s.order_id ? s.order_id : `Service - ${s.service_name || ''}`;
    const customer = s.customer?.name ?? 'N/A';
    const qty = saleQty(s);
    const gross = toMoney(s.total_amount);
    const discounts = discountDisplay(s);
    const cost = toMoney(s.total_cost || 0);
    const profit = toMoney(saleProfit(s));

    return [ i + 1, date, orderNumber, customer, qty.toString(), gross, discounts, cost, profit ];
  });

  const totalQty = sales.value.reduce((a, s) => a + saleQty(s), 0);
  const totalGross = sales.value.reduce((a, s) => a + Number(s.total_amount || 0), 0);
  const totalDiscounts = sales.value.reduce((a, s) => a + customDiscountLkr(s), 0);
  const totalCost = sales.value.reduce((a, s) => a + Number(s.total_cost || 0), 0);
  const totalProfit = sales.value.reduce((a, s) => a + saleProfit(s), 0);

  doc.setFontSize(16);
  doc.text("Sales Report", 14, 12);
  doc.setFontSize(10);
  doc.text(`Date range: ${dateRangeLabel.value} • Generated: ${now.toLocaleString()}`, 14, 18);

  const head = [[ "#","Date","Order Number","Customer","Qty","Gross (LKR)","Discounts","Cost (LKR)","Profit (LKR)" ]];

  const foot = [[
    { content: "Totals:", colSpan: 4, styles: { halign: "right", fontStyle: "bold" } },
    { content: totalQty.toLocaleString(), styles: { halign: "center", fontStyle: "bold" } },
    toMoney(totalGross),
    toMoney(totalDiscounts),
    toMoney(totalCost),
    toMoney(totalProfit)
  ]];

  doc.autoTable({
    head,
    body: rowData,
    foot,
    startY: 24,
    theme: "striped",
    styles: { fontSize: 9, cellPadding: 2 },
    headStyles: { fillColor: [51, 65, 85], textColor: 255 },
    columnStyles: {
      0: { cellWidth: 10 },
      1: { cellWidth: 22 },
      2: { cellWidth: 36 },
      3: { cellWidth: 36 },
      4: { cellWidth: 14, halign: "center" },
      5: { cellWidth: 28, halign: "right" },
      6: { cellWidth: 28, halign: "right" },
      7: { cellWidth: 28, halign: "right" },
      8: { cellWidth: 28, halign: "right" },
    },
    margin: { top: 18, left: 8, right: 8 },
    didParseCell: (data) => {
      if (data.section === "foot") {
        if (data.column.index === 4) data.cell.styles.halign = "center"; // Qty
        if (data.column.index >= 5)  data.cell.styles.halign = "right";  // Money columns
        if (data.column.index <= 3)  data.cell.styles.halign = "right";  // "Totals:" span
      }
    }
  });

  const safe = (s) => String(s).replace(/[^\dA-Za-z-]/g, "_");
  doc.save(`Sales_Report_${safe(dateRangeLabel.value)}.pdf`);
};

// ===== Stock table PDF =====
const downloadStockTablePDF = () => {
  const tableEl = document.getElementById("stockQtyTbl");
  if (!tableEl) return;

  const toNumber = (txt) => Number(String(txt).replace(/[^\d.\-]/g, "")) || 0;
  const rows = [];
  const $ = window.$;

  if ($ && $.fn && $.fn.dataTable && $.fn.dataTable.isDataTable("#stockQtyTbl")) {
    const dt = $("#stockQtyTbl").DataTable();
    dt.rows({ search: "applied" }).every(function () {
      const tr = this.node();
      const cells = Array.from(tr.querySelectorAll("td")).map(td => (td.textContent || "").trim());
      rows.push([cells[0], cells[1], cells[2], cells[3], cells[4], cells[5], cells[6], cells[7]]);
    });
  } else {
    tableEl.querySelectorAll("tbody tr").forEach((tr) => {
      const cells = Array.from(tr.querySelectorAll("td")).map(td => (td.textContent || "").trim());
      rows.push([cells[0], cells[1], cells[2], cells[3], cells[4], cells[5], cells[6], cells[7]]);
    });
  }

  const filteredTotals = rows.reduce(
    (acc, r) => {
      acc.qty += Number(r[2]) || 0;
      acc.sales += toNumber(r[3]);
      acc.profit += toNumber(r[7]);
      return acc;
    },
    { qty: 0, sales: 0, profit: 0 }
  );

  const doc = new jsPDF("l", "mm", "a4");
  doc.setFontSize(16);
  doc.text("Top Products Stock Report", 14, 12);
  doc.setFontSize(10);
  doc.text(`Date range: ${dateRangeLabel.value} • Generated: ${new Date().toLocaleString()}`, 14, 18);

  const head = [[ "#","Product","Sales Qty","Total Sales Value (LKR)","Price (LKR)","Discount","Price After Discount","Profit" ]];

  doc.autoTable({
    head, body: rows, startY: 24, theme: "striped",
    styles: { fontSize: 9 }, headStyles: { fillColor: [33, 102, 197], textColor: 255 },
    columnStyles: {
      0: { cellWidth: 10 }, 1: { cellWidth: 60 }, 2: { cellWidth: 22, halign: "right" },
      3: { cellWidth: 34, halign: "right" }, 4: { cellWidth: 28, halign: "right" },
      5: { cellWidth: 28, halign: "center" }, 6: { cellWidth: 34, halign: "right" }, 7: { cellWidth: 34, halign: "right" },
    },
    margin: { top: 18, left: 8, right: 8 },
  });

  const totalsRow = [
    "", "Totals:", filteredTotals.qty.toLocaleString(),
    filteredTotals.sales.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) + " LKR",
    "—", "—", "—",
    filteredTotals.profit.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) + " LKR",
  ];

  doc.autoTable({
    body: [totalsRow], startY: doc.lastAutoTable ? doc.lastAutoTable.finalY + 2 : 24,
    theme: "plain", styles: { fontSize: 10, fontStyle: "bold" },
    columnStyles: {
      0: { cellWidth: 10 }, 1: { cellWidth: 60, halign: "right" }, 2: { cellWidth: 22, halign: "right" },
      3: { cellWidth: 34, halign: "right" }, 4: { cellWidth: 28 }, 5: { cellWidth: 28 }, 6: { cellWidth: 34 },
      7: { cellWidth: 34, halign: "right" },
    },
    margin: { left: 8, right: 8 },
  });

  const safe = (s) => s.replace(/[^\dA-Za-z-]/g, "_");
  doc.save(`Top_Products_Stock_${safe(dateRangeLabel.value)}.pdf`);
};

// ===== DataTables init =====
onMounted(() => {
  const jq = window.$;

  // NEW: Product sales items table
  const $prod = jq && jq("#productSalesTbl");
  if ($prod && jq.fn?.dataTable) {
    if (jq.fn.dataTable.isDataTable($prod)) $prod.DataTable().destroy();
    const dp = $prod.DataTable({
      dom: "Bfrtip",
      paging: false,
      buttons: [],
      columnDefs: [{ targets: 0, searchable: false, orderable: false }],
      initComplete: function () {
        const $input = jq("div.dataTables_filter input");
        $input.attr("placeholder", "Search product sales…");
        $input.on("keypress", function (e) {
          if (e.which == 13) dp.search(this.value).draw();
        });
      },
      language: { search: "" },
    });
  }

  // Stock table
  const $stock = jq && jq("#stockQtyTbl");
  if ($stock && jq.fn.dataTable) {
    if (jq.fn.dataTable.isDataTable($stock)) $stock.DataTable().destroy();
    const dt = $stock.DataTable({
      dom: "Bfrtip",
      paging: false,
      buttons: [],
      columnDefs: [{ targets: 0, searchable: false, orderable: false }],
      initComplete: function () {
        const $input = jq("div.dataTables_filter input");
        $input.attr("placeholder", "Search stock...");
        $input.on("keypress", function (e) {
          if (e.which == 13) dt.search(this.value).draw();
        });
      },
      language: { search: "" },
    });
  }

  // Expenses table
  const $exp = jq && jq("#expenseTbl");
  if ($exp && jq.fn.dataTable) {
    if (jq.fn.dataTable.isDataTable($exp)) $exp.DataTable().destroy();
    const de = $exp.DataTable({
      dom: "Bfrtip",
      paging: false,
      buttons: [],
      columnDefs: [{ targets: 0, searchable: false, orderable: false }],
      initComplete: function () {
        const $input = jq("div.dataTables_filter input");
        $input.attr("placeholder", "Search expenses...");
        $input.on("keypress", function (e) {
          if (e.which == 13) de.search(this.value).draw();
        });
      },
      language: { search: "" },
    });
  }
});

// re-init Product items DataTable when any filter changes
watch([selectedProductId, monthFilter, prodStart, prodEnd], () => {
  const jq = window.$;
  if (!jq?.fn?.dataTable) return;
  const table = jq("#productSalesTbl");
  if (!table.length) return;
  if (jq.fn.dataTable.isDataTable(table)) {
    const inst = table.DataTable();
    inst.clear();
    setTimeout(() => inst.rows.add(jq("#productSalesTbl tbody tr")).draw(), 0);
  }
});
</script>

<style>
/* DataTables cosmetics */
.dataTables_wrapper .dataTables_paginate {
  display: flex; justify-content: center; align-items: center; margin-top: 20px;
}
#salesTbl_filter, #stockQtyTbl_filter, #expenseTbl_filter, #productSalesTbl_filter {
  display: flex; justify-content: flex-end; align-items: center; margin-bottom: 16px; float: left;
}
#salesTbl_filter label, #stockQtyTbl_filter label, #expenseTbl_filter label, #productSalesTbl_filter label {
  font-size: 17px; color: #000000; display: flex; align-items: center;
}
#salesTbl_filter input[type="search"], #stockQtyTbl_filter input[type="search"], #expenseTbl_filter input[type="search"], #productSalesTbl_filter input[type="search"] {
  font-weight: 400; padding: 9px 15px; font-size: 14px; color: #000000cc;
  border: 1px solid rgb(209 213 219); border-radius: 5px; background: #fff;
  outline: none; transition: all 0.5s ease;
}
#salesTbl_filter input[type="search"]:focus, #stockQtyTbl_filter input[type="search"]:focus, #expenseTbl_filter input[type="search"]:focus, #productSalesTbl_filter input[type="search"]:focus {
  border: 1px solid #4b5563; box-shadow: none;
}
.dataTables_wrapper { margin-bottom: 10px; }
</style>

<style scoped>
.chart-container {
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  width: 100%; height: calc(100% - 50px); position: relative;
}
thead { position: sticky; top: 0; z-index: 10; }
.max-h-64 { max-height: 16rem; }
</style>
