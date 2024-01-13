<script>
	import { Alert, Input, Label, Button } from 'sveltestrap';
	let isDetectedOnce = false;
	/**
	 * @type {number} latitude 緯度
	 */
	let latitude;

	/**
	 * @type {number} latitude 緯度
	 */
	let longitude;

	/**
	 * @type {number} grantMeters 許容されるメートル
	 */
	let grantMeters = 100;

	/**
	 * @type {boolean} enableWaiting ウェイトを有効にするか（動作検証用）
	 */
	let enableWaiting;

	/**
	 * @type {Promise<any>} checkCurrentPositionPromise 現在位置判定中のプロミスオブジェクト
	 */
	let checkCurrentPositionPromise;

	/**
	 * @type {Promise<any>} getCurrentPositionButtonPromise 現在位置取得中のプロミスオブジェクト
	 */
	let getCurrentPositionButtonPromise;

	/**
	 * @type {string} errorMessage サイト内で発生しているエラー
	 */
	let errorMessage;

	/**
	 * @type {Object<number, string>} getCurrentPositionErrorMessageMap getCurrentPositionのエラーコードとエラーメッセージのマッピング
	 */
	const getCurrentPositionErrorMessageMap = {
		0: '位置情報の取得に失敗しました。',
		1: '位置情報の取得を許可してください。',
		2: '電波状況をご確認の上、再度取得してください。',
		3: '位置情報の取得がタイムアウトしました。'
	};

	const sleep = () => {
		return new Promise((resolve) => setTimeout(resolve, 1000));
	};

	const getCurrentPosition = () => {
		return new Promise((resolve, reject) => {
			navigator.permissions.query({ name: 'geolocation' }).then((result) => {
				if (result.state === 'denied') {
					errorMessage = '位置情報の取得を許可してください。';
					reject({ message: errorMessage });
				}
			});
			const options = {
				enableHighAccuracy: true
			};
			navigator.geolocation.getCurrentPosition(
				resolve,
				(error) => {
					reject({ message: getCurrentPositionErrorMessageMap[error.code] });
				},
				options
			);
		});
	};

	/**
	 * 指定された範囲内にいるかどうか判定する
	 */
	const checkCurrentPositionInSpecifiedArea = async () => {
		isDetectedOnce = true;
		errorMessage = '';
		if (enableWaiting) {
			await sleep();
		}
		const promise = await new Promise((resolve, reject) => {
			getCurrentPosition()
				.then((position) => {
					const currentLatitude = position.coords.latitude;
					const currentLongitude = position.coords.longitude;
					const distance = haversineDistance(
						[latitude, longitude],
						[currentLatitude, currentLongitude]
					);
					errorMessage = '';
					if (distance > grantMeters / 1000) {
						resolve(
							`現在位置は、入力された緯度・経度の${grantMeters}m以内ではありません。${
								distance * 1000
							}m離れているようです。`
						);
					}
					resolve(`現在位置は、入力された緯度・経度の${grantMeters}m以内です。`);
				})
				.catch((error) => {
					errorMessage = error.message;
					return reject(errorMessage);
				});
		});
		return promise;
	};

	/**
	 * ハバーサインの公式を使って、2点間の距離（km）を求める。
	 * @param {Array<number>} point1 1点目の[緯度, 経度]
	 * @param {Array<number>} point2 2点目の[緯度, 経度]
	 * @returns {number} 2点間の距離（km）
	 */
	function haversineDistance([lat1, lon1], [lat2, lon2]) {
		const R = 6371; // 地球の半径（km）
		const dLat = ((lat2 - lat1) * Math.PI) / 180;
		const dLon = ((lon2 - lon1) * Math.PI) / 180;
		const a =
			Math.sin(dLat / 2) * Math.sin(dLat / 2) +
			Math.cos((lat1 * Math.PI) / 180) *
				Math.cos((lat2 * Math.PI) / 180) *
				Math.sin(dLon / 2) *
				Math.sin(dLon / 2);
		const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
		return R * c; // 距離（km）
	}

	/**
	 * 現在位置を取得し、緯度と経度に代入する
	 */
	const getCurrentPositionAndAssign = async () => {
		if (enableWaiting) {
			await sleep();
		}
		const promise = new Promise((resolve, reject) => {
			getCurrentPosition()
				.then((position) => {
					latitude = position.coords.latitude;
					longitude = position.coords.longitude;
					return resolve();
				})
				.catch((error) => {
					errorMessage = error.message;
				});
		});
		return promise;
	};

	/**
	 * 「判定」ボタンのハンドラ
	 */
	const handleCheckCurrentPositionButtonClick = () => {
		checkCurrentPositionPromise = checkCurrentPositionInSpecifiedArea();
	};

	/**
	 * 「現在の緯度・経度を反映」のハンドラ
	 */
	const handleGetCurrentPositionButtonClick = () => {
		errorMessage = '';
		getCurrentPositionButtonPromise = getCurrentPositionAndAssign();
	};
</script>

{#if errorMessage}
	<Alert color="danger">
		{errorMessage}
	</Alert>
{/if}

<h1>GeoLocation APIの素振り</h1>
<p>緯度と経度、許容する距離を入力すると、現在位置が許容する距離以内であるかどうかを判定します。</p>
<p>※ 未入力は0として扱われます。</p>

<div class="row">
	<div class="col-6">
		<Label for="latitude">緯度</Label>
		<Input id="latitude" type="number" bind:value={latitude} />
	</div>
	<div class="col-6">
		<Label for="longitude">緯度</Label>
		<Input id="longitude" type="number" bind:value={longitude} />
	</div>
	<div class="col-6">
		<Label for="grantMeters">許容する距離（m）</Label>
		<Input id="grantMeters" type="number" bind:value={grantMeters} />
	</div>
	<div class="col-12 mt-2">
		<Input
			type="checkbox"
			label="取得にウェイトを入れて、awaitの動作チェックをする（1秒sleepします）"
			bind:checked={enableWaiting}
		/>
	</div>
</div>
<div class="mt-2">
	<Button outline color="primary" on:click={handleGetCurrentPositionButtonClick}>
		{#await getCurrentPositionButtonPromise}
			反映中…
		{:then}
			現在の緯度・経度を反映
		{/await}</Button
	>
	<Button outline color="primary" on:click={handleCheckCurrentPositionButtonClick}>判定</Button>
</div>

<div class="mt-3">
	<h2>判定結果</h2>
	<div class="result-area">
		{#if !isDetectedOnce}
			<span>「判定」ボタンを押してください。</span>
		{:else}
			{#await checkCurrentPositionPromise}
				<span>判定中…</span>
			{:then result}
				<span>
					{result}
				</span>
			{:catch}
				<span> 位置情報の取得に失敗しました。</span>
			{/await}
		{/if}
	</div>
</div>

<style lang="scss">
	.result-area {
		background-color: gainsboro;
		text-align: center;
		padding: 30px 0;
	}
</style>
