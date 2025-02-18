<script>
	import JSONNode from 'svelte-json-tree';
	import ConsoleTable from './ConsoleTable.svelte';

	export let log;
	export let level = 1;

	function toggleGroupCollapse() {
		log.collapsed = !log.collapsed;
	}
</script>

{#if log.level === 'table'}
	<ConsoleTable data={log.args[0]} columns={log.args[1]} />
{/if}

<div class="log console-{log.level}" style="padding-inline-start: {level * 15}px" on:click={log.level === 'group' ? toggleGroupCollapse : undefined}>
	{#if log.count > 1}
		<span class="count">{log.count}x</span>
	{/if}

	{#if log.level === 'trace' || log.level === 'assert'}
		<div class="arrow" class:expand={!log.collapsed} on:click={toggleGroupCollapse}>▶</div>
	{/if}

	{#if log.level === 'assert'}
		<span class="assert">Assertion failed:</span>
	{/if}

	{#if log.level === 'clear'}
		<span class="info">Console was cleared</span>
	{:else if log.level === 'unclonable'}
		<span class="info error">Message could not be cloned. Open devtools to see it</span>
	{:else if log.level === 'group'}
		<div class="arrow" class:expand={!log.collapsed}>▶</div>
		<span class="title">{log.label}</span>
	{:else if log.level.startsWith('system')}
		{#each log.args as arg}
			{arg}
		{/each}
	{:else if log.level === 'table'}
		<JSONNode value={log.args[0]} />
	{:else}
		{#each log.args as arg}
			<JSONNode value={arg} />
		{/each}
	{/if}
	{#each new Array(level - 1) as _, idx}
		<div class="outline" style="inset-inline-start: {idx * 15 + 15}px" />
	{/each}
</div>

{#if log.level === 'group' && !log.collapsed}
	{#each log.logs as childLog}
		<svelte:self log={childLog} level={level + 1}/>
	{/each}
{/if}

{#if (log.level === 'trace' || log.level === 'assert') && !log.collapsed}
	<div class="trace">
		{#each log.stack.split('\n').slice(2) as stack}
			<div>{stack.replace(/^\s*at\s+/, '')}</div>
		{/each}
	</div>
{/if}

<style>
	.log {
		border-block-end: 1px solid #eee;
		padding-inline: 10px;
		padding-block: 5px 0;
		display: flex;
		position: relative;
		font-size: 12px;
		font-family: var(--font-mono);
	}

	.log > :global(*) {
		margin-inline-end: 10px;
		font-family: var(--font-mono);
	}

	.console-warn, .console-system-warn {
		background: #fffbe6;
		border-color: #fff4c4;
	}

	.console-error, .console-assert {
		background: #fff0f0;
		border-color: #fed6d7;
	}

	.console-group, .arrow {
		cursor: pointer;
		user-select: none;
	}

	.console-trace, .console-assert {
		border-block-end: none;
	}

	.console-assert + .trace {
		background: #fff0f0;
		border-color: #fed6d7;
	}

	.trace {
		border-block-end: 1px solid #eee;
		font-size: 12px;
		font-family: var(--font-mono);
		padding-block: 4px 2px;
	}

	.trace > :global(div) {
		margin-inline-start: 15px;
	}

	.count {
		color: #999;
		font-size: 12px;
		line-height: 1.2;
	}

	.info {
		color: #666;
		font-family: var(--font) !important;
		font-size: 12px;
	}

	.error {
		color: #da106e; /* todo make this a var */
	}

	.outline {
		border-inline-start: 1px solid #9c9cab;
		position: absolute;
		inset-block-start: 0;
		inset-block-end: -1px;
	}

	.arrow {
		position: absolute;
		font-size: 0.6em;
		transition: 150ms;
		transform-origin: 50% 50%;
		transform: translateY(1px) translateX(-50%);
	}

	.arrow.expand {
		transform: translateY(1px) translateX(-50%) rotateZ(90deg);
	}

	.title {
		font-family: var(--font-mono);
		font-size: 13px;
		font-weight: bold;
		padding-inline-start: 11px;
		block-size: 19px;
	}

	.assert {
		padding-inline-start: 11px;
		font-weight: bold;
		color: #da106e;
	}
</style>