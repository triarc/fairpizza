<script lang='ts'>
	import Map from '../map/Map.svelte';
	import prices from '../../../static/prices.json';

	const combinedPrices = Object.keys(prices).reduce((map, zip) => {
		const sourceZip = zip.length === 4 ? parseInt(zip) : parseInt(zip.substr(0, 4));
		if (!map[sourceZip]) {
			map[sourceZip] = {};
		}

		const targetPrices = prices[zip];
		for (const priceKey of Object.keys(targetPrices)) {
			const targetZip = zip.length === 4 ? parseInt(priceKey) : parseInt(priceKey.substr(0, 4));
			const price = targetPrices[priceKey];
			if (map[sourceZip][targetZip] === undefined) {
				map[sourceZip][targetZip] = price;
			} else if (map[sourceZip][targetZip] < price) {
				map[sourceZip][targetZip] = price;
			}
		}

		return map;
	}, {});

	let zone = 1;
	let zipCode = 8005;
	let zone1 = [];
	let zone2 = [];
	let zone3 = [];
	let selectedZone = [];
	let deliverySplit = 0.5;
	let pricings = [];

	let splits = [0, 0.25, 0.5, 0.75, 1];

	function updateZipCode(evt) {
		zipCode = parseInt(evt.detail);
		update();
	}

	function onChange(event) {
		zone = parseInt(event.currentTarget.value);
		update();
	}

	function update() {
		pricings = getPricings();
		zone1 = Object.keys(combinedPrices[zipCode]).filter(target => target === zipCode.toString() || combinedPrices[zipCode][target] <= 16);
		zone2 = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] <= 26 && combinedPrices[zipCode][target] > 16 && target !== zipCode.toString());
		zone3 = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] > 26);
		selectedZone = zone === 1 ? zone1 : zone === 2 ? zone2 : zone3;
	}

	function getZonePrice() {
		if (zone === 1) {
			return 14;
		} else if (zone === 2) {
			return 18;
		} else if (zone === 3) {
			return 27;
		}
	}

	function getPricings() {
		const zonePrice = getZonePrice();
		return [30, 40, 50, 60, 70, 80, 90, 100, 150, 200, 250, 300].map(orderValue => {
			const orderFee = orderValue * 0.029 + 1.3 + (zonePrice * (1 - deliverySplit));
			const revenue = orderValue - orderFee;
			return {
				orderValue,
				orderFee: orderFee.toFixed(2),
				deliveryFee: (zonePrice * deliverySplit).toFixed(2),
				revenue: revenue.toFixed(2),
				percentage: (100 * orderFee / orderValue).toFixed(2)
			};
		});
	}

	function updateSplit(split) {
		deliverySplit = split;
		update();
	}

	update();
</script>

<div class='py-16 bg-gray-50 overflow-hidden lg:py-24' id='delivery'>
	<div class='relative max-w-xl mx-auto px-4 sm:px-6 lg:px-8 lg:max-w-7xl'>

		<div class='relative'>
			<h2 class='text-center text-3xl leading-8 font-extrabold tracking-tight text-gray-900 sm:text-4xl'>
				Unsere Lieferlösung
			</h2>
			<p class='mt-4 max-w-3xl mx-auto text-center text-xl text-gray-500'>
				Fair Pizza arbeitet mit lokalen Velokurieren zusammen und ermöglicht die Lieferung innerhalb der Stadt Zürich.
			</p>
		</div>

		<div class='relative mt-12 lg:mt-24 lg:grid lg:grid-cols-2 lg:gap-8'>
			<div class='relative'>
				<h3 class='text-2xl font-extrabold text-gray-900 tracking-tight sm:text-3xl'>
					Lieferkonditionen
				</h3>
				<p class='mt-3 text-lg text-gray-500'>
					Die Lieferkonditionen hängem vom Standort des Restaurants sowie dem Lieferort des Kunden ab.
					Du hast die Möglichkeiten die Lieferzonen zu konfigurieren, um festzulegen wohin geliefert werden darf und zu
					welchem Mindestbestellpreis.
				</p>

				<dl class='mt-10 space-y-10'>
					<fieldset>
						<div class='bg-white rounded-md -space-y-px'>
							<label
								class="{ zone === 1 ? 'rounded-tl-md rounded-tr-md relative border p-4 flex items-center cursor-pointer focus:outline-none bg-red-50 border-red-200 z-10' : 'rounded-tl-md rounded-tr-md relative border p-4 flex items-center cursor-pointer focus:outline-none border-gray-200'}">
								<input type='radio' name='zone' value={1} checked={zone===1} on:change={onChange}
											 class='h-4 w-4 mt-0.5 cursor-pointer text-red-600 border-gray-300 focus:ring-red-500'
											 aria-labelledby='privacy-setting-0-label' aria-describedby='privacy-setting-0-description'>
								<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="{ zone === 1 ? 'block text-sm font-medium text-red-900' : 'block text-sm font-medium text-gray-900'}">
										Zone 1 für {zipCode}
									</span>
									<span class="{zone === 1 ? 'block text-sm text-red-700' : 'block text-sm text-gray-500'}">
										Beinhaltet:
										{#each zone1 as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>14</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</label>

							{#if zone2.length > 0}
							<label
								class="{ zone === 2 ? 'relative border p-4 flex items-center cursor-pointer focus:outline-none bg-red-50 border-red-200 z-10' : 'relative border p-4 flex items-center cursor-pointer focus:outline-none border-gray-200'}">
								<input type='radio' name='zone' value={2} checked={zone===2} on:change={onChange}
											 class='h-4 w-4 mt-0.5 cursor-pointer text-red-600 border-gray-300 focus:ring-red-500'
											 aria-labelledby='privacy-setting-0-label' aria-describedby='privacy-setting-0-description'>
								<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="{ zone === 2 ? 'block text-sm font-medium text-red-900' : 'block text-sm font-medium text-gray-900'}">
										Zone 2 für {zipCode}
									</span>
									<span class="{zone === 2 ? 'block text-sm text-red-700' : 'block text-sm text-gray-500'}">
										Beinhaltet:
										{#each zone2 as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>18</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</label>
							{/if}

							{#if zone3.length > 0}
							<label
								class="{ zone === 3 ? 'rounded-bl-md rounded-br-md relative items-center border p-4 flex cursor-pointer focus:outline-none bg-red-50 border-red-200 z-10' : 'rounded-bl-md rounded-br-md relative items-center border p-4 flex cursor-pointer focus:outline-none border-gray-200'}">
								<input type='radio' name='zone' value={3} checked={zone===3} on:change={onChange}
											 class='h-4 w-4 mt-0.5 cursor-pointer text-red-600 border-gray-300 focus:ring-red-500'
											 aria-labelledby='privacy-setting-0-label' aria-describedby='privacy-setting-0-description'>
								<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="{ zone === 3 ? 'block text-sm font-medium text-red-900' : 'block text-sm font-medium text-gray-900'}">
										Zone 3 für {zipCode}
									</span>
									<span class="{zone === 3 ? 'block text-sm text-red-700' : 'block text-sm text-gray-500'}">
										Beinhaltet:
										{#each zone3 as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>27</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</label>
							{/if}
						</div>
					</fieldset>
				</dl>

				<div class='mt-6 md:mt-12'>
					<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>Aufteilung</h4>

					<p class='mt-3 text-lg text-gray-500'>
						Die Aufteilung steuert wieviel der Lieferkosten dem Kunden transparent als Lieferkosten zuzüglich dem
						Bestellwert verrechnet wird.
					</p>
					<span class='relative z-0 inline-flex shadow-sm rounded-md mt-2'>
						{#each splits as split, i}
						<button type='button' class:-ml-px={i !== 0} class:rounded-l-md={i === 0}
										class:rounded-r-md={i +1 === splits.length} class:ring-1={split === deliverySplit}
										class:z-20={split === deliverySplit} class:ring-red-500={split === deliverySplit}
										class:border-red-500={split === deliverySplit} class:text-red-500={split === deliverySplit}
										class:text-gray-700={split !== deliverySplit}
										on:click={() => updateSplit(split)}
										class='relative inline-flex items-center px-4 py-2 border border-gray-300 bg-white text-sm font-medium outline-none hover:bg-gray-50 focus:outline-none'>
							{split * 100} %
						</button>
						{/each}
					</span>
				</div>
			</div>

			<div class='mt-10 -mx-4 relative lg:mt-0 flex flex-col items-center' aria-hidden='true'>
				<Map bind:zipCode={zipCode} bind:highlight={selectedZone} on:zipCodeChange={zip => updateZipCode(zip)}></Map>
			</div>
		</div>

		<div class='mt-6 md:mt-12'>
			<div class='flex justify-between'>
				<h3 class='text-2xl font-extrabold text-gray-900 tracking-tight sm:text-3xl'>
					Preistabelle
				</h3>
			</div>

			<div class='flex flex-col mt-6'>
				<div class='-my-2 overflow-x-auto sm:-mx-6 lg:-mx-8'>
					<div class='py-2 align-middle inline-block min-w-full sm:px-6 lg:px-8'>
						<div class='shadow overflow-hidden border-b border-gray-200 sm:rounded-lg'>
							<table class='min-w-full divide-y divide-gray-200'>
								<thead class='bg-gray-50'>
								<tr>
									<th scope='col'
											class='px-6 py-3 w-16 text-xs text-left font-medium text-gray-500 uppercase tracking-wider'>
										Bestellwert
									</th>
									<th scope='col'
											class='px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider'>
										Lieferkosten
									</th>
									<th scope='col'
											class='px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider'>
										Gebühr
									</th>
									<th scope='col'
											class='px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider'>
										Umsatz
									</th>
									<th scope='col'
											class='px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider'>
										Prozent
									</th>
								</tr>
								</thead>
								<tbody>
								{#each pricings as pricing, i}
									<tr class="{i % 2 === 0 ? 'bg-white': 'bg-gray-100'}">
										<td class='px-6 py-4 whitespace-nowrap text-sm text-right font-medium text-gray-900'>
											{ pricing.orderValue } CHF
										</td>
										<td class='px-6 py-4 whitespace-nowrap text-sm text-right text-gray-500'>
											{ pricing.deliveryFee } CHF
										</td>
										<td class='px-6 py-4 whitespace-nowrap text-sm text-right text-gray-500'>
											{ pricing.orderFee } CHF
										</td>
										<td class='px-6 py-4 whitespace-nowrap text-sm text-right text-gray-500'>
											{ pricing.revenue } CHF
										</td>
										<td class='px-6 py-4 whitespace-nowrap text-sm text-right text-gray-500'>
											{ pricing.percentage } %
										</td>
									</tr>
								{/each}
								</tbody>
							</table>
						</div>
						<div class='mt-2 mx-3 text-sm text-gray-600'>* Lieferkosten werden dem Kunden zusätzlich zum Bestellwert
							verrechnet, die Gebühr beinhaltet Zahlungsabwicklung- sowie Transportkosten welche vom Bestellwert
							abgezogen werden
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

<style>
</style>
