<template>
	<div class="container mx-auto p-4 bg-white shadow-lg rounded-lg">
		<h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">Nem tudod hogyan tehetnéd még rosszabbá a napod? <br><span class="underline">Egyszerű.</span> <br>Nézd meg mennyit költöttél el a Wolt-on!</h1>

		<!-- Bearer Token Input és Információs Gomb -->
		<div class="mb-6">
			<label for="bearer-token" class="block text-gray-700 text-sm font-bold mb-2">Bearer Token:</label>
			<div class="flex items-center">
				<input id="bearer-token" type="text" placeholder="Bearer Token"
					class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
					v-model="bearerToken">
				<button @click="toggleInfoModal"
					class="ml-2 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
					Info
				</button>
			</div>
		</div>
		<button @click="fetchData"
			class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" :disabled="fetching">
			Küldés
		</button>

		<!-- Információs Modal -->
		<div v-if="showInfoModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full z-50"
			@click.self="toggleInfoModal">
			<div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
				<div class="mt-3 text-center">
					<h3 class="text-lg leading-6 font-medium text-gray-900">Bearer Token Információ</h3>
					<div class="mt-2">
						<div class="p-4 max-w-xl mx-auto">
							<h2 class="text-lg font-semibold mb-4">Nem tudod mi az a Bearer Token?</h2>
							<p class="mb-4">Ezek szerint nagy valószínűséggel egy iStyle magyarországi dolgozó lehetsz.
								Szervusz Rigli! Az alábbiakban megtudhatod miért van erre szükség:</p>
							<h2 class="text-lg font-semibold mb-4">Mi az a Bearer Token?</h2>
							<p class="mb-4">A Bearer Token egy biztonsági mechanizmus, amelyet az API-k használnak a
								hitelesítés és azonosítás céljából. Ez egy hosszú karakterlánc, amely egyedi azonosítóként
								szolgál, lehetővé téve az alkalmazás számára, hogy biztonságosan hozzáférjen az API által
								biztosított adatokhoz.</p>
							<p class="mb-4">Amikor az alkalmazásunk a Wolt API-jához kapcsolódik, a Bearer Token biztosítja,
								hogy jogosultak vagyunk az adatok elérésére, és ezáltal védi az információkat az
								illetéktelen hozzáféréstől.</p>

							<h2 class="text-lg font-semibold mb-4">Hogyan Találhatod Meg a Bearer Tokent?</h2>
							<ol class="list-decimal list-inside mb-4">
								<li>Látogass el a <a href="https://wolt.com" class="text-blue-600 hover:text-blue-800"
										target="_blank">Wolt.com</a> weboldalra a böngésződben.</li>
								<li>Nyomd meg az F12 billentyűt, hogy megnyisd a fejlesztői eszközöket.</li>
								<li>Kattints a "Hálózat" (Network) fülre, ahol láthatod a weboldal által végrehajtott összes
									lekérést.</li>
								<li>Keress egy olyan lekérést, amely a <code>https://restaurant-api.wolt.com</code> címre
									lett küldve.</li>
								<li>Kattints a kiválasztott lekérésre, majd tekints meg a "Kérés fejlécei" (Request Headers)
									részt.</li>
								<li>Itt található az "Authorization" mező, amely a Bearer Tokent tartalmazza.</li>
								<li>Másold ki a token értékét, de ügyelj arra, hogy a "Bearer" szót hagyd el, csak a token
									maradjon.</li>
							</ol>
							<p class="mb-4"><a href="https://cors-anywhere.herokuapp.com/corsdemo" target="_blank" rel="noopener noreferrer">https://cors-anywhere.herokuapp.com/corsdemo</a></p>
						</div>

					</div>
					<div class="items-center px-4 py-3">
						<button id="ok-btn" @click="toggleInfoModal"
							class="px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md w-full shadow-sm hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-300">
							Ok
						</button>
					</div>
				</div>
			</div>
		</div>

		<!-- Hibaüzenet megjelenítése -->
		<div v-if="error" class="mt-4 text-red-600">
			{{ error }}
		</div>

		<!-- Rendelések Összegzése -->
		<div v-if="bearerToken && ordersFetched" class="mt-10">
			<h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">Wolt Rendeléseim Összegzése</h1>

			<!-- Összesítési időszak megjelenítése -->
			<div class="flex justify-center items-center space-x-2 mb-4">
				<label for="start-date" class="font-medium text-gray-600">Kezdete:</label>
				<input disabled type="date" id="start-date" class="p-2 border border-gray-300 rounded" v-model="formattedStartDate"
					>
				<span class="text-gray-500">és</span>
				<label for="end-date" class="font-medium text-gray-600">Vége:</label>
				<input disabled type="date" id="end-date" class="p-2 border border-gray-300 rounded" v-model="formattedEndDate"
					>
			</div>

			<div class="text-center mb-3">
				<h2 class="font-medium text-lg text-gray-700">Összes Rendelés:</h2>
				<p class="text-xl text-green-600 font-semibold">{{ orders.length }} db</p>
			</div>

			<div class="text-center mb-6">
				<h2 class="font-medium text-lg text-gray-700">Összesen Elköltött Összeg:</h2>
				<p class="text-xl text-green-600 font-semibold">{{ totalAmount.toLocaleString('hu-HU', {
					style: 'currency',
					currency: 'HUF'
				}) }}</p>
			</div>

			<div>
				<h2 class="font-medium text-lg text-gray-700 mb-3">Éttermek:</h2>
				<ul class="space-y-2">
					<li v-for="restaurant in sortedRestaurants" :key="restaurant.name"
						class="bg-gray-100 p-3 rounded shadow">
						<p class="font-semibold text-gray-800">{{ restaurant.name }}</p>
						<p class="text-gray-600">Rendelések száma: <span class="font-semibold">{{ restaurant.orders
						}}</span></p>
						<p class="text-gray-600">Összeg: <span class="font-semibold">{{
							(restaurant.amount / 100).toLocaleString('hu-HU', { style: 'currency', currency: 'HUF' })
						}}</span>
						</p>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios'

const bearerToken = ref('');
const showInfoModal = ref(false);
const ordersFetched = ref(false);
const error = ref(null);
const orders = ref([]);
const totalAmount = ref(0);
const fetching = ref(false);

const startDate = ref('');
const endDate = ref('');

const toggleInfoModal = () => {
	showInfoModal.value = !showInfoModal.value;
};

const formattedStartDate = computed(() => formatDate(startDate.value));
const formattedEndDate = computed(() => formatDate(endDate.value));


const fetchData = async () => {
	fetching.value = true

	ordersFetched.value = false
	error.value = null
	orders.value = []
	totalAmount.value = 0
	startDate.value = ''
	endDate.value = ''

	if (!bearerToken.value) {
		error.value = "Kérlek add meg a Bearer Tokent!";
		fetching.value = false
		return;
	}

	let skip = 0;
	const limit = 100;
	let hasMoreData = true;

	try {
		while (hasMoreData) {
			const response = await axios.get(`https://cors-anywhere.herokuapp.com/https://restaurant-api.wolt.com/v2/order_details/?limit=${limit}&skip=${skip}`, {
				headers: {
					'Authorization': `Bearer ${bearerToken.value}`
				}
			});

			if (response.data.length === 0) {
				hasMoreData = false;
			} else {
				orders.value = orders.value.concat(response.data);
				skip += limit;
			}
		}

		endDate.value = (new Date(orders.value[0]['payment_time']['$date'])).toLocaleDateString()
		startDate.value = new Date(orders.value[orders.value.length - 1]['payment_time']['$date'])

		console.log(endDate.value, startDate.value)

		totalAmount.value = orders.value.reduce((sum, order) => sum + order.payment_amount, 0) / 100;
		ordersFetched.value = true;
		error.value = null;
	} catch (e) {
		error.value = "Hiba történt a lekérés során. Ellenőrizd a Bearer Tokent!";
		ordersFetched.value = false;
	} finally {
		fetching.value = false
	}
};

const sortedRestaurants = computed(() => {
	let restaurants = {};
	orders.value.forEach(order => {
		const restaurantName = order.venue_name;
		if (!restaurants[restaurantName]) {
			restaurants[restaurantName] = { name: restaurantName, orders: 0, amount: 0 };
		}
		restaurants[restaurantName].orders++;
		restaurants[restaurantName].amount += order.payment_amount;
	});
	return Object.values(restaurants).sort((a, b) => b.orders - a.orders);
});

function formatDate(date) {
  const d = new Date(date);
  let month = '' + (d.getMonth() + 1);
  let day = '' + d.getDate();
  const year = d.getFullYear();

  if (month.length < 2) 
      month = '0' + month;
  if (day.length < 2) 
      day = '0' + day;

  return [year, month, day].join('-');
}

</script>