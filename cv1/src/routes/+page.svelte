<script lang="ts">
	type IonInputEvent = CustomEvent<{ value?: string | null }>;

	const defaultScriptUrl = 'https://homel.vsb.cz/~mor03/TAMZ/TAMZ22.php';

	let scriptUrl = defaultScriptUrl;
	let login = '';
	let secretToken = '';
	let decodedSecretToken = '';
	let finalMessage = '';
	let errorMessage = '';
	let activeRequest: 'idle' | 'getting-code' | 'sending-code' = 'idle';

	$: isGettingCode = activeRequest === 'getting-code';
	$: isSendingCode = activeRequest === 'sending-code';
	$: isBusy = activeRequest !== 'idle';
	$: canGetCode = !isBusy && scriptUrl.trim().length > 0 && login.trim().length > 0;
	$: canSendCode = !isBusy && scriptUrl.trim().length > 0 && secretToken.length > 0;

	async function getCode() {
		const normalizedUrl = scriptUrl.trim();
		const normalizedLogin = login.trim();

		if (!normalizedUrl || !normalizedLogin) {
			errorMessage = 'Enter both the script URL and your login first.';
			return;
		}

		try {
			new URL(normalizedUrl);
		} catch {
			errorMessage = 'Enter a valid script URL.';
			return;
		}

		activeRequest = 'getting-code';
		errorMessage = '';
		secretToken = '';
		decodedSecretToken = '';
		finalMessage = '';

		try {
			const response = await fetch(normalizedUrl, {
				method: 'POST',
				body: new URLSearchParams({
					user: normalizedLogin,
					timestamp: String(Date.now())
				})
			});

			const responseText = (await response.text()).trim();

			if (!response.ok) {
				throw new Error(responseText || `Request failed with status ${response.status}.`);
			}

			if (!responseText) {
				throw new Error('The server returned an empty token response.');
			}

			secretToken = responseText;

			try {
				decodedSecretToken = atob(responseText);
			} catch {
				throw new Error('The server response is not valid base64 data.');
			}
		} catch (error) {
			secretToken = '';
			decodedSecretToken = '';
			finalMessage = '';
			errorMessage = error instanceof Error ? error.message : 'Unable to request the secret token.';
		} finally {
			activeRequest = 'idle';
		}
	}

	async function sendCode() {
		const normalizedUrl = scriptUrl.trim();

		if (!normalizedUrl) {
			errorMessage = 'Enter the script URL first.';
			return;
		}

		if (!secretToken) {
			errorMessage = 'Request the secret token before sending the authorization header.';
			return;
		}

		try {
			new URL(normalizedUrl);
		} catch {
			errorMessage = 'Enter a valid script URL.';
			return;
		}

		activeRequest = 'sending-code';
		errorMessage = '';
		finalMessage = '';

		try {
			const response = await fetch(normalizedUrl, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${secretToken}`
				}
			});

			const responseText = (await response.text()).trim();

			if (!response.ok) {
				throw new Error(responseText || `Request failed with status ${response.status}.`);
			}

			if (!responseText) {
				throw new Error('The server returned an empty final message.');
			}

			finalMessage = responseText;
		} catch (error) {
			finalMessage = '';
			errorMessage =
				error instanceof Error ? error.message : 'Unable to send the authorization request.';
		} finally {
			activeRequest = 'idle';
		}
	}
</script>

<ion-header>
	<ion-toolbar>
		<ion-title>HTTP</ion-title>
	</ion-toolbar>
</ion-header>

<ion-content class="http-page">
	<ion-card>
		<ion-card-content>
			<ion-item>
				<ion-input
					label="Script URL"
					label-placement="stacked"
					placeholder="https://homel.vsb.cz/~mor03/TAMZ/TAMZ22.php"
					value={scriptUrl}
					on:ionInput={(event: IonInputEvent) => {
						scriptUrl = event.detail.value ?? '';
						secretToken = '';
						decodedSecretToken = '';
						finalMessage = '';
						errorMessage = '';
					}}
				></ion-input>
			</ion-item>

			<ion-item>
				<ion-input
					label="Login"
					label-placement="stacked"
					placeholder="Enter your username"
					value={login}
					on:ionInput={(event: IonInputEvent) => {
						login = event.detail.value ?? '';
						secretToken = '';
						decodedSecretToken = '';
						finalMessage = '';
						errorMessage = '';
					}}
				></ion-input>
			</ion-item>
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-content>
			<ion-item>
				<ion-textarea
					label="Received Secret Data"
					label-placement="stacked"
					auto-grow={true}
					readonly={true}
					value={secretToken}
				></ion-textarea>
			</ion-item>

			<ion-item>
				<ion-textarea
					label="Decoded Secret Data (atob)"
					label-placement="stacked"
					auto-grow={true}
					readonly={true}
					value={decodedSecretToken}
				></ion-textarea>
			</ion-item>

			<ion-item>
				<ion-textarea
					label="Final Message"
					label-placement="stacked"
					auto-grow={true}
					readonly={true}
					value={finalMessage}
				></ion-textarea>
			</ion-item>

			{#if errorMessage}
				<ion-text color="danger" class="status-text">
					<p>{errorMessage}</p>
				</ion-text>
			{/if}
		</ion-card-content>
	</ion-card>

	<ion-card>
		<ion-card-content>
			<div class="button-stack">
				<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions: ion-button is an interactive Ionic custom element. -->
				<ion-button expand="block" on:click={getCode} disabled={!canGetCode}>
					{isGettingCode ? 'GETTING CODE...' : 'GET CODE'}
				</ion-button>

				<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions: ion-button is an interactive Ionic custom element. -->
				<ion-button expand="block" on:click={sendCode} disabled={!canSendCode}>
					{isSendingCode ? 'SENDING CODE...' : 'SEND CODE'}
				</ion-button>
			</div>
		</ion-card-content>
	</ion-card>
</ion-content>
