<script lang="ts">
	import logo from './icon-144.png';
</script>

<!--
  This example requires Tailwind CSS v2.0+

  This example requires some changes to your config:

  ```
  // tailwind.config.js
  module.exports = {
    // ...
    plugins: [
      // ...
      require('@tailwindcss/forms'),
    ]
  }
  ```
-->
<div class="relative bg-white overflow-hidden">

	<div class="relative py-3 border-b">
		<div>
			<nav class="relative max-w-7xl mx-auto flex items-center justify-between px-4 sm:px-6" aria-label="Global">
				<div class="flex items-center flex-1">
					<div class="flex items-center justify-between w-full md:w-auto">
						<a href="https://fairpizza.ch">
							<span class="sr-only">Fair Pizza</span>
							<img src={logo} alt="fairpizza" class='h-8 w-auto sm:h-10' />
						</a>
						<div class="-mr-2 flex items-center md:hidden">
							<button type="button" class="bg-white rounded-md p-2 inline-flex items-center justify-center text-gray-400 hover:text-gray-500 hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-inset focus:ring-indigo-500" aria-expanded="false">
								<span class="sr-only">Open main menu</span>
								<!-- Heroicon name: outline/menu -->
								<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
								</svg>
							</button>
						</div>
					</div>
					<div class="hidden md:block md:ml-10 md:space-x-10">

						<a href="#delivery" class="font-medium text-gray-500 hover:text-gray-900">Lieferung</a>

						<a href="#aboutUs" class="font-medium text-gray-500 hover:text-gray-900">Über uns</a>
<!--						<a href="#take-away" class="font-medium text-gray-500 hover:text-gray-900">Take-Away</a>-->

						<a href="#contact" class="font-medium text-gray-500 hover:text-gray-900">Kontakt</a>
					</div>
				</div>
				<div class="hidden md:block text-right">
					<a href="#demo" class="inline-flex items-center px-4 py-2 border border-transparent text-base font-medium rounded-md  text-red-500 hover:text-red-700">
						Live demo
					</a>
				</div>
			</nav>

			<!--
        Mobile menu, show/hide based on menu open state.

        Entering: "duration-150 ease-out"
          From: "opacity-0 scale-95"
          To: "opacity-100 scale-100"
        Leaving: "duration-100 ease-in"
          From: "opacity-100 scale-100"
          To: "opacity-0 scale-95"
      -->
			<div class="absolute top-0 inset-x-0 p-2 transition transform origin-top-right md:hidden">
				<div class="rounded-lg shadow-md bg-white ring-1 ring-black ring-opacity-5 overflow-hidden">
					<div class="px-5 pt-4 flex items-center justify-between">
						<div>
							<img src={logo} alt="fairpizza" class='h-8 w-auto' />
						</div>
						<div class="-mr-2">
							<button type="button" class="bg-white rounded-md p-2 inline-flex items-center justify-center text-gray-400 hover:text-gray-500 hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-inset focus:ring-indigo-500">
								<span class="sr-only">Close main menu</span>
								<!-- Heroicon name: outline/x -->
								<svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
								</svg>
							</button>
						</div>
					</div>
					<div class="px-2 pt-2 pb-3 space-y-1">

						<a href="#delivery" class="block px-3 py-2 rounded-md text-base font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-50">Lieferung</a>

						<a href="#aboutUs" class="block px-3 py-2 rounded-md text-base font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-50">Über uns</a>
<!--						<a href="#take-away" class="block px-3 py-2 rounded-md text-base font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-50">Take-Away</a>-->

						<a href="#contact" class="block px-3 py-2 rounded-md text-base font-medium text-gray-700 hover:text-gray-900 hover:bg-gray-50">Kontakt</a>
					</div>
					<a href="#demo" class="block w-full px-5 py-3 text-center font-medium text-red-500 hover:text-red-700">
						Live demo
					</a>
				</div>
			</div>
		</div>

	</div>
</div>

<style>
</style>
