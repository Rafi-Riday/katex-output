<script>
	import { tick } from "svelte";
	import Katexify from "./Katexify";

	let currentExpr = "";
	let fieldStr;
	let resetBtn;
	let loaded = false;
	setTimeout(() => {
		loaded = true;
	}, 1000);

	const copy2Clip = async () => {
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		resetBtn.focus();
	};

	const copy2ClipForCode = async () => {
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		currentExpr = currentExpr.trim().replaceAll("\\", "\\\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		resetBtn.focus();
	};

	let toBeCopied = [];

	const copy2ClipAll = async () => {
		let temp = currentExpr.trim();
		currentExpr = toBeCopied.map((tbc) => tbc.str).join("\n");
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		resetBtn.focus();
		currentExpr = temp;
	};

	const copy2ClipForCodeAll = async () => {
		let temp = currentExpr.trim();
		currentExpr = toBeCopied.map((tbc) => tbc.str).join("\n");
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		currentExpr = currentExpr.trim().replaceAll("\\", "\\\\");
		await tick();
		fieldStr.select();
		document.execCommand("copy");
		currentExpr = currentExpr.trim().replaceAll("\\\\", "\\");
		resetBtn.focus();
		currentExpr = temp;
	};

	const featureKey = async (key, cursor = 1) => {
		const { selectionStart, selectionEnd, value } = fieldStr;
		if (selectionStart === selectionEnd) {
			currentExpr =
				value.slice(0, selectionStart) +
				key +
				value.slice(selectionStart, value.length);
		}
		await tick();
		fieldStr.selectionStart = selectionStart + cursor;
		fieldStr.selectionEnd = selectionStart + cursor;
		fieldStr.focus();
	};

	let preExpression = [
		{ key: "+", cursor: 1 },
		{ key: "-", cursor: 1 },
		{ key: "×", cursor: 1 },
		{ key: "÷", cursor: 1 },
		{ key: "=", cursor: 1 },
		{ key: "^", cursor: 1 },
		{ key: "(", cursor: 1 },
		{ key: ")", cursor: 1 },
		{ key: "()", cursor: 1 },
		{ key: "{", cursor: 1 },
		{ key: "}", cursor: 1 },
		{ key: "{}", cursor: 1 },
		{ key: "[]", cursor: 1 },
		{ key: "\\", cursor: 1 },
		{ key: "°", cursor: 1 },
		{ key: "_", cursor: 1 },
		{ key: "$$", cursor: 1 },
		{ key: "\\text{}", cursor: 6 },
		{ key: "\\frac{}{}", cursor: 6 },
		{ key: "\\sqrt{}", cursor: 6 },
		{ key: "\\big", cursor: 4 },
		{ key: "\\newline ", cursor: 9 },
	];

	let demo = [
		"a^2",
		"a^{xyz}",
		"a_i",
		"\\big( \\big) \\big[ \\big]",
		"\\times\\div",
		"\\frac{a}{b}",
		"\\sqrt{ab}",
		"2x^2+11x+12",
		"\\frac{-b\\pm\\sqrt{b^2-4ac}}{2a}",
		"\\frac{b}{2}\\sqrt{a^2-\\frac{b^2}{4}}",
	];

	let tempStorage = "";
	let tempEdittingValue;

	let edittingIdx = null;
	$: if (edittingIdx !== null && toBeCopied[edittingIdx]) {
		toBeCopied[edittingIdx].str = currentExpr.trim();
		toBeCopied = toBeCopied;
	}

	const edittingDone = () => {
		if (toBeCopied[edittingIdx]) {
			toBeCopied[edittingIdx].str = currentExpr.trim();
			toBeCopied[edittingIdx].editting = false;
		}
		edittingIdx = null;
		currentExpr = tempStorage;
		tempStorage = "";
	};

	// empty item detector in toBeCopied
	$: if (edittingIdx === null) {
		toBeCopied = toBeCopied.filter((tbc) => tbc.str.trim() !== "");
	}
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://cdn.jsdelivr.net/npm/katex@0.16.7/dist/katex.min.css"
		integrity="sha384-3UiQGuEI4TTMaFmGIZumfRPtfKQ3trwQE2JgosJxCnGmQpL/lJdjpcHkaaFwHlcI"
		crossorigin="anonymous"
	/>
</svelte:head>

<main>
	<!-- output -->
	{#if currentExpr.trim() !== ""}
		<center class="output">
			{@html Katexify(currentExpr.trim())}
			<!-- displayMode -->
			<!-- .replace(/\\/g, "\\\\") -->
		</center>
	{/if}
	<!-- input -->
	<div class="inp-cont">
		<textarea
			placeholder="Write your expression here..."
			bind:this={fieldStr}
			bind:value={currentExpr}
		/>
	</div>
	<!-- commonly used keys -->
	<aside class="feature-key">
		{#each preExpression as expData}
			<button
				class="feature-key-btn btn"
				style="padding: 10px {expData.key.trim().length > 1
					? '10px'
					: '20px'}; {['()', '[]']
					.map((p) => expData.key.includes(p))
					.includes(true)
					? 'letter-spacing: 1px;'
					: ''}"
				on:click={() => {
					featureKey(expData.key, expData.cursor || 1);
				}}
			>
				{expData.key}
			</button>
		{/each}
	</aside>
	<!-- copy-reset btn -->
	{#if currentExpr.trim() !== ""}
		<div class="actions">
			{#if toBeCopied.map((tbc) => tbc.editting).includes(true)}
				<button
					class="btn"
					style="background: #4e4eeb; color: #fff"
					on:click={edittingDone}
				>
					Done?
				</button>
			{:else}
				<button
					class="btn"
					style="background: #74efff;"
					on:click={() => {
						if (
							!toBeCopied
								.map((tbc) => tbc.str)
								.includes(currentExpr.trim()) &&
							currentExpr.trim() !== ""
						) {
							toBeCopied.push({
								str: currentExpr.trim(),
								editting: false,
							});
							toBeCopied = toBeCopied;
							currentExpr = "";
							fieldStr.focus();
						}
					}}
				>
					{toBeCopied
						.map((tbc) => tbc.str)
						.includes(currentExpr.trim())
						? "Added"
						: "Add"}
				</button>
			{/if}
			<button
				class="btn"
				style="background: #74efff;"
				on:click={copy2Clip}>Copy</button
			>
			<button
				style="border: 0; background: #74efff; border-radius: 10px; color: #000; padding: 7.5px 16px; font-size: 18px; display: flex; align-items: center; justify-content: space-between;"
				on:click={copy2ClipForCode}>Copy&nbsp;<code>\\</code></button
			>
			<button
				bind:this={resetBtn}
				class="btn"
				style="background: #ffc6b5;"
				on:click={() => {
					currentExpr = "";
					fieldStr.focus();
				}}>Reset</button
			>
		</div>
	{/if}
	<!-- copy all -->
	{#if toBeCopied.length !== 0}
		<div class="actions">
			<button
				class="btn"
				style="background: #00b8ff;"
				on:click={copy2ClipAll}>Copy All</button
			>
			<button
				style="border: 0; background: #00b8ff; border-radius: 10px; color: #000; padding: 7.5px 16px; font-size: 18px; display: flex; align-items: center; justify-content: space-between;"
				on:click={copy2ClipForCodeAll}
				>Copy All&nbsp;<code>\\</code></button
			>
			<button
				bind:this={resetBtn}
				class="btn"
				style="background: #ff4747; color: #fff"
				on:click={() => {
					toBeCopied = [];
				}}>Clear List</button
			>
		</div>
	{/if}
	<!-- to be copied -->
	{#if toBeCopied.length > 0}
		<aside class="demo">
			<b style="margin: 5px 0;font-size: 17px;"
				>To be copied for JavaScript Syntax</b
			>
			<div style="display: flex; flex-direction: column; gap: 8px">
				{#each toBeCopied as t, idx (idx)}
					<div
						style="margin-left: 5px; display: flex; flex-wrap: wrap; flex-direction: row; justify-content: start; align-items: center; gap: 8px;"
					>
						<span style="font-weight: 900;">{idx + 1}.</span>
						{@html Katexify(t.str)}
						{#if t.editting}
							<!-- tick -->
							<button
								style="background-color: #4e4eeb; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
								on:click={edittingDone}
							>
								<svg
									width="13"
									height="13"
									stroke-width="0"
									fill="#fff"
									style="transform: translate(0.5px, 1px);"
									viewBox="0 0 512 512"
									xmlns="http://www.w3.org/2000/svg"
									><path
										d="M173.898 439.404l-166.4-166.4c-9.997-9.997-9.997-26.206 0-36.204l36.203-36.204c9.997-9.998 26.207-9.998 36.204 0L192 312.69 432.095 72.596c9.997-9.997 26.207-9.997 36.204 0l36.203 36.204c9.997 9.997 9.997 26.206 0 36.204l-294.4 294.401c-9.998 9.997-26.207 9.997-36.204-.001z"
									/>
								</svg>
							</button>
							<!-- cancel -->
							<button
								style="background-color: #4e4eeb; color: #fff; border: 0; border-radius: 25%; outline-color: #9c9cff; width:25px; height:25px; font-size: 20px; font-weight: 900;"
								on:click={() => {
									edittingDone();
									t.str = tempEdittingValue;
								}}
							>
								×
							</button>
							<!-- remove -->
							<button
								style="background-color: #f00; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
								on:click={() => {
									edittingIdx = null;
									currentExpr = tempStorage;
									toBeCopied = toBeCopied.filter(
										(ti) => ti.str !== t.str
									);
								}}
							>
								<svg
									viewBox="0 0 38 38"
									width="30"
									height="30"
									stroke-width="0"
									fill="#fff"
									style="transform: translate(-2.5px, 1.4px);"
									xmlns="http://www.w3.org/2000/svg"
									><path
										d="M5 20a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8h2V6h-4V4a2 2 0 0 0-2-2H9a2 2 0 0 0-2 2v2H3v2h2zM9 4h6v2H9zM8 8h9v12H7V8z"
									/><path d="M9 10h2v8H9zm4 0h2v8h-2z" /></svg
								>
							</button>
						{:else}
							<!-- edit -->
							<button
								style="background-color: #4e4eeb; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
								on:click={() => {
									tempEdittingValue = t.str;
									toBeCopied.forEach(
										(tbc) => (tbc.editting = false)
									);
									t.editting = true;
									if (edittingIdx === null) {
										tempStorage = currentExpr.trim();
									}
									currentExpr = t.str;
									edittingIdx = idx;
									fieldStr.focus();
								}}
							>
								<svg
									width="20"
									height="20"
									stroke-width="0"
									fill="#fff"
									style="transform: translate(-2px, 0px);"
									xmlns="http://www.w3.org/2000/svg"
									><path
										d="M 3 15 L 11 7 L 13 9 L 5 17 L 3 17 L 3 15 M 12 6 L 14 8 L 15 7 L 13 5 L 12 6"
									/></svg
								>
							</button>
						{/if}
						{#if !t.editting}
							<!-- remove -->
							<button
								style="background-color: #f00; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
								on:click={() => {
									toBeCopied = toBeCopied.filter(
										(ti) => ti.str !== t.str
									);
								}}
							>
								<svg
									viewBox="0 0 38 38"
									width="30"
									height="30"
									stroke-width="0"
									fill="#fff"
									style="transform: translate(-2.5px, 1.4px);"
									xmlns="http://www.w3.org/2000/svg"
									><path
										d="M5 20a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8h2V6h-4V4a2 2 0 0 0-2-2H9a2 2 0 0 0-2 2v2H3v2h2zM9 4h6v2H9zM8 8h9v12H7V8z"
									/><path d="M9 10h2v8H9zm4 0h2v8h-2z" /></svg
								>
							</button>
						{/if}
						<!-- moving starts -->
						{#if edittingIdx === null}
							<!-- move up -->
							{#if idx !== 0}
								<button
									style="background-color: #323232df; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
									on:click={() => {
										let tmp = toBeCopied[idx - 1];
										toBeCopied[idx - 1] = toBeCopied[idx];
										toBeCopied[idx] = tmp;
									}}
								>
									<svg
										viewBox="0 0 28 28"
										width="20"
										height="20"
										stroke-width="0"
										fill="#fff"
										style="transform: translate(-0.6px, 7px);"
										xmlns="http://www.w3.org/2000/svg"
										><path
											d="M 0 10 L 20 10 L 10 0 L 0 10"
										/></svg
									>
								</button>
							{/if}
							<!-- move down -->
							{#if idx !== toBeCopied.length - 1}
								<button
									style="background-color: #323232df; border: 0; border-radius: 25%; outline: none; width:25px; height:25px;"
									on:click={() => {
										let tmp = toBeCopied[idx + 1];
										toBeCopied[idx + 1] = toBeCopied[idx];
										toBeCopied[idx] = tmp;
									}}
								>
									<svg
										viewBox="0 0 28 28"
										width="20"
										height="20"
										stroke-width="0"
										fill="#fff"
										style="transform: translate(-0.6px, 9px);"
										xmlns="http://www.w3.org/2000/svg"
										><path
											d="M 0 0 L 20 0 L 10 10 L 0 0"
										/></svg
									>
								</button>
							{/if}
							<!-- moving ends -->
						{/if}
					</div>
				{/each}
			</div>
		</aside>
	{/if}
	<!-- demo -->
	{#if loaded}
		<aside class="demo">
			<b style="margin: 5px 0;font-size: 20px;">Demo</b>
			{#each demo as d}
				<div>
					<button class="code" on:click={() => (currentExpr = d)}
						>{d}</button
					>
					=
					{@html Katexify(d)}
				</div>
			{/each}
			<p style="display: flex; flex-direction: column; gap: 12px">
				<a
					href="https://katex.org/docs/supported.html"
					target="_blank"
					rel="noreferrer"
				>
					<b>More Demo...</b>
				</a>
				<a
					href="https://detexify.kirelabs.org/classify.html"
					target="_blank"
					rel="noreferrer"
				>
					<b>Detect a Symbol by Drawing...</b>
				</a>
			</p>
		</aside>
	{/if}
</main>

<style>
	:global(html, body) {
		height: 100%;
		width: 100%;
		color: #000;
		background-color: #323232;
		margin: 0;
		padding: 0;
	}
	main {
		padding: 10px;
		display: flex;
		flex-direction: column;
		align-items: stretch;
		justify-content: center;
		flex-wrap: wrap;
		gap: 10px;
	}
	.output {
		font-size: 18px;
		background-color: #fff;
		border-radius: 10px;
		padding: 5px;
		display: inline;
	}
	.inp-cont {
		border-bottom: 2px solid #fffb;
	}
	textarea {
		background-color: #fff;
		width: calc(100% - 16px);
		/* width: 100%; */
		height: 75px;
		/* border: 2px dashed #444; */
		border: 2px solid #ff3300;
		/* margin: 0; */
		border-radius: 10px;
		padding: 5px;
		outline: 0;
	}
	.actions {
		display: flex;
		flex-wrap: wrap;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		gap: 10px;
	}
	.btn {
		border: 0;
		border-radius: 10px;
		color: #000;
		padding: 10px 16px;
		font-size: 18px;
	}
	.feature-key {
		max-width: 380px;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		align-items: center;
		gap: 10px;
		border-radius: 10px;
		margin: 0 auto 0 auto;
	}
	.feature-key-btn {
		font-size: 18px;
		padding: 3px 5px;
	}
	.demo {
		background-color: #fff;
		padding: 10px;
		border-radius: 10px;
		display: flex;
		flex-direction: column;
		gap: 5px;
	}
	.code {
		display: inline-block;
		color: #ff3300;
		font-weight: 600;
		background-color: #ff3e0010;
		border: 2px solid #ff3e0020;
		padding: 2px 5px;
		border-radius: 10px;
	}
	a {
		color: #0058aa;
	}
	code {
		background: #fffb;
		border-radius: 5px;
		border: 1.5px solid #ff3e00aa;
		letter-spacing: -7px;
		padding-right: 7px;
	}
</style>
