<script>
	import { Input, Label, Button } from 'sveltestrap';
	let isDetected = false;
	/**
	 * 緯度と経度の100m以内に自分がいるかどうか判定する
	 * @param {Number} longitude 緯度
	 * @param {Number} latitude 経度
	 */
	const checkCurrentPositionInSpecifiedArea = (longitude, latitude) => {};
	// 続き async化
</script>

<h1>GeoLocation APIの素振り</h1>
<p>緯度と経度を入力すると、自分のいる場所がその緯度と経度の100m付近かどうかを判定します。</p>
<div class="w-25">
	<Label for="latitude">緯度</Label>
	<Input id="latitude" type="number" />
</div>
<div class="w-25">
	<Label for="longitude">緯度</Label>
	<Input id="longitude" type="number" />
</div>
<div class="mt-2">
	<Button outline color="primary">判定</Button>
</div>

<div class="mt-3">
	<h2>判定結果</h2>
	<div class="result-area">
		<span>「判定」ボタンを押してください</span>
	</div>
	
</div>

<style lang="scss">
	.result-area {
		background-color: gainsboro;
		text-align: center;
		padding: 30px 0;
	}
</style>
