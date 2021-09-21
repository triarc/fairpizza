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
	let zoneDeliveryPrice = getZonePrice()
	let zipCode = 8005;
	let zone1ZipCodes = [];
	let zone2ZipCodes = [];
	let zone3ZipCodes = [];
	let zoneZipCodes = [];
	let deliverySplit = 0.5;
	let pricings = [];
	let selectedPricing = 40;

	let splits = [0, 0.25, 0.5, 0.75, 1];

	function updatePricing(pricing) {
		selectedPricing = pricing.orderValue
	}

	function updateZipCode(evt) {
		zipCode = parseInt(evt.detail);
		update();
	}

	function onChange(event) {
		zone = parseInt(event.currentTarget.value);
		zoneDeliveryPrice = getZonePrice()
		update();
	}

	function update() {
		pricings = getPricings();
		zone1ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => target === zipCode.toString() || combinedPrices[zipCode][target] < 18);
		zone2ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] < 26 && combinedPrices[zipCode][target] >= 18 && target !== zipCode.toString());
		zone3ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] >= 26);
		zoneZipCodes = zone === 1 ? zone1ZipCodes : zone === 2 ? zone2ZipCodes : zone3ZipCodes;
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
		return [32, 40, 50, 60, 70, 80, 90, 100, 150, 200].map(orderValue => {
			const orderFee = (orderValue + (zoneDeliveryPrice * deliverySplit)) * 0.029 + 1.3 + (zoneDeliveryPrice * (1 - deliverySplit));
			const revenue = orderValue - orderFee;
			const eatFee = 0.3 * orderValue;
			return {
				orderValue,
				orderFee: orderFee.toFixed(2),
				deliveryFee: (zoneDeliveryPrice * deliverySplit).toFixed(2),
				eatFee: eatFee.toFixed(2),
				vsEatFee: (orderValue * 0.3 - orderFee).toFixed(2),
				eatRevenue: (orderValue - eatFee).toFixed(2),
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
										{#each zone1ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>14</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</label>

							{#if zone2ZipCodes.length > 0}
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
										{#each zone2ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>18</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</label>
							{/if}

							{#if zone3ZipCodes.length > 0}
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
										{#each zone3ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
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
			</div>

			<div class='mt-10 -mx-4 relative lg:mt-0 flex flex-col items-center' aria-hidden='true'>
				<Map bind:zipCode={zipCode} bind:highlight={zoneZipCodes} on:zipCodeChange={zip => updateZipCode(zip)}></Map>
			</div>
		</div>

		<div class='mt-6 md:mt-12'>
			<h3 class='text-2xl font-extrabold text-gray-900 tracking-tight sm:text-3xl'>Kostenrechner</h3>
			<p class='mb-6 mt-3 text-lg text-gray-500'>
				Mit unserem Kostenrechner kannst du die Kosten für deine Konsumenten und dein Restaurant transparent ausrechnen und vergleichen.
			</p>

			<div class='grid grid-cols-1 lg:grid-cols-2 gap-x-12 gap-y-12'>
				<div class=''>
					<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>Aufteilung der Lieferkosten</h4>

					<p class='mt-3 text-lg text-gray-500'>
						Die Aufteilung steuert wieviel der Lieferkosten dem Kunden transparent als Lieferkosten zuzüglich dem
						Bestellwert verrechnet wird.
					</p>

					<div class='flex justify-between'>
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
				<div>
					<div class='flex justify-between'>
						<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
							Bestellwert
						</h4>
					</div>

					<p class='mt-3 text-lg text-gray-500'>
						Der Bestellwert aus dem Warenkorb des Konsumentens.
					</p>
					<div class='relative z-0 grid shadow-sm rounded-md mt-2' style='grid-template-columns: {`repeat(${pricings.length/2}, minmax(0, 1fr))`}'>
						{#each pricings as pricing, i}
						<button type='button' class:-ml-px={i !== 0} class:rounded-tl-md={i === 0} class:rounded-tr-md={i === pricings.length/2-1}
										class:rounded-bl-md={i === pricings.length/2} class:rounded-br-md={i+1 === pricings.length}
										class:ring-1={pricing.orderValue === selectedPricing}
										class:z-20={pricing.orderValue === selectedPricing} class:ring-red-500={pricing.orderValue === selectedPricing}
										class:border-red-500={pricing.orderValue === selectedPricing} class:text-red-500={pricing.orderValue === selectedPricing}
										class:text-gray-700={pricing.orderValue !== selectedPricing}
										on:click={() => updatePricing(pricing)}
										class='items-center px-4 py-2 border border-gray-300 bg-white text-sm font-medium outline-none hover:bg-gray-50 focus:outline-none'>
							{pricing.orderValue} CHF
						</button>
						{/each}
					</div>
				</div>

				<div>
					<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
						Kosten für den Konsumenten
					</h4>
					<p class='mt-3 text-lg text-gray-500'>
						Die Kosten, welche dem Konsumenten beim Bestellen aufgelistet und verrechnet werden.
					</p>


					<fieldset>
						<div class='bg-white rounded-md -space-y-px'>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 bg-gray-100 rounded-t text-gray-900">
								<div class='flex flex-col flex-grow block text-sm font-medium'>

								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									Eat.ch
								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									FairPizza
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Bestellwert
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Lieferkosten
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{(0).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{(zoneDeliveryPrice * deliverySplit).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									MWST.
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{(selectedPricing * 0.024).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{(zoneDeliveryPrice * deliverySplit).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 rounded-b">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700 font-bold'>
									Total
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing + selectedPricing * 0.024).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing + selectedPricing * 0.024 + zoneDeliveryPrice * deliverySplit).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
						</div>
					</fieldset>
				</div>
				<div>
					<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
						Umsatz des Restaurants
					</h4>
					<p class='mt-3 text-lg text-gray-500'>
						Der Umsatz basierend aus Bestellwert abzüglich der Komissionsbeträge.
					</p>



					<fieldset>
						<div class='bg-white rounded-md -space-y-px'>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 bg-gray-100 rounded-t text-gray-900">
								<div class='flex flex-col flex-grow block text-sm font-medium'>

								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									Eat.ch
								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									FairPizza
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Bestellwert
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Komission
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-{(0.3 * selectedPricing).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-{((zoneDeliveryPrice * (1 - deliverySplit)) + ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) + 1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Lieferkomission
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{(zoneDeliveryPrice * (1 - deliverySplit)).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Kreditkartenkomission
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Bestellkomission
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{(1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
							<label class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 rounded-b">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700 font-bold'>
									Umsatz
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing - selectedPricing*0.3).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing - (zoneDeliveryPrice * (1 - deliverySplit)) - ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) - 1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</label>
						</div>
					</fieldset>
				</div>
			</div>

		</div>
	</div>
</div>

<style>
</style>
