<template>
	<a class="mint-cell" :class="classed">
		<span class="mint-cell-mask" v-if="isLink"></span>
		<div class="mint-cell-title" v-if="istitle">
			<slot name="icon">
				<i v-if="icon" class="iconfont" :class="'icon-wx-' + icon"></i>
			</slot>
			<slot name="title">
				<span class="mint-cell-text" :class="{'leh-red-dot':reddot}" v-text="title"></span>
				<span v-if="label" class="mint-cell-label" v-text="label"></span>
			</slot>
		</div>
		<div class="mint-cell-value">
			<slot>
				<span :class="{'leh-c-black':blackfont}" v-text="value"></span>
			</slot>
			<span  v-if="isIcon" class="iconfont" :class="'icon-wx-'+icons"></span>
		</div>
		<i v-if="isLink" class="mint-cell-allow-right"></i>
	</a>
</template>

<script>

	/**
	 * mt-cell
	 * @module components/cell
	 * @desc 单元格
	 * @param {string} [icon] - 图标，提供 more, back，或者自定义的图标（传入不带前缀的图标类名，最后拼接成 .mintui-xxx）
	 * @param {string} [title] - 标题
	 * @param {string} [label] - 备注信息
	 * @param {boolean} [is-link=false] - 可点击的链接
	 * @param {string} [value] - 右侧显示文字
	 * @param {slot} - 同 value, 会覆盖 value 属性
	 * @param {slot} [title] - 同 title, 会覆盖 title 属性
	 * @param {slot} [icon] - 同 icon, 会覆盖 icon 属性，例如可以传入图片
	 *
	 * @example
	 * <mt-cell title="标题文字" icon="back" is-link value="描述文字"></mt-cell>
	 * <mt-cell title="标题文字" icon="back">
	 *   <div slot="value">描述文字啊哈</div>
	 * </mt-cell>
	 */
	import cx from 'classnames'
	export default {
		name: 'mt-cell',

		props: {
			icon: String,
			title: String,
			label: String,
			isLink: Boolean,
			isIcon: Boolean,
			value: String,
			istitle: Boolean,
			blackfont: Boolean,
			reddot: Boolean,
			icons: String,
			className: String
		},

		computed: {
			classed () {
				return cx({
					[this.className]: !!this.className
				})
			}
		},
	};
</script>

<style lang="css">

</style>
