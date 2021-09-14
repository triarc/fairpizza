<script context="module" lang="ts">
	export const prerender = true;
</script>

<script lang="ts">
	import Delivery from '$lib/delivery/Delivery.svelte';
	import AboutUs from '../lib/about-us/AboutUs.svelte';
	import Contact from '../lib/Contact.svelte';
	import { onMount } from 'svelte';

	onMount(async () => {
		const Plyr = await import('plyr')
		const player = new Plyr.default('#demo', {
			controls: ['play-large'],
			autoplay: true,
			loop: { active: true},
		})
	})
</script>

<svelte:head>
	<title>Fair Pizza</title>
</svelte:head>

<main class="my-16 mx-auto max-w-7xl px-4 sm:mt-24 sm:px-6 lg:mt-32">
	<div class="lg:grid lg:grid-cols-12 lg:gap-8">
		<div class="sm:text-center md:max-w-2xl md:mx-auto lg:col-span-6 lg:text-left">
			<h1>
						<span class="mt-1 block text-4xl tracking-tight font-extrabold sm:text-5xl xl:text-6xl">
              <span class="block text-gray-900">Die Onlinelösung</span>
              <span class="block text-red-500">für Pizza-Bestellung</span>
            </span>
			</h1>
			<p class="mt-3 text-base text-gray-500 sm:mt-5 sm:text-xl lg:text-lg xl:text-xl">
				Fair Pizza ermöglicht die eigenständige Onlinepräsenz.
				Erweitere deinen Kundenkreis und baue dir ein zweites Standbein auf mit Onlinebestellung oder Take-Away.
			</p>
			<div class="pt-6 pb-8">
				<dl class="mt-6 grid grid-cols-1 md:grid-cols-2 gap-x-4 gap-y-6">
					<div class="relative">
						<dt>
							<p class="text-lg leading-6 font-medium text-gray-900">Nutze deine Kurierlösung</p>
						</dt>
						<dd class="mt-2 text-base text-gray-500">
							Verwende FairPizza mit deinem vorhandenen Lieferpartner oder übernimm die Lieferung selbst.
						</dd>
					</div>

					<div class="relative">
						<dt>
							<p class="text-lg leading-6 font-medium text-gray-900">Nutze unseren Velokurier</p>
						</dt>
						<dd class="mt-2 text-base text-gray-500">
							Nutze unseren Velokurier und konzentriere dich auf dein Restaurant und die Bestellung.
						</dd>
					</div>

					<div class="relative">
						<dt>
							<p class="text-lg leading-6 font-medium text-gray-900">Onlinebezahlung</p>
						</dt>
						<dd class="mt-2 text-base text-gray-500">
							Vereinfacht den Prozess und stellt sicher, dass im Vorraus bezahlt ist.
						</dd>
					</div>

					<div class="relative">
						<dt>
							<p class="text-lg leading-6 font-medium text-gray-900">Bleibe unabhängig</p>
						</dt>
						<dd class="mt-2 text-base text-gray-500">
							Behalte die Kontrolle über deine Onlinepräsenz und bleibe uneingeschränkt.
						</dd>
					</div>
				</dl>
			</div>
			<div class="mt-10 sm:flex sm:justify-center lg:justify-start">
				<div class="rounded-md shadow">
					<a href="mailto:dh@triarc-labs.com" class="w-full flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-md text-white bg-red-600 hover:bg-indigo-700 md:py-4 md:text-lg md:px-10">
						Kontaktiere uns
					</a>
				</div>
				<div class="mt-3 rounded-md shadow sm:mt-0 sm:ml-3">
					<a href="https://demo.fairpizza.ch" target='_blank' class="w-full flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-md text-red-600 bg-white hover:bg-gray-50 md:py-4 md:text-lg md:px-10">
						Live demo
					</a>
				</div>
			</div>
		</div>
		<div class="mt-12 relative sm:max-w-lg sm:mx-auto lg:mt-0 lg:max-w-none lg:mx-0 lg:col-span-6 lg:flex lg:items-center">
			<div class="relative mx-auto w-full rounded-lg lg:max-w-md flex flex-col">
				<div class='bg-white rounded w-100 h-12 border-gray-700 border text-gray-600 flex items-center justify-center'><span class='text-red-500'>deinname</span>.fairpizza.ch</div>
				<video id='demo' playsinline class="object-cover z-10 flex-grow" style='border-bottom-left-radius: 32px; border-bottom-right-radius: 32px' autoplay loop>
					<source src='https://storage.googleapis.com/triarc-website/fairpizza.mp4' type='video/mp4' >
				</video>
			</div>
		</div>
	</div>
</main>

<Delivery></Delivery>
<!--<TakeAway></TakeAway>-->
<AboutUs></AboutUs>
<Contact></Contact>

<style style lang="postcss">
    @import 'plyr/dist/plyr.css';
</style>
