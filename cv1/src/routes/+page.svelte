<script lang="ts">
	import { onMount } from 'svelte';

	const DATES = {
		LETNI_SEMESTR_START: new Date(2026, 1, 16),
		KONEC_VYUKY_LETNI: new Date(2026, 4, 16),
		ZKOUSKOVE_LETNI_START: new Date(2026, 4, 18),
		ZKOUSKOVE_LETNI_END: new Date(2026, 5, 27)
	};

	const DAY_MS = 24 * 60 * 60 * 1000;

	let actualDate = '00/00/0000';
	let selectedDate = '---';
	let semesterProgress = 0;
	let semesterProgressText = 'Prubeh semestru: 0 %';

	let konecSemestruColor = 'inherit';
	let zkouskaColor = 'inherit';
	let servisColor = 'inherit';

	function formatCsDate(date: Date) {
		return date.toLocaleDateString('cs-CZ');
	}

	function startOfDay(date: Date) {
		return new Date(date.getFullYear(), date.getMonth(), date.getDate());
	}

	function updateSemesterProgress(referenceDate: Date) {
		const startSemestru = DATES.LETNI_SEMESTR_START;
		const konecSemestru = DATES.KONEC_VYUKY_LETNI;

		const totalMs = konecSemestru.getTime() - startSemestru.getTime();
		const elapsedMs = referenceDate.getTime() - startSemestru.getTime();
		let progress = elapsedMs / totalMs;

		if (progress < 0) {
			progress = 0;
		} else if (progress > 1) {
			progress = 1;
		}

		semesterProgress = progress;
		semesterProgressText = `Prubeh semestru: ${Math.round(progress * 100)} %`;
	}

	function getColorByDiff(diffMs: number) {
		if (diffMs < 0) {
			return 'gray';
		}
		if (diffMs < 7 * DAY_MS) {
			return 'red';
		}
		if (diffMs < 30 * DAY_MS) {
			return 'orange';
		}
		return 'green';
	}

	function updateDeadlineColors(referenceDate: Date) {
		const today = startOfDay(referenceDate);

		const konecSemestruDiff = DATES.KONEC_VYUKY_LETNI.getTime() - today.getTime();
		const zkouskaDiff = DATES.ZKOUSKOVE_LETNI_START.getTime() - today.getTime();
		const servisDiff = DATES.ZKOUSKOVE_LETNI_END.getTime() - today.getTime();

		konecSemestruColor = getColorByDiff(konecSemestruDiff);
		zkouskaColor = getColorByDiff(zkouskaDiff);
		servisColor = getColorByDiff(servisDiff);
	}

	function parseIonDate(value: string) {
		const [year, month, day] = value.slice(0, 10).split('-').map(Number);
		if (year && month && day) {
			return new Date(year, month - 1, day);
		}
		return new Date();
	}

	function handleDateChange(event: Event) {
		const target = event.currentTarget as HTMLIonDatetimeElement;
		const rawValue = String(target?.value || '');
		const date = parseIonDate(rawValue);

		selectedDate = formatCsDate(date);
		updateSemesterProgress(startOfDay(date));
		updateDeadlineColors(date);
	}

	async function showNearDeadlinesToast() {
		const today = startOfDay(new Date());
		const deadlines = [
			{ label: 'Konec vyuky v letnim semestru', date: DATES.KONEC_VYUKY_LETNI },
			{ label: 'Zacatek zkouskoveho obdobi', date: DATES.ZKOUSKOVE_LETNI_START },
			{ label: 'Konec zkouskoveho obdobi', date: DATES.ZKOUSKOVE_LETNI_END }
		];

		const upcoming = deadlines
			.map((deadline) => ({
				label: deadline.label,
				daysLeft: Math.round((deadline.date.getTime() - today.getTime()) / DAY_MS)
			}))
			.filter((deadline) => deadline.daysLeft >= 0 && deadline.daysLeft <= 7);

		if (upcoming.length === 0) {
			return;
		}

		const listUpcoming = upcoming
			.map((deadline) => `${deadline.label} (${deadline.daysLeft} dni)`)
			.join(', ');

		const toast = document.createElement('ion-toast');
		toast.message = listUpcoming;
		toast.buttons = [{ text: 'OK', role: 'cancel' }];

		document.body.appendChild(toast);
		await toast.present();
		toast.addEventListener(
			'didDismiss',
			() => {
				toast.remove();
			},
			{ once: true }
		);
	}

	onMount(() => {
		const now = new Date();
		actualDate = formatCsDate(now);
		updateSemesterProgress(startOfDay(now));
		updateDeadlineColors(now);
		void showNearDeadlinesToast();
	});
</script>

<ion-header>
	<ion-toolbar>
		<ion-title>Terminy</ion-title>
	</ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
	<ion-card>
		<ion-card-content>
			<ion-text style="display: block; font-size: 110%; padding-bottom: 5px;">
				Dnesni datum: {actualDate}
			</ion-text>
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-content>
			<ion-text style="display: block; font-size: 150%; text-align: center;">Zadej datum</ion-text>
			<p style="text-align: center; padding-bottom: 5px;">
				<ion-datetime-button datetime="datetime"></ion-datetime-button>
				<ion-modal>
					<ion-datetime
						id="datetime"
						locale="cs-CZ"
						presentation="date"
						show-default-buttons={true}
						on:ionChange={handleDateChange}
					></ion-datetime>
				</ion-modal>
			</p>
			<ion-text style="display: block; font-size: 110%; padding-top: 5px; padding-bottom: 5px;">
				Zvolene datum: {selectedDate}
			</ion-text>
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-content>
			<ion-text
				style={`display: block; font-size: 110%; padding-bottom: 5px; color: ${konecSemestruColor};`}
			>
				Konec vyuky v letnim semestru: {formatCsDate(DATES.KONEC_VYUKY_LETNI)}
			</ion-text>

			<ion-text
				style={`display: block; font-size: 110%; padding-bottom: 5px; color: ${zkouskaColor};`}
			>
				Zacatek zkouskoveho obdobi: {formatCsDate(DATES.ZKOUSKOVE_LETNI_START)}
			</ion-text>

			<ion-text
				style={`display: block; font-size: 110%; padding-bottom: 5px; color: ${servisColor};`}
			>
				Konec zkouskoveho obdobi: {formatCsDate(DATES.ZKOUSKOVE_LETNI_END)}
			</ion-text>
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-content>
			<ion-text>{semesterProgressText}</ion-text>
			<ion-progress-bar value={semesterProgress}></ion-progress-bar>
		</ion-card-content>
	</ion-card>
</ion-content>
