<script lang='ts'>
	import Map from '../map/Map.svelte';
	import prices from '../../prices.json';

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

	let selectedZone = 1;
	let ordersPerDay = 20;
	let zoneDeliveryPrice = getZonePrice()
	let zipCode = 8005;
	let zone1ZipCodes = [];
	let zone2ZipCodes = [];
	let zone3ZipCodes = [];
	let deliverySplit = 0.5;
	let pricings = [];
	let selectedPricing = 40;
	let availableZipCodes = Object.keys(prices).map(k => parseInt(k))

	let splits = [0, 0.25, 0.5, 0.75, 1];
	let zones = [1,2,3]

	function updatePricing(pricing) {
		selectedPricing = pricing.orderValue
	}

	function updateZipCode(evt) {
		zipCode = evt;
		update();
	}

	function selectZone(zone) {
		selectedZone = zone;
		zoneDeliveryPrice = getZonePrice()
		update();
	}

	function update() {
		pricings = getPricings();
		zone1ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] === 14);
		zone2ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] === 18);
		zone3ZipCodes = Object.keys(combinedPrices[zipCode]).filter(target => combinedPrices[zipCode][target] === 24);
	}

	function getZonePrice() {
		if (selectedZone === 1) {
			return 14;
		} else if (selectedZone === 2) {
			return 18;
		} else if (selectedZone === 3) {
			return 24;
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

	function formatNumber(value) {
		if (!value) {
			return value
		}
		return value.toFixed(2).replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1\'')
	}

	function formatNumberWithoutDecimal(value) {
		if (!value) {
			return value
		}
		return value.toFixed(0).replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1\'')
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

		<div class='grid grid-cols-1 lg:grid-cols-2 gap-x-12 gap-y-12 mt-12'>
			<div>
				<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
					Dein Standort
				</h4>

				<p class='mt-3 text-lg text-gray-500 mb-3'>
					Die Lieferkonditionen hängen vom Standort des Restaurants, sowie dem Lieferort des Kunden ab.
					Wähle deinen Standort und erfahre, welche Zonentarife für dich gelten.
				</p>

				<div class='flex'>

				<span class='relative z-0 inline-flex shadow-sm rounded-md mt-2 mb-4'>
					{#each availableZipCodes as availableZipCode, i}
					<button type='button' class:-ml-px={i !== 0} class:rounded-l-md={i === 0}
									class:rounded-r-md={i +1 === splits.length} class:ring-1={availableZipCode === zipCode}
									class:z-20={availableZipCode === zipCode} class:ring-red-500={availableZipCode === zipCode}
									class:border-red-500={availableZipCode === zipCode} class:text-red-500={availableZipCode === zipCode}
									class:text-gray-700={availableZipCode !== zipCode}
									on:click={() => updateZipCode(availableZipCode)}
									class='relative inline-flex items-center px-4 py-2 border border-gray-300 bg-white text-sm font-medium outline-none hover:bg-gray-50 focus:outline-none'>
						{availableZipCode}
					</button>
					{/each}
				</span>
				</div>

				<Map bind:zipCode={zipCode} bind:zone1={zone1ZipCodes} bind:zone2={zone2ZipCodes} bind:zone3={zone3ZipCodes}></Map>
			</div>
			<div class='relative'>
				<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
					Lieferzonen
				</h4>
				<p class='mt-3 text-lg text-gray-500'>
					Du hast die Möglichkeiten die Lieferzonen zu konfigurieren, um festzulegen, wohin geliefert werden darf und zu
					welchem Mindestbestellpreis.
				</p>

				<dl class='mt-10 space-y-10'>
					<fieldset>
						<div class='bg-white rounded-md -space-y-px'>
							<div
								class="rounded-tl-md rounded-tr-md relative border p-4 flex items-center cursor-pointer focus:outline-none bg-red-700 border-red-700 z-10">
								<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="block text-sm font-medium text-white">
										Zone 1 für {zipCode}
									</span>
									<span class="block text-sm text-white">
										Beinhaltet Lieferungen in an die Postleitzahlen<br/>
										<div class='text-xs'>
										{#each zone1ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
										</div>
									</span>
								</div>
								<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
									<h4>14</h4>
									<span class='text-xs ml-1'>CHF</span>
								</div>
							</div>

							{#if zone2ZipCodes.length > 0}
								<div
									class="relative border p-4 flex items-center cursor-pointer focus:outline-none bg-red-400 border-red-400 z-10">
									<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="block text-sm font-medium text-black">
										Zone 2 für {zipCode}
									</span>
										<span class="block text-sm text-black">
										Beinhaltet Lieferungen in an die Postleitzahlen<br/>
											<div class='text-xs'>
												{#each zone2ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
											</div>

									</span>
									</div>
									<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
										<h4>18</h4>
										<span class='text-xs ml-1'>CHF</span>
									</div>
								</div>
							{/if}

							{#if zone3ZipCodes.length > 0}
								<div
									class="rounded-bl-md rounded-br-md relative items-center border p-4 flex cursor-pointer focus:outline-none bg-red-100 border-red-200 z-10">
									<div class='ml-3 flex flex-col flex-grow'>
									<span
										class="block text-sm font-medium text-red-900">
										Zone 3 für {zipCode}
									</span>
										<span class="block text-sm text-red-700">
										Beinhaltet Lieferungen in an die Postleitzahlen<br/>
											<div class='text-xs'>
											{#each zone3ZipCodes as zip, i}{#if i !== 0}<span>,</span>{/if} {zip}{/each}
											</div>
									</span>
									</div>
									<div class='flex items-center justify-center px-3 py-2 rounded-md bg-black text-white'>
										<h4>24</h4>
										<span class='text-xs ml-1'>CHF</span>
									</div>
								</div>
							{/if}
						</div>
					</fieldset>
				</dl>


			</div>
		</div>


		<div class='mt-6 md:mt-12'>

			<div class='w-1/2 md:w-2/3 text-center mx-auto my-24'>
				<h3 class='text-2xl font-extrabold text-gray-900 tracking-tight sm:text-3xl'>Kostenrechner</h3>
				<p class='mb-6 mt-3 text-lg text-gray-500'>
					Mit unserem Kostenrechner kannst du die Kosten für deine Kunden und dein Restaurant transparent ausrechnen und vergleichen.
				</p>
			</div>

			<div class='grid grid-cols-1 lg:grid-cols-2 gap-x-12 gap-y-12 mt-12'>

				<div>
					<div class='flex justify-between mt-3'>
						<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
							Der Standort deines Kunden
						</h4>
					</div>
					<p class='mt-3 text-lg text-gray-500'>
						Die Zone, in welche die Bestellung geliefert werden soll.
					</p>
					<span class='relative z-0 inline-flex shadow-sm rounded-md mt-2 mb-4'>
						{#each zones as zone, i}
						<button type='button' class:-ml-px={i !== 0} class:rounded-l-md={i === 0}
										class:rounded-r-md={i +1 === zones.length} class:ring-1={zone === selectedZone}
										class:z-20={zone === selectedZone} class:ring-red-500={zone === selectedZone}
										class:border-red-500={zone === selectedZone} class:text-red-500={zone === selectedZone}
										class:text-gray-700={zone !== selectedZone}
										on:click={() => selectZone(zone)}
										class='relative inline-flex items-center px-4 py-2 border border-gray-300 bg-white text-sm font-medium outline-none hover:bg-gray-50 focus:outline-none'>
							Zone {zone}
						</button>
						{/each}
					</span>

					<div class='flex justify-between mt-16'>
						<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
							Bestellwert
						</h4>
					</div>

					<p class='mt-3 text-lg text-gray-500'>
						Der Bestellwert aus dem Warenkorb des Kunden.
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


					<div>
						<h4 class='text-xl mt-16 font-extrabold text-gray-900 tracking-tight sm:text-1xl'>Aufteilung der Lieferkosten</h4>

						<p class='mt-3 text-lg text-gray-500'>
							Die Aufteilung steuert, wieviel dem Kunden als Lieferkosten zuzüglich dem
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


				</div>



				<div class='flex flex-col justify-end'>


					<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl mt-6'>
						Umsatz des Restaurants
					</h4>
					<p class='mt-3 text-lg text-gray-500'>
						Der Umsatz basierend auf Bestellwert abzüglich der Komissionsbeträge.
					</p>



					<fieldset>
						<div class='bg-white rounded-md -space-y-px'>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 bg-gray-100 rounded-t text-gray-900">
								<div class='flex flex-col flex-grow block text-sm font-medium'>

								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									Eat.ch
								</div>
								<div class='flex-shrink-0 w-32 text-right text-sm'>
									Fair Pizza
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Bestellwert
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
									Komission
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-{(0.3 * selectedPricing).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-{((zoneDeliveryPrice * (1 - deliverySplit)) + ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) + 1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Lieferkomission <span class='ml-2 text-gray-400'>({(1-deliverySplit)*100}% von {zoneDeliveryPrice.toFixed(2)} CHF)</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{(zoneDeliveryPrice * (1 - deliverySplit)).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Kreditkartenkomission <span class='ml-2 text-gray-400'>(2.9% + 0.30 CHF)</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
								<div class='flex flex-grow block font-medium text-gray-700 text-xs ml-3'>
									Bestellkomission <span class='ml-2 text-gray-400'>(1.00 CHF pro Bestellung)</span>
								</div>
								<div class='flex-shrink-0 w-32 text-right'>
									-
								</div>
								<div class='flex-shrink-0 w-32 text-right text-xs'>
									-{(1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
							<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 rounded-b">
								<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700 font-bold'>
									Umsatz
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing - selectedPricing*0.3).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
								<div class='flex-shrink-0 font-bold w-32 text-right'>
									{(selectedPricing - (zoneDeliveryPrice * (1 - deliverySplit)) - ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) - 1).toFixed(2)} <span class='text-sm'>CHF</span>
								</div>
							</div>
						</div>
					</fieldset>


					<div class='mt-12'>
						<h4 class='text-xl font-extrabold text-gray-900 tracking-tight sm:text-1xl'>
							Kosten für den Kunden
						</h4>
						<p class='mt-3 text-lg text-gray-500'>
							Die Kosten, welche dem Konsumenten beim Bestellen aufgelistet und verrechnet werden.
						</p>


						<fieldset>
							<div class='bg-white rounded-md -space-y-px'>
								<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 bg-gray-100 rounded-t text-gray-900">
									<div class='flex flex-col flex-grow block text-sm font-medium'>

									</div>
									<div class='flex-shrink-0 w-32 text-right text-sm'>
										Eat.ch
									</div>
									<div class='flex-shrink-0 w-32 text-right text-sm'>
										Fair Pizza
									</div>
								</div>
								<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
									<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
										Bestellwert
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{selectedPricing.toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
								</div>
								<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
									<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
										Lieferkosten
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{(0).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{(zoneDeliveryPrice * deliverySplit).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
								</div>
								<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200">
									<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700'>
										MWST.
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{(selectedPricing * 0.024).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
									<div class='flex-shrink-0 w-32 text-right'>
										{((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.024).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
								</div>
								<div class="relative border px-4 py-2 mt-2 flex items-center cursor-pointer focus:outline-none border-gray-200 rounded-b">
									<div class='flex flex-col flex-grow block text-sm font-medium text-gray-700 font-bold'>
										Total
									</div>
									<div class='flex-shrink-0 font-bold w-32 text-right'>
										{(selectedPricing + selectedPricing * 0.024).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
									<div class='flex-shrink-0 font-bold w-32 text-right'>
										{(selectedPricing + selectedPricing * 0.024 + zoneDeliveryPrice * deliverySplit).toFixed(2)} <span class='text-sm'>CHF</span>
									</div>
								</div>
							</div>
						</fieldset>
					</div>
				</div>
			</div>

			<div class="bg-gray-50 pt-12 sm:pt-16">
				<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
					<div class="max-w-4xl mx-auto text-center">
						<h2 class="text-3xl font-extrabold text-gray-900 sm:text-4xl">
							Steigere deinen Umsatz
						</h2>
						<p class="mt-3 text-xl text-gray-500 sm:mt-4">
							Im Vergleich zu Eat.ch ist bei einer Anzahl von
							<input type='number' bind:value={ordersPerDay} class='w-16 shadow-sm mx-2 focus:ring-red-500 focus:border-red-500 sm:text-sm border-gray-300 rounded-md'> Bestellungen pro Tag,<br/> dein Umsatz um folgende Betrag höher.
						</p>
					</div>
				</div>
				<div class="mt-10 pb-12 sm:pb-16">
					<div class="relative">
						<div class="absolute inset-0 h-1/2 bg-gray-50"></div>
						<div class="relative max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
							<div class="max-w-4xl mx-auto">
								<dl class="rounded-lg bg-white shadow-lg sm:grid sm:grid-cols-3">
									<div class="flex flex-col border-b border-gray-100 p-6 text-center sm:border-0 sm:border-r">
										<dt class="order-2 mt-2 text-lg leading-6 font-medium text-gray-500">
											CHF / pro Bestellung
										</dt>
										<dd class="order-1 text-5xl font-extrabold text-green-600">
											+ {formatNumber((selectedPricing - (zoneDeliveryPrice * (1 - deliverySplit)) - ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) - 1 - (selectedPricing - selectedPricing*0.3)))}
										</dd>
									</div>
									<div class="flex flex-col border-t border-b border-gray-100 p-6 text-center sm:border-0 sm:border-l sm:border-r">
										<dt class="order-2 mt-2 text-lg leading-6 font-medium text-gray-500">
											CHF / pro Monat
										</dt>
										<dd class="order-1 text-5xl font-extrabold text-green-600">
											+ {formatNumberWithoutDecimal(ordersPerDay * 30 * (selectedPricing - (zoneDeliveryPrice * (1 - deliverySplit)) - ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) - 1 - (selectedPricing - selectedPricing*0.3)))}
										</dd>
									</div>
									<div class="flex flex-col border-t border-gray-100 p-6 text-center sm:border-0 sm:border-l">
										<dt class="order-2 mt-2 text-lg leading-6 font-medium text-gray-500">
											CHF / pro Jahr
										</dt>
										<dd class="order-1 text-5xl font-extrabold text-green-600">
											+ {formatNumberWithoutDecimal(ordersPerDay * 30 * 12 * (selectedPricing - (zoneDeliveryPrice * (1 - deliverySplit)) - ((selectedPricing + zoneDeliveryPrice * deliverySplit) * 0.029 + 0.3) - 1 - (selectedPricing - selectedPricing*0.3)))}
										</dd>
									</div>
								</dl>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

<style>
</style>
