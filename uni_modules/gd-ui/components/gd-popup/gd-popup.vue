<template>
	<view class="gd-popup-wrap" ref="refMask"
		:style="[{ background: maskBackground, zIndex: zIndex - 2 }, getMaskStyle]" @tap.stop="close()">
		<view ref="refPopup" class="gd-popup" :style="getPopupStyle">
			<slot></slot>
		</view>
	</view>
</template>

<script lang="uts" setup>
	const props = defineProps({
		anchorViewId: {
			type: String,
			required: true
		},
		placement: {
			type: String,
			default: 'bottom'
		},
		background: {
			type: String,
			default: '#FFFFFF'
		},
		radius: {
			type: Number,
			default: 0
		},
		maskClosable: {
			type: Boolean,
			default: true
		},
		maskBackground: {
			type: String,
			default: 'rgba(0, 0, 0, 0.6)'
		},
		zIndex: {
			type: Number,
			default: 1998
		},
	});

	const visible = defineModel<boolean>({ type: Boolean, default: false });
	const anchorViewNode = ref<NodeInfo>({} as NodeInfo);
	const windowSafeArea = ref<SafeArea>({ left: 0, top: 0, right: 0, bottom: 0, width: 0, height: 0 } as SafeArea);

	const refMask = ref<UniElement | null>(null);
	const refPopup = ref<UniElement | null>(null);

	const isPlacementTop = () : boolean => props.placement.toLowerCase() == 'top';
	const isPlacementBottom = () : boolean => props.placement.toLowerCase() == 'bottom';

	const getMaskStyle = computed(() : Map<string, any> => {
		const style = new Map<string, any>();
		const anchor = anchorViewNode.value;
		const anchorTop = anchor.top ?? 0;
		const anchorBottom = anchor.bottom ?? 0;
		const safe = windowSafeArea.value;

		if (isPlacementTop()) {
			style.set('left', '0px');
			style.set('top', '0px');
			style.set('right', '0px');
			style.set('height', `${anchorTop}px`);
		} else if (isPlacementBottom()) {
			style.set('left', '0px');
			style.set('top', `${anchorBottom}px`);
			style.set('right', '0px');
			style.set('bottom', '0px');
		}

		return style;
	});

	const getPopupStyle = computed(() : Map<string, any> => {
		const style = new Map<string, any>();

		style.set('borderRadius', `${props.radius}px`);
		style.set('background', props.background);

		return style;
	});

	const getAnchorNodeInfo = () => {
		uni.createSelectorQuery()
			.select(`#${props.anchorViewId}`)
			.boundingClientRect()
			.exec((ret) => {
				if (ret.length > 0) {
					anchorViewNode.value = ret[0] as NodeInfo;
				}
			});
	};

	const getSafeArea = () => {
		const result = uni.getWindowInfo();
		windowSafeArea.value = result.safeArea;
	};

	const close = () => {
		const _refMask = refMask.value;
		const _refPopup = refPopup.value;

		if (_refMask == null || _refPopup == null) {
			return;
		}

		visible.value = false;

		_refMask.style.setProperty('opacity', '0');
		_refPopup.style.setProperty('opacity', '0');
		setTimeout(() => {
			_refMask.style.setProperty('visibility', 'hidden');
			_refPopup.style.setProperty('visibility', 'hidden');
		}, 300);
	};

	const show = () => {
		const _refMask = refMask.value;
		const _refPopup = refPopup.value;

		if (_refMask == null || _refPopup == null) {
			return;
		}

		visible.value = true;

		_refMask.style.setProperty('opacity', '1');
		_refMask.style.setProperty('visibility', 'visible');
		_refPopup.style.setProperty('opacity', '1');
		_refPopup.style.setProperty('visibility', 'visible');
	};

	watch(() : boolean => visible.value, (newVisible : boolean) => {
		if (newVisible) {
			show()
		} else {
			close();
		}
	}, {
		immediate: true
	});

	onMounted(() => {
		getAnchorNodeInfo();
		getSafeArea();
	});
</script>

<style lang="scss" scoped>
	.gd-popup {
		width: 100%;
		transition-property: transform, opacity;
		transition-timing-function: ease-in-out;
		transition-duration: 0.3s;
		min-height: 20rpx;
		overflow: hidden;

		&-wrap {
			position: fixed;
			display: flex;
			flex-direction: row;
			align-items: flex-start;
			justify-content: center;
			transition-property: opacity;
			transition-timing-function: ease-in-out;
			transition-duration: 0.3s;
			visibility: hidden;
			overflow: hidden;
			opacity: 0;
		}
	}
</style>