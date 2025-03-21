<script lang="ts">
	import { getPrefixCls } from '@ikun-ui/utils';
	import { clsx } from 'clsx';
	import type { KTourProps } from './types';
	import { KPopover } from '@ikun-ui/popover';
	import { onMount, tick } from 'svelte';
	import { createEventDispatcher } from 'svelte';
	import { KIcon } from '@ikun-ui/icon';
	import { KButton } from '@ikun-ui/button';
	import { KIndicators } from '@ikun-ui/indicators';
	// @ts-ignore
	import { BROWSER } from 'esm-env';
	export let steps: KTourProps['steps'] = [];
	export let mask: KTourProps['mask'] = true;
	export let closeIcon: KTourProps['closeIcon'] = 'i-carbon-close';
	export let placement: KTourProps['placement'] = 'top';
	export let current: KTourProps['current'] = 0;
	export let scrollIntoViewOptions: KTourProps['scrollIntoViewOptions'] = true;
	export let zIndex: KTourProps['zIndex'] = 1001;
	export let open: KTourProps['open'] = false;
	export let cls: KTourProps['cls'] = undefined;
	export let contentCls: KTourProps['contentCls'] = undefined;
	export let attrs: KTourProps['attrs'] = {};
	export let prevBtnText: KTourProps['prevBtnText'] = 'prev';
	export let nextBtnText: KTourProps['nextBtnText'] = 'next';
	export let finishBtnText: KTourProps['finishBtnText'] = 'finish';
	const dispatch = createEventDispatcher();
	let index = current;
	$: {
		initIndex(current, steps.length);
	}
	function initIndex(cur: number, max: number) {
		if (cur < 0) {
			index = 0;
		}
		if (current > max) {
			index = max;
		}
	}
	const handleNext = () => {
		index++;
		initIndex(index, steps.length);
		onIndexChange();
	};

	const handlePrev = () => {
		index--;
		initIndex(index, steps.length);
		onIndexChange();
	};
	const onIndexChange = async () => {
		if (index >= steps.length) {
			await doClose(false, index - 1);
			dispatch('finish');
			return;
		}
		await setTriggerElStyle();
		await handleAppend();
		popoverRef.forceUpdated();
		await tick();
		resolveShowArrow = showArrow;
		await tick();
		popoverRef.updatedArrow();
		dispatch('change', index);
	};

	async function handleAppend() {
		const { target } = steps[index];
		const el: null | HTMLElement = popoverRef.getPopoverContainerRef();
		if (target && el) {
			(target as HTMLElement).appendChild(el);
			const rect = target.getBoundingClientRect();
			if (isElementNotInViewport(rect)) {
				(scrollIntoViewOptions || scrollIntoViewOptions === false) &&
					target.scrollIntoView(scrollIntoViewOptions);
			}
		}
	}

	let popoverRef: any = null;
	let popoverWidth: string = '';
	let maskWidth = '';
	let maskHeight = '';
	let maskBorderTopWidth = '';
	let maskBorderLeftWidth = '';
	let maskBorderRightWidth = '';
	let maskBorderBottomWidth = '';
	let maskRootBg = 'transparent';
	let maskRootTransition = '';
	let showArrow = true;
	let resolveShowArrow = showArrow;
	async function setTriggerElStyle() {
		// transition = false
		const { target } = steps[index];
		const el: null | HTMLElement = popoverRef.getPopoverContainerRef();
		function setFullScreen() {
			if (el) {
				el.style.position = 'fixed';
				el.style.left = el.style.top = `50%`;
				el.style.transform = `translate(-50%, -50%)`;
				// maskRootTransition = transition ? 'border-width .3s' : '';
				maskBorderBottomWidth =
					maskBorderRightWidth =
					maskBorderTopWidth =
					maskBorderLeftWidth =
						`0`;
				maskHeight = maskWidth = `100%`;
				maskHeight = `100%`;
				if (mask) {
					maskRootBg = 'rgba(0,0,0,0.5)';
				}
				showArrow = false;
			}
		}
		if (el && target) {
			const rect = target.getBoundingClientRect();
			const { left, top, width, height } = rect;
			if (isElementNotInViewport(rect)) {
				setFullScreen();
				return;
			}

			el.style.width = `${width}px`;
			popoverWidth = `${width}px`;
			el.style.height = `${height}px`;
			el.style.position = 'fixed';
			el.style.top = `${top}px`;
			el.style.left = `${left}px`;
			el.style.transform = `translate(0, 0)`;
			if (!mask) {
				el.style.zIndex = '-9999';
			}

			// maskRootTransition = transition ? 'border-width .3s' : '';
			const gap = 4;
			maskWidth = `${width + 2 * gap}px`;
			maskHeight = `${height + 2 * gap}px`;
			if (mask) {
				maskBorderTopWidth = `${Math.max(top - gap, 0)}px`;
				maskBorderLeftWidth = `${Math.max(left - gap, 0)}px`;
				const { marginRight, marginLeft, marginBottom, marginTop } = window.getComputedStyle(
					document.body
				);
				const widthWithMargin =
					document.body.offsetWidth + parseInt(marginLeft) + parseInt(marginRight);
				const heightWithMargin =
					document.body.offsetHeight + parseInt(marginBottom) + parseInt(marginTop);
				maskBorderBottomWidth = `${Math.max(heightWithMargin - height - top, 0)}px`;
				maskBorderRightWidth = `${Math.max(widthWithMargin - width - left, 0)}px`;
			}
			maskRootBg = 'transparent';
			showArrow = true;
		} else if (el && !target) {
			setFullScreen();
		}
		await tick();
		setMaskCls = mask && isShow;
	}
	function isElementNotInViewport(rect: DOMRect) {
		return (
			rect.bottom < 0 ||
			rect.right < 0 ||
			rect.left > window.innerWidth ||
			rect.top > window.innerHeight
		);
	}

	async function doShow(show: boolean) {
		isShow = show;
		await tick();
		if (popoverRef) {
			if (isShow) {
				await setTriggerElStyle();
				await handleAppend();
				await tick();
			}
			popoverRef.updateShow(show);
			resolveShowArrow = showArrow;
			await tick();
			popoverRef.updatedArrow();
		}
	}

	async function doClose(isDispatch: boolean, i: number) {
		const { target } = steps[i];
		const el: null | HTMLElement = popoverRef.getPopoverContainerRef();
		if (target && el) {
			(target as HTMLElement).removeChild(el);
		}

		index = current;
		popoverRef.updateShow(false);
		isShow = open = false;
		if (isDispatch) {
			dispatch('close');
		}
	}

	let isShow = open;
	$: {
		doShow(open);
	}

	async function doSetTriggerElStyle() {
		setTriggerElStyle();
		resolveShowArrow = showArrow;
		await tick();
		popoverRef.updatedArrow();
	}
	$: {
		if (isShow) {
			BROWSER && window.addEventListener('scroll', doSetTriggerElStyle);
			BROWSER && window.addEventListener('resize', doSetTriggerElStyle);
		} else {
			BROWSER && window.removeEventListener('scroll', doSetTriggerElStyle);
			BROWSER && window.removeEventListener('resize', doSetTriggerElStyle);
		}
	}
	onMount(() => {
		isShow && doShow(isShow);
	});

	const prefixCls = getPrefixCls('tour');
	let setMaskCls = false;
	$: cnames = clsx(
		prefixCls,
		{
			[`${prefixCls}-mask`]: setMaskCls
		},
		cls
	);
	$: contentClass = clsx(`${prefixCls}-content`, contentCls);
	const headerCls = clsx(`${prefixCls}-header`);
	const closeCls = clsx(`${prefixCls}-close`);
	const bodyCls = clsx(`${prefixCls}-body`);
	const footerCls = clsx(`${prefixCls}-footer`);
	const indicatorsCls = clsx(`${prefixCls}-indicators`);
	const btnCls = clsx(`${prefixCls}-btn`);
</script>

{#if isShow}
	<div
		class={cnames}
		{...$$restProps}
		{...attrs}
		style:width={maskWidth}
		style:background-color={maskRootBg}
		style:height={maskHeight}
		style:border-left-width={maskBorderLeftWidth}
		style:border-top-width={maskBorderTopWidth}
		style:border-bottom-width={maskBorderBottomWidth}
		style:border-right-width={maskBorderRightWidth}
		style:transition={maskRootTransition}
		style:z-index={zIndex}
	>
		<KPopover
			bind:this={popoverRef}
			trigger="manual"
			width={popoverWidth}
			{placement}
			arrow={resolveShowArrow}
		>
			<div slot="contentEl" class={contentClass}>
				<div class={headerCls}>
					<slot name="title" current={index} title={steps[index].title}>
						{#if steps[index].title}
							<span>{steps[index].title}</span>
						{/if}
					</slot>

					<slot name="closeIcon" handleClose={() => doClose(true, index)}>
						<button
							class={closeCls}
							on:click={() => doClose(true, index)}>
							<KIcon icon={closeIcon} width="auto" height="auto"></KIcon>
						</button>
					</slot>
				</div>

				<div class={bodyCls}>
					<slot current={index} name="description" description={steps[index].description}>
						{#if steps[index].description}
							<span>{steps[index].description}</span>
						{/if}
					</slot>
				</div>

				<slot name="footer" {handleNext} {handlePrev} current={index}>
					<div class={footerCls}>
						<slot name="indicators" current={index}>
							<KIndicators defaultPageIndex={index} count={steps.length} cls={indicatorsCls}
							></KIndicators>
						</slot>

						<div class={btnCls}>
							<slot name="prevButton" {handlePrev}>
								<KButton on:click={handlePrev} type="info" ghost size="sm">
									{prevBtnText}
								</KButton>
							</slot>
							<slot name="nextButton" {handleNext}>
								<KButton on:click={handleNext} type="primary" size="sm">
									{index === steps.length - 1 ? finishBtnText : nextBtnText}
								</KButton>
							</slot>
						</div>
					</div>
				</slot>
			</div>
		</KPopover>
	</div>
{/if}
