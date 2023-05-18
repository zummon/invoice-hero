<script>
	import { onMount } from "svelte";
	export let data;

	let l = data[""].label[""];
	let q = data[""].q;

	let saveink = false

	const price = number => {
		number = Number(number);
		if (number === 0 || isNaN(number)) {
			return "";
		}
		return `${q.currency} ${number.toLocaleString(undefined, {
			minimumFractionDigits: 2,
		})}`;
	};
	const qty = number => {
	  number = Number(number);
	  if (number === 0 || isNaN(number)) {
	    return "";
	  }
	  return number.toLocaleString(undefined, {
	    minimumFractionDigits: 0,
	  });
	};
	const rate = rate => {
	  rate = Number(rate) * 100;
	  if (!Number.isInteger(rate)) {
	    rate = rate.toFixed(2);
	  }
	  return `${rate} %`;
	};
	const addItem = () => {
	  q.itemDesc.push("");
	  q.itemPrice.push("");
	  q.itemQty.push("");
	  q = q;
	};
	const removeItem = () => {
	  q.itemDesc.pop();
	  q.itemPrice.pop();
	  q.itemQty.pop();
	  q = q;
	};

	onMount(() => {
	  const s = new URLSearchParams(location.search);
	  let obj = q;
	  Object.keys(q).forEach(key => {
	    const values = s.getAll(key);
	    if (values.length > 0) {
	      if (Array.isArray(q[key])) {
	        obj[key] = values;
	        return;
	      }
	      obj[key] = values[0];
	    }
	  });
	  q = { ...data[q.lang].q, ...obj };
	});

	$: l = {
	  ...data[q.lang].label[""],
	  ...data[q.lang].label[q.doc]
	};
	$: q.itemAmount = q.itemPrice.map((pr, i) => {
	  const num = Number(pr) * Number(q.itemQty[i]);
	  return num ? num : "";
	});
	$: q.totalAmount = q.itemAmount.reduce((a, b) => {
	  const num = Number(a) + Number(b);
	  return num ? num : "";
	}, 0);
	$: q.totalVat = Number(q.totalAmount) * Number(q.vatRate);
	$: q.totalWht = Number(q.totalAmount) * Number(q.whtRate);
	$: q.totalFinal =
	  Number(q.totalAmount) +
	  Number(q.totalVat) +
	  Number(q.totalWht) +
	  Number(q.totalAdjust);
</script>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
{#each Object.keys(data) as lng, i (`lang-${i}`)}
		<button class="block duration-300 p-4 {q.lang === lng ? "bg-red-600 text-gray-100" : "text-gray-900 bg-gray-100 hover:bg-red-600 focus:bg-red-600 hover:text-gray-100 focus:text-gray-100"}" on:click={() => {
			q.lang = lng
		}}>
			{data[lng]['']}
		</button>
	{/each}
	{#each Object.keys(data[q.lang].label) as dc, i (`doc-${i}`)}
		<button class="block duration-300 p-4 {q.doc === dc ? "bg-red-600 text-gray-100" : "text-gray-900 bg-gray-100 hover:bg-red-600 focus:bg-red-600 hover:text-gray-100 focus:text-gray-100"}" on:click={() => {
			q.doc = dc
		}}>
			{data[q.lang].label[dc].title}
		</button>
	{/each}
</div>

<div class="font-sans bg-white text-gray-900 max-w-[60rem] mx-auto print:max-w-none print:mx-0">
	<div class="flex px-4 py-8 font-bold {saveink ? '' : 'text-white bg-red-500'}">
		<div class="">
			<h1 class="text-5xl mb-4">{l.title}</h1>
			<div class="grid grid-cols-2 gap-2">
				<div class="">{l.ref}</div>
				<div class="" contenteditable="true" bind:textContent={q.ref}></div>
				<div class="">{l.date}</div>
				<div class="" contenteditable="true" bind:textContent={q.date}></div>
				{#if q.doc !== 'receipt'}
					<div class="">{l.duedate}</div>
					<div class="" contenteditable="true" bind:textContent={q.duedate}></div>
				{/if}
			</div>
		</div>
		<div class="flex-grow text-right">
			<h2 class="text-2xl" contenteditable="true" bind:textContent={q.vendorName}></h2>
			<p class="" contenteditable="true" bind:textContent={q.vendorId}></p>
			<p class="" contenteditable="true" bind:textContent={q.vendorAddress}></p>
		</div>
	</div>
	<div class="grid grid-cols-2 gap-4 p-4">
		<div class="">
			<p class="font-bold mb-2">{l.paymethod}</p>
			<p class="p-2 rounded border border-red-500 mb-4" contenteditable="true" bind:textContent={q.paymethod}></p>

			<p class="font-bold mb-2">{l.subject}</p>
			<p class="p-2 rounded border border-red-500" contenteditable="true" bind:textContent={q.subject}></p>
		</div>
		<div class="">
			<p class="font-bold mb-2">{l.client}</p>
			<p class="p-2 rounded border border-red-500" contenteditable="true" bind:textContent={q.clientName}></p>
			<p class="px-2 py-1 rounded border-b border-l border-r" contenteditable="true" bind:textContent={q.clientId}></p>
			<p class="px-2 py-1 rounded border-b border-l border-r" contenteditable="true" bind:textContent={q.clientAddress}></p>
		</div>
	</div>
	<div class="px-4">
		<table class="w-full">
			<thead class="border-b-2 border-red-500">
				<tr class="">
					<th class="py-1 px-2 text-left w-px whitespace-nowrap">{l.itemNo}</th>
					<th class="text-left" colspan="2">
						<div class="flex">
							<p class="py-1 px-2 flex-grow">{l.itemDesc}</p>
							<button class="py-1 px-2 text-gray-900 focus:bg-red-100 hover:bg-red-100 transition duration-300 ease-in-out rounded-full print:hidden" on:click={addItem}>
								<!-- heroicons outline duplicate -->
								<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
								</svg>
							</button>
							<button class="py-1 px-2 text-gray-900 focus:bg-red-100 hover:bg-red-100 transition duration-300 ease-in-out rounded-full print:hidden" on:click={removeItem}>
								<!-- heroicons outline trash -->
								<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
								</svg>
							</button>
						</div>
					</th>
					<th class="py-1 px-2 text-right w-px whitespace-nowrap">{l.itemPrice}</th>
					<th class="py-1 px-2 text-right w-px whitespace-nowrap">{l.itemQty}</th>
					<th class="py-1 px-2 text-right w-px whitespace-nowrap">{l.itemAmount}</th>
				</tr>
			</thead>
			<tbody class="">
				{#each q.itemDesc as _, i (`item-${i}`)}
					<tr class="border-b">
						<td class="py-1 px-2 whitespace-nowrap" contenteditable="true">{i + 1}</td>
						<td class="py-1 px-2" colspan="2" contenteditable="true" bind:textContent={q.itemDesc[i]}></td>
						<td class="py-1 px-2 text-right whitespace-nowrap" contenteditable="true" 
							on:focus={(e) => e.target.textContent = q.itemPrice[i]}
							on:input={(e) => q.itemPrice[i] = e.target.textContent}
							on:blur={(e) => e.target.textContent = price(q.itemPrice[i])}
						>
							{price(q.itemPrice[i])}
						</td>
						<td class="py-1 px-2 text-right whitespace-nowrap" contenteditable="true" 
							on:focus={(e) => e.target.textContent = q.itemQty[i]}
							on:input={(e) => q.itemQty[i] = e.target.textContent}
							on:blur={(e) => e.target.textContent = qty(q.itemQty[i])}
						>
							{qty(q.itemQty[i])}
						</td>
						<td class="py-1 px-2 text-right whitespace-nowrap">
							{price(q.itemAmount[i])}
						</td>
					</tr>
				{/each}
			</tbody>
			<tfoot class="border-t-2 border-red-500 text-right">
				<tr class="">
					<td class="text-left pr-4" colspan="2" rowspan={q.doc === 'receipt' ? 5 : 4}>
						<p class="font-bold mb-2">{l.note}</p>
						<p class="p-2 rounded border border-red-500" contenteditable="true" bind:textContent={q.note}></p>
					</td>
					<th class="py-1 px-2 border-b" colspan="3">{l.totalAmount}</th>
					<th class="py-1 px-2 border-b whitespace-nowrap">
						{price(q.totalAmount)}
					</th>
				</tr>
				<tr class="border-b">
					<th class="py-1 px-2" colspan="3">
						<span class="">{l.totalVat}</span>
						<span class=""> </span>
						<span class="" contenteditable="true" 
							on:focus={(e) => e.target.textContent = q.vatRate}
							on:input={(e) => q.vatRate = e.target.textContent}
							on:blur={(e) => e.target.textContent = rate(q.vatRate)}
						>
							{rate(q.vatRate)}
						</span>
					</th>
					<th class="py-1 px-2 whitespace-nowrap">
						{price(q.totalVat)}
					</th>
				</tr>
				{#if q.doc === 'receipt'}
					<tr class="border-b">
						<th class="py-1 px-2" colspan="3">
							<span class="">{l.totalWht}</span>
							<span class=""> </span>
							<span class="" contenteditable="true" 
								on:focus={(e) => e.target.textContent = q.whtRate}
								on:input={(e) => q.whtRate = e.target.textContent}
								on:blur={(e) => e.target.textContent = rate(q.whtRate)}
							>
								{rate(q.whtRate)}
							</span>
						</th>
						<th class="py-1 px-2 whitespace-nowrap">
							{price(q.totalWht)}
						</th>
					</tr>
				{/if}
				<tr class="border-b">
					<th class="py-1 px 2" colspan="3">{l.totalAdjust}</th>
					<th class="py-1 px-2" contenteditable="true" 
						on:focus={(e) => e.target.textContent = q.totalAdjust}
						on:input={(e) => q.totalAdjust = e.target.textContent}
						on:blur={(e) => e.target.textContent = price(q.totalAdjust)}
					>
						{price(q.totalAdjust)}
					</th>
				</tr>
				<tr class="">
					<th class="py-1 px 2" colspan="3">{l.totalFinal}</th>
					<th class="py-1 px-2 whitespace-nowrap">
						{price(q.totalFinal)}
					</th>
				</tr>
			</tfoot>
		</table>
	</div>
	<div class="grid grid-cols-3 gap-4 px-4 mb-4">
		<div class="">
			<p class="font-bold text-sm mb-2">{l.vendorSign}</p>
			<p class="px-2 py-1 rounded border border-red-500" contenteditable="true"><br><br></p>
			<p class="px-2 py-1 rounded border-b border-l border-r" contenteditable="true"></p>
		</div>
		<div class="">
			<p class="font-bold text-sm mb-2">{l.clientSign}</p>
			<p class="px-2 py-1 rounded border border-red-500" contenteditable="true"><br><br></p>
			<p class="px-2 py-1 rounded border-b border-l border-r" contenteditable="true"></p>
		</div>
		<div class="text-center">
			<h3 class="text-xl font-bold text-red-500 py-8">{l.thankMessage}</h3>
			<div class="">
				<img class="mx-auto" src={q.vendorStamp} alt="" width="" height="">
			</div>
		</div>
	</div>
</div>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden gap-4">
	<label>
		<span>Currency</span>
		<input class="border border-red-600 w-12" type="text" bind:value={q.currency} />
	</label>
	<button class="block duration-300 p-4 text-gray-100 bg-red-600 hover:bg-gray-100 focus:bg-gray-100 hover:text-gray-900 focus:text-gray-900" on:click={() => {window.print()}}>
		Print
	</button>
	<label>
		<span>Save ink</span>
		<input class="accent-red-600" type="checkbox" bind:checked={saveink} />
	</label>
</div>