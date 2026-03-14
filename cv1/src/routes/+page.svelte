<script lang="ts">
	type Gender = 'male' | 'female';

	type BMIResult = {
		value: string;
		category: string;
		guidance: string;
		profile: string;
	};

	let username = '';
	let age: number | null = null;
	let gender: Gender | null = null;
	let height = 185;
	let weight = 45;
	let error = '';
	let bmiResult: BMIResult | null = null;

	function calculateBMI() {
		if (!username) {
			error = 'Enter your username before calculating BMI.';
			bmiResult = null;
			return;
		}

		if (age === null || !Number.isFinite(age) || age < 1) {
			error = 'Enter a valid age greater than zero.';
			bmiResult = null;
			return;
		}

		if (gender === null) {
			error = 'Select a gender to complete the profile.';
			bmiResult = null;
			return;
		}

		const bmi = weight / ((height / 100) * (height / 100));
		let category = '';
		let guidance = '';

		if (bmi < 18.5) {
			category = 'Underweight';
			guidance = 'A few more nutrient-dense calories could help move you into a healthier range.';
		} else if (bmi < 25) {
			category = 'Healthy range';
			guidance = 'Your balance looks strong. Keep the same routine of sleep, movement, and meals.';
		} else if (bmi < 30) {
			category = 'Overweight';
			guidance =
				'Small changes in activity and food quality can steadily bring your BMI back down.';
		} else {
			category = 'Obese range';
			guidance = 'A doctor or nutrition coach can help you build a safe plan for gradual progress.';
		}

		error = '';
		bmiResult = {
			value: bmi.toFixed(1),
			category,
			guidance,
			profile: `${username}, ${age} years, ${gender[0].toUpperCase()}${gender.slice(1)}`
		};
	}
</script>

<ion-header>
	<ion-toolbar>
		<ion-title>BMI Calculator</ion-title>
	</ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
	<ion-input
		label="Username"
		placeholder="Enter username"
		value={username}
		on:ionInput={(event) => (username = (event.detail.value ?? '').trim())}
	></ion-input>

	<ion-input
		inputmode="numeric"
		label="Age"
		min="1"
		placeholder="Enter age"
		type="number"
		step="1"
		value={age}
		on:ionInput={(event) => {
			const nextValue = (event.detail.value ?? '').trim();
			const nextAge = Number(nextValue);

			age = nextValue === '' || !Number.isFinite(nextAge) ? null : nextAge;
		}}
	></ion-input>

	<ion-radio-group
		value={gender}
		on:ionChange={(event) => {
			const nextValue = event.detail.value;

			gender = nextValue === 'male' || nextValue === 'female' ? nextValue : null;
		}}
	>
		<ion-item>
			<ion-radio slot="start" value="male"></ion-radio>
			<ion-label>Male</ion-label>
		</ion-item>

		<ion-item>
			<ion-radio slot="start" value="female"></ion-radio>
			<ion-label>Female</ion-label>
		</ion-item>
	</ion-radio-group>

	<ion-range
		label={`Height: ${height} cm`}
		max={220}
		min={120}
		pin={true}
		step={1}
		value={height}
		on:ionInput={(event) => {
			if (typeof event.detail.value === 'number') {
				height = event.detail.value;
			}
		}}
	></ion-range>

	<ion-range
		class="ion-margin-top"
		label={`Weight: ${weight} kg`}
		max={180}
		min={30}
		pin={true}
		step={1}
		value={weight}
		on:ionInput={(event) => {
			if (typeof event.detail.value === 'number') {
				weight = event.detail.value;
			}
		}}
	></ion-range>

	{#if error}
		<ion-text color="danger">
			<p>{error}</p>
		</ion-text>
	{/if}

	<ion-button
		class="ion-margin-top"
		expand="block"
		on:click={calculateBMI}
		on:keydown={(event) => {
			if (event.key === 'Enter' || event.key === ' ') {
				event.preventDefault();
				calculateBMI();
			}
		}}
		role="button"
		tabindex="0">Calculate BMI</ion-button
	>

	{#if bmiResult}
		<ion-card class="ion-margin-top">
			<ion-card-header>
				<ion-card-title>Result</ion-card-title>
				<ion-card-subtitle>{bmiResult.profile}</ion-card-subtitle>
			</ion-card-header>

			<ion-card-content>
				<ion-row>
					{bmiResult.value}
				</ion-row>
				<ion-row>
					{bmiResult.category}
				</ion-row>
				<ion-row>
					{bmiResult.guidance}
				</ion-row>
			</ion-card-content>
		</ion-card>
	{/if}
</ion-content>
