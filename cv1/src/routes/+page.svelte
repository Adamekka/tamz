<script lang="ts">
	import type {
		InputChangeEventDetail,
		InputInputEventDetail,
		SelectChangeEventDetail
	} from '@ionic/core';
	import { onDestroy } from 'svelte';

	type AmountSource = 'czk' | 'foreign';

	type CnbRecord = {
		country_label: string;
		curr_label: string;
		unit: string;
		code: string;
		rate: string;
	};

	type CnbResponse = {
		date: string;
		data: CnbRecord[];
	};

	type ExchangeRate = {
		countryLabel: string;
		currencyLabel: string;
		unit: number;
		code: string;
		rate: number;
	};

	const API_URL = 'http://linedu.vsb.cz/~mor03/TAMZ/cnb_json.php';
	const amountFormatter = new Intl.NumberFormat('en-US', {
		maximumFractionDigits: 6,
		useGrouping: false
	});
	const now = new Date();
	const todayDate = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;

	let selectedDate = todayDate;
	let rates: ExchangeRate[] = [];
	let selectedCurrencyCode = '';
	let isLoading = false;
	let errorMessage = '';
	let czkAmount = '1000';
	let foreignAmount = '';
	let lastEdited: AmountSource = 'czk';
	let activeController: AbortController | null = null;

	let selectedRate: ExchangeRate | null;
	let ratePerForeignUnit: number | null;

	$: selectedRate = rates.find((entry) => entry.code === selectedCurrencyCode) ?? null;
	$: ratePerForeignUnit = selectedRate ? selectedRate.rate / selectedRate.unit : null;
	$: if (selectedDate) {
		void loadRates(selectedDate);
	}

	onDestroy(() => {
		activeController?.abort();
	});

	function syncAmounts(): void {
		const activeRate = rates.find((entry) => entry.code === selectedCurrencyCode) ?? null;
		const currentRatePerForeignUnit = activeRate ? activeRate.rate / activeRate.unit : null;

		if (!currentRatePerForeignUnit) {
			if (lastEdited === 'czk') {
				foreignAmount = '';
			} else {
				czkAmount = '';
			}

			return;
		}

		if (lastEdited === 'czk') {
			const normalizedAmount = czkAmount.replace(',', '.').trim();
			const amount = normalizedAmount ? Number(normalizedAmount) : Number.NaN;

			foreignAmount = Number.isFinite(amount)
				? amountFormatter.format(amount / currentRatePerForeignUnit)
				: '';
			return;
		}

		const normalizedAmount = foreignAmount.replace(',', '.').trim();
		const amount = normalizedAmount ? Number(normalizedAmount) : Number.NaN;

		czkAmount = Number.isFinite(amount)
			? amountFormatter.format(amount * currentRatePerForeignUnit)
			: '';
	}

	async function loadRates(dateValue: string): Promise<void> {
		activeController?.abort();
		const controller = new AbortController();
		activeController = controller;
		isLoading = true;
		errorMessage = '';

		try {
			const url = new URL(API_URL);
			url.searchParams.set('date', dateValue);
			url.searchParams.set('lang', 'en');

			const response = await fetch(url.toString(), { signal: controller.signal });

			if (!response.ok) {
				throw new Error('Could not load exchange rates right now.');
			}

			const payload = (await response.json()) as CnbResponse;
			const nextRates = payload.data
				.map((record) => {
					const unit = Number(record.unit);
					const rate = Number(record.rate);

					if (!Number.isFinite(unit) || unit <= 0 || !Number.isFinite(rate) || rate <= 0) {
						return null;
					}

					return {
						countryLabel: record.country_label,
						currencyLabel: record.curr_label,
						unit,
						code: record.code,
						rate
					};
				})
				.filter((entry): entry is ExchangeRate => entry !== null);

			if (nextRates.length === 0) {
				throw new Error('No exchange rates are available for the selected settings.');
			}

			rates = nextRates;
			selectedCurrencyCode = nextRates.some((entry) => entry.code === selectedCurrencyCode)
				? selectedCurrencyCode
				: nextRates[0].code;
			syncAmounts();
		} catch (error) {
			if (error instanceof DOMException && error.name === 'AbortError') {
				return;
			}

			rates = [];
			selectedCurrencyCode = '';
			if (lastEdited === 'czk') {
				foreignAmount = '';
			} else {
				czkAmount = '';
			}
			errorMessage =
				error instanceof Error && error.message
					? error.message
					: 'Could not load exchange rates right now.';
		} finally {
			if (activeController === controller) {
				isLoading = false;
				activeController = null;
			}
		}
	}
</script>

<ion-header>
	<ion-toolbar>
		<ion-title>Currency Converter</ion-title>
	</ion-toolbar>
</ion-header>

<ion-content class="converter-page" fullscreen>
	<ion-card>
		<ion-card-content>
			<ion-item>
				<ion-input
					label="Date"
					type="date"
					max={todayDate}
					value={selectedDate}
					on:ionChange={(event: CustomEvent<InputChangeEventDetail>) => {
						const nextDate =
							event.detail.value === null || event.detail.value === undefined
								? ''
								: String(event.detail.value);

						if (nextDate) {
							selectedDate = nextDate;
						}
					}}
				></ion-input>
			</ion-item>

			<ion-item>
				<ion-select
					label="Pick currency"
					value={selectedCurrencyCode}
					disabled={rates.length === 0}
					on:ionChange={(event: CustomEvent<SelectChangeEventDetail<string>>) => {
						selectedCurrencyCode = event.detail.value ?? '';
						syncAmounts();
					}}
				>
					{#each rates as rate (rate.code)}
						<ion-select-option value={rate.code}>
							{rate.countryLabel} - {rate.currencyLabel} ({rate.code})
						</ion-select-option>
					{/each}
				</ion-select>
			</ion-item>

			<ion-item>
				<ion-input
					label="CZK"
					inputmode="decimal"
					placeholder="Enter amount in CZK"
					value={czkAmount}
					on:ionInput={(event: CustomEvent<InputInputEventDetail>) => {
						lastEdited = 'czk';
						czkAmount =
							event.detail.value === null || event.detail.value === undefined
								? ''
								: String(event.detail.value);
						syncAmounts();
					}}
				></ion-input>
			</ion-item>

			<ion-item>
				<ion-input
					label={selectedRate?.code ?? 'CUR'}
					inputmode="decimal"
					placeholder="Enter amount in foreign currency"
					value={foreignAmount}
					disabled={!selectedRate}
					on:ionInput={(event: CustomEvent<InputInputEventDetail>) => {
						lastEdited = 'foreign';
						foreignAmount =
							event.detail.value === null || event.detail.value === undefined
								? ''
								: String(event.detail.value);
						syncAmounts();
					}}
				></ion-input>
			</ion-item>
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-header>
			<ion-card-title>Exchange rate</ion-card-title>
			{#if selectedRate && ratePerForeignUnit}
				<ion-card-subtitle
					>{selectedRate.countryLabel} - {selectedRate.currencyLabel}</ion-card-subtitle
				>
			{/if}
		</ion-card-header>
		<ion-card-content>
			{#if isLoading}
				<ion-spinner name="crescent"></ion-spinner>
				<ion-note>Loading exchange rates...</ion-note>
			{:else if errorMessage}
				<p class="status-copy">{errorMessage}</p>
			{:else if selectedRate && ratePerForeignUnit}
				<ion-row>
					<ion-note
						>{selectedRate.unit}
						{selectedRate.code} = {amountFormatter.format(selectedRate.rate)} CZK</ion-note
					>
				</ion-row>
				<ion-row>
					<ion-note
						>1 CZK = {amountFormatter.format(1 / ratePerForeignUnit)} {selectedRate.code}</ion-note
					>
				</ion-row>
			{:else}
				<ion-note>No exchange rates are available for the selected settings.</ion-note>
			{/if}
		</ion-card-content>
	</ion-card>
</ion-content>
