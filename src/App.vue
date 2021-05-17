<template>
	<div id="main" :class="isDay ? 'day' : 'night'">
		<div class="container my-5">
			<h2 class="title text-center">Weather In</h2>
			<!-- choose unit-->
			<select class="unit-select" v-model="unit" @change="getWeather">
				<option disabled value="" class="font-black">Select Unit</option>
				<option class="font-black">metric</option>
				<option class="font-black">imperial</option>
			</select>
			<form class="search-location" v-on:submit.prevent="getWeather">
				<input 
					type="text"
					class="form-control text-muted form-rounded p-4 shadow-sm"
					placeholder="City Name"
					v-model="citySearch"
					autocomplete="off"
				>
			</form>
			<p class="text-center my-3" v-if="cityFound">No City Found</p>
			<div
				class="card my-3 shadow-lg back-card overflow-hidden" 
				v-if="visible"
				style="height: 45rem;"
			>
				<!-- weather animations -->
				<div>
					<div icon="sunny" v-if="clearSky && isDay">
						<span class="sun"></span>
					</div>

					<div icon="snowy" v-if="snowy && isDay">
						<ul>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
						</ul>
					</div>

					<div icon="stormy" v-if="(stormy || rainy) && isDay">
						<span class="cloud"></span>
						<ul>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
							<li></li>
						</ul>
					</div>

					<div icon="cloudy" v-if="cloudy && isDay">
						<span class="cloud"></span>
						<span class="cloud"></span>
					</div>

					<div icon="nightmoon" v-if="!isDay">
						<span class="moon"></span>
						<span class="meteor"></span>
					</div>
				</div>

				<!-- card top -->
				<div class="card-header text-center">
					<p class="card-text">{{ weather.cityName }}</p>
					<p class="card-small">{{ weather.country }}</p>
				</div>
				<!-- card body -->
				<div class="card-body">
					<div class="card-mid">
						<div class="row">
							<div class="col-12 text-center temp">
								<span>{{ weather.temperature }}{{decodeUnit(unit)}}</span>
								<p class="my-4">{{ weather.description }}</p>
								<img :src="require('./assets/'+ logoUrl)" class="icon-logo text-center" />
							</div>
						</div>
						<div class="row">
							<!-- flex layout -->
							<div class="col box">
								<div>
									<span>HIGH TEMP:</span>
									<p>{{ weather.high }}{{decodeUnit(unit)}}</p>
								</div>
								<div>
									<span>LOW TEMP:</span>
									<p>{{ weather.low }}{{decodeUnit(unit)}}</p>
								</div>
							</div>
						</div>
					</div>
				</div>
				<!-- card footer -->
				<div class="card-footer">

					<div class="col box">
						<div>
							<span>FEELS LIKE:</span>
							<p>{{ weather.feels_like }}{{decodeUnit(unit)}}</p>
						</div>
						<div>
							<span>HUMIDITY:</span>
							<p>{{ weather.humidity }}%</p>
						</div>
					</div>
					<div class="col box">
						<div>
							<span>SUNRISE:</span>
							<p>{{changeTimeStamp(weather.sunrise)}}</p>
						</div>
						<div>
							<span>SUNSET:</span>
							<p>{{changeTimeStamp(weather.sunset)}}</p>
						</div>
					</div>
				</div>
			</div>
		</div>
		
	</div>
</template>

<!-- reactive data -->
<script>
export default {
	data() {
		return{
			cityFound: false,
			visible: false,
			stormy: false,
			cloudy: false,
			clearSky: false,
			snowy: false,
			rainy: false,

			isDay: true,
			citySearch: "",
			lastCity: "",
			unit: "metric",
			logoUrl: 'day.svg',
			weather: {
				cityName: "New York",
				country: "US",
				temperature: 12,
				description: "cloudy",
				high: 19,
				low: 8,
				feels_like: 15,
				humidity: 55,
				sunrise: "08:10",
				sunset: "19:00",
			},
		};
	},

	methods: {
		decodeUnit: function(unit) {
			var txt = document.createElement("textarea");
			if(unit === "metric"){
				txt.innerHTML = "&deg;C";
			}
			else{
				txt.innerHTML = "&deg;F";
			}
			return txt.value;
		},

		changeTimeStamp: function(unixStamp) {
			var date_ob = new Date(unixStamp * 1000);
			var hours = ("0" + date_ob.getHours()).slice(-2);
			var minutes = ("0" + date_ob.getMinutes()).slice(-2);
			const res = hours + ":" + minutes;
			return res;
		},	

		getWeather: async function(){
			console.log(this.unit, this.citySearch);
			const APIkey = "cdc7b7162377fb92b4347c46219324d8";
			const url = `http://api.openweathermap.org/data/2.5/weather?q=${this.citySearch}&appid=${APIkey}&units=${this.unit}`;

			//fetch data
			try{
				const response = await fetch(url);
				const data = await response.json();
				console.log(data);
				this.citySearch = data.name;
				this.weather.cityName = data.name;
				this.weather.country = data.sys.country;
				this.weather.temperature = Math.round(data.main.temp);
				this.weather.description = data.weather[0].description;
				this.weather.high = Math.round(data.main.temp_max);
				this.weather.low = Math.round(data.main.temp_min);
				this.weather.feels_like = Math.round(data.main.feels_like);
				this.weather.humidity = Math.round(data.main.humidity);
				this.weather.sunrise = data.sys.sunrise;
				this.weather.sunset = data.sys.sunset;

				const time = data.weather[0].icon; //weather -> array[1]
				this.isDay = time.includes("n") ? false : true;

				const mainWeather = data.weather[0].main;
				//icons
				if(mainWeather.includes("Clouds")){
					this.clearSky = false;
					this.cloudy = true; //02d 02n
					this.stormy = false;
					this.snowy = false;
					this.rainy = false;
				}

				if(mainWeather.includes("Rain")){
					this.clearSky = false;
					this.cloudy = false; 
					this.stormy = false;
					this.snowy = false;
					this.rainy = true; //10d 10n
				}

				if(mainWeather.includes("Thunderstorm")){
					this.clearSky = false;
					this.cloudy = false; 
					this.stormy = true; //11d 11n
					this.snowy = false;
					this.rainy = false;
				}

				if(mainWeather.includes("Clear")){
					this.clearSky = true; //01d 01n
					this.cloudy = false; 
					this.stormy = false; 
					this.snowy = false;
					this.rainy = false;
				}


				if(mainWeather.includes("Snow")){
					this.clearSky = false; 
					this.cloudy = false; 
					this.stormy = false; 
					this.snowy = true; //13d 13n
					this.rainy = false;
				}


				if(this.clearSky && !this.isDay){
					this.logoUrl = 'night.svg';
				}

				if(this.cloudy){
					this.logoUrl = this.isDay ? 'cloudy-day-2.svg' : 'cloudy-night-2.svg';
				}

				if(this.rainy){
					this.logoUrl = this.isDay ? 'rainy-3.svg' : 'rainy-6.svg';
				}

				if(this.stormy){
					this.logoUrl = 'thunder.svg';
				}

				if(this.snowy){
					this.logoUrl = this.isDay ? 'snowy-3.svg' : 'snowy-5.svg';
				}

				this.visible = true;
				this.cityFound = false;

			} catch(error){
				console.log(error);
				this.cityFound = true;
				this.visible = false;
			}
		},


	},
};
</script>

<style scoped>
	@import "./assets/overview.css"; /*import overview css*/
	@import "./assets/animations.css"; /*import animations css*/
</style>